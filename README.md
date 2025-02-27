package eu.olky.bankInfo.service;

import eu.olky.bankInfo.dto.FindBankResponse;
import eu.olky.bankInfo.dto.IbanValidationResponse;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.web.reactive.function.client.WebClient;
import org.springframework.web.reactive.function.client.WebClient.RequestHeadersSpec;
import org.springframework.web.reactive.function.client.WebClient.RequestHeadersUriSpec;
import org.springframework.web.reactive.function.client.WebClient.ResponseSpec;
import reactor.core.publisher.Mono;
import reactor.test.StepVerifier;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.eq;
import static org.mockito.Mockito.when;

@ExtendWith(MockitoExtension.class)
class BankInfoValidationServiceTest {

    @Mock
    private WebClient webClient;

    @Mock
    private WebClient.RequestHeadersUriSpec<?> requestHeadersUriSpecMock;

    @Mock
    private WebClient.RequestHeadersSpec<?> requestHeadersSpecMock;

    @Mock
    private WebClient.ResponseSpec responseSpecMock;

    @InjectMocks
    private BankInfoValidationService bankInfoValidationService;

    @BeforeEach
    void setUp() {
        // Fix Mockito issue with generic return types
        Mockito.<RequestHeadersUriSpec<?>>when(webClient.get()).thenReturn(requestHeadersUriSpecMock);

        // Mock URI handling
        when(requestHeadersUriSpecMock.uri(any(String.class), any(Object[].class)))
                .thenReturn(requestHeadersSpecMock);

        // Mock request retrieval
        when(requestHeadersSpecMock.retrieve()).thenReturn(responseSpecMock);
    }

    @Test
    void validateIban_ShouldReturnValidResponse() {
        // Given
        String iban = "DE89370400440532013000";
        IbanValidationResponse mockResponse = new IbanValidationResponse(true, "Valid IBAN");

        when(responseSpecMock.bodyToMono(IbanValidationResponse.class))
                .thenReturn(Mono.just(mockResponse));

        // When
        Mono<IbanValidationResponse> result = bankInfoValidationService.validateIban(iban);

        // Then - Verify using StepVerifier
        StepVerifier.create(result)
                .expectNextMatches(response -> response.isValid() && response.getMessage().equals("Valid IBAN"))
                .verifyComplete();
    }

    @Test
    void findBankByBic_ShouldReturnBankInfo() {
        // Given
        String bic = "DEUTDEFF";
        FindBankResponse mockResponse = new FindBankResponse("DEUTDEFF", "Deutsche Bank", "Germany");

        when(responseSpecMock.bodyToMono(FindBankResponse.class))
                .thenReturn(Mono.just(mockResponse));

        // When
        Mono<FindBankResponse> result = bankInfoValidationService.findBankByBic(bic);

        // Then - Verify using StepVerifier
        StepVerifier.create(result)
                .expectNextMatches(response -> response.getBic().equals("DEUTDEFF") &&
                        response.getBankName().equals("Deutsche Bank"))
                .verifyComplete();
    }
}