---
name: java-coding-guide
description: Comprehensive Java backend coding standards for enterprise/Spring Boot projects — includes naming conventions, Javadoc rules, BusinessException + ErrorCode pattern, logging best practices, Lombok usage, code layout, enum standards, and anti-patterns to avoid.
license: Apache-2.0
metadata:
  author: Niveclo
  version: "1.0.0"
---

# Java Coding Guide

This skill provides a comprehensive Java coding guide to ensure code readability, consistency, and maintainability.

## When to Use This Skill
This skill should be invoked when:
- **Writing new Java code**: To ensure adherence to naming, layout, and structure standards.
- **Refactoring**: To modernize code using Lombok, Stream API, and improved error handling.
- **Code Reviews**: To verify that the implementation follows the project's established best practices.
- **Troubleshooting**: To ensure logging and exceptions are handled according to the enterprise standard.

> **Note on Code Examples**: To maintain conciseness, all code snippets in this document omit Javadoc. In actual production environments, Javadoc must be provided in accordance with the standards defined in Section 1.

## 1. Javadoc Standards

### 1.1 Class-Level Javadoc
Use the following tags for classes, interfaces, and enums:
- `@since`: Specifies the version or date since the class was introduced.
- `@see`: (If applicable) References related classes or methods.
- `@deprecated`: (If applicable) Indicates that the class is no longer recommended for use, with an explanation of the alternative.

**Note:** `@author` and `@version` tags are **deprecated** and should not be used.

### 1.2 Method-Level Javadoc
Use the following tags for public methods:
- `@param`: Describes the method parameters.
- `@return`: Describes the return value.
- `@throws`: Lists exceptions that the method might throw.
- `@see`: (If applicable) References related methods or classes.
- `@deprecated`: (If applicable) Explains why the method is deprecated and what to use instead.

### 1.3 Formatting and Style
- **Language Consistency**: Use the same language (e.g., Chinese or English) as the existing comments in the project or file.
- **HTML Tags**: Use HTML tags (e.g., `<p>`, `<b>`, `{@code}`, `<ul>`, `<li>`) to improve readability.
- **Punctuation**: Every sentence in Javadoc must end with a period (`.`), regardless of the language.
- **Spacing**: Add a space between Chinese characters and English letters/numbers (e.g., "使用 Java 8" instead of "使用Java8").

## 2. Comments

- **Match Language**: Match the language of the existing code base when writing new comments.
- **No End-of-Line Comments**: Do not add comments at the end of a line of code. Comments should occupy their own line above the code they describe.
- **Single-Line Comments**: Use `//` followed by a single space before the comment text (e.g., `// This is a valid comment`).
- **Comment Tags (TODO/FIXME)**: Use standardized formats for actionable items:
  - `TODO/FIXME(Name, YYYY-MM-DD): Description`
  - Example: `// TODO(Niveclo, 2026-02-13): Add caching layer for this expensive query.`

## 3. Naming Conventions

### 3.1 Interface & Implementation
- **Contract First**: Prefer defining interfaces for Service and DAO layers to clarify available operations at a glance.
- **No Prefixes/Suffixes**: 
  - **Interfaces**: DO NOT use the `I` prefix (e.g., use `UserService`, not `IUserService`).
  - **Implementations**: DO NOT use the `Impl` suffix (e.g., use `DefaultUserService`, not `UserServiceImpl`).
- **Naming Implementation**: 
  - If there is only one primary implementation, use the `Default` prefix (e.g., `DefaultUserService`).
  - If there are multiple implementations, use a descriptive prefix based on its strategy or technology (e.g., `MysqlUserRepository`, `RedisUserRepository`).

### 3.2 General Naming
- **No Abbreviations**: Avoid using abbreviations unless they are widely accepted industry standards (e.g., `ID`, `URL`, `JSON`, `HTTP`).
  - ❌ `userNo`, `authMsg`, `addrStr`
  - ✅ `userNumber`, `authenticationMessage`, `addressString`
- **Meaningful Names**: Use names that clearly describe the purpose. Avoid generic names like `data`, `info`, or `process`.

## 4. Code Length & Complexity

- **Class Length**: Aim for **100–200 lines**. Ideally, a class should not exceed **500 lines**. 
  - **Design Red Flags**: If a class approaches the upper limit, it often indicates a violation of the **Single Responsibility Principle (SRP)**. Such classes should be refactored into smaller, cohesive units.
  - **Dependency Bloat**: Monitor the number of injected dependencies. A class with excessive dependencies (e.g., > 5-7) often becomes a mere **"Orchestrator"** or **"God Object"** that lacks its own domain logic. This is a clear signal for architectural refactoring.
- **Method Length**: Aim for **5–15 lines**. Ideally, a method should not exceed **50 lines**. Each method should do only one thing.

## 5. Project Structure

- **Domain-Driven Design (DDD)**: Prioritize a DDD-based package structure (e.g., `domain`, `application`, `infrastructure`, `interfaces`) unless the project explicitly dictates otherwise.
- **Package Naming**: Use lowercase, dot-separated names (e.g., `com.company.project.module`).

## 6. Best Practices

- **Avoid Magic Numbers**: Extract numeric literals into meaningful constants (e.g., `private static final int MAX_RETRY_COUNT = 3;`).
- **Modern Java Features**: 
  - Use `Stream API` for collection processing.
  - Use `try-with-resources` for automatic resource management.
  - Use `Optional` to avoid `null` returns and checks.
- **Immutability**: Prefer `final` for variables and fields that do not change. Use unmodifiable collections where possible.

## 7. Error Handling & Exceptions

Exception handling should be centralized, informative, and closely integrated with logging.

### 7.1 Business Exceptions
- **Internal Exception Design**: This design is tailored for **internal exceptions**. When caught globally, only `BusinessException.getMessage()` is exposed to the frontend. Therefore, error code Enums do not require a numeric `code` field.
- **Error Code Interface**: Define a simple interface for all error code Enums.
  ```java
  public interface ErrorCode {
  
      String getDescription();
  
  }
  ```
- **Enum Implementation**: Use Enums to categorize error messages. The Enum constant name itself serves as the machine-readable code if needed internally.
  ```java
  public enum UserErrorCode implements ErrorCode {
  
      UNKNOWN("Unknown user error"),
  
      USER_NOT_FOUND("User does not exist");
  
      private final String description;
  
      UserErrorCode(String description) {
          this.description = description;
      }
  
      @Override
      public String getDescription() {
          return description;
      }
  
  }
  ```
- **Business Exception Class**: The `BusinessException` should handle the mapping between the `ErrorCode` and the exception message/code.
  ```java
  public class BusinessException extends RuntimeException {
  
      private final String code;
  
      public BusinessException(ErrorCode errorCode) {
          super(errorCode.getDescription());
          this.code = errorCode instanceof Enum<?> e ? e.name() : "UNKNOWN";
      }
  
      public BusinessException(ErrorCode errorCode, String detail) {
          super(errorCode.getDescription() + (detail != null ? "：" + detail : ""));
          this.code = errorCode instanceof Enum<?> e ? e.name() : "UNKNOWN";
      }
  
      public BusinessException(ErrorCode errorCode, Throwable cause) {
          super(errorCode.getDescription(), cause);
          this.code = errorCode instanceof Enum<?> e ? e.name() : "UNKNOWN";
      }
  
      public BusinessException(ErrorCode errorCode, String detail, Throwable cause) {
          super(errorCode.getDescription() + (detail != null ? "：" + detail : ""), cause);
          this.code = errorCode instanceof Enum<?> e ? e.name() : "UNKNOWN";
      }
  
  }
  ```
- **Throwing Exceptions**:
  ```java
  if (user == null) {
      throw new BusinessException(UserErrorCode.USER_NOT_FOUND);
  }
  ```

### 7.2 Global Exception Handling
- **Centralized Management**: Use `@ControllerAdvice` (in Spring) or a similar global mechanism to catch and format exception responses.
- **Standardized Response**: Ensure all error responses follow a consistent JSON structure (e.g., `{ "code": 404, "message": "...", "timestamp": "..." }`).

### 7.3 Exception & Logging
- **Log with Context**: Always log exceptions with enough context to debug. Use `ERROR` level for unexpected failures and `WARN` for handled business logic deviations.
- **Avoid Stack Trace Silencing**: Never swallow exceptions. If you catch an exception, either rethrow it, wrap it in a business exception, or log the full stack trace.
  ```java
  try {
      // risk code
  } catch (IOException e) {
      log.error("Failed to process file: {}", fileName, e); // Pass 'e' as the last argument for stack trace
      throw new BusinessException(ErrorCode.FILE_PROCESS_ERROR);
  }
  ```

## 8. Logging Standards

- **Log Every Key Step**: Record logs for every significant operation, including inputs, outputs, and critical state changes.
- **Contextual Information**: Ensure logs include enough parameters to identify and debug issues (e.g., `userId`, `orderId`).
- **Appropriate Log Levels**:
  - `ERROR`: System failures or exceptions that need immediate attention.
  - `WARN`: Unexpected behavior that isn't a failure yet (e.g., business validation failures).
  - `INFO`: Significant business events (e.g., "User login successful").
  - `DEBUG`: Detailed information for troubleshooting.
- **SLF4J Abstraction**: Use SLF4J (Simple Logging Facade for Java) to decouple the application from the logging implementation (e.g., Logback, Log4j2).
- **Parameterized Logging**: Use placeholders `{}` to avoid unnecessary string concatenation (e.g., `log.info("Processing order: {}", orderId);`).
- **Sensitivity**: Never log sensitive data like passwords, PII (Personally Identifiable Information), or secret keys.

## 9. Code Layout

- **Blank Lines**: Use blank lines to separate fields, constructors, and methods. Also, ensure there are blank lines after the class opening brace and before the closing brace.
  
  ```java
  public class User {
  
      private String username;
  
      private String email;
  
      public User(String username, String email) {
          this.username = username;
          this.email = email;
      }
  
  }
  ```
- **Member Ordering**:
  1. **Fields**: All fields (static and instance) at the top.
  2. **Constructors**: Immediately following fields.
  3. **Public Methods**: Grouped by functionality. If implementing an interface, `@Override` methods should follow the order defined in the interface.
  4. **Private Methods**: All private helper methods should be placed after all public methods.
  5. **Inner Classes/Enums**: Placed at the very bottom of the class.

## 10. Enum Standards

- **Field Naming**: Use `code` (int) and `description` (String) for enum properties.
- **Default Value**: Always include an `UNKNOWN` constant with `code = 0` as the first entry to represent an unknown or default state.
- **The `of` Method**: Implement a static `of(int code)` method to retrieve an enum constant by its code. If no match is found, return `UNKNOWN`.
  
  ```java
  public enum UserStatus {
  
      UNKNOWN(0, "Unknown"),
  
      ACTIVE(1, "Active"),
  
      INACTIVE(2, "Inactive");
  
      private final int code;
  
      private final String description;
  
      UserStatus(int code, String description) {
          this.code = code;
          this.description = description;
      }
  
      public static UserStatus of(int code) {
          for (UserStatus status : values()) {
              if (status.code == code) {
                  return status;
              }
          }
          return UNKNOWN;
      }
  
      public int getCode() {
          return code;
      }
  
      public String getDescription() {
          return description;
      }
  
  }
  ```
- **Immutability**: Enum fields should be `private final` to ensure thread safety and consistency.

## 11. Boilerplate Reduction (Lombok & SLF4J)

If the project includes dependencies for **Lombok** or **SLF4J**, prioritize using their annotations to reduce boilerplate code and improve maintainability:

- **Lombok**: Use annotations like `@Data`, `@Getter`, `@Setter`, `@NoArgsConstructor`, and `@AllArgsConstructor` instead of manually writing these methods.
- **SLF4J**: Use the `@Slf4j` annotation to automatically generate a logger instance (usually named `log`), rather than manually defining a static logger field.
- **Dependency Injection**: For Spring-based or similar projects, prefer using `@RequiredArgsConstructor` combined with `final` fields for constructor-based dependency injection. This ensures immutability and facilitates testing.
  ```java
  @Service
  @RequiredArgsConstructor
  public class UserService {
  
      private final UserRepository userRepository;
  
      private final LogService logService;
  
  }
  ```
