# Cursor Rules

Configuration and prompts for Cursor IDE when working on bndy.

## .cursorrules File

Place this in the repo root:

```
# bndy Cursor Rules

## Project Context
bndy is an AI-native grassroots music discovery platform.
Read `bndy-brain/README.md` for full context.

## Code Style

### TypeScript
- Strict mode always
- No `any`, use `unknown`
- No `as` assertions
- Prefer interfaces for objects
- Use Zod for runtime validation

### Testing
- TDD: write tests first
- Use factories for test data
- Test behaviour, not implementation
- Aim for 80%+ coverage

### Architecture
- Domain-driven design
- Ports and adapters
- Pure functions in domain layer
- Side effects at boundaries

### Naming
- Files: kebab-case
- Classes: PascalCase
- Functions: camelCase
- Constants: SCREAMING_SNAKE

## Git
- Single line commits
- Format: `type: description`
- Types: feat, fix, refactor, test, docs

## AWS/Lambda
- Serverless-first
- DynamoDB single-table design
- Validate input, format output
- Handle errors explicitly

## Don't
- Don't use `any`
- Don't mutate objects
- Don't swallow errors
- Don't skip tests
- Don't use magic strings
```

## Cursor Chat Prompts

### For Architecture Questions

```
Read the architecture docs in bndy-brain/04-architecture/ and answer based on our actual stack and patterns.
```

### For Feature Implementation

```
Before implementing:
1. Check bndy-brain/03-backlog/now.md for acceptance criteria
2. Check bndy-brain/05-entities/ for data models
3. Write tests first per bndy-brain/07-prompts/claude-code-rules.md
```

### For Code Review

```
Review this code against:
- TypeScript strict rules (no any, no assertions)
- DDD layer separation
- Test coverage
- Error handling patterns

Reference: bndy-brain/07-prompts/claude-code-rules.md
```

## Cursor Composer Prompts

### For New Lambda

```
Create a new Lambda function following bndy patterns:
- Handler in src/interfaces/http/
- Use case in src/application/
- Domain logic in src/domain/
- DynamoDB access in src/infrastructure/

Include:
- Input validation with Zod
- Error handling
- Unit tests
- Integration test

Reference architecture: bndy-brain/04-architecture/aws-stack.md
```

### For New Entity

```
Create a new entity following bndy patterns:
- Entity in src/domain/entities/
- Value objects for IDs and complex values
- Repository interface in domain
- DynamoDB implementation in infrastructure

Reference model: bndy-brain/05-entities/
```

## Related

- [[claude-code-rules]]
- [[chatgpt-product-review]]
