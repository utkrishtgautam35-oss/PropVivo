# propVivo Implementation Plan: Auth Backend & Module Integration

This plan outlines the steps to build a robust authentication backend and replace mock data across 8 feature modules in the propVivo HRMS project.

## Phase 1: Authentication Backend (Foundation)

### 1.1. Domain & Data Layer
- **Extend Employee Entity**: Add `PasswordHash` and `PasswordSalt` fields to the existing `Employee` entity in `UserFeature.Domain`.
- **Repository Update**: Add `GetEmployeeByEmailAsync` to `IEmployeeRepository` and its implementation in `EmployeeRepository`.
- **Data Seeding**: Implement `DataSeeder` to create a default admin user (`admin@propvivo.com` / `Admin@123`) with a hashed password on startup.

### 1.2. Auth Feature Module
- **Scaffold AuthFeature**: Create `AuthFeature.Application`, `AuthFeature.Infrastructure`, and `AuthFeature.GraphQL` projects.
- **DTOs**: Define `LoginRequest`, `LoginResponse`, `RefreshTokenRequest`, and `UserDto`.
- **JWT Service**: Implement `JwtService` for generating Access Tokens (JWT) and Refresh Tokens.
- **MediatR Handlers**: Create `AuthHandler` to process `LoginRequest` (verify credentials, issue tokens) and `RefreshRequest`.
- **REST Controller**: Implement `AuthController` with `/auth/login` and `/auth/refresh` endpoints to match existing frontend expectations.

### 1.3. Global Configuration
- **JWT Middleware**: Configure `services.AddAuthentication().AddJwtBearer()` in `Startup.cs`.
- **AppSettings**: Add JWT configuration (Secret Key, Issuer, Audience, Expiration) to `appsettings.json`.
- **Dependency Injection**: Register `IJwtService` and `AuthHandler` in the DI container.

---

## Phase 2: Frontend Auth Integration

### 2.1. Environment Setup
- **Config**: Update `.env.local` to point to the real backend and set `NEXT_PUBLIC_DISABLE_AUTH=false`.

### 2.2. Auth Service & State
- **Verify `authService.ts`**: Ensure it correctly calls the new `/auth/login` and `/auth/refresh` endpoints.
- **Session Context**: Ensure `SessionContext.tsx` correctly manages the logged-in state and token persistence.

---

## Phase 3: Feature Module Integration (Mock to Real)

For each of the 8 modules, follow this pattern:

### 3.1. Backend API Completion
- **Ensure GraphQL Queries**: Verify each module (Attendance, Leave, Documents, etc.) has its `getAll*` query implemented and registered.
- **Database Schema**: Ensure Postgres tables for each feature exist (EF Core `EnsureCreated` will handle this based on entities).

### 3.2. Frontend GraphQL Wiring
- **Define GraphQL Queries**: Create `.ts` files in `frontend/graphql/query/` for each module (e.g., `getAttendance.ts`).
- **Update Pages**: Replace `generateMockData()` with `useQuery` from Apollo Client.
- **Map Data**: Transform backend DTOs to match the frontend's expected component shapes.

### 3.3. Module-Specific Tasks:
1.  **Attendance**: Connect `getAllAttendance` to `AttendancePage`.
2.  **Leave**: Connect `getAllLeaves` to `LeavePage`.
3.  **Documents**: Connect `getAllDocuments` to `DocumentsPage`.
4.  **Payroll**: Connect `getAllPayrolls` to `PayrollPage`.
5.  **Expenses**: Connect `getAllExpenses` to `ExpensesPage`.
6.  **Team**: Connect `getAllEmployees` (with filtering) to `TeamPage`.
7.  **Announcements**: Connect `getAllAnnouncements` to `AnnouncementsPage`.
8.  **Dashboard/Employees**: Connect stats and employee lists to the main Dashboard.

---

## Phase 4: Testing & Cleanup

- **End-to-End Test**: Verify Login -> Redirect -> Data Fetching for all modules.
- **Error Handling**: Implement proper error states for failed API calls or expired sessions.
- **Cleanup**: Remove all mock data generators and unused boilerplate files.
- **Final Build**: Ensure both Frontend and Backend build without warnings.
