
---
title: Software Design
---

<div class="nav-bar">

[<a href="/CS499/">Home</a>] | 
[<a href="/CS499/code-review">Code Review</a>] | 
[<a href="/CS499/software-design">Software Design and Engineering</a>] | 
[<a href="/CS499/databases">Databases</a>] |
[<a href="/CS499/algorithms">Algorithms and Data Structures</a>]  | 


</div>

## Overview
## What I Reviewed

This code review is for *Contact Management System* completed in CS320.


- Key modules/components reviewed:
  - Contact.java
    - Immutable contactId using final keyword
    - Validation at construction time
    - Re-validation at update time
    - Proper encaspulation with private fields
    - Consistent Validation rules across constructor and setters
  - ContactService.java
    - Uses HashMap for efficient 0(1) lookups
    - Prevents duplicate contact IDs in addContact
    - Validates existence before delete and update operations
    - Supports partial updates in updateContact
    - In-memory storage only
    - No persistence
    - Direct Coupling to HashMap implementation
  - ContactTest.java
    - Tests both valid and invalid inputs
    - Comprehensive boundary testing
    - Verifies all validation rules work correctly
    - Uses JUnit 5 assertions
  - Contact ServiceTest.java
    - Tests all CRUD operations
    - Tests error conditions (duplicates, not found)
    - Verifies partials update functionality
    - Good coverage of both happy path and error cases

---
## Strengths Observed


- Proper data validation at both construction and update time
- CRUD operations through the serice layer
- Immutable contactIDs
- Efficient HashMap-based storage with O(1) lookups
- Comprehensive unit test coverage
- Clear separation between domain model (Contact) and service layer (ContactService)
- Code is readable with clear method names and logical structure
- Encapsulation is properly implemented with private fields and public accessors
  
---
## Issues / Risks Identified
- **1: Architecural Issues**
    - The contact Service class is tightly coupled to HashMap storage implementation. There is no abstraction layer, which means if I wanted to change from in-memory storage to a database, file system or cloud storage, I would need to modify the ContactService Class directly.
  - *No Dependency Injection*
    - ContactService creates its own HashMap directly in the class. This makes it difficult to test in isolation and impossible to swap out dependencies without modifying the code.
  - *Missing Separation of Concerns*
    - No clear layered architecture. The service layer is directly handling data storage logic. In a well designed system there should be distinct layers: presentation, business logic, data access, and domain model. 
- **2: Lack of Abstraction**
  - No interfaces defined anywhere in this codebase. This creates several problems:
    - Cannot easily create alternative implementations
    - Cannot use dependency inversion for loose coupling
    - Makes unit testing more difficult 
    - reduce code flexibility and extensibility
- **3: Poor Error Handling Strategy**
  - Code uses generic IllegalArgumentException for all error casses. 
  - Makes it difficult for calling code to distringuish between different tuypes of errors. 
- ** 4: No persistence Layer**
  - In-memory HashMap means all data is lost when the application stops. No mechanism for:
    - Saving Data to disk
    - Loading data at startup
    - Backing up data
    - Recovering from failures
- **5: Limited Testability**
  - Even though there is testing the tight coupling makes it impossible to test the service layer without also testing the stroage. I cannot mock the repository, cannot test differetn storage scenarios, and cannot easily test edge cases related to data persistence. 
-**6: No design Patterns**
  - Code does employ any established design patterns:
    - no repostiory pattern for data access abstraction
    - No factory pattern for object creation
    - No builder pattern for complex object construction
    - No strategy Pattern for different validation or storage strategies.
- **7: Critical Security vulnerabilities**
  - *Thread Safety Issues*
    - HashMap is not thread-safe. If multiple threads try to access the ContactService simultaneiously which is common in web applications or multi-threaded environments there could be race conditions.
  - *No Input Sanitization*
    - When field lengths are checked, there is no validation for malicious content
      - Special characters like ```Java <script>``` , SQL injeciton patterns, or command injection strings are accepted
      - If data is later displayed in a web interface, it could enable XSS attacks
      - Contact names could contain unicode control characters or null bytes that could cause parsing issues
      - PHone numbers are validated as digits, but addresses and names accept any characters
      - No protection against homograph attacks using similar-locking characters
  - *No authentications or authorization*
    - No users or permissions
      - anyone with access to the ContactService can add, modify, or delete ANY contact
      - No way to restrict access based on user roles or ownership
      - No way to implement privacy - Contacts are globally accessed 
      - Violation of least privilege
  - *No Audit Logging*
    - Cannot determine who added, modified, or deleted a contact 
    - No timestamp tracking for changes
    - Cannot investigate security incidents or data breaches
    - No compliance with regulations like GDPR that require audit trails
    - Cannot detect or investigate unauthorized access patterns
- *No input validation on Null Contact Object*
  - In ```Java ContactService.addContact``` its never checked if the contact parameter itself is null:
  - 
  ```java 
    public void addContact(Contact contact){
        if (contacts.containsKey(contact.getContactId()))
        //NullPointerException if contact is null
  ```

- *Denial of Service Vulnerability*
  - NO limit on number of contacts
    - An attacker could add millions of contacts, exhausting memory
    - No rate limiting on operations
    - No resourse quotas or throttling
  
## Target Areas for Improvement

*1: Architecture and Structure*
  - Implement layered architecture with clear separation between presentation, business, data access and domin layers
  - Add abstration layers using interfaces
  - Implement dependency injeciton for loose coupling
  - Apply SOLID principles throughout the codebase

*2. Design Patterns*
  - Implement Repository pattern to abstract data access
  - Add builder pattern for flexible Contact object creation
  - Consider Factory Pattern for creating different types of contacts or services
  - Use stragtegy pattern if multiple validation or storage strategies are needed. 

*3: Error Handling*
  - Create custom exception hieracrchy for specific error types
  - Implement consistent error handling strategy across all layers
  - Add proper error messages with context
  - Ensure exceptions are descriptive and actionable

*4: Persistence and Data Management*
  - Add persistence layer abstraction through repository interfaces
  - Create ability to plug in different storage implementations
  - Ensure data can be saved and loaded
  - Add transaction support for complex operations

*5: Code Quality and Maintainability*
  - Add comprehensive JavaDoc documentation
  - IMprove code organization with proper package structure
  - Add logging framework for observability
  - Implement configuration management for environment specific settings 

*6: Testing and Reliability*
  - Enhance unit tests to use mocking and test in isolation
  - Add integration tests for end-to-end scenarios
  - Improve test coverage to include edge cases
  - -Test different storage implementations

*7: Security*
  - Thread Safety: Use thread-safe collections like Concurrent Hashmap or synchronization mechanisms
  - Input Sanitization: Implement validation for malicious patterns (IE., XSS, injection attacks, special chars)
  - Null Validation: Add null checks for method parameters to prevent Null pointer exploits
  - Audit Logging: Track all CRUD operations with timestamps and user context
  - Data Encryption: Encrypt sensitive fields like phone numbers and addresses
  - Rate Limiting: IMplement throttling to prevent DoS attacks 
  - Secure Error Handling: Avoid information disclosure in error messages
  - Access Control: IMplement ownership model so users can only access their own contacts
  - Resource Limits: Add maximum contact limits per user to prevent memory exhaustion


### Planned Enhancements

- **Enhancement 1: Implement Repository Pattern with Interface Abstraction**
  - I will create an IContactRepository interface that defines the contract for data access operations:
    - save
    - findById
    - existsById
    - deleteById
    - findAll
  - Then I will refactor the existing HashMap code into an InMemoryContactRepository class that implements this interface

*Skills Demonstrated*
  - Understanding of abstraction and interface design
  - Application of Repository design pattern
  - Separation of concerns between business logic and data access

*Course Outcome Alignment*
  - Supports course outcome 3: "Design and evaluate computing solutions wthat solve a given problem using algorithmic principles and computer science practicies and standards appropriate to its solution while managing the trade-offs involved in design choices"

---

**Enhancemen 2: Refactor Service Layer with Dependency Injection**

- I will create an IContactService interface, rename the current ContactService to ContactServiceImpl and refactor it to accept an IcontactRepository through constructor injeciton rather than creating its own HashMap.

**Skills Demonstrated:**
- Dependency Injection principle
- Inversion of Control
- Interface-based programming
- Improved testability through dependency management

**Course Outcome Alignment:**
This supports Course Outcome 1: "Employ strategies for building collaborative environments that enable diverse audiences to support organizational decision-making in the field of computer science."

By making the code more modular and testable, I'm creating an environment where multiple developers can work on different components independently - one team could work on the repository implementation while another works on business logic.

---

### Enhancement 3: Create Custom Exception Hierarchy


I will create a base ContactException class and specific exception types: ContactNotFoundException, DuplicateContactException, and ContactValidationException. I'll refactor all the code to throw these specific exceptions instead of generic IllegalArgumentException.

**Skills Demonstrated:**
- Exception hierarchy design
- Proper error handling strategy
- API design for clear error communication

**Course Outcome Alignment:**
This supports Course Outcome 4: "Demonstrate an ability to use well-founded and innovative techniques, skills, and tools in computing practices for the purpose of implementing computer solutions that deliver value and accomplish industry-specific goals."

Custom exceptions are an industry-standard practice that improves code clarity, debugging, and error handling throughout the application.

---

### Enhancement 4: Implement Builder Pattern for Contact Creation


I will create a ContactBuilder class that provides a fluent interface for constructing Contact objects. This will use method chaining and make it easier to create contacts with optional field updates.

**Skills Demonstrated:**
- Builder design pattern implementation
- Fluent API design
- Improved code readability and usability

**Course Outcome Alignment:**
This supports Course Outcome 3 by demonstrating evaluation of design patterns and choosing appropriate solutions for object creation complexity.

---

### Enhancement 5: Add Comprehensive JavaDoc and Documentation


I will add complete JavaDoc comments to all public classes, interfaces, and methods.

**Skills Demonstrated:**
- Technical documentation
- API documentation standards
- Clear communication of design decisions

**Course Outcome Alignment:**
This strongly supports Course Outcome 2 by providing professional-quality written technical communication.

---
### Enhancement 6: Add Logging Framework


I will integrate SLF4J with Logback as the logging implementation. I'll add appropriate logging at different levels (INFO, DEBUG, WARN, ERROR) throughout the application to track operations, errors, and system behavior.

**Skills Demonstrated:**
- Application observability
- Debugging support
- Production-ready code practices

**Course Outcome Alignment:**
This supports Course Outcome 4 by implementing industry-standard tools and practices for delivering production-quality software.

---

### Enhancement 7: Improve Unit Tests with Mocking


I will refactor the unit tests to use Mockito for mocking dependencies. The ContactServiceImpl tests will mock the repository, allowing true unit testing without depending on the actual storage implementation.

**Skills Demonstrated:**
- Advanced testing techniques
- Test isolation and independence
- Proper unit testing methodology

**Course Outcome Alignment:**
This supports Course Outcome 3 by demonstrating understanding of testing practices and standards for evaluating computing solutions.

---

### Enhancement 8: Implement Critical Security Improvements


I will address the security vulnerabilities identified in my analysis:

1. **Thread Safety:** Replace HashMap with ConcurrentHashMap in the InMemoryContactRepository to prevent race conditions and data corruption in multi-threaded environments.

2. **Input Sanitization:** Create a ContactValidator class that checks for:
   - XSS patterns (script tags, javascript: protocols)
   - Special characters that could cause injection attacks
   - Unicode control characters and null bytes
   - Implement whitelist validation for names (letters, spaces, hyphens only)
   - Ensure addresses don't contain HTML or script content

3. **Null Parameter Validation:** Add null checks at the service layer:
```java
public void addContact(Contact contact) {
    if (contact == null) {
        throw new ContactValidationException("Contact cannot be null");
    }
    // ... rest of method
}
```

1. **Audit Logging:** Integrate the logging framework to track all operations:
   - Log contact creation with timestamp and contact ID
   - Log updates with before/after values
   - Log deletions with contact details
   - Log failed operations and validation errors
   - Include operation metadata for security auditing

2. **Secure Error Messages:** Refactor exception handling to avoid information disclosure:
   - Don't reveal whether contact IDs exist in error messages to unauthorized users
   - Provide detailed errors only in logs, generic messages to clients
   - Never expose stack traces to end users

3. **Resource Limits:** Add maximum contact limits to prevent DoS:
   - Define MAX_CONTACTS constant (e.g., 10,000 per instance)
   - Check size before adding new contacts
   - Throw appropriate exception when limit reached

**Skills Demonstrated:**
- Security-first development mindset
- Understanding of OWASP Top 10 vulnerabilities
- Defensive programming techniques
- Thread-safe programming
- Input validation and sanitization
- Security logging and auditing
- Resource management and DoS prevention

**Course Outcome Alignment:**
This strongly supports Course Outcome 5: "Develop a security mindset that anticipates adversarial exploits in software architecture and designs to expose potential vulnerabilities, mitigate design flaws, and ensure privacy and enhanced security of data and resources."

By systematically identifying and addressing security vulnerabilities, I'm demonstrating the ability to think like an attacker, anticipate exploits, and implement proper security controls. This includes:
- Anticipating race conditions in concurrent access
- Recognizing injection attack vectors
- Understanding information disclosure risks
- Preventing denial of service attacks
- Implementing defense in depth through multiple security layers

**Security Enhancement Pseudocode:**

```pseudocode
CLASS ContactValidator:
    PRIVATE CONSTANT XSS_PATTERNS = ["<script", "javascript:", "onerror=", ...]
    PRIVATE CONSTANT MAX_CONTACT_NAME_LENGTH = 10

    METHOD validateForMaliciousContent(input: String) -> boolean:
        IF input IS null THEN
            RETURN false
        END IF

        FOR EACH pattern IN XSS_PATTERNS:
            IF input.toLowerCase().contains(pattern) THEN
                THROW ContactValidationException("Input contains unsafe content")
            END IF
        END FOR

        // Check for control characters
        FOR EACH char IN input:
            IF char.isControlCharacter() AND char != '\n' AND char != '\t' THEN
                THROW ContactValidationException("Input contains invalid characters")
            END IF
        END FOR

        RETURN true
    END METHOD

    METHOD validateName(name: String) -> boolean:
        IF NOT name.matches("^[a-zA-Z\\s\\-']+$") THEN
            THROW ContactValidationException("Name contains invalid characters")
        END IF
        RETURN true
    END METHOD
END CLASS

// Thread-safe repository implementation
CLASS InMemoryContactRepository:
    PRIVATE contacts: ConcurrentHashMap<String, Contact>
    PRIVATE CONSTANT MAX_CONTACTS = 10000
    PRIVATE logger: Logger

    METHOD save(contact: Contact):
        IF contacts.size() >= MAX_CONTACTS THEN
            logger.warn("Maximum contact limit reached")
            THROW ResourceLimitException("Cannot add more contacts")
        END IF

        contacts.put(contact.getContactId(), contact)
        logger.info("Contact saved: ID={}", contact.getContactId())
    END METHOD
END CLASS

// Secure service implementation
CLASS ContactServiceImpl:
    PRIVATE repository: IContactRepository
    PRIVATE validator: ContactValidator
    PRIVATE auditLogger: Logger

    METHOD addContact(contact: Contact):
        // Null validation
        IF contact IS null THEN
            auditLogger.warn("Attempted to add null contact")
            THROW ContactValidationException("Contact cannot be null")
        END IF

        // Sanitize inputs
        validator.validateName(contact.getFirstName())
        validator.validateName(contact.getLastName())
        validator.validateForMaliciousContent(contact.getAddress())

        // Check for duplicates
        IF repository.existsById(contact.getContactId()) THEN
            auditLogger.warn("Duplicate contact ID attempted: {}", contact.getContactId())
            THROW DuplicateContactException("Contact already exists")
        END IF

        // Save and log
        repository.save(contact)
        auditLogger.info("Contact added successfully: ID={}, Name={} {}",
                        contact.getContactId(),
                        contact.getFirstName(),
                        contact.getLastName())
    END METHOD

    METHOD deleteContact(contactId: String):
        IF contactId IS null OR contactId.isEmpty() THEN
            THROW ContactValidationException("Contact ID required")
        END IF

        Contact contact = repository.findById(contactId)
        IF contact IS null THEN
            // Don't reveal whether ID exists in error message
            auditLogger.warn("Attempted deletion of non-existent contact: {}", contactId)
            THROW ContactNotFoundException("Contact not found")
        END IF

        repository.deleteById(contactId)
        auditLogger.info("Contact deleted: ID={}, Name={} {}",
                        contactId,
                        contact.getFirstName(),
                        contact.getLastName())
    END METHOD
END CLASS
```

This enhancement transforms the application from a security-naive prototype into a system that follows secure coding best practices and anticipates real-world attack vectors.

---
### SKILLS DEMONSTRATION AND COURSE OUTCOMES

Let me summarize the specific skills I'll demonstrate through these enhancements:

### Software Design Skills:
- Layered architecture design
- Application of SOLID principles
- Interface-based programming
- Design pattern implementation (Repository, Builder)
- System architecture documentation

### Software Engineering Skills:
- Dependency management with Maven
- Professional project structure organization
- Build automation and configuration
- Version control best practices
- Code review and refactoring techniques

### Code Quality Skills:
- Writing maintainable, readable code
- Comprehensive documentation
- Logging and observability
- Custom exception design
- API design principles

### Testing Skills:
- Unit testing with isolation
- Mocking and test doubles
- Test-driven thinking
- Code coverage improvement

### Security Skills:
- Threat modeling and vulnerability identification
- Understanding OWASP Top 10 vulnerabilities (Broken Access Control, Injection, Security Misconfiguration)
- Defensive programming and input validation
- Thread-safe programming for concurrent environments
- Security logging and audit trails
- Secure error handling to prevent information disclosure
- Resource management and DoS prevention
- Input sanitization against XSS and injection attacks
- Security-first design thinking

### Professional Practice Skills:
- Following industry standards and conventions
- Understanding and applying trade-offs in design decisions
- Creating collaborative development environments
- Technical communication through documentation

### Course Outcomes Addressed:

**Course Outcome 1** - Collaborative environments: My modular, interface-based design allows teams to work independently on different layers

**Course Outcome 2** - Professional communication: Through comprehensive documentation, JavaDoc, README, and clear code organization

**Course Outcome 3** - Design and evaluation: Through application of design patterns, SOLID principles, and conscious architecture decisions with trade-off analysis

**Course Outcome 4** - Industry techniques and tools: Through Maven, logging frameworks, testing frameworks, and design patterns that are industry standards

**Course Outcome 5** - Security mindset: Through comprehensive security enhancements including:
- Identifying and mitigating nine critical security vulnerabilities in the original code
- Implementing thread-safe collections to prevent race conditions and data corruption
- Creating input sanitization to protect against XSS and injection attacks
- Adding audit logging to track all operations and support forensic analysis
- Implementing secure error handling that prevents information disclosure
- Adding resource limits to prevent denial of service attacks
- Validating all inputs including null checks to prevent NullPointerException exploits
- Applying defense-in-depth principles with multiple security layers
- Demonstrating anticipation of adversarial exploits and implementing appropriate countermeasures

This represents a complete security transformation from a vulnerable prototype to a security-hardened application following industry best practices and OWASP guidelines.

---

## CONCLUSION

In conclusion, while my original Contact Management System demonstrated basic programming competency with proper validation and testing, the planned enhancements will transform it into a professionally architected, secure, maintainable, and scalable application that follows industry best practices.

These enhancements demonstrate my growth in software engineering from writing functional code to designing robust, production-ready, and security-hardened systems. The improvements address real-world concerns like maintainability, testability, flexibility, security vulnerabilities, and professional development practices that are essential in the software industry.

Most importantly, my systematic identification and mitigation of nine critical security vulnerabilities demonstrates a mature security mindset - the ability to think like an attacker, anticipate exploits, and implement defense-in-depth strategies that protect user data and system resources.

---