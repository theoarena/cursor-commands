---
name: development
description: Global development rules and architectural patterns for Laravel + Inertia + React stack. Always apply when developing with Laravel, Inertia, or React, working with controllers, models, services, managers, React components, Inertia pages, hooks, or when implementing features in Laravel + Inertia + React projects. Defines strict architectural flows (Controller → Manager → Service → Model for backend, Component → Hook → Service for frontend), form handling with Inertia forms, validation patterns, and coding standards for the full stack.
---

# Global Development Rules

## 1. Core Architecture (MANDATORY)

### Backend Flow (Laravel)
**Strict Flow:** `Controller` → `Manager` → `Service` → `Model`

1.  **Controller**: HTTP entry point. Validates input (FormRequest). Calls **Manager ONLY**. Returns `Inertia::render` for pages or `to_route()` for store/update/destroy actions. NEVER calls Service/Model.
2.  **Manager**: Business logic orchestration. Calls Services or other Managers. NEVER calls Models.
3.  **Service**: Persistence & Data retrieval. Calls Models. Uses **Custom Query Builders** (not raw `where` chains).
4.  **Model**: Eloquent definition ONLY.

### Frontend Flow (React/Inertia)
**Strict Flow:** `UI Component` → `Custom Hook` → `Service` → `Inertia/Axios`

1.  **UI Component**: Pure presentation. Calls Hooks. NEVER calls API directly.
2.  **Custom Hook**: State manager & Orchestrator. Calls Services.
3.  **Service**: API definition. Uses `Inertia.visit` or `axios`.
4.  **Zustand**: Global state only (User/Theme).

### Query Parameters & Filters
-   **Backend Source of Truth**: Query parameters MUST be processed in Controllers and passed as props (e.g., `filters`). NEVER parse `window.location.search` or `page.url` in components.
-   **Frontend Consumption**: Access filters via `usePage().props.filters`. This ensures SSR compatibility and maintains single source of truth.
-   **Filter Updates**: Filter changes MUST go through the Hook → Service flow. The Service uses `router.get()` with query params. Components receive filter change callbacks as props from parent Pages.
-   **Pattern**: `Component (receives callback)` → `Page (uses Hook)` → `Hook` → `Service (router.get)` → `Backend processes` → `Props updated`

---

## 2. Coding Standards & Best Practices

-   **SOLID**: Strictly adhere to SOLID principles. Favor Composition over Inheritance.
-   **Dependency Injection**: Required for all classes.
-   **Encapsulation**: Private properties, public accessors.
-   **Simplicity**:
    -   Avoid defensive coding if types cover it.
    -   Use Early Returns to flatten `if/else` structures.
    -   Single source of truth for state (avoid syncing multiple state variables).
-   **Imports**: All `use`/`import` at top of file.
-   **Documentation**: Objective docblocks for all public methods.

---

## 3. Directory Structure & Responsibilities

### Backend (`app/`)
-   `Http/Controllers`: Handle Request/Response. Slim code.
-   `Http/Requests`: **MANDATORY** for validation. Use end-to-end (Controller->Manager->Service). DTOs encouraged.
-   `Managers`: Complex business logic, workflow orchestration.
-   `Services`: DB operations, External APIs.
-   `Models`: Eloquent, Relations, Scopes.
-   `Http/Resources`: API/Inertia Response transformation.
-   **Response**: Use `Inertia::render` for pages. Use `to_route()` for store/update/destroy actions (always redirect to index route).

### Frontend (`resources/js/`)
-   `pages/`: Inertia Views. Receive data via Props (SSR).
-   `layouts/`: Structural wrappers.
-   `components/`: Reusable UI. Use **Compound Components** (`Card.Header`, `Card.Body` etc.) for complex UIs.
-   `hooks/`: Logic extraction. Two types:
    -   **Feature-specific hooks**: One hook per feature context (e.g., `useCotistas`, `useDiagnostics`). These hooks manage state and orchestrate Services for a specific domain.
    -   **Generic hooks**: Reusable React hooks used across the system (e.g., `useLocalStorage`, `useMobile`, `useClipboard`). These hooks provide generic functionality and MUST be centralized in `hooks/`.
-   `services/`: Axios/Inertia wrappers.
-   `lib/`: Pure utility functions (no React hooks). Centralized reusable functions used across the system.
    -   `lib/utils.ts`: General utilities (e.g., `cn()`, `toUrl()`, string helpers, type guards).
    -   `lib/date.ts`: Date utilities (e.g., `formatDateToString()`, `formatDateToLocale()`, `parseStringToDate()`).   
    -   `lib/storage.ts`: Storage utilities (e.g., `getStorage()`, `setStorage()`) - Note: For React hooks that sync with storage, use `hooks/useLocalStorage` instead.
    -   `lib/validation/`: Joi Schemas and validation utilities.
    -   `lib/notifications/`: Centralized notification keys and utilities.
    -   `lib/permissions.ts`: Permission checking utilities (e.g., `can()`, `hasRole()`).
    -   **Examples**: Date formatters, currency formatters, string helpers, type guards, URL helpers, data transformers, cache management, storage helpers etc.
-   **Page-Specific Components**: Components that are used exclusively by a single page MUST be placed in the same directory as that page. Only truly reusable components should be in `components/`. For example, if `CompanyForm` and `CompanyList` are only used by `pages/admin/companies/Index.tsx`, they should be in `pages/admin/companies/`.

---

## 4. Advanced Patterns (Strict Implementation)

### Backend
-   **Pipeline Pattern**: MUST use `Illuminate\Pipeline` for complex workflows (e.g., Checkout).
-   **Custom Query Builders**: Services must call semantic scopes (e.g., `->active()`), never raw `->where()`. Register in Model via `newEloquentBuilder`.
-   **Value Objects**: Use immutable classes for domain values (Money, CPF) in DTOs/Services. No primitive obsession.

### Frontend
-   **Optimistic UI**: Update UI state immediately before API response when possible.
-   **Sidebar Forms**: CRUD Create/Edit forms MUST open in a Sidebar (OffCanvas), not new pages.
-   **Validation**:
    -   **Client**: Joi Schemas (`useJoiValidation`).
    -   **Server**: Laravel FormRequest.
    -   **Display**: Combine `errors` (Inertia) || `clientErrors` (Joi).

---

## 5. Specific Implementation Rules

### Notifications
-   **System**: Use `useNotifier()` hook.
-   **Keys**: MUST be defined in `lib/notifications.ts`.
-   **Format**: `entity.action.result` (e.g., `cotista.update.success`).
-   **Prohibited**: Hardcoded strings or direct `Toast` usage in components.

### React Specifics
-   **No Routing**: Never use `react-router`. All routing is Laravel-side (`routes/*.php`).
-   **No Direct Fetch**: Components must not use `fetch/axios`. Delegate to Hooks -> Services.
-   **Types**: Strong TypeScript interfaces for all Models/Props in `types/`.
-   **Relationship Types**: All relationship fields (foreign keys) in type definitions MUST reference the related entity type using indexed access types (e.g., `company_id: Company['id']` instead of `company_id: number`). This ensures type safety and maintains explicit relationships between entities.
-   **Standardized Callbacks**: Use `ServiceCallbacks` (from `types/api.ts`) for service functions and `HookActionCallbacks` (from `types/api.ts`) for hook action methods. Do not create custom callback types per service or hook.

### Forms (Frontend & Backend)
-   **Inertia Form Pattern (MANDATORY)**: All forms MUST use `useForm` hook from `@inertiajs/react`. Forms must call `post`/`put` directly, never use `onSubmit` props or custom submission handlers. See examples in `examples.mdc` section 4.
-   **Shadcn UI Components (MANDATORY)**: All form components (input, textarea, select, etc.) MUST use Shadcn UI components from `@/components/ui/`. Use `Field`, `FieldLabel`, `FieldError` for field structure, and `Input`, `Textarea`, `Select` for form controls. Never use native HTML form elements directly.
-   **Form State Management**: Use `data`, `setData`, `errors`, and `processing` from `useForm`. Never use `useState` for form data.
-   **Dynamic Form Values**: When form needs to update based on props (e.g., editing mode), use `useEffect` with `setData()` for each field. The `useEffect` is necessary because components are not remounted when props change.
-   **Always Send All Fields**: Forms MUST send all fields present in the form, regardless of their values. Empty/unfilled fields must be sent as `null`.
-   **No Conditional Sending**: Do not use conditional logic to exclude fields (e.g., `if (data.email) formData.append('email', data.email)`). Always append all fields.
-   **Backend Direct Persistence**: After validation, Services should persist data directly without redundant existence checks (e.g., `array_key_exists()`). Trust validated data from FormRequest.
-   **Exceptions**: Special validation or conditional logic should be handled case-by-case when explicitly required.

### Laravel Specifics
-   **Controller Responses**: Methods `store()`, `update()`, and `destroy()` MUST always return `to_route()` redirecting to the index route. Never return JSON responses for these actions. Use `abort(404)` for not found cases instead of returning JSON error responses.
-   **Eloquent Observers**: Use PHP Attributes `#[ObservedBy]`.
-   **Helpers**: Prefer `auth()->id()` over `Auth::id()`.
-   **Pivot Tables**: Alphabetical order (`project_role`).
-   **Exceptions**: Handled globally. Controllers do not use `try/catch` for standard flows.
