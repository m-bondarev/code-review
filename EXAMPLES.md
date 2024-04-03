1) Repeated not null check
```java
@Data
@Entity
@Table(name = "BK_Transaction")
public class Transaction {

  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  @NotNull
  @Column(nullable = false)
  private String currency; // should be Currency not String

  @NotNull
  @Column(nullable = false)
  private BigDecimal amount;

  @NotNull
  @Column(nullable = false)
  private String sourceaccountid; // incorrect naming. Should be sourceAccountId
}
```

And then in the controller we don't need to check for not null since we've already annotated the fields using **@NotNull** annotation: 
```java
@PatchMapping(path = "/{transactionId}", consumes = "application/json")
public Transaction patchTransaction(
			@PathVariable("transactionId") Long transactionId,
			@RequestBody Transaction patch
) {
  Transaction transaction = transactionRepo.findById(transactionId).get();

  if (patch.getCurrency() != null) {
    transaction.setCurrency(patch.getCurrency());
  }
  if (patch.getAmount() != null) {
    transaction.setAmount(patch.getAmount());
  }
  if (patch.getSourceaccountid() != null) {
    transaction.setSourceaccountid(patch.getSourceaccountid());
  }

  return transactionRepo.save(transaction);
}
```

2) Use **ifPresentOrElse** method for Optional:
```java
  if (transaction.isPresent()) {
    logger.info(LOG_PREFIX + ".getTransactionById | Transaction with ID : {} found : {}", id,
    transaction.get().toString());
  } else {
    logger.info(LOG_PREFIX + ".getTransactionById | Transaction NOT found with ID : {}", id);
  }
```

For example:
```java
 transaction.ifPresentOrElse(
     o -> log.info(LOG_PREFIX + ".getTransactionById | Transaction with ID : {} found : {}", id, transaction.get().toString()),
     () -> log.info(LOG_PREFIX + ".getTransactionById | Transaction NOT found with ID : {}", id)
 );
```
3) Using Integer type for id:
```java
public interface TransactionRepository extends JpaRepository<Transaction,Integer>  {

}
```
It would be better to use Long or UUID

4) Using double for money amount:
```java
  private Double amount;
```

```java
  private Double fee = transaction.getAmount() * FeeConstants.FACTOR;
```

5) Incorrect controller mappings:

```java
  @PostMapping("/saveTransaction")
  public Transaction saveTransaction(@RequestBody Transaction transaction) {
    
    return transactionRepository.save(transaction);
  }
  
  @GetMapping("/getTransactions")
  public List<Transaction> getAllTransactions(){
    return transactionRepository.findAll();
  }
```
It's better to use a noun e.g. **transactions**:
```java
@RestController
@RequestMapping("/transactions")
public class TransactionController {

  @GetMapping
  public List<Transaction> getAllTransactions(){
    return transactionRepository.findAll();
  }

  @PostMapping
  @ResponseStatus(HttpStatus.OK)
  public Transaction saveTransaction(@RequestBody Transaction transaction) {

    return transactionRepository.save(transaction);
  }
}
```

6) Using @SpringBootTest for junit tests
It should be used for integration tests when the whole context should be started
```java
@SpringBootTest
class TransactionProjectsApplicationTests {

  @Mock
  TransactionRepository transactionRepository;

  @InjectMocks
  TransactionController transactionController;

  void someMethod() { // junit test
    when(transactionRepository.findAll()).thenReturn(List.of());
  
    var transactions = transactionController.getAllTransactions();
    
    assertThat(transactions).isEmpty();
  }
}
```

7) Creating your own enum for Currency
   There is already java.util.Currency with ISO 4217 - Currency codes

   http://www.iso.org/iso/home/standards/currency_codes.htm

```java
public enum Currency {
    AED("AED"), AFN("AFN"), ALL("ALL"), AMD("AMD"), ANG("ANG"), AOA("AOA"), ARS("ARS"), AUD("AUD"), AWG("AWG"), AZN("AZN"), BAM("BAM"), BBD("BBD"),
    BDT("BDT"), BGN("BGN"), BHD("BHD"), BIF("BIF"), BMD("BMD"), BND("BND"), BOB("BOB"), BOV("BOV"), BRL("BRL"), BSD("BSD"), BTN("BTN"), BWP("BWP"),
    BYN("BYN"), BYR("BYR"), BZD("BZD"), CAD("CAD"), CDF("CDF"), CHE("CHE"), CHF("CHF"), CHW("CHW"), CLF("CLF"), CLP("CLP"), CNY("CNY"), COP("COP"),
    COU("COU"), CRC("CRC"), CUC("CUC"), CUP("CUP"), CVE("CVE"), CZK("CZK"), DJF("DJF"), DKK("DKK"), DOP("DOP"), DZD("DZD"), EGP("EGP"), ERN("ERN"),
    ETB("ETB"), EUR("EUR"), FJD("FJD"), FKP("FKP"), GBP("GBP"), GEL("GEL"), GHS("GHS"), GIP("GIP"), GMD("GMD"), GNF("GNF"), GTQ("GTQ"), GYD("GYD"),
    HKD("HKD"), HNL("HNL"), HRK("HRK"), HTG("HTG"), HUF("HUF"), IDR("IDR"), ILS("ILS"), INR("INR"), IQD("IQD"), IRR("IRR"), ISK("ISK"), JMD("JMD"),
    JOD("JOD"), JPY("JPY"), KES("KES"), KGS("KGS"), KHR("KHR"), KMF("KMF"), KPW("KPW"), KRW("KRW"), KWD("KWD"), KYD("KYD"), KZT("KZT"), LAK("LAK"),
    LBP("LBP"), LKR("LKR"), LRD("LRD"), LSL("LSL"), LYD("LYD"), MAD("MAD"), MDL("MDL"), MGA("MGA"), MKD("MKD"), MMK("MMK"), MNT("MNT"), MOP("MOP"),
    MRO("MRO"), MUR("MUR"), MVR("MVR"), MWK("MWK"), MXN("MXN"), MXV("MXV"), MYR("MYR"), MZN("MZN"), NAD("NAD"), NGN("NGN"), NIO("NIO"), NOK("NOK"),
    NPR("NPR"), NZD("NZD"), OMR("OMR"), PAB("PAB"), PEN("PEN"), PGK("PGK"), PHP("PHP"), PKR("PKR"), PLN("PLN"), PYG("PYG"), QAR("QAR"), RON("RON"),
    RSD("RSD"), RUB("RUB"), RWF("RWF"), SAR("SAR"), SBD("SBD"), SCR("SCR"), SDG("SDG"), SEK("SEK"), SGD("SGD"), SHP("SHP"), SLL("SLL"), SOS("SOS"),
    SRD("SRD"), SSP("SSP"), STD("STD"), SYP("SYP"), SZL("SZL"), THB("THB"), TJS("TJS"), TMT("TMT"), TND("TND"), TOP("TOP"), TRY("TRY"), TTD("TTD"),
    TWD("TWD"), TZS("TZS"), UAH("UAH"), UGX("UGX"), USD("USD"), USN("USN"), UYI("UYI"), UYU("UYU"), UZS("UZS"), VEF("VEF"), VND("VND"), VUV("VUV"),
    WST("WST"), XAF("XAF"), XAG("XAG"), XAU("XAU"), XBA("XBA"), XBB("XBB"), XBC("XBC"), XBD("XBD"), XCD("XCD"), XDR("XDR"), XFU("XFU"), XOF("XOF"),
    XPD("XPD"), XPF("XPF"), XPT("XPT"), XSU("XSU"), XTS("XTS"), XUA("XUA"), XXX("XXX"), YER("YER"), ZAR("ZAR"), ZMW("ZMW");

    private String iso4217Code = "";

    Currency(String code) {
        this.iso4217Code = code;
    }

    @Override
    public String toString() {
        return iso4217Code;
    }
}
```

8) Using HttpStatus.OK for failed response. Optional for account?

```java
if (account == null) {
    return new ResponseEntity<>(Constants.CREATE_ACCOUNT_FAILED, HttpStatus.OK);
} else {
    return new ResponseEntity<>(account, HttpStatus.OK);
}
```

9) Incorrect variable naming and be careful using synchronized.
Also I would suggest to use UUID or another random type for transaction reference instead of creating your own method for that:
```java
public static String transactionReferenceNumber() {
  byte[] lock = new byte[0];
  long w = 100000000;
  long r = 0;
  synchronized (lock) {
    r = (long) ((Math.random() + 1) * w);
  }

  return System.currentTimeMillis() + String.valueOf(r).substring(1);
}
```

10) Complex statement:
```java
if (request != null && request.getCurrency() != null 
    && !request.getCurrency().isBlank() 
    && request.getAmount() != null
    && !request.getAmount().equals("") 
    && request.getAccountId() != null && !request.equals(" ")
) {
  log.info("Inside saveTranaction method of TransactionController");
  return  transactionService.saveTranaction(request);
} else {
  throw new BusinessException(HttpStatus.BAD_REQUEST, "Please provide the required fields.");
}
```

11) Incorrect end of line
```java
public class Simple {

  public static void main(String[] args) {
    int ival = 10;

    if(ival>0); // ; must be removed, otherwise the logic inside the brackets will never be executed
    {
      // some logic here
    }
  }
}
```

