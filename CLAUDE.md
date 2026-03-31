# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an AI-assisted development template with standardized configurations for Java Spring Boot applications, following Alibaba development standards and the .ai-config convention for AI tool integration.

## Technology Stack

- **Java Version**: Java 8 (1.8) only
  - Use Lambda, Stream API, Optional, and interface default methods
  - Never use Java 14+ syntax (record, var, switch expressions, text blocks)
- **Framework**: Spring Boot 2.7.x
- **Build Tool**: Maven 3.6+
- **ORM**: MyBatis / MyBatis-Plus or Spring Data JPA
- **Utilities**: Lombok (@Data, @Builder, @Slf4j, @RequiredArgsConstructor), Hutool/Apache Commons, Jackson

## Strict Constraints

- Forbidden: `javax.annotation.PostConstruct` (use jakarta or standard packages)
- Forbidden: `System.out.println` (use Logger instead)
- Forbidden: `SimpleDateFormat` (use java.time or DateUtils - thread safety)
- Forbidden: Database queries in loops (N+1 problem)
- Forbidden: Raw type collections (use generics like `List<String>`)
- Required: Specify initial capacity for ArrayList/HashMap when size is known
- Required: Use `ThreadPoolExecutor` for thread pools (not `Executors` - OOM risk)

## Code Style

### Naming Conventions
- Variables/Functions: camelCase (`userName`, `getUserInfo`)
- Classes/Components: PascalCase (`UserProfile`, `OrderController`)
- Constants: UPPER_SNAKE_CASE (`MAX_LENGTH`, `API_BASE_URL`)
- Booleans: is/has prefix (`isActive`, `hasPermission`)
- File names: PascalCase for components (`UserProfile.jsx`), camelCase for utilities (`dateUtils.js`)

### Directory Structure (Alibaba Standard)
```
com.example.project
├── common/           # General components (Result, BusinessException, Constants)
├── config/           # Configuration classes (Swagger, MyBatis, Cors)
├── framework/        # Framework related (filters, interceptors)
├── modules/
│   └── user/
│       ├── controller/    # Web layer (param validation, response wrapping)
│       ├── service/       # Business layer interfaces
│       │   └── impl/      # Business layer implementations
│       ├── manager/       # General business processing (optional, third-party calls)
│       ├── mapper/        # Data access layer
│       ├── entity/        # Database entities (UserDO/Entity)
│       ├── dto/           # Data transfer objects (UserDTO)
│       └── vo/            # View objects (UserVO)
└── Application.java   # Application entry point
```

### Code Formatting
- Indentation: 4 spaces
- Line width: Max 120 characters
- Always use braces for code blocks
- Use semicolons in JavaScript/TypeScript
- Organize imports: built-in → third-party → local

## Security Requirements

- **SQL Injection**: Use parameterized queries or ORM
- **Authentication**: JWT with proper token generation/validation/refresh
- **Password Storage**: bcrypt or similar secure hashing
- **XSS Prevention**: HTML escape user output, set CSP
- **CSRF Protection**: CSRF tokens, SameSite cookies
- **Environment Variables**: Never hardcode API keys, use .env files
- **HTTPS Required**: All data transmission

## Git Workflow

### Commit Format
```
<type>(<scope>): <subject>

<body>

<footer>
```
Types: feat, fix, docs, style, refactor, test, chore

### Branch Strategy
- main/master: Production
- develop: Development
- feature/*: Features
- bugfix/*: Bug fixes
- hotfix/*: Emergency fixes
- release/*: Releases

### Merge Strategy
- Pull Request required
- At least one code reviewer
- CI/CD must pass
- Squash merge recommended

## Development Workflow

**5 stages with strict sequential execution - no skipping stages:**

1. **Requirements & Prototyping**: Understand requirements, create HTML+CSS prototypes, customer confirmation
2. **Technical Design**: Architecture design, tech selection, database/API design, team confirmation
3. **Coding Implementation**: Environment setup, backend API, frontend) UI, integration testing
4. **Self-Testing**: Functional, performance, security, compatibility testing
5. **Manual Review**: Code review, architecture review, documentation review, architect confirmation

**Critical**: Each stage requires human confirmation before proceeding to the next stage.

## AI Configuration Structure

`.ai-config/` contains:
- **rules/**: Development standards (tech stack, code style, security, git workflow)
- **skills/**: Reusable skills (write-plan, code-review, generate-prototype, database-dba)
- **agents/**: Specialized agents (architect, frontend_dev, backend_dev, qa_engineer, prototype_designer, dba)
- **mcp/**: External tool configurations
- **hooks.json**: Lifecycle hooks for automation (on_file_save, on_commit, on_pr_create, etc.)

## Code Review Checkpoints

- Naming conventions followed
- Code formatting consistent
- Comments clear and complete
- Business logic correct
- No performance issues
- No security vulnerabilities
- Maintainable structure