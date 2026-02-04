---
applyTo: '**/*.java'
---
# GitHub Copilot Instructions for eIDM Project

## Project Overview
- **Java Version**: 21 (use modern Java 21 features: records, pattern matching, switch expressions, text blocks)
- **Spring Boot**: 3.5.5
- **Spring Cloud**: 2025.0.0
- **Build Tool**: Maven
- **Database**: Oracle (JPA/Hibernate)
- **Mapper**: MapStruct 1.5.5
- **API Docs**: SpringDoc OpenAPI 2.8.13
- **Testing**: JUnit 5, Mockito

## Code Standards

### Naming Conventions
- **Entities**: Suffix with entity name (e.g., `EidmMsgLog`)
- **DTOs**: Suffix with `Dto` (e.g., `EidmMsgLogDto`, `RegistrationDto`)
- **Repositories**: Suffix with `Repository` (e.g., `EidmMsgLogRepository`)
- **Services**: Suffix with `Service` (e.g., `TidMessageService`)
- **Controllers**: Suffix with `Controller` (e.g., `TidMessageController`)
- **Mappers**: Suffix with `Mapper` (e.g., `TidOperationMapper`, `EidmMsgLogMapper`)
- **Operations**: Use `Operation` suffix for operation DTOs (e.g., `CardOperation`, `DriverOperation`)

### Method Naming
- Use `get` prefix for simple getters
- Use `find` prefix for repository queries (e.g., `findById`, `findByCode`)
- Use `generate` prefix for factory methods that create complex objects
- Use `create` prefix for methods that persist new entities
- Use `enquire` prefix for search/query operations (e.g., `enquireMessageLogs`)

### Java 21 Features Usage
- **Use Java Records** for DTOs and immutable data classes (see `CardOperation`, `DriverOperation`, `CompanyOperation`)
- **Use Switch Expressions** with yield for multi-branch logic
- **Use Text Blocks** for multi-line strings (XML, SQL)
- **Use Pattern Matching** where appropriate
- **Constructor Injection** over field injection (use final fields)

## Architecture Patterns

### Data Flow Patterns
Follow these standard data flow patterns for consistency:

**CREATE/UPDATE Operations (Input DTO → Database)**
```
InputDTO → Controller → Service → Mapper (DTO→Entity) → Repository → Database
```
Example:
```java
// Controller receives InputDTO
@PostMapping
public BaseResponse<CompanyDto> create(@Valid @RequestBody CompanyInputDto inputDto) {
    return BaseResponse.wrap(companyService.create(inputDto));
}

// Service uses mapper to convert DTO to entity, then saves
@Transactional
public CompanyDto create(CompanyInputDto inputDto) {
    EidmCompany entity = companyMapper.toEntity(inputDto);
    EidmCompany saved = companyRepository.save(entity);
    return companyMapper.toDto(saved);
}
```

**READ/GET Operations (Database → Output DTO)**
```
Database → Repository → Entity → Service → Mapper (Entity→DTO) → Controller → OutputDTO
```
Example:
```java
// Controller calls service and returns wrapped DTO
@GetMapping("/{id}")
public BaseResponse<CompanyDto> getById(@PathVariable Long id) {
    return BaseResponse.wrap(companyService.getById(id));
}

// Service fetches entity, converts to DTO using mapper
@Transactional(readOnly = true)
public CompanyDto getById(Long id) {
    EidmCompany entity = companyRepository.findById(id)
        .orElseThrow(() -> new RuntimeException("Company not found: " + id));
    return companyMapper.toDto(entity);
}
```

**Key Principles:**
- Controllers never directly interact with repositories or entities
- All entity-DTO conversions happen in the service layer using MapStruct mappers
- Entities never leave the service layer - always convert to DTOs
- Input DTOs and Output DTOs may differ (separate concerns)

### Service Layer
- Use `@Service` annotation
- Use `@Transactional` for write operations
- Use `@Transactional(readOnly = true)` for read-only operations
- Constructor-based dependency injection with final fields
- Use Apache Commons Logging (`Log log = LogFactory.getLog(ClassName.class)`)

### Controller Layer
- Use `@RestController` and `@RequestMapping`
- **Always wrap responses** in `BaseResponse<T>` using `BaseResponse.wrap(payload)`
- Use `@PathVariable`, `@RequestParam`, `@RequestBody` appropriately
- Use `@RequestHeader` for optional headers with defaults (e.g., `@RequestHeader(value = "X-Terminal-Recipient", defaultValue = TidMessageConstant.MTL)`)
- Use `produces = MediaType.APPLICATION_JSON_VALUE` for JSON endpoints
- Use `produces = MediaType.APPLICATION_XML_VALUE` for XML endpoints

### Response Wrapper Pattern
```java
// Success response
@GetMapping("/{id}")
public BaseResponse<EidmMsgLogDto> getById(@PathVariable Long id) {
    return BaseResponse.wrap(service.getById(id));
}

// List/Page response
@GetMapping
public BaseResponse<Page<EidmMsgLogDto>> list(Enquiry enquiry) {
    return BaseResponse.wrap(service.list(enquiry));
}
```

### Exception Handling
- Use custom exceptions extending `RuntimeException` (e.g., `EidmException`, `EidmEntityNotFoundException`)
- Global exception handler via `@ControllerAdvice` (see `ControllerExceptionHandler`)
- Exceptions auto-wrapped in `BaseResponse` with status `FAILURE`
- Use descriptive error messages: `throw new RuntimeException("Message log not found for ID: " + id)`

### DTO and Entity Mapping
- **Always use MapStruct** for entity-to-DTO mapping
- Define mapper interfaces with `@Mapper(componentModel = "spring")`
- Use descriptive mapper names (e.g., `EidmMsgLogMapper`, `TidOperationMapper`)
- **Never expose JPA entities in REST responses** - always convert to DTOs
- For collections, define list mapping methods: `List<EidmMsgLogDto> toEidmMsgLogDtoList(List<EidmMsgLog> entities)`

### Repository Layer
- Extend `JpaRepository<Entity, ID>`
- Use Spring Data JPA method naming conventions
- Use `Specification<Entity>` for dynamic queries
- Custom queries return `Optional<Entity>` for single results

### Validation
- Use Bean Validation annotations: `@NotNull`, `@NotBlank`, `@Valid`
- Validate at controller boundaries with `@Valid` on `@RequestBody`
- Use `@Validated` on classes that need validation groups

## Logging Standards
- Use Apache Commons Logging: `Log log = LogFactory.getLog(ClassName.class)`
- Log method entry: `log.info("methodName invoked. Param: " + param)`
- Log important business events: `log.info("Operation completed. Result: " + result)`
- Log errors with stack traces: `log.error("Error message", exception)`
- Use structured log messages with key context

## Testing Standards
- Use JUnit 5 (`@Test`, `@BeforeEach`, `@DisplayName`)
- Use Mockito for mocking: `@Mock`, `when().thenReturn()`
- Initialize mocks: `MockitoAnnotations.openMocks(this)`
- Use static imports for assertions: `import static org.junit.jupiter.api.Assertions.*`
- Test naming: descriptive names or use `@DisplayName`
- Mock all external dependencies (repositories, external services)

## Transaction Management
- Use `@Transactional` on service methods that modify data
- Use `@Transactional(readOnly = true)` for read-only operations
- Avoid transactions in controllers
- Be explicit about transaction boundaries

## Constants and Configuration
- Use constant classes with `public static final` fields
- Group related constants (e.g., `TidMessageConstant`)
- Use descriptive constant names (e.g., `MSG_STATUS_NEW`, `COMPANY_ACTION_CREATE`)

## Code Style
- Use constructor-based dependency injection (final fields)
- Prefer immutability where possible (Java records, final fields)
- Use meaningful variable names (avoid single letters except in short lambdas)
- Keep methods focused (single responsibility)
- Add JavaDoc for public APIs and complex logic
- Use inline comments sparingly, only for non-obvious logic

## Common Patterns

### Factory Pattern for Complex Object Creation
```java
public static CardOperation create(String cardId, String plateNumber) {
    return new CardOperation(cardId, plateNumber, null, null,
            TidMessageConstant.CARD_ACTION_CREATE, 
            TidMessageConstant.CARD_STATUS_ACTIVE);
}
```

### Dynamic Query Building with Specifications
```java
private Specification<Entity> buildSpecification(Enquiry enquiry) {
    return (root, query, cb) -> {
        List<Predicate> predicates = new ArrayList<>();
        if (enquiry.getField() != null) {
            predicates.add(cb.equal(root.get("field"), enquiry.getField()));
        }
        return cb.and(predicates.toArray(new Predicate[0]));
    };
}
```

### Pagination and Enquiry Pattern
- Use custom `Page<T>` class (not Spring's)
- Use `BaseEnquiry` as base class for search criteria
- Convert Spring Page to custom Page using `BaseEnquiry.convertPage()`

## Security
- Spring Security with OAuth2 resource server
- Do not log sensitive data (credentials, tokens)
- Validate all user inputs

## API Documentation
- Use SpringDoc OpenAPI annotations where needed
- Keep API versioned via properties
- Generate OpenAPI specs to `doc/openapi/`

## Don't Do
- ❌ Don't return JPA entities from controllers
- ❌ Don't use field injection (`@Autowired` on fields)
- ❌ Don't catch exceptions without logging them
- ❌ Don't use magic strings - use constants
- ❌ Don't use old Java patterns (avoid Optional.get() without isPresent check)
- ❌ Don't create deep inheritance hierarchies
- ❌ Don't use `System.out.println` - use logging

## Do
- ✅ Use Java 21 features (records, switch expressions, pattern matching, text blocks)
- ✅ Use constructor injection with final fields
- ✅ Wrap all REST responses in `BaseResponse<T>`
- ✅ Use MapStruct for entity-DTO mapping
- ✅ Use descriptive error messages in exceptions
- ✅ Add `@Transactional` annotations appropriately
- ✅ Log important operations with context
- ✅ Write unit tests with Mockito
- ✅ Use Bean Validation at boundaries
- ✅ Keep code DRY - extract common logic to helper methods
