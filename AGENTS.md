# AGENTS.md - Analytics Proto Project Guidelines

Guidelines for agentic coding assistants working on the analytics-proto project.

## Project Overview
- **Type**: Java Maven Protobuf library
- **Language**: Protocol Buffers (proto3) with Java generated code
- **Parent POM**: `dev.crashteam:library-parent-pom:1.0.0`
- **Purpose**: Protobuf definitions for marketplace analytics services

## Build Commands

### Essential Commands
```bash
# Clean compile (regenerates protobuf Java classes)
mvn clean compile

# Compile only
mvn compile

# Install to local repository
mvn install

# Deploy to remote repository (requires credentials)
mvn deploy
```

### Proto Generation
- Java classes generated during `mvn compile` via `protobuf-maven-plugin`
- Always run `mvn clean compile` after modifying `.proto` files
- Generated code in `target/generated-sources/protobuf/java/`

### CI/CD
- GitHub Actions run on PRs (`build.yml`)
- Deployment to Maven Central on main push (`deploy.yml`)
- Uses reusable workflows from `crashteamdev/jvm-workflows`

## Proto Style Guidelines

### File Structure
- Use `proto3` syntax, files in `src/main/proto/`
- One service per proto file (or logical grouping)
- Import common definitions from `base.proto`

### Package & Java Options
```protobuf
package marketplace.external.analytics.category;
option java_multiple_files = true;
option java_outer_classname = "ExternalCategoryAnalyticsProto";
option java_package = "dev.crashteam.mp.external.analytics.category";
```

### Naming Conventions
- **Messages**: PascalCase (`CategoryAnalytics`, `GetCategoryAnalyticsRequest`)
- **Fields**: snake_case (`category_id`, `revenue_percentage`)
- **Enum Types**: PascalCase (`DatePeriod`, `ErrorCode`)
- **Enum Values**: UPPER_SNAKE_CASE with prefix (`DATE_PERIOD_UNSPECIFIED`, `ERROR_CODE_NOT_FOUND`)

### Service Definitions
```protobuf
service ExternalCategoryAnalyticsService {
  rpc getCategoryAnalytics(GetCategoryAnalyticsRequest) returns (GetCategoryAnalyticsResponse);
}
```

## Error Handling Pattern

### Response Structure
Use `oneof` pattern for success/error responses:
```protobuf
message GetCategoryAnalyticsResponse {
  oneof response {
    SuccessResponse success_response = 1;
    ErrorResponse error_response = 2;
  }
  message SuccessResponse { repeated CategoryAnalyticsInfo categories = 1; }
  message ErrorResponse {
    ErrorCode errorCode = 1;
    optional string description = 2;
    enum ErrorCode {
      ERROR_CODE_UNSPECIFIED = 0;
      ERROR_CODE_NOT_FOUND = 1;
      ERROR_CODE_BAD_REQUEST = 2;
    }
  }
}
```

### Error Code Conventions
- Always include `ERROR_CODE_UNSPECIFIED = 0` as first enum value
- Use descriptive error codes (`ERROR_CODE_NOT_FOUND`, `ERROR_CODE_FORBIDDEN`)
- Optional `description` field for additional context

## Field Number Conventions
- Reserve field numbers 1-15 for frequently used fields
- Never change field numbers of existing fields
- Add new fields at the end of message definitions

## Import Organization
1. Google protobuf imports
2. Local base imports
3. External dependencies
```protobuf
import "google/protobuf/timestamp.proto";
import "base.proto";
```

## Java Generated Code Style
- **Classes**: PascalCase (`Money`, `CategoryAnalyticsInfo`)
- **Methods**: camelCase (`getCurrencyCode()`, `newBuilder()`)
- **Fields**: camelCase with underscore suffix (`currencyCode_`)
- **Builder Pattern**: `Message.newBuilder()` and `Message.Builder`
- All generated classes are `final` and immutable (use builders)
- Package: `dev.crashteam.mp.*` matching `java_package`

## Testing
- No existing tests in the codebase
- If adding tests: use JUnit 5, place in `src/test/java/`
- Test generated code functionality and service definitions

## Code Quality & Validation
- No local linting configuration detected
- Parent POM may include checkstyle/spotless configurations
- Rely on CI/CD pipeline for quality checks

### Validation Steps
1. Ensure proto files compile without errors
2. Verify generated Java code compiles
3. Check for backward compatibility when modifying existing protos
4. Validate field number assignments don't conflict

## Common Pitfalls
- Never edit generated Java files directly - all changes in `.proto` files
- Avoid renaming fields used in production
- Use relative imports for local proto files
- Avoid circular dependencies between proto files
- Keep base definitions in `base.proto`

## Workflow for Agents

### When Making Changes
1. **Modify `.proto` files** in `src/main/proto/`
2. **Run `mvn clean compile`** to regenerate Java code
3. **Verify compilation** with `mvn compile`
4. **Check for errors** in generated code
5. **Consider backward compatibility** for existing services

### Before Committing
- Ensure all proto files compile
- Verify no syntax errors in generated Java
- Check field numbering consistency
- Validate import statements

## Editor & Tool Configurations
- No Cursor rules (`.cursor/rules/` or `.cursorrules`) found
- No GitHub Copilot instructions (`.github/copilot-instructions.md`) found
- No local `.editorconfig` file
- Follow proto style guidelines and parent POM conventions

## References
- [Protocol Buffers Style Guide](https://developers.google.com/protocol-buffers/docs/style)
- [Google Cloud APIs Design Guide](https://cloud.google.com/apis/design)
- [gRPC Best Practices](https://grpc.io/docs/guides/)

---
*Last updated: January 2025*