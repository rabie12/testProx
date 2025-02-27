@ExtendWith(MockitoExtension.class)
class BankInfoValidationServiceTest {

    @Mock
    private WebClient webClient;

    @Mock
    private WebClient.RequestHeadersUriSpec<?> requestHeadersUriSpecMock;

    @Mock
    private WebClient.RequestHeadersSpec<?> requestHeadersSpecMock;

    @Mock
    private WebClient.ResponseSpec responseSpecMock;

    @InjectMocks
    private BankInfoValidationService bankInfoValidationService;

    @Test
    void validateIban_ShouldReturnValidResponse() {
        // Given
        String iban = "DE89370400440532013000";
        IbanValidationResponse expectedResponse = new IbanValidationResponse(true, "Valid IBAN");

        Mockito.when(webClient.get()).thenReturn(requestHeadersUriSpecMock);
        Mockito.when(requestHeadersUriSpecMock.uri("/validate_iban/{iban}", iban.trim()))
               .thenReturn(requestHeadersSpecMock);
        Mockito.when(requestHeadersSpecMock.retrieve()).thenReturn(responseSpecMock);
        Mockito.when(responseSpecMock.bodyToMono(IbanValidationResponse.class))
               .thenReturn(Mono.just(expectedResponse));

        // When
        Mono<IbanValidationResponse> responseMono = bankInfoValidationService.validateIban(iban);

        // Then
        StepVerifier.create(responseMono)
                .expectNext(expectedResponse)
                .verifyComplete();
    }

    @Test
    void findBankByBic_ShouldReturnFindBankResponse() {
        // Given
        String bic = "DEUTDEFF";
        FindBankResponse expectedResponse = new FindBankResponse("Deutsche Bank", bic, "Germany");

        Mockito.when(webClient.get()).thenReturn(requestHeadersUriSpecMock);
        Mockito.when(requestHeadersUriSpecMock.uri("https://rest.sepatools.eu/find_bank///" + bic + "/5"))
               .thenReturn(requestHeadersSpecMock);
        Mockito.when(requestHeadersSpecMock.retrieve()).thenReturn(responseSpecMock);
        Mockito.when(responseSpecMock.bodyToMono(FindBankResponse.class))
               .thenReturn(Mono.just(expectedResponse));

        // When
        Mono<FindBankResponse> responseMono = bankInfoValidationService.findBankByBic(bic);

        // Then
        StepVerifier.create(responseMono)
                .expectNext(expectedResponse)
                .verifyComplete();
    }
}