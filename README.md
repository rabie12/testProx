@ExtendWith(MockitoExtension.class)
class BankInfoValidationServiceTest {

    @Mock
    private WebClient webClient;

    @Mock
    private WebClient.RequestHeadersUriSpec<?> requestHeadersUriSpec;

    @Mock
    private WebClient.RequestHeadersSpec<?> requestHeadersSpec;

    @Mock
    private WebClient.ResponseSpec responseSpec;

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

        // Properly mock WebClient behavior
        when(webClient.get()).thenReturn(requestHeadersUriSpec);
        when(requestHeadersUriSpec.uri(anyString(), any())).thenReturn(requestHeadersSpec);
        when(requestHeadersSpec.retrieve()).thenReturn(responseSpec);
        when(responseSpec.bodyToMono(IbanValidationResponse.class)).thenReturn(Mono.just(mockResponse));

        // When & Then
        StepVerifier.create(bankInfoValidationService.validateIban(iban))
                .expectNextMatches(response -> response.isValid())
                .verifyComplete();
    }
}