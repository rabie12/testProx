
Voici la classe IbanValidationResponse avec les annotations @JsonProperty pour s’assurer que les noms des champs correspondent aux propriétés JSON retournées par l’API :

import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.List;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class IbanValidationResponse {
    
    @JsonProperty("iban")
    private String iban;
    
    @JsonProperty("result")
    private String result;
    
    @JsonProperty("return_code")
    private int returnCode;
    
    @JsonProperty("checks")
    private List<String> checks;
    
    @JsonProperty("length_check")
    private String lengthCheck;
    
    @JsonProperty("account_check")
    private String accountCheck;
    
    @JsonProperty("bank_code_check")
    private String bankCodeCheck;
    
    @JsonProperty("iban_checksum_check")
    private String ibanChecksumCheck;
    
    @JsonProperty("account_validation_method")
    private String accountValidationMethod;
    
    @JsonProperty("account_validation")
    private String accountValidation;
    
    @JsonProperty("iban_candidate")
    private String ibanCandidate;
    
    @JsonProperty("IBANformat")
    private String ibanFormat;
    
    @JsonProperty("formatcomment")
    private String formatComment;
    
    @JsonProperty("bic_candidates")
    private List<BicCandidate> bicCandidates;
    
    @JsonProperty("all_bic_candidates")
    private List<BicCandidate> allBicCandidates;
    
    @JsonProperty("country")
    private String country;
    
    @JsonProperty("bank_code")
    private String bankCode;
    
    @JsonProperty("bank_and_branch_code")
    private String bankAndBranchCode;
    
    @JsonProperty("bank")
    private String bank;
    
    @JsonProperty("bank_address")
    private String bankAddress;
    
    @JsonProperty("bank_street")
    private String bankStreet;
    
    @JsonProperty("bank_city")
    private String bankCity;
    
    @JsonProperty("bank_state")
    private String bankState;
    
    @JsonProperty("bank_postal_code")
    private String bankPostalCode;
    
    @JsonProperty("bank_url")
    private String bankUrl;
    
    @JsonProperty("branch")
    private String branch;
    
    @JsonProperty("branch_code")
    private String branchCode;
    
    @JsonProperty("in_scl_directory")
    private String inSclDirectory;
    
    @JsonProperty("sct")
    private String sct;
    
    @JsonProperty("sdd")
    private String sdd;
    
    @JsonProperty("cor1")
    private String cor1;
    
    @JsonProperty("b2b")
    private String b2b;
    
    @JsonProperty("scc")
    private String scc;
    
    @JsonProperty("sct_inst")
    private String sctInst;
    
    @JsonProperty("sct_inst_readiness_date")
    private String sctInstReadinessDate;
    
    @JsonProperty("account_number")
    private String accountNumber;
    
    @JsonProperty("data_age")
    private String dataAge;
    
    @JsonProperty("iban_listed")
    private String ibanListed;
    
    @JsonProperty("iban_www_occurrences")
    private int ibanWwwOccurrences;
    
    @JsonProperty("www_seen_from")
    private String wwwSeenFrom;
    
    @JsonProperty("www_seen_until")
    private String wwwSeenUntil;
    
    @JsonProperty("iban_url")
    private String ibanUrl;
    
    @JsonProperty("url_rank")
    private int urlRank;
    
    @JsonProperty("url_category")
    private String urlCategory;
    
    @JsonProperty("url_min_depth")
    private int urlMinDepth;
    
    @JsonProperty("url_prominence")
    private String urlProminence;
    
    @JsonProperty("iban_reported_to_exist")
    private int ibanReportedToExist;
    
    @JsonProperty("iban_last_reported")
    private String ibanLastReported;
    
    @JsonProperty("iban_first_seen")
    private String ibanFirstSeen;
    
    @JsonProperty("balance")
    private int balance;

    public IbanValidationResponse(String iban, String result) {
        this.iban = iban;
        this.result = result;
    }
}

Et voici la classe BicCandidate pour représenter les BICs associés :

import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class BicCandidate {
    
    @JsonProperty("bic")
    private String bic;
    
    @JsonProperty("wwwcount")
    private int wwwCount;
    
    @JsonProperty("sampleurl")
    private String sampleUrl;
    
    @JsonProperty("city")
    private String city;
}

Cette classe permettra de mapper directement la réponse JSON de l’API en un objet Java, facilitant son utilisation dans ton microservice.