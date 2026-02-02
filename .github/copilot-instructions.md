#GitHub Copilot Instructions

## Project Context

## Project Context

This is a modern full-stack web application project.
**Tech Stack:** TypeScript, React 18, Node.js, Vite, Tailwind CSS
**Current Phase:** Active Development

## Core Principles (Order of Priority)

1. **Correctness** - Code must work reliably and handle edge cases
2. **Separation of Concerns** - Separation of Concerns between components, services, and utilities
3. **Security** - Validate inputs, sanitize outputs, protect sensitive data
4. **Maintainability** - Future developers (including me) should understand it easily
5. **Performance** - Optimize intelligently, but only after correctness is proven
6. **User Experience** - Intuitive and responsive interfaces

## Workflow: Every Task Follows This Pattern

### 1️⃣ PLAN (Always Start Here)

- Break the task into clear, manageable steps
- List all files to create or modify
- Identify potential risks, edge cases, and assumptions
- Propose the approach and wait for confirmation before proceeding
- For complex tasks, outline the architecture or data flow

### 2️⃣ IMPLEMENT

- Write clean, well-documented code following the style guide below
- Include comprehensive error handling
- Add or update tests for new functionality
- Follow single responsibility principle
- Prefer small, focused functions over large ones

### 3️⃣ VERIFY

- Suggest specific verification steps (build, test, lint)
- Propose exact terminal commands to run
- List manual checks needed (UI testing, API endpoints, etc.)
- Confirm expected behavior and outputs
- Flag any assumptions that need validation

# Coding Standards

### General Style

- **TypeScript:** Always use strict mode; no implicit `any`
- **Patterns:** Prefer functional patterns and composition over classes/inheritance
- **Functions:** Maximum 30 lines per function; prefer pure functions when possible
- **Lines:** Maximum 100 characters per line
- **Naming Conventions:**
  - `camelCase` for variables and functions
  - `PascalCase` for types, interfaces, and classes
  - `UPPER_SNAKE_CASE` for constants
  - Use descriptive names over short abbreviations

### Code Quality Rules

- Use early returns to reduce nesting
- Avoid deep nesting (max 3 levels)
- Prefer `const` over `let`; never use `var`
- Use modern syntax: async/await over callbacks, destructuring, spread operators
- No magic numbers - use named constants
- Avoid code duplication - extract reusable logic

### Documentation Standards

Every public function, class, or complex logic must have JSDoc comments:

````typescript
/**
 * Fetches user data from the API and transforms it for the UI
 *
 * @param userId - Unique identifier for the user (UUID v4 format)
 * @param options - Optional configuration for the request
 * @returns Promise resolving to a formatted User object
 * @throws {UserNotFoundError} When the user doesn't exist in the database
 * @throws {ValidationError} When userId format is invalid
 * @throws {NetworkError} When API request fails
 *
 * @example
 * ```typescript
 * const user = await fetchUserData('123e4567-e89b-12d3-a456-426614174000');
 * console.log(user.displayName);
 * ```
 */
async function fetchUserData(
  userId: string,
  options?: RequestOptions,
): Promise<User> {
  // Implementation
}
````

**Documentation Guidelines:**

- Add inline comments ONLY for non-obvious logic or complex algorithms
- Update README.md when adding features or changing architecture
- Document WHY, not just WHAT (the code shows what it does)
- Keep comments up-to-date with code changes

### Testing Approach

**Requirements:**

- Write unit tests for all new business logic
- Write integration tests for API endpoints and critical flows
- Test happy paths, edge cases, and error scenarios
- Aim for >80% coverage on business logic (not boilerplate)

**Testing Pattern (TDD-like):**

1. Plan what needs to be tested
2. Write failing tests first (when applicable)
3. Implement the feature
4. Verify tests pass
5. Refactor if needed

**Test Organization:**

- Test files: `__tests__/` folder or co-located `*.test.ts` files
- Use descriptive test names: `it('should throw ValidationError when email is invalid')`
- Group related tests with `describe` blocks
- Use test frameworks: Jest, Vitest, or React Testing Library

**Before Finalizing:**

- Always suggest running the test suite
- Verify all tests pass
- Check for new edge cases that need tests

### Error Handling

**Rules:**

- Always wrap async operations in try-catch blocks
- Use custom error classes for domain-specific errors
- Never swallow errors silently (`catch (e) {}`)
- Always log errors with context
- Provide meaningful error messages for users
- Include stack traces in development, sanitize in production

**Pattern:**

```typescript
// ✅ DO: Proper error handling
async function processPayment(orderId: string): Promise<PaymentResult> {
  if (!orderId?.trim()) {
    throw new ValidationError("Order ID is required");
  }

  try {
    const order = await fetchOrder(orderId);
    const result = await paymentGateway.charge(order);

    logger.info("Payment processed successfully", {
      orderId,
      transactionId: result.id,
    });

    return result;
  } catch (error) {
    logger.error("Payment processing failed", {
      orderId,
      error: error instanceof Error ? error.message : "Unknown error",
      stack: error instanceof Error ? error.stack : undefined,
    });

    if (error instanceof NetworkError) {
      throw new PaymentGatewayError("Payment service unavailable", {
        cause: error,
      });
    }

    throw new PaymentProcessingError("Failed to process payment", {
      orderId,
      cause: error,
    });
  }
}

// ❌ DON'T: Silent failures or untyped errors
async function processPayment(orderId) {
  try {
    return await paymentGateway.charge(orderId);
  } catch (e) {
    console.log("error");
    return null;
  }
}
```

### File Structure Conventions

- Follow the existing project structure strictly
- Standard layout: `src/`, `tests/`, `components/`, `utils/`, `types/`, etc.
- Group related files in feature folders when applicable
- Configuration files in root or dedicated config folders
- Do NOT create new folders without explicit discussion and approval
- Keep import paths clean (use path aliases like `@/components`)

---

## Code Examples & Patterns

### ✅ Preferred Patterns (DO)

```typescript
// Clear, typed async function with proper error handling
async function fetchUser(id: string): Promise<User> {
  if (!id?.trim()) {
    throw new ValidationError("User ID is required and must not be empty");
  }

  try {
    const response = await api.get<UserResponse>(`/users/${id}`);
    return transformUserResponse(response.data);
  } catch (error) {
    logger.error("Failed to fetch user", { id, error });

    if (error instanceof ApiError && error.status === 404) {
      throw new UserNotFoundError(id);
    }

    throw new UserFetchError(`Failed to fetch user ${id}`, { cause: error });
  }
}

// Small, focused, single-responsibility functions
function calculateTotalPrice(items: CartItem[]): number {
  return items.reduce((total, item) => total + item.price * item.quantity, 0);
}

function applyDiscount(price: number, discountPercent: number): number {
  if (discountPercent < 0 || discountPercent > 100) {
    throw new ValidationError("Discount must be between 0 and 100");
  }
  return price * (1 - discountPercent / 100);
}

// Early returns for readability
function validateEmail(email: string): ValidationResult {
  if (!email) {
    return { valid: false, error: "Email is required" };
  }

  if (!email.includes("@")) {
    return { valid: false, error: "Email must contain @" };
  }

  if (email.length > 255) {
    return { valid: false, error: "Email is too long" };
  }

  return { valid: true };
}
```

### ❌ Anti-Patterns (DON'T)

```typescript
// ❌ Untyped parameters and return values
function fetchUser(id) {
  return api.get("/users/" + id).then((r) => r.data);
}

// ❌ Large, multi-responsibility functions
function handleUserSubmit(data) {
  // 150 lines of validation, API calls, state updates, navigation...
}

// ❌ Deep nesting
function processOrder(order) {
  if (order) {
    if (order.items) {
      if (order.items.length > 0) {
        if (order.customer) {
          // actual logic buried 4 levels deep
        }
      }
    }
  }
}

// ❌ Magic numbers and unclear variable names
function calc(p, d) {
  return p * 0.85 - 10; // What is 0.85? What is 10?
}

// ❌ Silent error handling
try {
  await dangerousOperation();
} catch (e) {
  // Nothing - error is lost
}

// ❌ Using 'any' type
function processData(data: any): any {
  return data.map((x: any) => x.value);
}
```

---

## Communication Style & Workflow

### ✅ DO These Things:

**Planning & Communication:**

- **Always propose a clear plan** before writing code for complex tasks
- **Explain WHY** you chose an approach, not just WHAT the code does
- **Show trade-offs** for different implementation options
- **Ask for confirmation** before making large changes or refactoring
- **Flag assumptions and risks** proactively
- **Suggest verification steps** after every implementation

**Code Quality:**

- **Think step-by-step** before generating code
- **Use existing dependencies** already in the project
- **Justify new packages** before suggesting installation (consider bundle size, maintenance, security)
- **Prefer established libraries** over custom implementations for complex problems
- **Show focused code snippets** or diffs rather than entire files when possible

**Learning & Adaptation:**

- **Learn from corrections** - if I fix something repeatedly, note it and avoid it
- **Keep responses concise** and actionable
- **Adapt to my preferences** as you learn them through our interactions
- **Accept feedback gracefully** and adjust quickly

### ❌ DON'T Do These Things:

**Avoid:**

- Generating code without a plan for complex features
- Using placeholder comments like `// TODO: implement this` or `// Add logic here`
- Suggesting unverified or untested solutions
- Assuming I understand context you haven't explained
- Installing packages without justification
- Making assumptions about requirements - ask for clarification
- Overusing formatting (excessive bold, lists, headers) in responses
- Repeating the same mistakes after I've corrected them

---

## My Specific Preferences

> Customize this section based on your actual preferences

**Development Workflow:**

- I review all code before running it
- I appreciate explanations of performance implications for data operations
- Warn me about bundle size impact (packages >50kb warrant discussion)
- I prefer to understand WHY patterns exist, not just how to use them
- Explain security implications when handling user data or auth

**Code Preferences:**

- Prefer composition over inheritance
- Prefer React hooks over Higher-Order Components (HOCs)
- Prefer functional components over class components
- I like explicit over implicit (clarity over cleverness)
- Favor immutability - avoid mutating data structures

**Communication:**

- Be direct and concise
- Use examples to illustrate concepts
- Suggest alternatives when there are multiple valid approaches
- Don't apologize excessively - just fix and move forward

---

## Project-Specific Rules

> Add specific rules for YOUR project here

**Examples:**

- All API calls must go through the `apiClient` utility (centralized error handling)
- Use Zod for runtime validation of API responses and form inputs
- State management: Zustand for global state, useState for local component state
- Forms: React Hook Form + Zod validation schema
- Styling: Tailwind utility classes only (no custom CSS unless absolutely necessary)
- Icons: Use lucide-react icon library
- Date handling: Use date-fns library
- Environment variables: Must be prefixed with `VITE_` to be exposed to client

---

## Tool Usage Guidelines

**When to use tools:**

- **bash/terminal:** For testing, building, linting, running scripts, git operations
- **file operations:** For creating new files, refactoring across multiple files
- **web search:** For checking latest API docs, finding current best practices

**Before using tools:**

- Show me the plan and what commands you'll run
- Explain why the tool is needed
- For multiple operations, list them all first

---

## Pre-Commit Checklist

Before marking any task as complete, ensure:

- [ ] Code compiles/builds without errors
- [ ] All tests pass (unit + integration)
- [ ] Linter passes with no warnings
- [ ] TypeScript has no errors (strict mode)
- [ ] Documentation is updated (README, JSDoc, inline comments)
- [ ] No `console.log()` or debug code remains
- [ ] No `any` types (or justified with comment if absolutely necessary)
- [ ] Error handling is comprehensive
- [ ] Edge cases are considered and tested
- [ ] Verification steps are provided

---

## Security Checklist

For any code touching sensitive areas:

- [ ] User inputs are validated and sanitized
- [ ] SQL queries use parameterized statements (no string concatenation)
- [ ] Authentication tokens are stored securely
- [ ] Sensitive data is not logged
- [ ] CORS is properly configured
- [ ] Rate limiting is considered for public endpoints
- [ ] File uploads are validated (type, size, content)
- [ ] XSS vulnerabilities are prevented (sanitize HTML)
- [ ] CSRF protection is in place for state-changing operations

---

## Learning Loop & Iteration

**Continuous Improvement:**
When I correct or adjust something you've suggested, note the pattern and apply it going forward.

**Common Corrections to Remember:**

> This section will be updated as we work together

Example format:

- ❌ **Issue:** You suggested using `Array.forEach()` for transformations
- ✅ **Correction:** Use `Array.map()` for transformations, `forEach()` only for side effects
- 📝 **Reason:** More functional, easier to test, clearer intent

---

## Examples of Good Interactions

### Example 1: Feature Request

**User:** "Add user authentication to the app"

**Good Response:**

```
I'll help you add authentication. Here's my proposed plan:

PLAN:
1. Install & configure authentication library (suggest: NextAuth.js or Clerk)
2. Create auth context/provider for app-wide auth state
3. Implement login/signup pages with form validation
4. Add protected route wrapper component
5. Update API routes to verify auth tokens
6. Add user session management

CONSIDERATIONS:
- Need to choose between JWT or session-based auth
- Should we support OAuth providers (Google, GitHub)?
- Password requirements: min length, special chars?
- Do we need email verification?

FILES TO MODIFY:
- src/App.tsx (wrap with AuthProvider)
- src/pages/Login.tsx (new)
- src/pages/Signup.tsx (new)
- src/components/ProtectedRoute.tsx (new)
- src/lib/auth.ts (new - auth utilities)
- src/api/middleware/auth.ts (new - API auth check)

DEPENDENCIES:
- @auth/core or similar (~15kb)
- bcrypt for password hashing

Does this approach work for you? Any preferences on the questions above?
```

### Example 2: Bug Fix

**User:** "The form isn't validating email properly"

**Good Response:**

```
I'll investigate the email validation issue.

ANALYSIS:
Looking at src/components/SignupForm.tsx, I see the validation regex:
`/^[^\s@]+@[^\s@]+$/`

PROBLEM:
This regex is too permissive - it allows invalid formats like "user@domain"
(missing TLD) and doesn't handle edge cases.

SOLUTION:
Replace with more robust validation:
1. Use a proper email regex or validation library
2. Add additional checks (max length, no consecutive dots, etc.)

RECOMMENDED FIX:
[show code with proper validation]

VERIFICATION STEPS:
1. Test with valid emails: user@domain.com, user+tag@domain.co.uk
2. Test with invalid emails: @domain.com, user@, user@domain
3. Run existing test suite: `npm test`
4. Add new test cases for edge cases

Would you like me to implement this fix?
```

---

## Final Notes

**Philosophy:**

- **Better slow and correct than fast and broken**
- **Clarity over cleverness**
- **Explicit over implicit**
- **Verified over assumed**

**Remember:**

- Prioritize safety, security, and reliability
- Think before coding
- Verify your own suggestions before presenting them
- Learn and adapt based on feedback

**When Uncertain:**

- Propose 2-3 alternative approaches with trade-offs
- Ask clarifying questions
- Research current best practices
- Admit uncertainty rather than guessing

---

Follow these instructions strictly for every interaction in this repository.
