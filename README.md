Mockito is currently self-attaching to enable the inline-mock-maker. This will no longer work in future releases of the JDK. Please add Mockito as an agent to your build what is described in Mockito's documentation: https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#0.3
WARNING: A Java agent has been loaded dynamically (C:\Users\RHABACHI\.m2\repository\net\bytebuddy\byte-buddy-agent\1.15.11\byte-buddy-agent-1.15.11.jar)
WARNING: If a serviceability tool is in use, please run with -XX:+EnableDynamicAgentLoading to hide this warning
WARNING: If a serviceability tool is not in use, please run with -Djdk.instrument.traceUsage for more information
WARNING: Dynamic loading of agents will be disallowed by default in a future release
OpenJDK 64-Bit Server VM warning: Sharing is only supported for boot loader classes because bootstrap classpath has been appended

org.mockito.exceptions.misusing.NotAMockException: Argument should be a mock, but is null!

	at eu.olky.bankInfo.web.BankInfoValidationControllerTest.setUp(BankInfoValidationControllerTest.java:26)
	at java.base/java.lang.reflect.Method.invoke(Method.java:580)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1596)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1596)
package eu.olky.bankInfo.service;

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

