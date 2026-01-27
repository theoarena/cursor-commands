---
name: examples
description: Examples for Laravel + Inertia + React development
---
# Development Patterns & Examples

## 1. Backend: Architectural Flow
**Context:** Updating a resource (e.g., Cotista).
**Rule:** Controller → Manager → Service → Model.

### Database Queries in Managers

```php
// ❌ WRONG: Manager calling Model directly
class AccommodationTypeManager {
    public function findByWpTermId(array $resortAttributes): ?Hotel {
        $termId = array_key_first($resortAttributes);
        return Hotel::where('wp_term_id', $termId)->first(); // Direct Model call
    }
}

// ✅ CORRECT: Manager delegates to Service
class AccommodationTypeManager {
    public function __construct(
        private readonly HotelManager $hotelManager
    ) {}

    public function findByWpTermId(int $termId): ?Hotel {
        return $this->hotelManager->findByWpTermId($termId);
    }
}

// Service handles database queries
class HotelService {
    public function findByWpTermId(int $termId): ?Hotel {
        return Hotel::where('wp_term_id', $termId)->first();
    }
}
```

### Checklist Before Creating Methods

- **Database query?** → Must be in Service
- **Business logic orchestration?** → Must be in Manager
- **HTTP validation?** → Must be in FormRequest
- **Response formatting?** → Must be in Resource

```php
// ❌ WRONG: Skipping layers + Returning JSON
public function update(Request $request, Cotista $cotista): JsonResponse {
    // Controller logic mixed with DB logic
    $cotista->update($request->all()); // Direct Model call
    return response()->json($cotista);
}

public function store(CompanyRequest $request): JsonResponse {
    $company = $this->manager->create($request->validated());
    return response()->json([
        'message' => 'Empresa criada com sucesso.',
        'data' => new CompanyResource($company),
    ], 201);
}

public function update(CompanyRequest $request, int $id): JsonResponse {
    $company = $this->manager->find($id);
    if (! $company) {
        return response()->json(['message' => 'Empresa não encontrada.'], 404);
    }
    $this->manager->update($company, $request->validated());
    return response()->json(['message' => 'Empresa atualizada com sucesso.']);
}

// ✅ CORRECT: Strict Layer Separation + Redirect to route
// 1. Controller: HTTP Handling (store/update/destroy always redirect)
public function store(CompanyRequest $request) {
    $this->manager->create($request->validated());
    return to_route('admin.companies.index');
}

public function update(CompanyRequest $request, int $id) {
    $company = $this->manager->find($id);
    if (! $company) {
        abort(404, 'Empresa não encontrada.');
    }
    $this->manager->update($company, $request->validated());
    return to_route('admin.companies.index');
}

public function destroy(int $id) {
    $company = $this->manager->find($id);
    if (! $company) {
        abort(404, 'Empresa não encontrada.');
    }
    $this->manager->delete($company);
    return to_route('admin.companies.index');
}

// 2. Manager: Business Logic Orchestration
public function update(Cotista $cotista, array $data): Cotista {
    // Logic: Check permissions, dispatch jobs, etc.
    if (!auth()->user()->can('update', $cotista)) {
        throw new AuthorizationException();
    }
    return $this->service->update($cotista, $data);
}

// 3. Service: Persistence
public function update(Cotista $cotista, array $data): Cotista {
    $cotista->update($data); // Eloquent Call
    return $cotista->refresh();
}
```

**Key Rules:**
- **Store/Update/Destroy**: Always use `to_route('resource.index')` - never return JSON
- **Not Found**: Use `abort(404)` instead of JSON error responses
- **No JsonResponse**: Remove `JsonResponse` return type hints from store/update/destroy methods

---

## 2. Code Simplicity & Defensive Logic

### Null Checks

```ts
// ❌ WRONG: Redundant defensive checks
if (value !== null && value !== undefined) {
  doSomething(value);
}

// ✅ CORRECT: Trust the Type System / Contracts
doSomething(value);

```

### State Management (Single Source of Truth)

```ts
// ❌ WRONG: Syncing two states manually
const [amount, setAmount] = useState<number>(0);
const [display, setDisplay] = useState<string>('0.00'); // Redundant

// ✅ CORRECT: Derive state from truth
const [amount, setAmount] = useState<number>(0);
const display = formatCurrency(amount); // Calculated on render

```

### Early Return (Flattening)

```ts
// ❌ WRONG: Deep nesting
if (hotelId) {
    loadRooms(hotelId);
} else {
    reset();
}

// ✅ CORRECT: Guard clause
if (!hotelId) {
    reset();
    return;
}
loadRooms(hotelId);

```

### Arrow Functions & Inline Expressions

```tsx
// ❌ WRONG: Verbose with explicit return and braces
const template = (rowData: Rate) => {
    if (!rowData.accommodation_type?.title) {
        return <span className="text-gray-400">-</span>;
    }
    return <span>{rowData.accommodation_type.title}</span>;
};

const button = (rowData: Rate) => {
    return (
        <Button
            icon="pi pi-ellipsis-v"
            onClick={(e) => handleClick(e, rowData)}
        />
    );
};

// ✅ CORRECT: Single expression with inline fallback
const template = (rowData: Rate) => (
    <span>{rowData.accommodation_type?.title || "-"}</span>
);

const button = (rowData: Rate) => (
    <Button
        icon="pi pi-ellipsis-v"
        onClick={(e) => handleClick(e, rowData)}
    />
);
```

**Rules:**
- Single expression → Use expression syntax `() => (JSX)` without `{}` or `return`
- Inline fallback → Use `||` or `??` operators instead of if/else
- Less is more → Prefer concise, readable code over verbose structures

---

## 3. Frontend: Data Flow & Inertia

### Fetching Data

```tsx
// ✅ Pattern A: SSR (Initial Load)
// Data comes strictly from Controller -> Inertia Props
const { cotistas } = usePage<{ cotistas: PaginatedResponse }>().props;

// ✅ Pattern B: Async Action (Client Interaction)
// Component -> Hook -> Service -> Inertia
const { fetchList } = useCotistas();
useEffect(() => {
    fetchList(filters); // Triggers router.get() inside Service
}, [filters]);

```

### Interaction Flow (Delete Example)

```tsx
// ❌ WRONG: Component calls Service/Axios directly
<Button onClick={() => axios.delete(`/api/cotistas/${id}`)} />

// ✅ CORRECT: Component delegates to Hook
// UI Component
<Button onClick={() => remove(id)} />

// Hook (useCotistas.ts)
const remove = (id: number) => {
    // Manage local loading state + Call Service
    setLoading(true);
    CotistaService.delete(id, {
        onFinish: () => setLoading(false)
    });
};

```

---

## 4. Filters in CRUD Listings

**Context:** Implementing filters in list pages following the Inertia pattern.

### Complete Filter Pattern

```php
// ✅ Backend: Controller processes query params and passes as props
public function index(): Response
{
    $companyId = request()->query('company_id');
    $users = $this->manager->list(companyId: $companyId ? (int) $companyId : null);

    return Inertia::render('admin/users/Index', [
        'users' => [
            'data' => UserResource::collection($users->items())->resolve(),
            'current_page' => $users->currentPage(),
            'last_page' => $users->lastPage(),
            'per_page' => $users->perPage(),
            'total' => $users->total(),
        ],
        'filters' => [
            'company_id' => $companyId ? (int) $companyId : null,
        ],
    ]);
}
```

```tsx
// ✅ Frontend: Page uses Hook and passes callback to Component
// Index.tsx (Page)
export default function UsersIndex() {
  const { users, filters, fetchList } = useUsers();

  const handleCompanyFilterChange = (companyId: number | null) => {
    fetchList(companyId); // Hook method
  };

  return (
    <UserList
      users={users}
      selectedCompanyId={filters.company_id}
      onCompanyFilterChange={handleCompanyFilterChange}
    />
  );
}

// ✅ Component receives callback as prop
// UserList.tsx (Component)
interface UserListProps {
  selectedCompanyId: number | null;
  onCompanyFilterChange: (companyId: number | null) => void;
}

export default function UserList({ selectedCompanyId, onCompanyFilterChange }: UserListProps) {
  const handleChange = (value: string) => {
    const companyId = value !== 'all' ? parseInt(value, 10) : null;
    onCompanyFilterChange(companyId); // Calls parent callback
  };

  return <Select onValueChange={handleChange} value={selectedCompanyId?.toString() || 'all'} />;
}

// ✅ Hook manages state and calls Service
// useUsers.ts (Hook)
export function useUsers() {
  const { filters } = usePage<UsersPageProps>().props;

  const fetchList = (companyId?: number | null) => {
    UserService.fetchList(companyId); // Service method
  };

  return { filters, fetchList };
}

// ✅ Service performs router.get call
// UserService.ts (Service)
static fetchList(companyId?: number | null): void {
  const queryParams: Record<string, string | number> = {};
  if (companyId) {
    queryParams.company_id = companyId;
  }

  router.get('/admin/users', queryParams, {
    preserveState: true,
    preserveScroll: true,
  });
}
```

### ❌ WRONG: Direct router.get in Component

```tsx
// ❌ WRONG: Component calls router directly
export default function UserList() {
  const handleChange = (value: string) => {
    router.get('/admin/users', { company_id: value }); // Violates architecture
  };
}
```

---

## 5. Forms & Validation (Inertia Pattern)

**Context:** All forms MUST use `useForm` from Inertia. This ensures proper validation error handling and form state management.

### Basic Form Pattern (MANDATORY)

```tsx
import { useEffect } from 'react';
import { useForm } from '@inertiajs/react';
import { Field, FieldLabel, FieldError } from '@/components/ui/field';
import { Input } from '@/components/ui/input';
import { Button } from '@/components/ui/button';
import { store, update } from '@/actions/App/Http/Controllers/CompanyController';

export default function CompanyForm({ company, onClose, open }: CompanyFormProps) {
  // ✅ CORRECT: Use useForm hook from Inertia
  const { data, setData, post, put, errors, processing } = useForm({
    name: '',
    metadata: null,
  });

  // ✅ CORRECT: Update form values when company prop changes
  useEffect(() => {
    if (company) {
      setData('name', company.name);
      setData('metadata', (company.metadata || null) as any);
    } else {
      setData('name', '');
      setData('metadata', null);
    }
  }, [company, setData]);

  // ✅ CORRECT: Call post/put directly, no onSubmit prop
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (company) {
      put(update(company.id).url, {
        onSuccess: () => onClose(),
      });
    } else {
      post(store().url, {
        onSuccess: () => onClose(),
      });
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <Field>
        <FieldLabel htmlFor="name">Nome</FieldLabel>
        <Input
          id="name"
          value={data.name}
          onChange={(e) => setData('name', e.target.value)}
          disabled={processing}
          aria-invalid={errors.name ? 'true' : 'false'}
        />
        <FieldError>{errors.name}</FieldError>
      </Field>

      <Button type="submit" disabled={processing}>
        {processing ? 'Salvando...' : company ? 'Atualizar' : 'Criar'}
      </Button>
    </form>
  );
}
```

### ❌ WRONG: Using useState and onSubmit prop

```tsx
// ❌ WRONG: Using useState for form data
const [formData, setFormData] = useState({ name: '', email: '' });

// ❌ WRONG: Passing onSubmit prop
interface FormProps {
  onSubmit: (data: FormData) => void;
  loading?: boolean;
}

// ❌ WRONG: Custom submission handler
const handleSubmit = (e: React.FormEvent) => {
  e.preventDefault();
  onSubmit(formData); // Delegating to parent
};
```

### ✅ CORRECT: Using useForm directly

```tsx
// ✅ CORRECT: useForm manages all form state
const { data, setData, post, put, errors, processing } = useForm({
  name: '',
  email: '',
});

// ✅ CORRECT: Form calls post/put directly
const handleSubmit = (e: React.FormEvent) => {
  e.preventDefault();
  post(store().url, {
    onSuccess: () => {
      onClose();
    },
  });
};
```

### Form with Client-Side Validation (Joi)

**Context:** When using client-side validation with Joi, combine with Inertia form pattern.

```tsx
import { useJoiValidation } from '@/lib/validation';
import { cotistaSchema } from '@/lib/validation/schemas';

// Inside Component
const { validate, errors: clientErrors, clearErrors } = useJoiValidation(cotistaSchema);
const { data, setData, post, errors: serverErrors } = useForm({
  name: '',
  email: '',
});

const handleSubmit = (e: React.FormEvent) => {
  e.preventDefault();
  clearErrors();

  // 1. Client Validation (Immediate)
  if (!validate(data)) return;

  // 2. Server Submission
  post(store().url);
};

// Error Display: Combine Joi (client) + Laravel (server) errors
<Field>
  <FieldLabel htmlFor="email">Email</FieldLabel>
  <Input
    id="email"
    aria-invalid={clientErrors.email || serverErrors.email ? 'true' : 'false'}
  />
  <FieldError>{clientErrors.email || serverErrors.email}</FieldError>
</Field>
```

### Key Rules

1. **Always use `useForm`**: Never use `useState` for form data
2. **Shadcn UI components**: Use `Field`, `FieldLabel`, `FieldError`, `Input`, `Textarea`, `Select` etc. from `@/components/ui/`. Never use native HTML form elements
3. **Direct submission**: Forms call `post`/`put` directly, never use `onSubmit` props
4. **Dynamic values**: Use `useEffect` with `setData()` when form values depend on props
5. **Error display**: Use `FieldError` component with `errors.field` from `useForm`
6. **Processing state**: Use `processing` from `useForm` to disable buttons/inputs during submission

---

## 6. Notifications (Centralized System)

**Context:** Using the standardized `useNotifier` hook.

```tsx
import { useNotifier } from '@/lib/notifications';

// ❌ WRONG: Hardcoded strings or direct Toast usage
toast.show({ detail: 'Saved successfully' });
notify('User saved'); // Key doesn't exist

// ✅ CORRECT: Use predefined semantic keys
const notify = useNotifier();

// Simple success
notify('cotista.create.success');

// With dynamic parameters
notify('booking.voucher.sent', { email: 'user@example.com' });

// Error with override
notify('system.error.generic', undefined, { detail: 'API timeout' });
```

---

## 7. Type Definitions & Relationships

**Context:** Type definitions must use indexed access types for relationship fields to maintain type safety and explicit relationships.

```typescript
// ❌ WRONG: Using primitive types for relationship fields
export interface Diagnostic {
  id: number;
  company_id: number; // No type reference
  questionnaire_template_id: number; // No type reference
}

// ✅ CORRECT: Using indexed access types for relationships
import type { Company } from './company';
import type { QuestionnaireTemplate } from './questionnaire';

export interface Diagnostic {
  id: number;
  company_id: Company['id']; // References Company type
  questionnaire_template_id: QuestionnaireTemplate['id']; // References QuestionnaireTemplate type
}

export interface DiagnosticFormData {
  questionnaire_template_id: QuestionnaireTemplate['id']; // Also in form data
}
```

## 8. Standardized Callbacks (Services & Hooks)

**Context:** Service functions and hook action methods must use standardized callback types from `types/api.ts`.

### Service Functions```typescript
// ✅ CORRECT: Use ServiceCallbacks type
import type { ServiceCallbacks } from '@/types/api';

export function createCotista(
    data: CreateCotistaData,
    callbacks?: ServiceCallbacks
): void {
    router.post(cotistasRoutes.store(), formData, {
        onSuccess: () => callbacks?.onSuccess?.(),
        onError: (errors) => callbacks?.onError?.(errors as Record<string, string>),
    });
}

// ❌ WRONG: Inline callback type definitions
export function createCotista(
    data: CreateCotistaData,
    callbacks?: {
        onSuccess?: () => void;
        onError?: (errors: Record<string, string>) => void;
    }
): void {
    // ...
}
```

### Hook Action Methods

```typescript
// ✅ CORRECT: Use HookActionCallbacks type
import type { HookActionCallbacks } from '@/types/api';

export function useCotistas() {
    const create = async (
        data: CreateCotistaData,
        callbacks?: HookActionCallbacks
    ): Promise<void> => {
        // Hook logic with callbacks
        await createCotista(data, {
            onSuccess: () => {
                callbacks?.onSuccess?.();
            },
            onError: (errors) => {
                const errorMessage = getFirstErrorMessage(errors) || 'Error';
                callbacks?.onError?.(errors, errorMessage);
            },
        });
    };
}

// ❌ WRONG: Custom callback type per hook
export type CotistaActionCallbacks = {
    onSuccess?: () => void;
    onError?: (errors: Record<string, string>, errorMessage: string) => void;
};
```

