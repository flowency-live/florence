# Claude Code Rules

System prompt and rules for Claude Code when working on bndy.

## Context Loading

When starting a bndy session, Claude Code should read:

1. `README.md` - Project overview
2. `01-strategy/bndy-strategy.md` - Strategic context
3. `03-backlog/now.md` - Current work
4. Relevant entity docs from `05-entities/`
5. Relevant architecture docs from `04-architecture/`

## Core Rules

### 1. Test-Driven Development

Every feature starts with a failing test. No exceptions.

```
1. Write failing test
2. Write minimal code to pass
3. Refactor
4. Repeat
```

### 2. TypeScript Strict Mode

- No `any` - use `unknown` and narrow
- No `as` assertions
- No `!` non-null assertions
- Prefer `interface` over `type` for objects

### 3. Domain-Driven Design

```
src/
├── domain/           # Pure business logic, no dependencies
│   ├── entities/     # Venue, Artist, Event
│   ├── value-objects/ # VenueId, Coordinates, Genre
│   └── services/     # Domain services
├── application/      # Use cases, orchestration
├── infrastructure/   # AWS, external APIs
└── interfaces/       # HTTP handlers, CLI
```

### 4. Validation at Boundaries

External data (HTTP, events, env vars) must be validated with Zod:

```typescript
const VenueInput = z.object({
  name: z.string().min(1),
  city: z.string().min(1),
  postcode: z.string().optional(),
});
```

### 5. Immutability

Never mutate. Create new objects.

```typescript
// Bad
venue.name = "New Name";

// Good
const updatedVenue = { ...venue, name: "New Name" };
```

### 6. Error Handling

Handle or throw. Never swallow.

```typescript
// Bad
try {
  await fetchVenue(id);
} catch (e) {
  // silence
}

// Good
try {
  await fetchVenue(id);
} catch (e) {
  throw new VenueNotFoundError(id, { cause: e });
}
```

### 7. Value Objects Over Primitives

```typescript
// Bad
function createEvent(venueId: string) { ... }

// Good
function createEvent(venueId: VenueId) { ... }
```

## Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Files | kebab-case | `venue-repository.ts` |
| Classes | PascalCase | `VenueRepository` |
| Functions | camelCase | `findVenueById` |
| Constants | SCREAMING_SNAKE | `MAX_RETRY_COUNT` |
| Types/Interfaces | PascalCase | `VenueEntity` |

## Git Workflow

**IMPORTANT: Commit and push regularly. No permission required.**

After completing any logical unit of work:
1. `git add` the changed files
2. `git commit -m "type: description"`
3. `git push origin main`

Do this automatically. Do not ask for permission. The repo should always reflect current state.

### Commit Format

Single line only. Format: `type: short description`

```
feat: add venue search endpoint
fix: handle missing postcode in venue input
refactor: extract entity resolution to service
test: add venue duplicate detection tests
docs: update architecture diagram
```

### When to Commit

- After creating or updating any document
- After completing a backlog item
- After adding/modifying an ADR
- After any meaningful change

Don't batch up changes. Commit frequently.

## Lambda Patterns

```typescript
// Handler structure
export const handler = async (event: APIGatewayEvent): Promise<APIGatewayResult> => {
  const input = parseAndValidate(event);   // Validate early
  const result = await useCase(input);      // Domain logic
  return formatResponse(result);            // Format late
};
```

## Testing Patterns

```typescript
// Use factories
const venue = createTestVenue({ city: "London" });

// Test behaviour, not implementation
it("should reject duplicate venue names in same city", async () => {
  await createVenue({ name: "Blue Note", city: "London" });

  await expect(
    createVenue({ name: "Blue Note", city: "London" })
  ).rejects.toThrow(DuplicateVenueError);
});
```

## Related

- [[cursor-rules]]
- [[chatgpt-product-review]]
