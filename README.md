@ExtendWith(MockitoExtension.class)
class BankInfoValidationServiceTest {

    @Mock
    private WebClient webClient; // Mocking WebClient only

    @Mock
    private WebClient.RequestHeadersUriSpec<?> requestHeadersUriSpecMock;

    @Mock
    private WebClient.RequestHeadersSpec<?> requestHeadersSpecMock;

    @Mock
    private WebClient.ResponseSpec responseSpecMock;

    private BankInfoValidationService bankInfoValidationService;

    @BeforeEach
    void setUp() {
        bankInfoValidationService = new BankInfoValidationService(webClient);
    }

    @Test
    void validateIban_ShouldReturnValidResponse() {
        // Given
        String iban = "DE89370400440532013000";
        IbanValidationResponse mockResponse = new IbanValidationResponse(iban, true);

        // Define mock behavior
        when(webClient.get()).thenReturn(requestHeadersUriSpecMock);
        when(requestHeadersUriSpecMock.uri(anyString(), any())).thenReturn(requestHeadersSpecMock);
        when(requestHeadersSpecMock.retrieve()).thenReturn(responseSpecMock);
        when(responseSpecMock.bodyToMono(IbanValidationResponse.class)).thenReturn(Mono.just(mockResponse));

        // When & Then
        StepVerifier.create(bankInfoValidationService.validateIban(iban))
                .expectNextMatches(response -> response.isValid())
                .verifyComplete();
    }

    @Test
    void findBankByBic_ShouldReturnValidResponse() {
        // Given
        String bic = "DEUTDEFF";
        FindBankResponse mockResponse = new FindBankResponse(bic, "Deutsche Bank", "Germany");

        // Define mock behavior
        when(webClient.get()).thenReturn(requestHeadersUriSpecMock);
        when(requestHeadersUriSpecMock.uri(anyString())).thenReturn(requestHeadersSpecMock);
        when(requestHeadersSpecMock.retrieve()).thenReturn(responseSpecMock);
        when(responseSpecMock.bodyToMono(FindBankResponse.class)).thenReturn(Mono.just(mockResponse));

        // When & Then
        StepVerifier.create(bankInfoValidationService.findBankByBic(bic))
                .expectNextMatches(response -> response.getBankName().equals("Deutsche Bank"))
                .verifyComplete();
    }
}