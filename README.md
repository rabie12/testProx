@Mapper(componentModel = "spring")
public interface IbanMapper {
    @Mapping(source = "iban", target = "iban")
    @Mapping(source = "bank", target = "bank")
    @Mapping(source = "bankAddress", target = "bankAddress")
    IbanEntity toEntity(IbanValidationResponse response);
}