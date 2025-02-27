
java.lang.NullPointerException: Cannot invoke "org.springframework.web.reactive.function.client.WebClient$RequestHeadersUriSpec.uri(String, Object[])" because the return value of "org.springframework.web.reactive.function.client.WebClient.get()" is null

	at eu.olky.bankInfo.service.BankInfoValidationService.validateIban(BankInfoValidationService.java:27)
	at eu.olky.bankInfo.service.BankInfoValidationServiceTest.validateIban_ShouldReturnValidResponse(BankInfoValidationServiceTest.java:51)
	at java.base/java.lang.reflect.Method.invoke(Method.java:580)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1596)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1596)
