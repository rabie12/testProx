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
        IbanValidationResponse ibanResponse = new IbanValidationResponse(iban, true);
        ApiResponse<IbanValidationResponse> expectedResponse = new ApiResponse<>(ibanResponse, "Valid IBAN");

        Mockito.when(ibanValidationService.validateIban(iban)).thenReturn(Mono.just(expectedResponse));

        // When
        StepVerifier.create(ibanValidationController.validateIban(iban))
                .expectNext(expectedResponse)
                .verifyComplete();

        Mockito.verify(ibanValidationService).validateIban(iban);
    }

    @Test
    void findBankByIban_ShouldReturnBankInfo() {
        // Given
        String iban = "DE89370400440532013000";
        FindBankResponse bankResponse = new FindBankResponse("BIC123", "Test Bank", "DE", "Test Address");
        ApiResponse<FindBankResponse> expectedResponse = new ApiResponse<>(bankResponse, "Bank found for IBAN");

        Mockito.when(ibanValidationService.findBankByIban(iban)).thenReturn(Mono.just(expectedResponse));

        // When
        StepVerifier.create(ibanValidationController.findBankByIban(iban))
                .expectNext(expectedResponse)
                .verifyComplete();

        Mockito.verify(ibanValidationService).findBankByIban(iban);
    }

    @Test
    void findBankByBic_ShouldReturnBankInfo() {
        // Given
        String bic = "BIC123";
        FindBankResponse bankResponse = new FindBankResponse(bic, "Test Bank", "DE", "Test Address");
        ApiResponse<FindBankResponse> expectedResponse = new ApiResponse<>(bankResponse, "Bank found for BIC");

        Mockito.when(ibanValidationService.findBankByBic(bic)).thenReturn(Mono.just(expectedResponse));

        // When
        StepVerifier.create(ibanValidationController.findBankByBic(bic))
                .expectNext(expectedResponse)
                .verifyComplete();

        Mockito.verify(ibanValidationService).findBankByBic(bic);
    }
}