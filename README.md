
import lombok.RequiredArgsConstructor;
import org.iban4j.IbanUtil;
import org.iban4j.IbanFormatException;
import org.iban4j.IbanCheckDigitException;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
@RequiredArgsConstructor
public class IbanValidationService {

    private final IbanValidationRepository repository;
    private final IbanValidationMapper mapper;
    private final RestTemplate restTemplate;

    private static final String IBAN_VALIDATION_URL = "https://rest.sepatools.eu/validate_iban/";

    public IbanValidation validateIban(String iban) {
        // 1. Validate IBAN format using iban4j
        try {
            IbanUtil.validate(iban);
        } catch (IbanFormatException | IbanCheckDigitException ex) {
            throw new IllegalArgumentException("Invalid IBAN format: " + iban, ex);
        }

        // 2. Check if IBAN exists in the DB
        return repository.findByIban(iban)
                .orElseGet(() -> {
                    // 3. Call the external API
                    IbanValidationResponse response = callIbanValidationApi(iban);
                    // 4. Map API response to our entity (only bank, bankAddress, and IBAN)
                    IbanValidation entity = mapper.toEntity(response);
                    // 5. Save the entity in the DB
                    repository.save(entity);
                    return entity;
                });
    }

    private IbanValidationResponse callIbanValidationApi(String iban) {
        ResponseEntity<IbanValidationResponse> responseEntity =
                restTemplate.getForEntity(IBAN_VALIDATION_URL + iban, IbanValidationResponse.class);
        return responseEntity.getBody();
    }
}