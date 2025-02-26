import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.web.servlet.MockMvc;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.mockito.BDDMockito.given;

@WebMvcTest(IbanValidationController.class)
class IbanValidationControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private IbanValidationService ibanValidationService;

    @Test
    void testValidateIban_Valid() throws Exception {
        // Arrange
        String validIban = "DE89370400440532013000";
        given(ibanValidationService.validateIban(validIban)).willReturn(true);

        // Act & Assert
        mockMvc.perform(get("/api/validateIban")
                .param("iban", validIban))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.valid").value(true));
    }

    @Test
    void testValidateIban_Invalid() throws Exception {
        // Arrange
        String invalidIban = "INVALIDIBAN";
        given(ibanValidationService.validateIban(invalidIban)).willReturn(false);

        // Act & Assert
        mockMvc.perform(get("/api/validateIban")
                .param("iban", invalidIban))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.valid").value(false));
    }
}