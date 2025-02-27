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