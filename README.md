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
package eu.olky.bankInfo.web;

import eu.olky.bankInfo.dto.FindBankResponse;
import eu.olky.bankInfo.dto.IbanValidationResponse;
import eu.olky.bankInfo.service.BankInfoValidationService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.media.ArraySchema;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Mono;


@RestController
@RequestMapping("/api/")
@Tag(name = "Bank Info", description = "Bank Info Validation")
public class BankInfoValidationController {

    private final BankInfoValidationService bankInfoValidationService;

    public BankInfoValidationController(BankInfoValidationService bankInfoValidationService) {
        this.bankInfoValidationService = bankInfoValidationService;
    }

    @Operation(summary = "Validate Iban")
    @ApiResponses(value = {
            @ApiResponse(responseCode = "200", description = "Validation completed",
                    content = @Content(array = @ArraySchema(schema = @Schema(implementation = IbanValidationResponse.class)))),
            @ApiResponse(responseCode = "401", description = "Non autorisé", content = @Content)
    })
    @GetMapping("/validate/{iban}")
    public Mono<IbanValidationResponse> validateIban(@PathVariable String iban) {
        return bankInfoValidationService.validateIban(iban);
    }

    @Operation(summary = "find bank by bic")
    @ApiResponses(value = {
            @ApiResponse(responseCode = "200", description = "Validation completed",
                    content = @Content(array = @ArraySchema(schema = @Schema(implementation = IbanValidationResponse.class)))),
            @ApiResponse(responseCode = "401", description = "Non autorisé", content = @Content)
    })
    @GetMapping("/bic/{bic}")
    public Mono<FindBankResponse> getBankByBic(@PathVariable String bic) {
        return bankInfoValidationService.findBankByBic(bic);
    }

}

