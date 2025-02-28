@Mapper(componentModel = "spring")
public interface IbanMapper {
    @Mapping(source = "iban", target = "iban")
    @Mapping(source = "bank", target = "bank")
    @Mapping(source = "bankAddress", target = "bankAddress")
    IbanEntity toEntity(IbanValidationResponse response);
}

@Entity
@Data
@NoArgsConstructor
@AllArgsConstructor
public class IbanEntity {
    @Id
    private String iban;
    private String bank;
    private String bankAddress;
}

@Mapper(componentModel = "spring")
public interface IbanMapper {
    @Mapping(source = "iban", target = "iban")
    @Mapping(source = "bank", target = "bank")
    @Mapping(source = "bankAddress", target = "bankAddress")
    IbanEntity toEntity(IbanValidationResponse response);
}