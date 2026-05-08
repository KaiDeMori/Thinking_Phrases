Java enterprise architecture sometimes feels like someone looked at:

```java
System.out.println("Hello");
```

…and thought:

> “This is dangerously simple. We need governance.”

Here are some lovingly exaggerated examples of Java’s tendency to turn one line of code into an organizational chart.

---

# 1. Enterprise Hello World™

## Simple version

```java
System.out.println("Hello, world!");
```

## Java Enterprise Edition version

```java
public interface GreetingMessageProvider {
    String provideGreetingMessage();
}
```

```java
public class DefaultGreetingMessageProvider implements GreetingMessageProvider {
    @Override
    public String provideGreetingMessage() {
        return "Hello, world!";
    }
}
```

```java
public interface GreetingMessageProviderFactory {
    GreetingMessageProvider createGreetingMessageProvider();
}
```

```java
public class DefaultGreetingMessageProviderFactory 
        implements GreetingMessageProviderFactory {

    @Override
    public GreetingMessageProvider createGreetingMessageProvider() {
        return new DefaultGreetingMessageProvider();
    }
}
```

```java
public interface GreetingOutputChannel {
    void output(String message);
}
```

```java
public class ConsoleGreetingOutputChannel implements GreetingOutputChannel {
    @Override
    public void output(String message) {
        System.out.println(message);
    }
}
```

```java
public interface GreetingOutputChannelFactory {
    GreetingOutputChannel createOutputChannel();
}
```

```java
public class ConsoleGreetingOutputChannelFactory 
        implements GreetingOutputChannelFactory {

    @Override
    public GreetingOutputChannel createOutputChannel() {
        return new ConsoleGreetingOutputChannel();
    }
}
```

```java
public class GreetingApplicationConfiguration {
    private final GreetingMessageProviderFactory messageProviderFactory;
    private final GreetingOutputChannelFactory outputChannelFactory;

    public GreetingApplicationConfiguration(
            GreetingMessageProviderFactory messageProviderFactory,
            GreetingOutputChannelFactory outputChannelFactory) {
        this.messageProviderFactory = messageProviderFactory;
        this.outputChannelFactory = outputChannelFactory;
    }

    public GreetingMessageProviderFactory getMessageProviderFactory() {
        return messageProviderFactory;
    }

    public GreetingOutputChannelFactory getOutputChannelFactory() {
        return outputChannelFactory;
    }
}
```

```java
public class GreetingApplication {
    private final GreetingApplicationConfiguration configuration;

    public GreetingApplication(GreetingApplicationConfiguration configuration) {
        this.configuration = configuration;
    }

    public void run() {
        GreetingMessageProvider provider =
            configuration.getMessageProviderFactory()
                         .createGreetingMessageProvider();

        GreetingOutputChannel output =
            configuration.getOutputChannelFactory()
                         .createOutputChannel();

        output.output(provider.provideGreetingMessage());
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        GreetingApplicationConfiguration config =
            new GreetingApplicationConfiguration(
                new DefaultGreetingMessageProviderFactory(),
                new ConsoleGreetingOutputChannelFactory()
            );

        GreetingApplication app = new GreetingApplication(config);
        app.run();
    }
}
```

Congratulations. You have printed a string.

The Jira ticket can now move from **In Progress** to **Architecture Review Pending**.

---

# 2. The Factory Factory

Sometimes a factory is not enough. What if you need a factory that creates factories?

```java
public interface ButtonFactory {
    Button createButton();
}
```

Reasonable.

```java
public interface ButtonFactoryFactory {
    ButtonFactory createButtonFactory();
}
```

Getting nervous.

```java
public interface ButtonFactoryFactoryProvider {
    ButtonFactoryFactory provideButtonFactoryFactory();
}
```

Please stop.

```java
public interface ButtonFactoryFactoryProviderManager {
    ButtonFactoryFactoryProvider getManagedProvider();
}
```

Security has been notified.

```java
public class DefaultButtonFactoryFactoryProviderManagerImpl
        extends AbstractConfigurableLifecycleAwareComponent
        implements ButtonFactoryFactoryProviderManager,
                   InitializingBean,
                   DisposableBean,
                   ApplicationContextAware,
                   BeanNameAware,
                   EnvironmentAware {

    private ApplicationContext applicationContext;
    private Environment environment;
    private String beanName;

    @Override
    public ButtonFactoryFactoryProvider getManagedProvider() {
        return new DefaultButtonFactoryFactoryProvider();
    }

    @Override
    public void afterPropertiesSet() {
        System.out.println("Button factory factory provider manager initialized.");
    }

    @Override
    public void destroy() {
        System.out.println("Button factory factory provider manager destroyed.");
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
    }

    @Override
    public void setBeanName(String name) {
        this.beanName = name;
    }

    @Override
    public void setEnvironment(Environment environment) {
        this.environment = environment;
    }
}
```

And somewhere, hidden in production:

```java
button.click();
```

---

# 3. Java Boolean Evaluation Framework

## Normal humans

```java
if (isEnabled) {
    doThing();
}
```

## Enterprise Java

```java
public interface BooleanValue {
    boolean getValue();
}
```

```java
public class ImmutableBooleanValue implements BooleanValue {
    private final boolean value;

    public ImmutableBooleanValue(boolean value) {
        this.value = value;
    }

    @Override
    public boolean getValue() {
        return value;
    }
}
```

```java
public interface BooleanValueEvaluator {
    EvaluationResult evaluate(BooleanValue value);
}
```

```java
public class StandardBooleanValueEvaluator implements BooleanValueEvaluator {
    @Override
    public EvaluationResult evaluate(BooleanValue value) {
        if (value.getValue()) {
            return EvaluationResultFactory.createPositiveEvaluationResult();
        }

        return EvaluationResultFactory.createNegativeEvaluationResult();
    }
}
```

```java
public interface EvaluationResult {
    boolean isAffirmative();
}
```

```java
public class PositiveEvaluationResult implements EvaluationResult {
    @Override
    public boolean isAffirmative() {
        return true;
    }
}
```

```java
public class NegativeEvaluationResult implements EvaluationResult {
    @Override
    public boolean isAffirmative() {
        return false;
    }
}
```

```java
public final class EvaluationResultFactory {
    private EvaluationResultFactory() {
        throw new UnsupportedOperationException(
            "Utility class must not be instantiated by unauthorized constructors."
        );
    }

    public static EvaluationResult createPositiveEvaluationResult() {
        return new PositiveEvaluationResult();
    }

    public static EvaluationResult createNegativeEvaluationResult() {
        return new NegativeEvaluationResult();
    }
}
```

Usage:

```java
BooleanValue enabledValue = new ImmutableBooleanValue(true);

BooleanValueEvaluator evaluator =
    BooleanValueEvaluatorFactoryProvider
        .getInstance()
        .getFactory()
        .createEvaluator();

EvaluationResult result = evaluator.evaluate(enabledValue);

if (result.isAffirmative()) {
    doThing();
}
```

Ticket comment:

> “Refactored boolean logic to improve testability and future extensibility.”

---

# 4. The Abstract Singleton Manager Provider Delegate

You want one thing.

```java
DatabaseConnection connection = Database.connect();
```

Java says:

```java
public interface ConnectionProvider {
    Connection provideConnection();
}
```

```java
public interface ConnectionProviderFactory {
    ConnectionProvider createConnectionProvider();
}
```

```java
public interface ConnectionProviderFactoryManager {
    ConnectionProviderFactory getConnectionProviderFactory();
}
```

```java
public interface ConnectionProviderFactoryManagerDelegate {
    ConnectionProviderFactoryManager delegateConnectionProviderFactoryManager();
}
```

```java
public abstract class AbstractConnectionProviderFactoryManagerDelegateSupport
        implements ConnectionProviderFactoryManagerDelegate {

    protected abstract ConnectionProviderFactoryManager resolveConnectionProviderFactoryManager();

    @Override
    public ConnectionProviderFactoryManager delegateConnectionProviderFactoryManager() {
        return resolveConnectionProviderFactoryManager();
    }
}
```

```java
public class DefaultConnectionProviderFactoryManagerDelegateImpl
        extends AbstractConnectionProviderFactoryManagerDelegateSupport {

    @Override
    protected ConnectionProviderFactoryManager resolveConnectionProviderFactoryManager() {
        return ConnectionProviderFactoryManagerRegistry
            .getInstance()
            .lookup("defaultConnectionProviderFactoryManager");
    }
}
```

Usage:

```java
Connection connection =
    new DefaultConnectionProviderFactoryManagerDelegateImpl()
        .delegateConnectionProviderFactoryManager()
        .getConnectionProviderFactory()
        .createConnectionProvider()
        .provideConnection();
```

The debugger stack trace looks like a CVS receipt.

---

# 5. Enterprise FizzBuzz

## Junior dev

```java
for (int i = 1; i <= 100; i++) {
    if (i % 15 == 0) System.out.println("FizzBuzz");
    else if (i % 3 == 0) System.out.println("Fizz");
    else if (i % 5 == 0) System.out.println("Buzz");
    else System.out.println(i);
}
```

## Senior Java Architect

```java
public interface IntegerSequenceProvider {
    IntegerSequence provideSequence();
}
```

```java
public interface IntegerSequence {
    boolean hasNext();
    Integer next();
}
```

```java
public interface DivisibilityRule {
    boolean appliesTo(Integer input);
    String getReplacementText();
}
```

```java
public class FizzDivisibilityRule implements DivisibilityRule {
    @Override
    public boolean appliesTo(Integer input) {
        return input % 3 == 0;
    }

    @Override
    public String getReplacementText() {
        return "Fizz";
    }
}
```

```java
public class BuzzDivisibilityRule implements DivisibilityRule {
    @Override
    public boolean appliesTo(Integer input) {
        return input % 5 == 0;
    }

    @Override
    public String getReplacementText() {
        return "Buzz";
    }
}
```

```java
public interface DivisibilityRuleAggregationStrategy {
    String aggregate(List<DivisibilityRule> matchingRules);
}
```

```java
public class ConcatenatingDivisibilityRuleAggregationStrategy
        implements DivisibilityRuleAggregationStrategy {

    @Override
    public String aggregate(List<DivisibilityRule> matchingRules) {
        StringBuilder builder = new StringBuilder();

        for (DivisibilityRule rule : matchingRules) {
            builder.append(rule.getReplacementText());
        }

        return builder.toString();
    }
}
```

```java
public interface FizzBuzzEvaluationService {
    String evaluate(Integer input);
}
```

```java
public class DefaultFizzBuzzEvaluationService 
        implements FizzBuzzEvaluationService {

    private final List<DivisibilityRule> rules;
    private final DivisibilityRuleAggregationStrategy aggregationStrategy;

    public DefaultFizzBuzzEvaluationService(
            List<DivisibilityRule> rules,
            DivisibilityRuleAggregationStrategy aggregationStrategy) {
        this.rules = rules;
        this.aggregationStrategy = aggregationStrategy;
    }

    @Override
    public String evaluate(Integer input) {
        List<DivisibilityRule> matchingRules = new ArrayList<>();

        for (DivisibilityRule rule : rules) {
            if (rule.appliesTo(input)) {
                matchingRules.add(rule);
            }
        }

        if (matchingRules.isEmpty()) {
            return input.toString();
        }

        return aggregationStrategy.aggregate(matchingRules);
    }
}
```

```java
public interface OutputSink {
    void write(String value);
}
```

```java
public class ConsoleOutputSink implements OutputSink {
    @Override
    public void write(String value) {
        System.out.println(value);
    }
}
```

```java
public class FizzBuzzOrchestrationManager {
    private final IntegerSequenceProvider sequenceProvider;
    private final FizzBuzzEvaluationService evaluationService;
    private final OutputSink outputSink;

    public FizzBuzzOrchestrationManager(
            IntegerSequenceProvider sequenceProvider,
            FizzBuzzEvaluationService evaluationService,
            OutputSink outputSink) {
        this.sequenceProvider = sequenceProvider;
        this.evaluationService = evaluationService;
        this.outputSink = outputSink;
    }

    public void executeFizzBuzzWorkflow() {
        IntegerSequence sequence = sequenceProvider.provideSequence();

        while (sequence.hasNext()) {
            Integer value = sequence.next();
            String result = evaluationService.evaluate(value);
            outputSink.write(result);
        }
    }
}
```

And then someone opens a PR comment:

> “Can we make the modulo operator injectable?”

---

# 6. The Injectable Modulo Operator

Because obviously `%` is a hard dependency.

```java
public interface RemainderCalculationStrategy {
    int calculateRemainder(int dividend, int divisor);
}
```

```java
public class NativeModuloRemainderCalculationStrategy 
        implements RemainderCalculationStrategy {

    @Override
    public int calculateRemainder(int dividend, int divisor) {
        return dividend % divisor;
    }
}
```

```java
public interface DivisorProvider {
    int provideDivisor();
}
```

```java
public class ThreeDivisorProvider implements DivisorProvider {
    @Override
    public int provideDivisor() {
        return 3;
    }
}
```

```java
public interface DivisibilityAssessmentService {
    boolean isDivisible(int candidate, DivisorProvider divisorProvider);
}
```

```java
public class DefaultDivisibilityAssessmentService 
        implements DivisibilityAssessmentService {

    private final RemainderCalculationStrategy strategy;

    public DefaultDivisibilityAssessmentService(RemainderCalculationStrategy strategy) {
        this.strategy = strategy;
    }

    @Override
    public boolean isDivisible(int candidate, DivisorProvider divisorProvider) {
        return strategy.calculateRemainder(
            candidate,
            divisorProvider.provideDivisor()
        ) == 0;
    }
}
```

Architecture review verdict:

> “Approved, but please add an interface for zero.”

---

# 7. Interface for Zero

```java
public interface ZeroProvider {
    int provideZero();
}
```

```java
public class DefaultZeroProvider implements ZeroProvider {
    @Override
    public int provideZero() {
        return 0;
    }
}
```

```java
public interface ZeroProviderFactory {
    ZeroProvider createZeroProvider();
}
```

```java
public class DefaultZeroProviderFactory implements ZeroProviderFactory {
    @Override
    public ZeroProvider createZeroProvider() {
        return new DefaultZeroProvider();
    }
}
```

```java
public interface ZeroProviderFactoryProvider {
    ZeroProviderFactory provideZeroProviderFactory();
}
```

```java
public class ApplicationContextAwareZeroProviderFactoryProvider
        implements ZeroProviderFactoryProvider {

    private final ApplicationContext context;

    public ApplicationContextAwareZeroProviderFactoryProvider(ApplicationContext context) {
        this.context = context;
    }

    @Override
    public ZeroProviderFactory provideZeroProviderFactory() {
        return context.getBean(ZeroProviderFactory.class);
    }
}
```

Then in production:

```java
if (remainder == zeroProviderFactoryProvider
        .provideZeroProviderFactory()
        .createZeroProvider()
        .provideZero()) {
    return true;
}
```

Performance optimization:

> Cache zero.

---

# 8. The User Service That Knows Too Much

A sane version:

```java
userRepository.save(user);
```

Enterprise version:

```java
public class UserCreationWorkflowOrchestrationServiceFacadeManagerImpl
        implements UserCreationWorkflowOrchestrationServiceFacadeManager {

    private final UserCreationRequestValidatorFactory validatorFactory;
    private final UserEntityAssemblerProvider assemblerProvider;
    private final UserPersistenceGatewayAdapterDelegate persistenceDelegate;
    private final UserCreationDomainEventPublisherFactory eventPublisherFactory;
    private final TransactionBoundaryExecutionTemplateProvider transactionTemplateProvider;
    private final UserCreationAuditTrailRecorderManager auditTrailRecorderManager;

    public UserCreationWorkflowOrchestrationServiceFacadeManagerImpl(
            UserCreationRequestValidatorFactory validatorFactory,
            UserEntityAssemblerProvider assemblerProvider,
            UserPersistenceGatewayAdapterDelegate persistenceDelegate,
            UserCreationDomainEventPublisherFactory eventPublisherFactory,
            TransactionBoundaryExecutionTemplateProvider transactionTemplateProvider,
            UserCreationAuditTrailRecorderManager auditTrailRecorderManager) {
        this.validatorFactory = validatorFactory;
        this.assemblerProvider = assemblerProvider;
        this.persistenceDelegate = persistenceDelegate;
        this.eventPublisherFactory = eventPublisherFactory;
        this.transactionTemplateProvider = transactionTemplateProvider;
        this.auditTrailRecorderManager = auditTrailRecorderManager;
    }

    @Override
    public UserCreationResponseDto executeUserCreationWorkflow(
            UserCreationRequestDto requestDto) {

        return transactionTemplateProvider
            .provideTransactionBoundaryExecutionTemplate()
            .executeWithinTransaction(() -> {

                validatorFactory
                    .createUserCreationRequestValidator()
                    .validateUserCreationRequest(requestDto);

                User user = assemblerProvider
                    .provideUserEntityAssembler()
                    .assembleUserEntityFromUserCreationRequestDto(requestDto);

                User persistedUser = persistenceDelegate
                    .getUserPersistenceGatewayAdapter()
                    .persistUserEntity(user);

                eventPublisherFactory
                    .createUserCreationDomainEventPublisher()
                    .publishUserCreationDomainEvent(
                        new UserCreationDomainEvent(persistedUser)
                    );

                auditTrailRecorderManager
                    .getAuditTrailRecorder()
                    .recordUserCreationAuditTrailEntry(
                        new UserCreationAuditTrailEntry(persistedUser)
                    );

                return new UserCreationResponseDto(persistedUser.getId());
            });
    }
}
```

The method name contains the entire project roadmap.

---

# 9. The Configuration File For Saying “Yes”

```yaml
application:
  feature:
    enablement:
      default:
        affirmative-response:
          lexical-token-provider:
            value: "yes"
          validation:
            strict-mode: true
          output:
            capitalization-strategy: LOWERCASE
          lifecycle:
            initialization:
              eager: false
              dependency-checking: runtime
```

Spring bean:

```java
@Configuration
public class AffirmativeResponseLexicalTokenProviderConfiguration {

    @Bean
    public AffirmativeResponseLexicalTokenProvider 
        affirmativeResponseLexicalTokenProvider(
                AffirmativeResponseLexicalTokenProperties properties) {

        return new ConfigurableAffirmativeResponseLexicalTokenProvider(
            properties.getValue()
        );
    }

    @Bean
    public AffirmativeResponseService affirmativeResponseService(
            AffirmativeResponseLexicalTokenProvider provider,
            AffirmativeResponseValidationStrategy validationStrategy,
            AffirmativeResponseCapitalizationStrategy capitalizationStrategy) {

        return new DefaultAffirmativeResponseService(
            provider,
            validationStrategy,
            capitalizationStrategy
        );
    }
}
```

Usage:

```java
affirmativeResponseService.generateAffirmativeResponse();
```

Output:

```text
yes
```

Sprint velocity: devastated.

---

# 10. The Legendary `SomethingManager`

Java devs love the word `Manager`.

Nobody knows what it manages.

```java
public class FileManager {
    public void manageFile(File file) {
        // Management intensifies
    }
}
```

Better:

```java
public class FileManagementManager {
    private final FileManager fileManager;

    public FileManagementManager(FileManager fileManager) {
        this.fileManager = fileManager;
    }

    public void manageFileManagement(File file) {
        fileManager.manageFile(file);
    }
}
```

More enterprise:

```java
public class FileManagementManagerManager {
    private final FileManagementManager fileManagementManager;

    public FileManagementManagerManager(FileManagementManager fileManagementManager) {
        this.fileManagementManager = fileManagementManager;
    }

    public void manageFileManagementManagerialResponsibilities(File file) {
        fileManagementManager.manageFileManagement(file);
    }
}
```

At some point you get:

```java
public class GlobalAbstractFileManagementManagerManagerFactoryProviderDelegate {
}
```

And it has one method:

```java
public File getFile() {
    return file;
}
```

---

# 11. The DTO Explosion

You start with:

```java
class User {
    String name;
}
```

Then enterprise reality arrives:

```java
UserDto
UserRequestDto
UserResponseDto
UserCreateRequestDto
UserCreateResponseDto
UserUpdateRequestDto
UserUpdateResponseDto
UserDeleteRequestDto
UserDeleteResponseDto
UserSummaryDto
UserDetailsDto
UserViewDto
UserProjectionDto
UserRepresentationModel
UserResource
UserEntity
UserDomainObject
UserAggregateRoot
UserPersistenceModel
UserTransportModel
UserApiModel
UserInternalModel
UserExternalModel
UserCanonicalModel
UserLegacyCompatibilityModel
UserV2BetaExperimentalModel
UserForTheFrontendButNotReallyDto
```

Then someone asks:

> “Can we add a mapper?”

So now:

```java
UserDtoMapper
UserEntityMapper
UserDomainMapper
UserRepresentationAssembler
UserResourceAssembler
UserModelTransformer
UserProjectionAdapter
UserViewConverter
UserTransportTranslator
UserCanonicalNormalizer
UserLegacyCompatibilityHydrator
```

And naturally:

```java
UserMapperFactory
UserMapperFactoryProvider
UserMapperFactoryProviderRegistry
UserMapperFactoryProviderRegistryPostProcessor
```

All to copy:

```java
target.name = source.name;
```

---

# 12. The Null Handling Framework

Normal person:

```java
if (thing != null) {
    thing.doStuff();
}
```

Enterprise Java:

```java
public interface NullabilityAssessmentService<T> {
    NullabilityAssessmentResult assess(T value);
}
```

```java
public enum NullabilityAssessmentResultType {
    NULL,
    NON_NULL,
    UNKNOWN,
    PROBABLY_NULL_BUT_LEGAL_SAID_NOT_TO_ASSUME
}
```

```java
public class NullabilityAssessmentResult {
    private final NullabilityAssessmentResultType type;

    public NullabilityAssessmentResult(NullabilityAssessmentResultType type) {
        this.type = type;
    }

    public boolean indicatesNonNullness() {
        return type == NullabilityAssessmentResultType.NON_NULL;
    }
}
```

```java
public class DefaultNullabilityAssessmentService<T>
        implements NullabilityAssessmentService<T> {

    @Override
    public NullabilityAssessmentResult assess(T value) {
        if (value == null) {
            return new NullabilityAssessmentResult(
                NullabilityAssessmentResultType.NULL
            );
        }

        return new NullabilityAssessmentResult(
            NullabilityAssessmentResultType.NON_NULL
        );
    }
}
```

Usage:

```java
NullabilityAssessmentResult result =
    nullabilityAssessmentService.assess(thing);

if (result.indicatesNonNullness()) {
    thing.doStuff();
}
```

Team lead:

> “Nice. But can we make it reactive?”

---

# 13. Reactive Null Handling Framework

```java
Mono<NullabilityAssessmentResult> resultMono =
    reactiveNullabilityAssessmentService
        .assessNullabilityAsync(Mono.justOrEmpty(thing));

resultMono
    .filter(NullabilityAssessmentResult::indicatesNonNullness)
    .flatMap(result -> reactiveThingInvocationGateway.invokeDoStuff(thing))
    .subscribe();
```

Bug report:

> Sometimes nothing happens.

Senior dev:

> That’s expected. The null was successfully non-blocking.

---

# 14. The Enum That Wanted To Be A Database

```java
public enum Status {
    ACTIVE,
    INACTIVE
}
```

Enterprise version:

```java
@Entity
@Table(name = "status_type_reference_data")
public class StatusTypeReferenceDataEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private Long statusTypeReferenceDataIdentifier;

    @Column(name = "status_type_reference_data_code")
    private String statusTypeReferenceDataCode;

    @Column(name = "status_type_reference_data_description")
    private String statusTypeReferenceDataDescription;

    @Column(name = "status_type_reference_data_effective_start_timestamp")
    private LocalDateTime effectiveStartTimestamp;

    @Column(name = "status_type_reference_data_effective_end_timestamp")
    private LocalDateTime effectiveEndTimestamp;

    @Column(name = "status_type_reference_data_is_active_indicator")
    private Boolean activeIndicator;
}
```

Rows:

| ID | Code | Description |
|---:|------|-------------|
| 1 | ACTIVE | Active |
| 2 | INACTIVE | Inactive |

Migration file:

```sql
INSERT INTO status_type_reference_data
(status_type_reference_data_code, status_type_reference_data_description)
VALUES
('ACTIVE', 'Active'),
('INACTIVE', 'Inactive');
```

Then someone adds caching:

```java
StatusTypeReferenceDataCacheWarmupLifecycleManager
```

The enum died so a table could live.

---

# 15. The Password Validator With 11 Layers

You want:

```java
password.length() >= 8
```

Enterprise demands:

```java
public interface PasswordValidationRule {
    PasswordValidationResult validate(PasswordCandidate candidate);
}
```

```java
public class MinimumLengthPasswordValidationRule 
        implements PasswordValidationRule {

    private final PasswordLengthPolicyProvider policyProvider;

    public MinimumLengthPasswordValidationRule(
            PasswordLengthPolicyProvider policyProvider) {
        this.policyProvider = policyProvider;
    }

    @Override
    public PasswordValidationResult validate(PasswordCandidate candidate) {
        int minimumLength = policyProvider
            .providePasswordLengthPolicy()
            .getMinimumAcceptablePasswordLength();

        if (candidate.getRawPasswordValue().length() >= minimumLength) {
            return PasswordValidationResult.success();
        }

        return PasswordValidationResult.failure(
            PasswordValidationFailureReason.MINIMUM_LENGTH_NOT_SATISFIED
        );
    }
}
```

Then:

```java
PasswordValidationRuleChainFactory
PasswordValidationRuleChainFactoryProvider
PasswordValidationRuleChainExecutionContext
PasswordValidationRuleChainExecutionContextFactory
PasswordValidationRuleChainExecutionManager
PasswordValidationRuleChainExecutionResultMapper
PasswordValidationRuleChainExecutionAuditTrailPublisher
PasswordValidationRuleChainExecutionMetricsCollector
PasswordValidationRuleChainExecutionTracingSpanDecorator
```

Final error message:

```text
Password invalid.
```

---

# 16. The `Impl` Disease

```java
public interface UserService {
    User getUser(Long id);
}
```

```java
public class UserServiceImpl implements UserService {
    @Override
    public User getUser(Long id) {
        return userRepository.findById(id);
    }
}
```

Then one day:

```java
public interface UserServiceImplFactory {
    UserServiceImpl createUserServiceImpl();
}
```

And somewhere:

```java
public class DefaultUserServiceImplFactoryImpl 
        implements UserServiceImplFactory {
}
```

This is how you get class names that sound like they were generated by a printer having a panic attack.

---

# 17. The Overly Honest Java Stack Trace

```text
Exception in thread "main" java.lang.NullPointerException
    at com.company.enterprise.platform.application.core.service.impl.DefaultUserAccountCreationWorkflowExecutionManagerImpl
    at com.company.enterprise.platform.application.core.service.impl.DefaultUserAccountCreationWorkflowExecutionManagerFactoryImpl
    at com.company.enterprise.platform.application.core.service.impl.DefaultUserAccountCreationWorkflowExecutionManagerFactoryProviderImpl
    at com.company.enterprise.platform.application.core.service.impl.DefaultUserAccountCreationWorkflowExecutionManagerFactoryProviderRegistryImpl
    at com.company.enterprise.platform.application.core.service.impl.DefaultUserAccountCreationWorkflowExecutionManagerFactoryProviderRegistryInitializationPostProcessorImpl
    at com.company.enterprise.platform.application.core.service.impl.DefaultUserAccountCreationWorkflowExecutionManagerFactoryProviderRegistryInitializationPostProcessorLifecycleCoordinatorImpl
    at com.company.enterprise.platform.application.Application
Caused by: java.lang.NullPointerException: Cannot invoke "String.trim()" because "name" is null
```

Root cause:

```java
name.trim()
```

But it took the stack trace 47 lines to admit it.

---

# 18. The “Utility” Class

```java
public final class StringUtils {
    private StringUtils() {
        throw new UnsupportedOperationException("Utility class");
    }

    public static boolean isEmpty(String value) {
        return value == null || value.length() == 0;
    }
}
```

Then the company needs “enterprise-grade string operations.”

```java
public interface StringEmptinessEvaluationService {
    boolean evaluateStringEmptiness(StringEvaluationCandidate candidate);
}
```

```java
public class DefaultStringEmptinessEvaluationService
        implements StringEmptinessEvaluationService {

    private final NullabilityAssessmentService<String> nullabilityService;
    private final StringLengthCalculationStrategy lengthCalculationStrategy;

    public DefaultStringEmptinessEvaluationService(
            NullabilityAssessmentService<String> nullabilityService,
            StringLengthCalculationStrategy lengthCalculationStrategy) {
        this.nullabilityService = nullabilityService;
        this.lengthCalculationStrategy = lengthCalculationStrategy;
    }

    @Override
    public boolean evaluateStringEmptiness(StringEvaluationCandidate candidate) {
        return nullabilityService.assess(candidate.getValue()).isNull()
            || lengthCalculationStrategy.calculateLength(candidate.getValue()) == 0;
    }
}
```

Usage:

```java
if (stringEmptinessEvaluationService
        .evaluateStringEmptiness(
            new StringEvaluationCandidateFactory()
                .createStringEvaluationCandidate(input))) {
    // yes, the string is empty
}
```

Someone in the meeting:

> “This will make testing easier.”

Nobody writes tests.

---

# 19. Enterprise Coffee Machine

You just want coffee.

```java
coffeeMachine.brew();
```

Enterprise architecture:

```java
public interface BeveragePreparationRequest {
    BeverageType getRequestedBeverageType();
}
```

```java
public interface BeveragePreparationRequestValidator {
    void validate(BeveragePreparationRequest request);
}
```

```java
public interface BeverageIngredientInventoryAssessmentService {
    InventoryAssessmentResult assessInventoryFor(BeverageType beverageType);
}
```

```java
public interface BeveragePreparationStrategy {
    PreparedBeverage prepare(BeveragePreparationContext context);
}
```

```java
public interface BeveragePreparationStrategyResolver {
    BeveragePreparationStrategy resolveStrategy(BeverageType beverageType);
}
```

```java
public interface ThermalEnergyDeliverySubsystem {
    void deliverThermalEnergy(WaterReservoir reservoir);
}
```

```java
public interface BeanGrindingSubsystemManager {
    GroundBeanPayload grindBeans(BeanGrindingRequest request);
}
```

```java
public interface BeverageDispensationActuatorGateway {
    void actuateDispensation(PreparedBeverage beverage);
}
```

```java
public class DefaultCaffeinatedBeveragePreparationWorkflowOrchestrationManager
        implements CaffeinatedBeveragePreparationWorkflowOrchestrationManager {

    private final BeveragePreparationRequestValidator validator;
    private final BeverageIngredientInventoryAssessmentService inventoryService;
    private final BeveragePreparationStrategyResolver strategyResolver;
    private final BeveragePreparationMetricsCollector metricsCollector;
    private final BeveragePreparationAuditEventPublisher auditEventPublisher;

    public DefaultCaffeinatedBeveragePreparationWorkflowOrchestrationManager(
            BeveragePreparationRequestValidator validator,
            BeverageIngredientInventoryAssessmentService inventoryService,
            BeveragePreparationStrategyResolver strategyResolver,
            BeveragePreparationMetricsCollector metricsCollector,
            BeveragePreparationAuditEventPublisher auditEventPublisher) {
        this.validator = validator;
        this.inventoryService = inventoryService;
        this.strategyResolver = strategyResolver;
        this.metricsCollector = metricsCollector;
        this.auditEventPublisher = auditEventPublisher;
    }

    @Override
    public PreparedBeverage execute(BeveragePreparationRequest request) {
        validator.validate(request);

        InventoryAssessmentResult inventoryResult =
            inventoryService.assessInventoryFor(request.getRequestedBeverageType());

        if (!inventoryResult.isSufficient()) {
            throw new BeveragePreparationInventoryConstraintViolationException();
        }

        BeveragePreparationStrategy strategy =
            strategyResolver.resolveStrategy(request.getRequestedBeverageType());

        PreparedBeverage beverage = strategy.prepare(
            new BeveragePreparationContext(request, inventoryResult)
        );

        auditEventPublisher.publish(new BeveragePreparedAuditEvent(beverage));
        metricsCollector.incrementPreparedBeverageCounter(beverage.getType());

        return beverage;
    }
}
```

Production issue:

```text
CoffeePreparationException: NoSuchBeanDefinitionException:
No qualifying bean of type 'CaffeineExtractionPolicyResolverFactoryProvider'
```

The machine is fine. Spring just can’t find the bean that tells it beans exist.

---

# 20. The Most Java Class Name Ever

```java
public class AbstractSingletonProxyFactoryBeanPostProcessorAdapterDelegateManagerImpl {
}
```

It does nothing.

But it does it in a very enterprise way.

---

# 21. A Typical Meeting

Developer:

> “We need to send an email.”

Architect:

> “Okay. We need an EmailService.”

Developer:

> “Sure.”

Architect:

> “And an EmailServiceImpl.”

Developer:

> “Fine.”

Architect:

> “But we should hide construction behind an EmailServiceFactory.”

Developer:

> “Why?”

Architect:

> “Testing.”

Developer:

> “We can mock the interface.”

Architect:

> “Then we need an EmailServiceFactoryInterface.”

Developer:

> “That’s just an interface.”

Architect:

> “Correct. And an AbstractEmailServiceFactory.”

Developer:

> “Why abstract?”

Architect:

> “Extensibility.”

Developer:

> “For what?”

Architect:

> “Future email.”

Developer:

> “As opposed to present email?”

Architect:

> “Exactly.”

Final package:

```text
com.company.messaging.email.service
com.company.messaging.email.service.impl
com.company.messaging.email.service.api
com.company.messaging.email.service.spi
com.company.messaging.email.factory
com.company.messaging.email.factory.impl
com.company.messaging.email.factory.provider
com.company.messaging.email.factory.provider.impl
com.company.messaging.email.manager
com.company.messaging.email.manager.impl
com.company.messaging.email.delegate
com.company.messaging.email.delegate.impl
com.company.messaging.email.configuration
com.company.messaging.email.configuration.properties
com.company.messaging.email.configuration.autoconfigure
```

Method called:

```java
emailService.send(email);
```

---

# 22. The “Clean Architecture” Onion That Became A Planet

```text
domain
 ├── model
 ├── aggregate
 ├── entity
 ├── valueobject
 ├── service
 ├── policy
 ├── specification
 └── exception

application
 ├── usecase
 ├── command
 ├── query
 ├── handler
 ├── mapper
 ├── dto
 ├── port
 ├── orchestration
 └── workflow

infrastructure
 ├── persistence
 ├── persistence.jpa
 ├── persistence.jpa.entity
 ├── persistence.jpa.repository
 ├── persistence.jpa.mapper
 ├── messaging
 ├── messaging.kafka
 ├── messaging.kafka.producer
 ├── messaging.kafka.consumer
 ├── messaging.kafka.mapper
 └── config

interfaces
 ├── rest
 ├── graphql
 ├── grpc
 ├── websocket
 ├── dto
 ├── mapper
 └── exceptionhandler
```

Feature: toggle dark mode.

Files changed: 89.

---

# 23. The Final Boss: Enterprise Addition

Goal:

```java
int result = 1 + 2;
```

Enterprise:

```java
public interface IntegerAdditionOperand {
    int getValue();
}
```

```java
public class ImmutableIntegerAdditionOperand implements IntegerAdditionOperand {
    private final int value;

    public ImmutableIntegerAdditionOperand(int value) {
        this.value = value;
    }

    @Override
    public int getValue() {
        return value;
    }
}
```

```java
public interface BinaryArithmeticOperationStrategy {
    int execute(IntegerAdditionOperand left, IntegerAdditionOperand right);
}
```

```java
public class IntegerAdditionBinaryArithmeticOperationStrategy
        implements BinaryArithmeticOperationStrategy {

    @Override
    public int execute(IntegerAdditionOperand left, IntegerAdditionOperand right) {
        return left.getValue() + right.getValue();
    }
}
```

```java
public interface BinaryArithmeticOperationStrategyFactory {
    BinaryArithmeticOperationStrategy createStrategy();
}
```

```java
public class IntegerAdditionBinaryArithmeticOperationStrategyFactory
        implements BinaryArithmeticOperationStrategyFactory {

    @Override
    public BinaryArithmeticOperationStrategy createStrategy() {
        return new IntegerAdditionBinaryArithmeticOperationStrategy();
    }
}
```

```java
public interface ArithmeticOperationExecutionManager {
    ArithmeticOperationResult execute(
        IntegerAdditionOperand left,
        IntegerAdditionOperand right
    );
}
```

```java
public class DefaultArithmeticOperationExecutionManager
        implements ArithmeticOperationExecutionManager {

    private final BinaryArithmeticOperationStrategyFactory strategyFactory;

    public DefaultArithmeticOperationExecutionManager(
            BinaryArithmeticOperationStrategyFactory strategyFactory) {
        this.strategyFactory = strategyFactory;
    }

    @Override
    public ArithmeticOperationResult execute(
            IntegerAdditionOperand left,
            IntegerAdditionOperand right) {

        BinaryArithmeticOperationStrategy strategy =
            strategyFactory.createStrategy();

        int result = strategy.execute(left, right);

        return new ImmutableArithmeticOperationResult(result);
    }
}
```

```java
public interface ArithmeticOperationResult {
    int getResultValue();
}
```

```java
public class ImmutableArithmeticOperationResult implements ArithmeticOperationResult {
    private final int resultValue;

    public ImmutableArithmeticOperationResult(int resultValue) {
        this.resultValue = resultValue;
    }

    @Override
    public int getResultValue() {
        return resultValue;
    }
}
```

Usage:

```java
IntegerAdditionOperand left =
    new ImmutableIntegerAdditionOperand(1);

IntegerAdditionOperand right =
    new ImmutableIntegerAdditionOperand(2);

ArithmeticOperationExecutionManager manager =
    new DefaultArithmeticOperationExecutionManager(
        new IntegerAdditionBinaryArithmeticOperationStrategyFactory()
    );

ArithmeticOperationResult result =
    manager.execute(left, right);

System.out.println(result.getResultValue());
```

Output:

```text
3
```

Enterprise achievement unlocked:

> Addition is now loosely coupled.

---

# 24. The PR Description

```text
Title:
Introduce generalized arithmetic operation execution workflow abstraction

Summary:
This PR introduces an extensible framework for binary arithmetic operation
execution using strategy, factory, manager, provider, and immutable result
abstractions.

Changes:
- Added IntegerAdditionOperand interface
- Added ImmutableIntegerAdditionOperand implementation
- Added BinaryArithmeticOperationStrategy interface
- Added IntegerAdditionBinaryArithmeticOperationStrategy implementation
- Added BinaryArithmeticOperationStrategyFactory interface
- Added IntegerAdditionBinaryArithmeticOperationStrategyFactory implementation
- Added ArithmeticOperationExecutionManager interface
- Added DefaultArithmeticOperationExecutionManager implementation
- Added ArithmeticOperationResult interface
- Added ImmutableArithmeticOperationResult implementation

Future work:
- Add subtraction support
- Add multiplication support
- Add configurable arithmetic operation strategy resolver
- Add arithmetic operation audit logging
- Add arithmetic operation observability
- Add arithmetic operation tracing
- Add arithmetic operation metrics
- Add arithmetic operation feature flagging
- Add arithmetic operation result DTO mapping
- Add arithmetic operation exception hierarchy
- Add arithmetic operation execution context
- Add arithmetic operation execution context factory
- Add arithmetic operation execution context factory provider
```

Reviewer comment:

```text
Looks good, but can we avoid using primitive int directly?
```

---

# 25. The Primitive Avoidance Refactor

```java
public interface NumericValue<T extends Number> {
    T unwrap();
}
```

```java
public class IntegerNumericValue implements NumericValue<Integer> {
    private final Integer value;

    public IntegerNumericValue(Integer value) {
        this.value = value;
    }

    @Override
    public Integer unwrap() {
        return value;
    }
}
```

```java
public interface NumericValueFactory<T extends Number> {
    NumericValue<T> create(T value);
}
```

```java
public class IntegerNumericValueFactory implements NumericValueFactory<Integer> {
    @Override
    public NumericValue<Integer> create(Integer value) {
        return new IntegerNumericValue(value);
    }
}
```

Final usage:

```java
NumericValue<Integer> one =
    integerNumericValueFactory.create(Integer.valueOf(1));

NumericValue<Integer> two =
    integerNumericValueFactory.create(Integer.valueOf(2));
```

A Python developer watching from across the room:

```python
1 + 2
```

Java architect:

> “That won’t scale.”

---

# 26. Enterprise `true`

```java
public interface TruthValueProvider {
    boolean provideTruthValue();
}
```

```java
public class ConstantTrueTruthValueProvider implements TruthValueProvider {
    @Override
    public boolean provideTruthValue() {
        return true;
    }
}
```

```java
public interface TruthValueProviderFactory {
    TruthValueProvider createTruthValueProvider();
}
```

```java
public class DefaultTruthValueProviderFactory implements TruthValueProviderFactory {
    @Override
    public TruthValueProvider createTruthValueProvider() {
        return new ConstantTrueTruthValueProvider();
    }
}
```

```java
public interface TruthValueProviderFactoryManager {
    TruthValueProviderFactory getManagedTruthValueProviderFactory();
}
```

```java
public class DefaultTruthValueProviderFactoryManager
        implements TruthValueProviderFactoryManager {

    @Override
    public TruthValueProviderFactory getManagedTruthValueProviderFactory() {
        return new DefaultTruthValueProviderFactory();
    }
}
```

Usage:

```java
boolean truth =
    truthValueProviderFactoryManager
        .getManagedTruthValueProviderFactory()
        .createTruthValueProvider()
        .provideTruthValue();
```

Output:

```text
true
```

Philosophy department:

> “Finally, truth has an abstraction.”

---

# 27. The Enterprise Way To Sleep

```java
Thread.sleep(1000);
```

No.

```java
public interface TemporalSuspensionDurationProvider {
    Duration provideTemporalSuspensionDuration();
}
```

```java
public interface ThreadExecutionSuspensionService {
    void suspendCurrentThreadExecution(
        TemporalSuspensionDurationProvider durationProvider
    );
}
```

```java
public class DefaultThreadExecutionSuspensionService
        implements ThreadExecutionSuspensionService {

    @Override
    public void suspendCurrentThreadExecution(
            TemporalSuspensionDurationProvider durationProvider) {

        try {
            Thread.sleep(durationProvider
                .provideTemporalSuspensionDuration()
                .toMillis());
        } catch (InterruptedException exception) {
            Thread.currentThread().interrupt();
            throw new ThreadExecutionSuspensionInterruptedRuntimeException(
                "Thread execution suspension was interrupted during temporal delay interval.",
                exception
            );
        }
    }
}
```

Usage:

```java
threadExecutionSuspensionService.suspendCurrentThreadExecution(
    new FixedTemporalSuspensionDurationProvider(Duration.ofSeconds(1))
);
```

Someone files a bug:

> “Application hangs for one second.”

Resolution:

> “Works as architected.”

---

# 28. The Ultimate Enterprise Class

```java
package com.company.platform.core.shared.common.internal.framework.support.factory.provider.manager.delegate.registry.configuration.lifecycle.impl;

public final class DefaultAbstractConfigurableSingletonThreadSafeLazyInitializingProxyAwareFactoryBeanProviderManagerDelegateRegistryLifecycleCoordinatorImpl
        extends AbstractConfigurableSingletonThreadSafeLazyInitializingProxyAwareFactoryBeanProviderManagerDelegateRegistryLifecycleCoordinatorSupport
        implements ConfigurableSingletonThreadSafeLazyInitializingProxyAwareFactoryBeanProviderManagerDelegateRegistryLifecycleCoordinator,
                   ApplicationContextAware,
                   BeanFactoryAware,
                   BeanNameAware,
                   EnvironmentAware,
                   InitializingBean,
                   DisposableBean,
                   SmartLifecycle,
                   Ordered,
                   PriorityOrdered {

    private boolean running;

    @Override
    public void start() {
        this.running = true;
    }

    @Override
    public void stop() {
        this.running = false;
    }

    @Override
    public boolean isRunning() {
        return running;
    }

    @Override
    public int getOrder() {
        return Ordered.LOWEST_PRECEDENCE;
    }

    @Override
    public void afterPropertiesSet() {
        // No-op intentionally left blank, required for lifecycle compatibility.
    }

    @Override
    public void destroy() {
        // No-op intentionally left blank, required for lifecycle compatibility.
    }
}
```

Its purpose:

```java
// TODO: determine if still needed
```

---

# 29. The Final Production Incident

Error:

```text
User cannot log in.
```

Investigation:

```text
AuthenticationRequestDtoMapper maps username to userName.
UserNameCanonicalizationService trims userName.
UserNameNormalizationStrategy lowercases userName.
UserIdentityResolutionManager resolves userName.
UserIdentityProviderFactory creates provider.
Provider delegates to repository adapter.
Repository adapter maps to persistence entity.
Persistence entity maps to database row.
Database row returns user_name.
UserNameHydrationService hydrates userName.
UserNameComparisonStrategy compares hydrated userName to normalized userName.
```

Root cause:

```java
username = "";
```

Fix:

```java
if (username.isBlank()) throw new BadRequestException();
```

Files changed: 37.

---

# 30. The Universal Java Punchline

Every Java project starts with:

```java
public static void main(String[] args) {
    System.out.println("Hello");
}
```

And after six months becomes:

```java
public final class ApplicationBootstrapLifecycleOrchestrationManagerImpl
        implements ApplicationBootstrapLifecycleOrchestrationManager {

    private final StartupContextFactoryProvider startupContextFactoryProvider;
    private final ApplicationRunnerDelegateRegistry applicationRunnerDelegateRegistry;
    private final EnvironmentValidationService environmentValidationService;
    private final ObservabilitySubsystemInitializationCoordinator observabilityCoordinator;
    private final GracefulShutdownHookRegistrationManager shutdownHookRegistrationManager;

    public ApplicationBootstrapLifecycleOrchestrationManagerImpl(
            StartupContextFactoryProvider startupContextFactoryProvider,
            ApplicationRunnerDelegateRegistry applicationRunnerDelegateRegistry,
            EnvironmentValidationService environmentValidationService,
            ObservabilitySubsystemInitializationCoordinator observabilityCoordinator,
            GracefulShutdownHookRegistrationManager shutdownHookRegistrationManager) {
        this.startupContextFactoryProvider = startupContextFactoryProvider;
        this.applicationRunnerDelegateRegistry = applicationRunnerDelegateRegistry;
        this.environmentValidationService = environmentValidationService;
        this.observabilityCoordinator = observabilityCoordinator;
        this.shutdownHookRegistrationManager = shutdownHookRegistrationManager;
    }

    public void executeApplicationBootstrapLifecycle() {
        StartupContext context = startupContextFactoryProvider
            .provideStartupContextFactory()
            .createStartupContext();

        environmentValidationService.validateEnvironment(context);
        observabilityCoordinator.initializeObservabilitySubsystem(context);
        shutdownHookRegistrationManager.registerGracefulShutdownHook(context);
        applicationRunnerDelegateRegistry
            .resolvePrimaryApplicationRunnerDelegate()
            .runApplication(context);
    }
}
```

And what does the app do?

```text
Prints "Hello".
```

But now it can print `"Hello"` in a distributed, cloud-native, testable, observable, dependency-injected, horizontally scalable, configuration-driven, strategy-resolved, lifecycle-managed, enterprise-compliant way.

And honestly?

That’s beautiful. Horrifying, but beautiful.
