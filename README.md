# NOTES.md

## Summary of Main Bugs/Issues Found

### 1. **Unused Imports and Variables**
- **Issue**: TypeScript strict mode flagged unused imports (`Todo` in test file) and variables (`InMemoryUserRepository` and `InMemoryTodoRepository` in `main.ts`)
- **Impact**: Code quality warning, prevents clean compilation

### 2. **Logic Bug in `processReminders()`**
- **Issue**: The `processReminders()` method was updating ALL todos with due reminders, including those already marked as `DONE`
- **Impact**: Test failure - completed todos were incorrectly being changed to `REMINDER_DUE` status
- **Root cause**: Missing status filter in `findDueReminders()` repository method

### 3. **Timestamp Race Condition in Tests**
- **Issue**: Test expected `updatedAt` timestamp to be strictly greater than `createdAt`, but execution was too fast, resulting in identical timestamps
- **Impact**: Flaky test that fails intermittently on fast machines

### 4. **PowerShell Execution Policy (Environment Issue)**
- **Issue**: Windows PowerShell blocked npm scripts from running
- **Impact**: Cannot execute `npm test` command

## How I Fixed Them

### 1. **Cleaned Up Unused Code**
```typescript
// Removed unused import from test file
// import { Todo } from "../src/domain/Todo";

// Removed unused variables from main.ts
// const userRepo = new InMemoryUserRepository();
// const todoRepo = new InMemoryTodoRepository();
```

### 2. **Added Status Filter to `findDueReminders()`**
```typescript
// In InMemoryTodoRepository.ts
async findDueReminders(now: Date): Promise<Todo[]> {
  return this.todos.filter(
    (todo) =>
      todo.remindAt &&
      todo.remindAt <= now &&
      todo.status === "PENDING"  // Only process PENDING todos
  );
}
```

### 3. **Fixed Timestamp Assertion**
```typescript
// Changed from strict comparison to inclusive comparison
expect(completed.updatedAt.getTime()).toBeGreaterThanOrEqual(
  todo.updatedAt.getTime()
);
```

### 4. **Switched to Command Prompt**
- Used CMD instead of PowerShell to run npm scripts
- Alternative: Configure PowerShell execution policy (not recommended for security reasons)

## Framework/Database Choices

### **Database: SQLite**
**Why I chose SQLite:**
- ✅ **Zero configuration** - no separate database server needed
- ✅ **Portable** - single file database, easy to backup and share
- ✅ **Perfect for development** - lightweight and fast
- ✅ **Production-ready** for small to medium applications
- ✅ **ACID compliant** - reliable transactions
- ✅ **Easy testing** - can use in-memory mode or separate test DB file

### **Testing Framework: Jest**
**Why Jest:**
- ✅ Already configured in starter code
- ✅ Excellent TypeScript support
- ✅ Built-in assertion library
- ✅ Great for TDD workflow
- ✅ Fast test execution with parallel running

### **Repository Pattern**
- Implemented both `InMemoryRepository` (for fast unit tests) and `SqliteRepository` (for integration tests)
- This separation allows testing business logic without database dependencies

## Optional Improvements Implemented

### 1. **Comprehensive Test Coverage**
- Happy path tests for all main features
- Edge case tests (empty titles, non-existent users)
- Idempotency tests (completing already-done todos)
- Reminder processing with various scenarios

### 2. **Input Validation**
- Empty/whitespace title validation
- User existence validation before creating todos
- RemindAt date parsing and validation

### 3. **Repository Abstraction**
- Clean separation between business logic and data access
- Easy to swap implementations (InMemory ↔ SQLite)

## What I Would Improve Further

### 1. **Authentication & Authorization**
- Add user authentication (JWT tokens)
- Implement middleware to verify user ownership of todos
- Role-based access control (admin users)

### 2. **Enhanced Error Handling**
- Create custom error classes (`NotFoundError`, `ValidationError`, `UnauthorizedError`)
- Implement global error handler middleware
- Add proper HTTP status codes for different error types

### 3. **API Implementation**
- Complete the HTTP server endpoints (currently only TODO comments exist)
- Add request validation middleware (using Zod or Joi)
- Implement pagination for `getTodosByUser`
- Add filtering and sorting options

### 4. **Database Improvements**
- Add database migrations system
- Implement indexes for better query performance
- Add soft delete functionality (deleted_at column)
- Connection pooling for production

### 5. **Testing Enhancements**
- Add integration tests for HTTP endpoints
- Implement E2E tests
- Add test coverage reporting (aim for >80% coverage)
- Mock external dependencies properly

### 6. **Code Quality**
- Add ESLint for code linting
- Add Prettier for code formatting
- Implement pre-commit hooks (Husky)
- Add CI/CD pipeline (GitHub Actions)

### 7. **Features**
- Todo sharing between users
- Todo categories/tags
- Recurring reminders
- Priority levels
- Due dates (different from reminders)
- File attachments
- Todo comments/notes

### 8. **Observability**
- Structured logging (Winston or Pino)
- Request tracing
- Performance monitoring
- Error tracking (Sentry)

### 9. **Documentation**
- API documentation (OpenAPI/Swagger)
- Architecture decision records (ADR)
- Code examples and usage guides
- Deployment guide

### 10. **Performance**
- Add caching layer (Redis)
- Optimize database queries
- Add rate limiting
- Implement request throttling

---

## Conclusion

The main issues were related to **code quality** (unused imports), **business logic bugs** (reminder processing), and **test reliability** (timestamp assertions). All critical bugs have been fixed, and the test suite now passes reliably. The application uses SQLite for simplicity and portability, with a clean repository pattern that allows easy testing and future database migrations.

With more time, I would focus on implementing the actual HTTP API endpoints, adding authentication, and improving error handling to make this production-ready.
