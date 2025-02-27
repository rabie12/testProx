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
