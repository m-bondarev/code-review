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

12) Incorrect enumeration naming
```java
public enum ACTION {  // Enum types should be named using singular nouns in UpperCamelCase, reflecting what the enumeration represents. E.g. Action
   DEPOSIT,
   WITHDRAW
}
```

13) Incorrect class naming
```java
public class constants { // Classes should be named in UpperCamelCase -> Constants

   public static final String SUCCESS =
           "Operation completed successfully";
   public static final String NO_ACCOUNT_FOUND =
           "Unable to find an account matching this sort code and account number";

}
```

14) if statement can be simplified
```java
 public static boolean isSearchTransactionValid(TransactionInput transactionInput) {

     if (!isSearchCriteriaValid(transactionInput.getSourceAccount()))
        return false;

     if (!isSearchCriteriaValid(transactionInput.getTargetAccount()))
        return false;

     if (transactionInput.getSourceAccount().equals(transactionInput.getTargetAccount()))
        return false;

     return true;
}
```

This code can be simplified:
```java
 public static boolean isSearchTransactionValid(TransactionInput transactionInput) {

     if (!isSearchCriteriaValid(transactionInput.getSourceAccount()))
        return false;

     if (!isSearchCriteriaValid(transactionInput.getTargetAccount()))
        return false;

     return !transactionInput.getSourceAccount().equals(transactionInput.getTargetAccount());
}
```

15) *public* modifier is redundant for interface methods
```java
public interface TransactionService {
   public TransactionResponse processTransaction(TransactionRequest transactionRequest) throws TransactionProcessingException;
}
```

16) Instantiable utility class
```java
public class InputValidator {

   public static boolean isAccountNoValid(String accountNo) {
      return Constants.ACCOUNT_NUMBER_PATTERN.matcher(accountNo).find();
   }

   public static boolean isCreateAccountCriteriaValid(CreateAccountInput createAccountInput) {
      return (!createAccountInput.getBankName().isBlank() && !createAccountInput.getOwnerName().isBlank());
   }
}
```

Default constructor for utility class should be suppressed:
```java
public class InputValidator {
    
   private InputValidator() {}
   
   public static boolean isAccountNoValid(String accountNo) {
      return Constants.ACCOUNT_NUMBER_PATTERN.matcher(accountNo).find();
   }

   public static boolean isCreateAccountCriteriaValid(CreateAccountInput createAccountInput) {
      return (!createAccountInput.getBankName().isBlank() && !createAccountInput.getOwnerName().isBlank());
   }
}
```

17) Synchronization on non-final field
```java
public class Counter {
    
   private Integer count = 0;
   
   public void increment() {
      synchronized (count) {
          count++;
      }
   }
}
```
Synchronizing on a non-final field is risky because the field can be changed, leading to unexpected behaviour.

18) String concatenation inside the loop
```java
public class StringUtils {
   
   public static String reverseString(String str) {
      String reversed = "";
      for (int i = str.length() - 1; i >= 0; i--) {
          reversed += str.charAt(i);
      }
      return reversed;
   }
}
```

String concatenation is used inside the loop which can result in performance issues when dealing with large strings.
It's better to use StringBuilder for better performance and efficiency:
```java
public class StringUtils {
   
   public static String reverseString(String str) {
      StringBuilder reversed = new StringBuilder();
      for (int i = str.length() - 1; i >= 0; i--) {
          reversed.append(str.charAt(i));
      }
      return reversed.toString();
   }
}
```

19) try-finally with AutoClosable resources
```java
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
       in.close();
    }
}
```

It's preferable to use try-with-resources instead:
```java
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
    OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}
```