# CLAUDE.md - Crossover Journals Project Guide

## Project Overview

**Crossover Journals** is a Spring Boot web application for managing and distributing academic/scientific journals. It provides functionality for:
- Publishers to upload and manage journal PDFs
- Users to browse and subscribe to journals by category
- Role-based access control (Publishers vs Public Users)
- PDF document viewing with subscription-based access

**Technology:** Java 8, Spring Boot 1.3.3, MySQL 5.6+, Maven, Thymeleaf

## Technology Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| Language | Java | 8 |
| Framework | Spring Boot | 1.3.3.RELEASE |
| ORM | Spring Data JPA / Hibernate | 5.x |
| Security | Spring Security | 4.x |
| View Engine | Thymeleaf | 2.x |
| Database | MySQL | 5.6+ |
| Build Tool | Maven | 3.3+ |
| Testing | Spring Boot Test, JSONPath | - |

## Architecture Overview

The application follows a **layered architecture** pattern:

```
┌─────────────────────────────────────┐
│     Presentation Layer              │
│  (Controllers, REST Services)       │
├─────────────────────────────────────┤
│     Service Layer                   │
│  (Business Logic)                   │
├─────────────────────────────────────┤
│     Repository Layer                │
│  (Data Access - JPA)                │
├─────────────────────────────────────┤
│     Domain Model                    │
│  (JPA Entities)                     │
└─────────────────────────────────────┘
```

## Directory Structure

```
crossoverjournal/
├── src/
│   ├── main/
│   │   ├── java/com/crossover/trial/journals/
│   │   │   ├── Application.java              # Main entry point
│   │   │   ├── config/                       # Configuration classes
│   │   │   │   ├── MvcConfig.java           # MVC configuration
│   │   │   │   └── WebSecurityConfig.java   # Security configuration
│   │   │   ├── controller/                   # Web controllers
│   │   │   │   ├── JournalController.java   # Journal viewing
│   │   │   │   └── PublisherController.java # Publisher operations
│   │   │   ├── rest/                         # REST API endpoints
│   │   │   │   ├── JournalRestService.java
│   │   │   │   └── CategoryRestService.java
│   │   │   ├── service/                      # Business logic layer
│   │   │   │   ├── JournalService.java      # Interface
│   │   │   │   ├── JournalServiceImpl.java  # Implementation
│   │   │   │   ├── UserService.java
│   │   │   │   ├── UserServiceImpl.java
│   │   │   │   ├── CurrentUser.java         # Security principal
│   │   │   │   ├── CurrentUserDetailsService.java
│   │   │   │   └── ServiceException.java    # Custom exception
│   │   │   ├── repository/                   # Data access layer
│   │   │   │   ├── JournalRepository.java
│   │   │   │   ├── CategoryRepository.java
│   │   │   │   ├── PublisherRepository.java
│   │   │   │   └── UserRepository.java
│   │   │   ├── model/                        # JPA entities
│   │   │   │   ├── Journal.java             # Main entity
│   │   │   │   ├── Category.java            # Journal categories
│   │   │   │   ├── Publisher.java           # Publishers
│   │   │   │   ├── User.java                # User accounts
│   │   │   │   ├── Subscription.java        # User subscriptions
│   │   │   │   └── Role.java                # User roles enum
│   │   │   └── dto/                          # Data transfer objects
│   │   │       └── SubscriptionDTO.java
│   │   └── resources/
│   │       ├── application.properties        # App configuration
│   │       ├── static/                       # Static assets
│   │       │   ├── css/
│   │       │   └── js/
│   │       └── templates/                    # Thymeleaf templates
│   │           ├── home.html
│   │           ├── login.html
│   │           ├── journals.html
│   │           ├── 403.html
│   │           └── publisher/
│   │               ├── browse.html
│   │               └── publish.html
│   └── test/
│       └── java/com/crossover/trial/journals/
│           └── rest/
│               ├── JournalServiceTest.java
│               └── CategoryRestServiceTest.java
├── PDFs/                                     # Sample PDF files
│   ├── 1/
│   └── 2/
├── pom.xml                                   # Maven configuration
├── Instructions.txt                          # Setup instructions
├── README.md                                 # Basic readme
└── .gitignore                               # Git ignore rules

```

## Domain Model

### Core Entities

**Journal** (`model/Journal.java`)
- Primary entity representing a published journal
- Fields: `id`, `name`, `publishDate`, `uuid`, `publisher`, `category`
- Relationships: ManyToOne with Publisher and Category
- Uses `@PrePersist` to auto-set publish date

**User** (`model/User.java`)
- Represents system users (publishers and public users)
- Fields: `id`, `loginName`, `pwd`, `enabled`, `role`, `subscriptions`
- Relationships: OneToMany with Subscription
- Role-based: PUBLISHER or USER

**Publisher** (`model/Publisher.java`)
- Extends User functionality for publishers
- Manages journal publications

**Category** (`model/Category.java`)
- Categorizes journals (e.g., Science, Engineering)
- Used for subscription management

**Subscription** (`model/Subscription.java`)
- Links users to categories they're subscribed to
- ManyToOne relationships with User and Category

**Role** (`model/Role.java`)
- Enum: PUBLISHER, USER

## Key Components

### Service Layer

**JournalService** (`service/JournalService.java`)
- Interface defining journal operations
- Methods:
  - `listAll(User user)` - List journals available to user
  - `publisherList(Publisher publisher)` - List publisher's journals
  - `publish(Publisher, Journal, categoryId)` - Publish new journal
  - `unPublish(Publisher, journalId)` - Remove journal

**Pattern:** Service layer uses interface + implementation pattern
- Interfaces in `service/`
- Implementations suffixed with `Impl`

### Repository Layer

**Pattern:** Spring Data JPA repositories
- Extend `CrudRepository<Entity, ID>`
- Custom query methods using naming conventions
- Example: `findByPublisher(Publisher)`, `findByCategoryIdIn(List<Long>)`

Located in: `repository/`

### Controller Layer

**JournalController** (`controller/JournalController.java`)
- Handles journal viewing (`/view/{id}`)
- Validates user subscriptions before serving PDFs
- Returns PDF as `application/pdf` response

**PublisherController** (`controller/PublisherController.java`)
- Handles publisher-specific operations
- Journal upload and management
- Restricted to users with PUBLISHER role

### REST Services

Located in: `rest/`
- `JournalRestService` - Journal REST API
- `CategoryRestService` - Category REST API

## Configuration

### Application Properties

**File:** `src/main/resources/application.properties`

Key configurations:
```properties
# Database
spring.datasource.url=jdbc:mysql://localhost:3306/journals?createDatabaseIfNotExist=true
spring.datasource.username=root
spring.datasource.password=mysql

# JPA/Hibernate
spring.jpa.hibernate.ddl-auto=create  # WARNING: Recreates schema on startup
spring.jpa.show-sql=true              # Logs SQL statements

# File Upload
multipart.maxFileSize=1MB
multipart.maxRequestSize=1MB
```

**IMPORTANT:**
- `ddl-auto=create` drops and recreates database on every startup
- Upload directory: Configurable via `-Dupload-dir=<path>` system property
- Default upload location: `${user.home}/upload`

### Security Configuration

**File:** `config/WebSecurityConfig.java`

- **Password Encoding:** BCrypt
- **Public Access:** `/home`, `/`, `/css/**`, `/js/**`, `/images/**`, `/public/rest/**`
- **Publisher Access:** `/publisher/**` requires `PUBLISHER` authority
- **Login:** Custom login page at `/login`
- **CSRF:** Disabled (`.csrf().disable()`)

**Authentication:**
- Form-based login
- UserDetailsService implementation via `CurrentUserDetailsService`
- Password stored using BCrypt hashing

## Development Workflows

### Prerequisites

1. Java 8
2. Maven 3.3+
3. MySQL 5.6+ running on localhost:3306
4. Internet connection (for Maven dependencies)

### Initial Setup

1. **Configure Database:**
   - Edit `src/main/resources/application.properties`
   - Set correct MySQL username/password
   - Database `journals` will be auto-created

2. **Install Dependencies:**
   ```bash
   mvn clean install
   ```

### Running the Application

**From project root:**
```bash
mvn spring-boot:run
```

**With custom upload directory:**
```bash
mvn spring-boot:run -Dupload-dir=/custom/path
```

**Access:** http://localhost:8080

### Building

**Create executable JAR:**
```bash
mvn clean package
```

**Output:** `target/journals-1.0.jar`

**Run JAR:**
```bash
java -jar target/journals-1.0.jar
```

### Testing

**Run all tests:**
```bash
mvn test
```

**Test files location:** `src/test/java/com/crossover/trial/journals/`

### Default User Accounts

**Publishers:**
- username: `publisher1` / password: `publisher1`
- username: `publisher2` / password: `publisher2`

**Public Users:**
- username: `user1` / password: `user1`
- username: `user2` / password: `user2`

### Sample Data

- Sample PDFs are in `PDFs/` folder
- Copy to upload directory to view predefined journals
- Default upload dir: `~/upload`

## Coding Conventions

### Java Code Style

1. **Package Structure:** Follow layered architecture
   - `model` - JPA entities only
   - `repository` - Data access interfaces
   - `service` - Business logic (interface + impl)
   - `controller` - Web controllers
   - `rest` - REST endpoints
   - `config` - Configuration classes
   - `dto` - Data transfer objects

2. **Naming Conventions:**
   - Entities: Singular nouns (Journal, User, Category)
   - Repositories: `<Entity>Repository`
   - Services: `<Entity>Service` (interface), `<Entity>ServiceImpl` (implementation)
   - Controllers: `<Entity>Controller`
   - REST Services: `<Entity>RestService`

3. **Annotations:**
   - Use `@Autowired` for dependency injection
   - Use `@Service`, `@Controller`, `@Repository` stereotypes
   - Use `@RequestMapping` for endpoint mapping
   - Use JPA annotations: `@Entity`, `@Id`, `@GeneratedValue`, etc.

4. **JPA Entities:**
   - Always include `@Entity` annotation
   - Use `@Id` and `@GeneratedValue` for primary keys
   - Use `@Column(nullable = false)` for required fields
   - Define relationships with `@ManyToOne`, `@OneToMany`, etc.
   - Use `@PrePersist`, `@PreUpdate` for lifecycle callbacks

### Spring Patterns

1. **Dependency Injection:** Constructor or field injection via `@Autowired`
2. **Repository Pattern:** Extend `CrudRepository<T, ID>`
3. **Service Layer:** Always use interface + implementation
4. **Transaction Management:** `@Transactional` on service methods (if needed)
5. **Exception Handling:** Custom `ServiceException` for business logic errors

### Security Patterns

1. **Authorization:** Use `@AuthenticationPrincipal` to get current user
2. **Password Storage:** Always use BCrypt (configured in `WebSecurityConfig`)
3. **Role Checks:** Use `@PreAuthorize` or check in service layer
4. **Current User:** Access via `CurrentUser` wrapper class

## Testing Conventions

### Test Structure

- **Location:** `src/test/java/com/crossover/trial/journals/`
- **Naming:** `<ClassName>Test.java`
- **Framework:** JUnit + Spring Boot Test
- **Mock Data:** Use test fixtures for entities

### Test Categories

1. **Service Tests:** Test business logic
2. **REST Tests:** Test API endpoints with `@WebMvcTest`
3. **Repository Tests:** Test data access (if needed)

**Example Test:**
```java
@RunWith(SpringRunner.class)
@WebMvcTest
public class JournalServiceTest {
    @Autowired
    private MockMvc mockMvc;

    // Test methods...
}
```

## Common Tasks for AI Assistants

### Adding a New Entity

1. Create entity class in `model/` with JPA annotations
2. Create repository interface in `repository/` extending `CrudRepository`
3. Create service interface in `service/`
4. Create service implementation in `service/` with `@Service`
5. Create controller in `controller/` or REST service in `rest/`
6. Add necessary templates in `resources/templates/`

### Adding a New REST Endpoint

1. Create/update REST service in `rest/`
2. Use `@RequestMapping` for path and method
3. Use `@ResponseBody` for JSON responses
4. Add security rules in `WebSecurityConfig` if needed
5. Write tests in `src/test/java/.../rest/`

### Modifying Security Rules

1. Edit `config/WebSecurityConfig.java`
2. Update `configure(HttpSecurity)` method
3. Use `.antMatchers()` for path patterns
4. Use `.hasAuthority()` for role-based access
5. Test with different user roles

### Adding Database Fields

1. Add field to entity class with `@Column`
2. Add getter/setter methods
3. Update service layer if business logic needed
4. Update DTOs if used in API
5. **NOTE:** Schema will auto-recreate on restart (ddl-auto=create)

### File Upload Handling

1. PDFs stored in upload directory (default: `~/upload`)
2. Filename pattern: `{publisherId}/{journalUuid}.pdf`
3. Use `Application.ROOT` constant for upload directory
4. Check `PublisherController.getFileName()` for path logic
5. Max file size: 1MB (configured in application.properties)

## Important Notes for AI Assistants

### Database Management

- **WARNING:** `spring.jpa.hibernate.ddl-auto=create` recreates schema on every startup
- All data is lost on application restart
- Consider changing to `update` or `validate` for production
- Use SQL scripts in `resources/` for initial data if needed

### File Storage

- PDFs are stored on filesystem, not in database
- Upload directory is configurable via system property
- Default location: `${user.home}/upload`
- Ensure directory exists and has write permissions
- File paths stored as `uuid` in Journal entity

### Security Considerations

- CSRF protection is DISABLED (`.csrf().disable()`)
- Passwords are BCrypt encoded
- Session-based authentication (no JWT/token)
- Role-based access: PUBLISHER vs USER
- Publishers can only see/delete their own journals

### Performance Considerations

- `User.subscriptions` uses `EAGER` fetching
- Consider `LAZY` loading for large subscription lists
- SQL queries logged when `show-sql=true`
- No caching configured by default

### Known Patterns

1. **Service Exceptions:** Use `ServiceException` for business rule violations
2. **Current User Access:** Use `@AuthenticationPrincipal Principal` in controllers
3. **File Serving:** Use `ResponseEntity<byte[]>` with `application/pdf` content type
4. **Repository Queries:** Prefer method name conventions over `@Query`

### Dependencies Version Notes

- Spring Boot 1.3.3 is outdated (released 2016)
- Consider upgrading for security patches
- MySQL connector 5.1.6 is very old
- Java 8 is minimum requirement

### When Making Changes

1. **Always** test with both PUBLISHER and USER roles
2. **Always** check file permissions for upload directory
3. **Always** verify database connection before running
4. **Always** consider subscription-based access for journal viewing
5. **Always** use BCrypt for any password handling
6. **Never** store passwords in plain text
7. **Never** bypass security checks in controllers
8. **Never** commit `application.properties` with real credentials

### Git Workflow

- **Ignored files:** `target/`, `*.class`, IDE files (see `.gitignore`)
- **Included:** Source code, templates, static assets, PDFs folder
- **Database:** Not in git, created automatically on startup
- **Uploads:** Not in git (user-specific data)

## Troubleshooting

### Common Issues

1. **Port 8080 already in use:**
   - Add `server.port=8081` to application.properties
   - Or stop other application using port 8080

2. **Database connection failed:**
   - Verify MySQL is running on localhost:3306
   - Check username/password in application.properties
   - Ensure MySQL user has CREATE DATABASE privilege

3. **File upload fails:**
   - Check upload directory exists
   - Verify write permissions
   - Check file size < 1MB

4. **403 Forbidden:**
   - Verify user has correct role (PUBLISHER for /publisher/**)
   - Check security configuration
   - Verify user is authenticated

5. **PDF not displaying:**
   - Ensure user has subscription to journal's category
   - Or user is the publisher of the journal
   - Check file exists in upload directory

## Additional Resources

- **Spring Boot 1.3.x Docs:** https://docs.spring.io/spring-boot/docs/1.3.x/reference/htmlsingle/
- **Spring Data JPA:** https://docs.spring.io/spring-data/jpa/docs/current/reference/html/
- **Spring Security:** https://docs.spring.io/spring-security/reference/
- **Thymeleaf:** https://www.thymeleaf.org/documentation.html

---

**Last Updated:** 2025-11-13
**Project Version:** 1.0
**Spring Boot Version:** 1.3.3.RELEASE
**Java Version:** 8
