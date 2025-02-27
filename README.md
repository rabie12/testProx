package eu.olky.bankInfo.web;

import eu.olky.bankInfo.dto.FindBankResponse;
import eu.olky.bankInfo.dto.IbanValidationResponse;
import eu.olky.bankInfo.service.BankInfoValidationService;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.junit.jupiter.MockitoExtension;
import reactor.core.publisher.Mono;
import reactor.test.StepVerifier;

import static org.mockito.ArgumentMatchers.anyString;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class BankInfoValidationControllerTest {

    @Mock
    private BankInfoValidationService bankInfoValidationService;

    @InjectMocks
    private BankInfoValidationController bankInfoValidationController;

    @BeforeEach
    void setUp() {
        Mockito.reset(bankInfoValidationService);
    }

    @Test
    void validateIban_ShouldReturnValidResponse() {
        // Given
        String iban = "DE89370400440532013000";
        IbanValidationResponse ibanResponse = new IbanValidationResponse(iban, true);

        when(bankInfoValidationService.validateIban(iban)).thenReturn(Mono.just(ibanResponse));

        // When & Then
        StepVerifier.create(bankInfoValidationController.validateIban(iban))
                .expectNext(ibanResponse)
                .verifyComplete();

        verify(bankInfoValidationService, times(1)).validateIban(iban);
    }

    @Test
    void validateIban_ShouldReturnErrorForInvalidIban() {
        // Given
        String invalidIban = "INVALID";
        when(bankInfoValidationService.validateIban(invalidIban)).thenReturn(Mono.error(new IllegalArgumentException("Invalid IBAN")));

        // When & Then
        StepVerifier.create(bankInfoValidationController.validateIban(invalidIban))
                .expectErrorMatches(throwable -> throwable instanceof IllegalArgumentException &&
                        throwable.getMessage().equals("Invalid IBAN"))
                .verify();

        verify(bankInfoValidationService, times(1)).validateIban(invalidIban);
    }

    @Test
    void getBankByBic_ShouldReturnBankInfo() {
        // Given
        String bic = "DEUTDEFF";
        FindBankResponse bankResponse = new FindBankResponse(bic, "Deutsche Bank", "Germany", "Taunusanlage 12");

        when(bankInfoValidationService.findBankByBic(bic)).thenReturn(Mono.just(bankResponse));

        // When & Then
        StepVerifier.create(bankInfoValidationController.getBankByBic(bic))
                .expectNext(bankResponse)
                .verifyComplete();

        verify(bankInfoValidationService, times(1)).findBankByBic(bic);
    }

    @Test
    void getBankByBic_ShouldReturnErrorForInvalidBic() {
        // Given
        String invalidBic = "INVALID";
        when(bankInfoValidationService.findBankByBic(invalidBic)).thenReturn(Mono.error(new IllegalArgumentException("Invalid BIC")));

        // When & Then
        StepVerifier.create(bankInfoValidationController.getBankByBic(invalidBic))
                .expectErrorMatches(throwable -> throwable instanceof IllegalArgumentException &&
                        throwable.getMessage().equals("Invalid BIC"))
                .verify();

        verify(bankInfoValidationService, times(1)).findBankByBic(invalidBic);
    }

    @Test
    void getBankByBic_ShouldHandleWebClientException() {
        // Given
        String bic = "DEUTDEFF";
        when(bankInfoValidationService.findBankByBic(bic))
                .thenReturn(Mono.error(new RuntimeException("Error fetching bank info")));

        // When & Then
        StepVerifier.create(bankInfoValidationController.getBankByBic(bic))
                .expectErrorMatches(throwable -> throwable instanceof RuntimeException &&
                        throwable.getMessage().equals("Error fetching bank info"))
                .verify();

        verify(bankInfoValidationService, times(1)).findBankByBic(bic);
    }
}

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

