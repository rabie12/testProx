package eu.olky.bankInfo.service;

import eu.olky.bankInfo.dto.FindBankResponse;
import eu.olky.bankInfo.dto.IbanValidationResponse;
import org.iban4j.BicFormatException;
import org.iban4j.BicUtil;
import org.iban4j.IbanFormatException;
import org.iban4j.IbanUtil;
import org.springframework.stereotype.Service;
import org.springframework.web.reactive.function.UnsupportedMediaTypeException;
import org.springframework.web.reactive.function.client.WebClient;
import org.springframework.web.reactive.function.client.WebClientResponseException;
import reactor.core.publisher.Mono;

@Service
public class BankInfoValidationService {

    private final WebClient webClient;

    public BankInfoValidationService(WebClient webClient) {
        this.webClient = webClient;
    }

    public Mono<IbanValidationResponse> validateIban(String iban) {
        if (isValidIban(iban)) {
            return webClient.get()
                    .uri("/validate_iban/{iban}", iban.trim())
                    .retrieve()
                    .bodyToMono(IbanValidationResponse.class)
                    .onErrorResume(WebClientResponseException.class, ex -> {
                        return Mono.error(new RuntimeException("Erreur lors de la validation de l'IBAN : " + ex.getMessage()));
                    });
        }
        return Mono.error(new IllegalArgumentException("Invalid IBAN"));


    }

    public Mono<FindBankResponse> findBankByBic(String bic) {
        // Validate BIC input
        if (isValidBic(bic)) {
            return webClient.get()
                    .uri("https://rest.sepatools.eu/find_bank///" + bic + "/5")
                    .retrieve()
                    .bodyToMono(FindBankResponse.class)
                    .onErrorResume(UnsupportedMediaTypeException.class, ex -> {
                        // Handle UnsupportedMediaTypeException
                        System.err.println("Unsupported media type: " + ex.getMessage());
                        return Mono.error(new RuntimeException("Unsupported media type: " + ex.getMessage()));
                    });
        }
        return Mono.error(new IllegalArgumentException("Invalid BIC"));

    }

    public static boolean isValidBic(String bic) {
        try {
            BicUtil.validate(bic);
            return true;
        } catch (BicFormatException e) {
            return false;
        }
    }

    public static boolean isValidIban(String bic) {
        try {
            IbanUtil.validate(bic);
            return true;
        } catch (IbanFormatException e) {
            return false;
        }
    }

}

