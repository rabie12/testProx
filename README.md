You’re seeing this error because Mockito’s inline mock maker isn’t properly set up, and some dependencies might be missing. Let’s fix it step by step.

Fix 1: Add Mockito Java Agent in pom.xml (Recommended)

Starting from Java 21, dynamic agent loading (which Mockito uses) is being restricted. To fix this, explicitly add Mockito’s Java agent:

<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>5.7.0</version>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-inline</artifactId>
    <version>5.2.0</version>
    <scope>test</scope>
</dependency>

Then, in maven-surefire-plugin, add this:

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.0.0-M7</version>
    <configuration>
        <argLine>-javaagent:"${settings.localRepository}/org/mockito/mockito-inline/5.2.0/mockito-inline-5.2.0.jar"</argLine>
    </configuration>
</plugin>

This ensures Mockito runs correctly in newer Java versions.

Fix 2: Ensure WebClient Mocking Works Properly

You’re getting NotAMockException because Mockito is expecting a mock but got null. Ensure correct annotation usage:

@ExtendWith(MockitoExtension.class)
class BankInfoValidationControllerTest {

    @Mock
    private BankInfoValidationService bankInfoValidationService;

    @InjectMocks
    private BankInfoValidationController bankInfoValidationController; // Ensure it's created with mocks injected

    @BeforeEach
    void setUp() {
        when(bankInfoValidationService.validateIban(anyString()))
                .thenReturn(Mono.just(new IbanValidationResponse("DE89370400440532013000", true)));

        when(bankInfoValidationService.findBankByBic(anyString()))
                .thenReturn(Mono.just(new FindBankResponse("DEUTDEFF", "Deutsche Bank", "Germany", "Taunusanlage 12")));
    }

    @Test
    void validateIban_ShouldReturnResponse() {
        StepVerifier.create(bankInfoValidationController.validateIban("DE89370400440532013000"))
                .expectNextMatches(response -> response.isValid())
                .verifyComplete();
    }

    @Test
    void getBankByBic_ShouldReturnResponse() {
        StepVerifier.create(bankInfoValidationController.getBankByBic("DEUTDEFF"))
                .expectNextMatches(response -> response.getBankName().equals("Deutsche Bank"))
                .verifyComplete();
    }
}

Why This Works

✔ Fixed the missing inline-mock-maker issue using mockito-inline
✔ Ensured proper dependency injection by initializing @InjectMocks
✔ Used .expectNextMatches() to verify fields dynamically

Now, StepVerifier should work correctly without the NotAMockException error. Let me know if you still have issues!
import eu.olky.bankInfo.dto.FindBankResponse;
import eu.olky.bankInfo.dto.IbanValidationResponse;
import org.iban4j.BicFormatException;
import org.iban4j.IbanFormatException;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.web.reactive.function.client.WebClient;
import org.springframework.web.reactive.function.client.WebClientResponseException;
import reactor.core.publisher.Mono;
import reactor.test.StepVerifier;

import static org.mockito.ArgumentMatchers.anyString;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class BankInfoValidationServiceTest {

    @Mock
    private WebClient webClient;

    @Mock
    private WebClient.RequestHeadersUriSpec<?> requestHeadersUriSpec;

    @Mock
    private WebClient.ResponseSpec responseSpec;

    @InjectMocks
    private BankInfoValidationService bankInfoValidationService;

    @BeforeEach
    void setUp() {
        lenient().when(webClient.get()).thenReturn(requestHeadersUriSpec);
        lenient().when(requestHeadersUriSpec.uri(anyString(), anyString())).thenReturn(requestHeadersUriSpec);
        lenient().when(requestHeadersUriSpec.retrieve()).thenReturn(responseSpec);
    }

    @Test
    void validateIban_ShouldReturnValidResponse() {
        // Given
        String iban = "DE89370400440532013000";
        IbanValidationResponse ibanResponse = new IbanValidationResponse(iban, true);

        when(responseSpec.bodyToMono(IbanValidationResponse.class)).thenReturn(Mono.just(ibanResponse));

        // When & Then
        StepVerifier.create(bankInfoValidationService.validateIban(iban))
                .expectNext(ibanResponse)
                .verifyComplete();

        verify(requestHeadersUriSpec, times(1)).uri("/validate_iban/{iban}", iban.trim());
        verify(responseSpec, times(1)).bodyToMono(IbanValidationResponse.class);
    }

    @Test
    void validateIban_ShouldReturnErrorForInvalidIban() {
        // Given
        String invalidIban = "INVALID";

        // When & Then
        StepVerifier.create(bankInfoValidationService.validateIban(invalidIban))
                .expectErrorMatches(throwable -> throwable instanceof IllegalArgumentException &&
                        throwable.getMessage().equals("Invalid IBAN"))
                .verify();

        verify(requestHeadersUriSpec, never()).uri(anyString(), anyString());
    }

    @Test
    void validateIban_ShouldHandleWebClientError() {
        // Given
        String iban = "DE89370400440532013000";
        when(responseSpec.bodyToMono(IbanValidationResponse.class))
                .thenReturn(Mono.error(new WebClientResponseException(500, "Server Error", null, null, null)));

        // When & Then
        StepVerifier.create(bankInfoValidationService.validateIban(iban))
                .expectErrorMatches(throwable -> throwable instanceof RuntimeException &&
                        throwable.getMessage().contains("Erreur lors de la validation de l'IBAN"))
                .verify();
    }

    @Test
    void findBankByBic_ShouldReturnValidResponse() {
        // Given
        String bic = "DEUTDEFF";
        FindBankResponse bankResponse = new FindBankResponse(bic, "Deutsche Bank", "Germany", "Taunusanlage 12");

        when(responseSpec.bodyToMono(FindBankResponse.class)).thenReturn(Mono.just(bankResponse));

        // When & Then
        StepVerifier.create(bankInfoValidationService.findBankByBic(bic))
                .expectNext(bankResponse)
                .verifyComplete();

        verify(requestHeadersUriSpec, times(1)).uri("https://rest.sepatools.eu/find_bank///" + bic + "/5");
        verify(responseSpec, times(1)).bodyToMono(FindBankResponse.class);
    }

    @Test
    void findBankByBic_ShouldReturnErrorForInvalidBic() {
        // Given
        String invalidBic = "INVALID";

        // When & Then
        StepVerifier.create(bankInfoValidationService.findBankByBic(invalidBic))
                .expectErrorMatches(throwable -> throwable instanceof IllegalArgumentException &&
                        throwable.getMessage().equals("Invalid BIC"))
                .verify();

        verify(requestHeadersUriSpec, never()).uri(anyString(), anyString());
    }

    @Test
    void findBankByBic_ShouldHandleWebClientError() {
        // Given
        String bic = "DEUTDEFF";
        when(responseSpec.bodyToMono(FindBankResponse.class))
                .thenReturn(Mono.error(new WebClientResponseException(500, "Server Error", null, null, null)));

        // When & Then
        StepVerifier.create(bankInfoValidationService.findBankByBic(bic))
                .expectErrorMatches(throwable -> throwable instanceof RuntimeException &&
                        throwable.getMessage().contains("Unsupported media type"))
                .verify();
    }
}

