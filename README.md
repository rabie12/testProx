

DE ♦ EN

Mass Calculation
Web Service
Countries
Security
Prices
IBAN-BIC.com (Theano GmbH)  » Web Service  » Without SOAP  » REST Documentation: validate_iban

webbanking
••••••••••••
Forgot password?

Contact
us

 Stay logged in		
New customer? Register

Validate IBANs: https://rest.sepatools.eu/validate_iban/{iban}
Example:
https://rest.sepatools.eu/validate_iban/IE92BOFI90001710027952

Purpose:

Validates the given IBAN, including its contained domestic account number and bank code, insofar as possible.
Adds information such as BIC code, bank name, bank address, bank URL.
Helps prevent fraud by checking the IBAN against a list of publicly visible IBANs which can be found on the WWW (and might therefore be used by fraudsters instead of their own IBANs) and against a list of IBANs for which a fraud suspicion has been reported.
Testing the call mechanism for free: for your first implementation steps, we recommend using the endpoint rest.sepatools.eu/validate_iban_dummy/AL90208110080000001039531801, which expects the same parameters, runs on the same servers, uses the same authentication mechanism, and returns the same data structure as validate_iban. However,

the endpoint validate_iban_dummy is free to call, i.e. your account balance remains unchanged,
and this dummy endpoint always validates the same IBAN, no matter which IBAN you pass as an input parameter.
There are some exceptions: for the IBANs DE27100777770209299700, DE11520513735120710131, DE11520513735120710132, and AT411100000237571500, the dummy endpoint returns the actual result. The purpose of these exceptions is to let you test scenarios such as an invalid IBAN and an IBAN found on the Web.
Once you see that your implementation works, change the endpoint name from validate_iban_dummy to validate_iban.

Test Data: for testing your client implementation with different scenarios (valid IBAN, invalid IBAN, suspicious IBAN, etc.), you might want to use these test data (including test data for fraud prevention).

Authentication: Basic Authentication using the user name and password you defined when signing up on iban-bic.com.

Supports GET and POST.

Input
Input Parameters:

iban the IBAN to be validated.
Output: validation results
The following output fields contain validation results:

iban: The IBAN that was checked.
result: 'passed' or 'failed' - for a formally correct or incorrect IBAN
return_code: a condensed numerical representation of the results of various checks. Not all checks are always carried out.
Possible values:
0 = all supported checks were done and passed.
Otherwise, the sum of one or more of the following numbers:
1 = a subaccount number was automatically appended.
2 = Account number does not contain a checksum.
4 = Checksum was not checked.
8 = Bank code was not checked.
32 = Warning: a subaccount number may need to be appended, but the necessity of this could not be automatically verified. Please verify manually.
128 = Checksum error in account number.
256 = Bankcode not found in directory.
512 = IBAN has incorrect length.
1024 = Bank code has incorrect length.
2048 = The IBAN checksum is incorrect (3rd and 4th characters of the IBAN)
4096 = Missing input data (for example, the country code).
8192 = Country not yet supported.
Interpretation: if the sum is

< 32. Result can be assumed correct;
32 ≤ sum ≤ 127. Result might be correct but should be verified;
≥ 128. There may be a more serious error.
= 65536. The return code is not set at all, which should not happen - please notify us of this error.
checks: an array of the checks performed (can contain elements such as 'length', 'bank_code', 'account_number', 'iban_checksum').
length_check:'passed' or 'failed' - shows if the IBAN is of the correct length for the given country.
account_check: (not provided for all countries) 'passed' or 'failed'. Gives the result of the checksum validation of the account number. If the algorithm is unknown, or if no checksum exists, the result is empty or 'passed'.
bank_code_check: (not provided for all countries) Directory lookup of domestic bank code was successful ('passed') or not ('failed').
iban_checksum_check: 'passed' or 'failed'. Correctness of the 3rd and 4th characters of the IBAN, i.e. the IBAN checksum.
account_validation_method: name of the validation algorithm for the domestic account number
account_validation: (DE and CH only) An explanation of the account number validation (in German)
iban_candidate: for Germany, the IBAN that would result from converting the domestic account number and bank code which are embedded in the given IBAN. There are some special IBAN rules which could make such a re-calculated IBAN different. For countries without such special IBAN rules, this field is left empty.
IBANformat: A string describing the IBAN format for the given country, for example: 'DEkk BBBB BBBB CCCC CCCC CC'.
formatcomment: Key to the IBANformat string, for example: 'B = sort code (BLZ), C = account No.'
Output: account and bank information (BIC code, bank address, extracted account number etc.)
The following output fields contain additional information about the validated IBAN, such as its BIC code and bank data:

bic_candidates: an array of BICs that can be associated with the given domestic bank code. May be empty or may contain one or more elements. Each BIC element is itself a complex data type that has the attributes 'bic', 'wwwcount', 'sampleurl', and 'city'.
Interpretation: If
wwwcount > 0. The BIC was harvested from the web (and found on as many pages as indicated by 'wwwcount', for example on the page indicated by 'sampleurl').
wwwcount = 0. The BIC comes from a national bank or the ECB.
If 'city' is given, this also indicates that the BIC comes from a national bank or the ECB.
Note: The given city does not necessarily reflect the location of the given branch - it can also be the location of the headquarters.

all_bic_candidates: an array of BIC candidates which includes the BICs listed above, plus possible some more BIC codes which were excluded from the previous array based on some heuristics. Use this array for a somewhat softer check of the validity of a BIC code.
country: the ISO country code (first two letters of the IBAN)
bank_code: the domestic bank code, if it exists. Returns first 4 characters of the BIC for NL (where no domestic bank codes exist), or the BC-Number for Switzerland.
bank_and_branch_code: the concatenated contents of the fields bank_code and branch_code plus a checksum for bank/branch code, if it exists. For example, for Hungary, this field will contain more information than the bank code and branch code fields together. For most other countries, however, this field contains just the contents of those two fields.
bank: Bank name, if known.
bank_address: Bank address, if known.
bank_street: the street from the bank_address field.
bank_city: the city from the bank_address field.
bank_state: the state (if present) from the bank_address field.
bank_postal_code: the postal code from the bank_address field.
bank_url: URL of bank website, if known.
branch: Branch name, if known.
branch_code: Branch code, if known.
in_scl_directory: if at least one BIC is returned (that is, if the array bic_candidates, which is mentioned above, contains at least one element), this field is set to 'yes' or 'no', depending on whether the first returned BIC is listed in the German Bundesbank's SCL directory. If no BIC is returned, this field is left blank.
sct: if in_scl_directory is set to 'yes', this field is set to 'yes' if a SEPA Credit Transfer is supported for the first returned BIC. If no SCT is supported, the field is set to 'no'. If no BIC is returned, the field is left blank.
sdd: if in_scl_directory is set to 'yes', this field is set to 'yes' if SEPA Direct Debit is supported for the first returned BIC. If no SDD is supported, the field is set to 'no'. If no BIC is returned, the field is left blank.
cor1: if in_scl_directory is set to 'yes', this field is set to 'yes' if Core 1 Direct Debit is supported for the first returned BIC. If no Core 1 is supported, the field is set to 'no'.
b2b: if in_scl_directory is set to 'yes', this field is set to 'yes' if SEPA Business to Business is supported for the first returned BIC. If no B2B is supported, the field is set to 'no'. If no BIC is returned, the field is left blank.
scc: if in_scl_directory is set to 'yes', this field is set to 'yes' if SEPA Card Clearing is supported for the first returned BIC. If no SCC is supported, the field is set to 'no'.
sct_inst: "yes" if instant SEPA Credit Transfer is supported, "no" if it is not supported, empty if we cannot find this information for the given IBAN (either because we were not able to determine a BIC or because the BIC is not in our SCT instant table).
sct_inst_readiness_date: the date from which this IBAN supports instant credit transfers, or empty if we cannot find this information.
account_number: Domestic account number.
data_age: (not provided for all countries.) Age of BIC and other bank data. (undefined for data harvested from the web). Format: yyyymmdd.
Output: Fraud prevention
The following output fields can be useful for judging if the validated IBAN is used fraudulently, for instance, harvested from the Web, generated by a fake IBAN generator, or already known to be related to fraud.

You may use these test IBANs for testing the various categories of blacklists: DE27100777770209299700, DE11520513735120710131, AT411100000237571500

iban_listed: either
empty if the IBAN is not on any of our lists or
"www" if it is on our list of IBANs which are publicly visible on the Web or
"fake" if we have found it to have been generated with a fake IBAN generator or
"blacklist" or "schummelrechnungen.de" if it is suspect (e.g., listed on oplichter.eu).
iban_www_occurrences: the number of times we have encountered this IBAN in our monthly web crawl. If greater than zero, the following 7 fields give more details.
www_seen_from: the first date when we encountered this IBAN online. This is most likely later than when it actually appeared online.
www_seen_until: the last date when we encountered this IBAN online. It might or might not be still online.
iban_url: the most prominent URL where we have found this IBAN. See below for how we assess prominence. If iban_www_occurrences is 1 and iban_url contains "IBAN_BLACKLISTED", the IBAN is not necessarily publicly visible but has been reported as suspicious by an unknown user.
url_rank: the number of other websites which receive more traffic than the website specified in iban_url.
url_category: the category of the most prominent website in natural language, if known.
url_min_depth: the smallest nesting depth of the page where the IBAN was found.
url_prominence: this field is reserved for a prominence score.
iban_reported_to_exist: if nonzero: this IBAN was reported by another user to exist and to belong to the account holder you specified.
iban_last_reported: the latest date when this IBAN was reported to exist, in the format YYYYMMDD.
iban_first_seen: the first date when this IBAN was validated on the ad-funded website ibancalculator.com, in the format YYYY-MM-DD HH:MM:SS. IBANs which are used for fraud often are very young and then thrown away. So, if there is a date reported in this field which is more than about 2 weeks old, chances are that this IBAN is legitimate. If the field is empty, this means we cannot say anything about the age of this IBAN.
Output: your account balance
The following output field contains your user account balance on iban-bic.com:

balance: Number of remaining API calls you can do before having to recharge your account. This does not apply to customers who pay their calculations retroactively.

@Data
@NoArgsConstructor
@AllArgsConstructor
public class IbanValidationResponse {
    private String iban;
    private String result;
    private int returnCode;
    private List<String> checks;
    private String lengthCheck;
    private String accountCheck;
    private String bankCodeCheck;
    private String ibanChecksumCheck;
    private String accountValidationMethod;
    private String accountValidation;
    private String ibanCandidate;
    private String ibanFormat;
    private String formatComment;
    private List<BicCandidate> bicCandidates;
    private List<BicCandidate> allBicCandidates;
    private String country;
    private String bankCode;
    private String bankAndBranchCode;
    private String bank;
    private String bankAddress;
    private String bankStreet;
    private String bankCity;
    private String bankState;
    private String bankPostalCode;
    private String bankUrl;
    private String branch;
    private String branchCode;
    private String inSclDirectory;
    private String sct;
    private String sdd;
    private String cor1;
    private String b2b;
    private String scc;
    private String sctInst;
    private String sctInstReadinessDate;
    private String accountNumber;
    private String dataAge;
    private String ibanListed;
    private int ibanWwwOccurrences;
    private String wwwSeenFrom;
    private String wwwSeenUntil;
    private String ibanUrl;
    private int urlRank;
    private String urlCategory;
    private int urlMinDepth;
    private String urlProminence;
    private int ibanReportedToExist;
    private String ibanLastReported;
    private String ibanFirstSeen;
    private int balance;

    public IbanValidationResponse(String iban, String result) {
        this.iban = iban;
        this.result = result;
    }
}

