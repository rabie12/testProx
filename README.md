Sure! I’ll provide the unit test implementation for IbanValidationController using Mockito and JUnit 5. The service will return either an IbanValidationResponse or a FindBankResponse, and we’ll mock the dependencies without using @Autowired. Instead, we’ll use constructor injection for better testability.

1. IbanValidationResponse & FindBankResponse DTOs

If these DTOs don’t already exist, we define them:

public record IbanValidationResponse(String iban, boolean isValid, String message) {}

public record FindBankResponse(String bic, String bankName, String country, String address) {}

2. IbanValidationService

We’ll define the service with methods returning either an IbanValidationResponse or a FindBankResponse.

@Service
@RequiredArgsConstructor
public class IbanValidationService {
    private final BankInfoClient bankInfoClient;

    public IbanValidationResponse validateIban(String iban) {
        // Assume some validation logic
        boolean isValid = iban.startsWith("DE") && iban.length() == 22;
        String message = isValid ? "Valid IBAN" : "Invalid IBAN";

        return new IbanValidationResponse(iban, isValid, message);
    }

    public FindBankResponse findBankByIban(String iban) {
        return bankInfoClient.getBankInfoByIban(iban);
    }
}

	•	validateIban: Validates IBAN format.
	•	findBankByIban: Calls BankInfoClient (FeignClient) to fetch bank details.

3. FeignClient to Call Bank Info Service

Since IbanValidationService depends on the Bank Info Service, we use FeignClient:

@FeignClient(name = "bank-info-service", path = "/api/bank")
public interface BankInfoClient {
    @GetMapping("/iban/{iban}")
    FindBankResponse getBankInfoByIban(@PathVariable String iban);
}

4. IbanValidationController

The controller injects IbanValidationService via constructor injection:

@RestController
@RequestMapping("/api/iban")
@RequiredArgsConstructor
public class IbanValidationController {
    private final IbanValidationService ibanValidationService;

    @GetMapping("/validate/{iban}")
    public IbanValidationResponse validateIban(@PathVariable String iban) {
        return ibanValidationService.validateIban(iban);
    }

    @GetMapping("/find-bank/{iban}")
    public FindBankResponse findBankByIban(@PathVariable String iban) {
        return ibanValidationService.findBankByIban(iban);
    }
}

5. Unit Test for IbanValidationController

We’ll use Mockito to mock IbanValidationService and verify interactions.

@ExtendWith(MockitoExtension.class)
class IbanValidationControllerTest {

    @Mock
    private IbanValidationService ibanValidationService;

    private IbanValidationController ibanValidationController;

    @BeforeEach
    void setUp() {
        ibanValidationController = new IbanValidationController(ibanValidationService);
    }

    @Test
    void validateIban_ShouldReturnValidResponse() {
        // Given
        String iban = "DE89370400440532013000";
        IbanValidationResponse expectedResponse = new IbanValidationResponse(iban, true, "Valid IBAN");

        Mockito.when(ibanValidationService.validateIban(iban)).thenReturn(expectedResponse);

        // When
        IbanValidationResponse response = ibanValidationController.validateIban(iban);

        // Then
        assertEquals(expectedResponse, response);
        Mockito.verify(ibanValidationService).validateIban(iban);
    }

    @Test
    void findBankByIban_ShouldReturnBankInfo() {
        // Given
        String iban = "DE89370400440532013000";
        FindBankResponse expectedResponse = new FindBankResponse("BIC123", "Test Bank", "DE", "Test Address");

        Mockito.when(ibanValidationService.findBankByIban(iban)).thenReturn(expectedResponse);

        // When
        FindBankResponse response = ibanValidationController.findBankByIban(iban);

        // Then
        assertEquals(expectedResponse, response);
        Mockito.verify(ibanValidationService).findBankByIban(iban);
    }
}

6. Explanation

✅ Mocks IbanValidationService instead of using @Autowired.
✅ Uses constructor injection for IbanValidationController.
✅ Verifies interactions using Mockito.verify().
✅ Tests both API endpoints (/validate/{iban} and /find-bank/{iban}).

Let me know if you need further refinements!