# Frontend Architecture

**Project:** [PROJECT_NAME]  
**Framework:** [Angular/React/Vue/Svelte]  
**Language:** [TypeScript/JavaScript]  
**Last Updated:** [DATE]  
**Next Review:** [DATE]

---

## Overview

This document defines frontend architecture standards for [PROJECT_NAME]. The frontend is **presentation-only**; all business logic lives in the backend.

**Core Philosophy:** Dumb Terminal Architecture applied to UI.

---

## Table of Contents

1. [Dumb Terminal for UI](#dumb-terminal-for-ui)
2. [Smart vs Dumb Components](#smart-vs-dumb-components)
3. [Data Flow Architecture](#data-flow-architecture)
4. [State Management](#state-management)
5. [Service Layer](#service-layer)
6. [API Integration](#api-integration)
7. [Common Violations](#common-violations)
8. [Testing Strategy](#testing-strategy)
9. [Performance Optimization](#performance-optimization)
10. [Anti-Patterns](#anti-patterns)

---

## Dumb Terminal for UI

### Principle

**The frontend is presentation-only.** All business logic lives in the backend.

The UI's job is to:
- Display data from backend
- Capture user input
- Send requests to backend
- Show results to user

The backend's job is to:
- Make all business decisions
- Validate all data
- Enforce all rules
- Protect sensitive operations

### What Belongs in Backend (NOT UI)

**Business Logic Layer:**
```
❌ Authentication (checking tokens)
❌ Authorization (who can do what)
❌ Data validation rules
❌ Workflow decisions
❌ Payment processing
❌ Encryption/decryption
❌ Audit logging
❌ Rate limiting
❌ Access control
❌ Permissions checking
❌ Business calculations
❌ Data transformation (complex)
```

**Examples:**

```javascript
// ❌ WRONG - Authorization in UI
if (user.role === 'admin') {
  showAdminPanel();
}
// User can hack this!

// ✅ RIGHT - Authorization from backend
const response = await api.get('/api/admin-panel');
if (response.status === 200) {
  showAdminPanel(response.data);
} else if (response.status === 403) {
  showAccessDenied();
}
```

```javascript
// ❌ WRONG - Price calculation in UI
const discount = price > 100 ? 0.1 : 0;
const total = price * (1 - discount);
// User can modify!

// ✅ RIGHT - Backend calculates price
const response = await api.post('/checkout', {
  items: cart
});
// Backend returns: { subtotal, discount, total, tax }
```

```javascript
// ❌ WRONG - Validation only in UI
if (email.includes('@')) {
  submit();
}
// User can bypass!

// ✅ RIGHT - Validation on both sides
if (email.includes('@')) {  // Quick UX feedback
  const response = await api.post('/users', { email });
  if (response.status === 400) {
    showError('Invalid email format');  // Server rejected
  }
}
```

### What Belongs in Frontend (UI Layer)

**Presentation Logic:**
```
✅ Format validation (format only, not business rules)
✅ Show/hide UI elements based on state
✅ Theme switching (light/dark mode)
✅ Sorting/filtering display data
✅ Form input masking
✅ Loading indicators
✅ Error message display
✅ Animation/transitions
✅ Responsive design
✅ Keyboard navigation
✅ Accessibility features
```

**Examples:**

```javascript
// ✅ Format validation is OK
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
if (!emailRegex.test(email)) {
  showError('Please enter a valid email format');
  return;
}
// But backend will also validate!
```

```javascript
// ✅ Show/hide based on data from backend
if (user.canDelete) {  // Backend decided this
  showDeleteButton();
}
```

```javascript
// ✅ Filtering display data is OK
const filtered = users.filter(u => u.name.includes(searchTerm));
// This is just presentation, not business logic
```

### Decision Tree: Where Does Logic Go?

```
New feature needed?

├─ Is it a business rule?
│  (Affects data, security, calculations)
│  └─ YES → Backend (100%)
│
├─ Does it need to be enforced?
│  (Someone might bypass it)
│  └─ YES → Backend (100%)
│
├─ Can users change it?
│  (Price, discount, permissions)
│  └─ YES → Backend (100%)
│
├─ Is it just presentation?
│  (Sort, filter, show/hide, format)
│  └─ YES → Frontend (100%)
│
└─ Is it user preference?
   (Theme, language, layout)
   └─ YES → Frontend (100%), store in localStorage
```

---

## UIController Pattern (MANDATORY)

**Frontend uses UIControllers for state management and API delegation.**

### UIController Requirements

UIControllers are the bridge between UI components and backend APIs. They MUST follow strict rules.

**UIController MUST:**
- ✅ Be framework-agnostic (ZERO [FRAMEWORK_NAME] imports)
- ✅ Have ZERO database access
- ✅ Communicate via HttpClient for REST API calls only
- ✅ Manage local state (forms, selections, pagination)
- ✅ Handle error responses gracefully
- ✅ Be injectable (constructor-based)
- ✅ Be testable without UI or database

**UIController MUST NOT:**
- ❌ Import UI framework classes ([FRAMEWORK_NAME], Swing, AWT)
- ❌ Import database libraries (JDBC, JPA, MongoDB driver)
- ❌ Instantiate services directly (`new ServiceClass()`)
- ❌ Access configuration or environment directly
- ❌ Import Spring context or Spring beans
- ❌ Have UI rendering logic

### Example: Correct UIController

```typescript
// ✅ CORRECT: Framework-agnostic UIController
export class CategoryEditorUIController {
    private readonly httpClient: HttpClient;
    private categories: CategoryDto[] = [];
    private currentCategory: CategoryDto | null = null;
    
    constructor(httpClient: HttpClient) {
        this.httpClient = httpClient;
    }
    
    public async loadCategories(): Promise<void> {
        this.categories = await this.httpClient.get('/api/categories');
    }
    
    public async save(dto: CategoryDto): Promise<void> {
        const result = await this.httpClient.post('/api/categories', dto);
        this.currentCategory = result;
    }
    
    public getCategories(): CategoryDto[] {
        return this.categories;
    }
}

// ✅ CORRECT: UI component delegates to UIController
export class CategoryEditorPanel {
    private controller: CategoryEditorUIController;
    
    constructor(controller: CategoryEditorUIController) {
        this.controller = controller;
    }
    
    public async onSave(): Promise<void> {
        const formData = this.getFormData();
        await this.controller.save(formData);  // Delegates only
        this.refreshDisplay();
    }
}
```

### Example: Wrong UIController

```typescript
// ❌ WRONG: Imports UI framework
import { Component } from '@angular/core';  // NO!
export class CategoryEditorUIController extends Component { ... }

// ❌ WRONG: Direct database access
import { getDatabase } from 'firebase';
public load() {
    const categories = getDatabase().ref('categories').get();  // NO!
}

// ❌ WRONG: Creates services directly
export class CategoryEditorUIController {
    private service = new CategoryService();  // NO!
}
```

### Testing UIControllers

UIControllers are fully testable without UI or database:

```typescript
@Test
public async testSaveCategory() {
    // Mock HttpClient
    const mockHttp = { post: jest.fn() };
    const controller = new CategoryEditorUIController(mockHttp);
    
    // Test business logic
    await controller.save({ id: 1, name: 'Electronics' });
    
    // Verify API call
    expect(mockHttp.post).toHaveBeenCalledWith(
        '/api/categories',
        { id: 1, name: 'Electronics' }
    );
}
```

---

## Smart vs Dumb Components

### Component Architecture

**Two types of components:**

| Aspect | Dumb (Presentational) | Smart (Container) |
|--------|---|---|
| **Data** | Via `@Input()` | Fetches from services |
| **Events** | Via `@Output()` | Handles & responds |
| **Services** | None (injectable deps) | Multiple services |
| **Logic** | None (display only) | API calls, business flow |
| **Testability** | Very easy (pure functions) | Harder (mocked services) |
| **Reusability** | High (generic) | Low (specific to flow) |
| **Location** | `/shared/components/` | `/features/*/components/` |

### Dumb Component Example

**Purpose:** Display data, handle user interaction, emit events.

```typescript
import { Component, Input, Output, EventEmitter } from '@angular/core';

// Dumb Component - Presentational Only
@Component({
  selector: 'app-user-card',
  template: `
    <div class="card">
      <img [src]="user.avatar" alt="{{ user.name }}">
      <h3>{{ user.name }}</h3>
      <p class="email">{{ user.email }}</p>
      <p class="role">{{ user.role }}</p>
      
      <div class="actions">
        <button (click)="onEdit()">Edit</button>
        <button 
          (click)="onDelete()"
          [disabled]="isDeleting"
          class="danger">
          {{ isDeleting ? 'Deleting...' : 'Delete' }}
        </button>
      </div>
    </div>
  `,
  styleUrls: ['./user-card.component.css']
})
export class UserCardComponent {
  // Receive data from parent
  @Input() user: User;
  @Input() isDeleting: boolean = false;
  
  // Emit events to parent
  @Output() edit = new EventEmitter<User>();
  @Output() delete = new EventEmitter<number>();
  
  // NO services injected
  // NO API calls
  // NO business logic
  
  onEdit() {
    this.edit.emit(this.user);  // Just emit
  }
  
  onDelete() {
    this.delete.emit(this.user.id);  // Just emit
  }
}
```

**Why this is Dumb:**
- ✅ No `UserService` injected
- ✅ No `@Injectable()` decorator
- ✅ No `constructor(private service: UserService)`
- ✅ No API calls (`this.service.delete()`)
- ✅ Just displays data and emits events
- ✅ Easy to test (pass data, check output)
- ✅ Highly reusable

---

### Smart Component Example

**Purpose:** Manage state, call services, orchestrate dumb components.

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { Observable, Subject } from 'rxjs';
import { takeUntil } from 'rxjs/operators';

// Smart Component - Container/Feature
@Component({
  selector: 'app-users-list',
  template: `
    <div class="container">
      <h2>Users</h2>
      
      <!-- Loading state -->
      <app-loading *ngIf="isLoading$ | async"></app-loading>
      
      <!-- Error state -->
      <app-error 
        *ngIf="error$ | async as error"
        [error]="error"
        (retry)="loadUsers()">
      </app-error>
      
      <!-- User list (dumb component) -->
      <div class="user-grid">
        <app-user-card
          *ngFor="let user of users$ | async"
          [user]="user"
          [isDeleting]="deletingIds.has(user.id)"
          (edit)="onEditUser($event)"
          (delete)="onDeleteUser($event)">
        </app-user-card>
      </div>
    </div>
  `
})
export class UsersListComponent implements OnInit, OnDestroy {
  // Observables for state
  users$: Observable<User[]>;
  isLoading$: Observable<boolean>;
  error$: Observable<string | null>;
  
  // Local state
  deletingIds = new Set<number>();
  
  private destroy$ = new Subject<void>();
  
  // MULTIPLE services injected
  constructor(
    private userService: UserService,
    private notificationService: NotificationService,
    private router: Router
  ) {}
  
  ngOnInit() {
    this.loadUsers();
  }
  
  loadUsers() {
    // Call backend API
    this.users$ = this.userService.getUsers()
      .pipe(takeUntil(this.destroy$));
  }
  
  onEditUser(user: User) {
    // Handle business logic
    this.router.navigate(['/users', user.id, 'edit']);
  }
  
  onDeleteUser(userId: number) {
    // Handle deletion with loading state
    this.deletingIds.add(userId);
    
    this.userService.delete(userId)
      .pipe(takeUntil(this.destroy$))
      .subscribe(
        () => {
          this.notificationService.success('User deleted');
          this.loadUsers();  // Refresh list
        },
        (error) => {
          this.notificationService.error('Failed to delete user');
          this.deletingIds.delete(userId);
        }
      );
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

**Why this is Smart:**
- ✅ Multiple services injected
- ✅ Calls backend APIs
- ✅ Manages application state
- ✅ Handles errors
- ✅ Orchestrates child components
- ✅ Contains business logic
- ✅ Specific to this feature

---

### Component Hierarchy Pattern

**Recommended structure:**

```
App (Smart)
  ├── Header (Dumb)
  ├── Sidebar (Dumb)
  └── Router Outlet
      └── Feature Module
          ├── FeatureContainer (Smart)
          │   ├── FeaturePage (Smart - page-specific logic)
          │   │   ├── FeatureForm (Smart - form state)
          │   │   │   ├── Input (Dumb)
          │   │   │   ├── Button (Dumb)
          │   │   │   └── Checkbox (Dumb)
          │   │   └── FeatureList (Smart - list state)
          │   │       └── FeatureItem (Dumb)
          │   └── FeatureSidebar (Dumb)
          └── Shared Components (All Dumb)
              ├── Button
              ├── Modal
              ├── LoadingSpinner
              └── ErrorMessage
```

**Rules:**
- 1 Smart component per feature
- Multiple Dumb components per feature
- Dumb components in `/shared/components/`
- Smart components in `/features/[feature]/`
- Never nest Smart in Dumb

---

## Data Flow Architecture

### Unidirectional Data Flow

**Data flows DOWN, events flow UP.**

```
┌─────────────────────────────────────────┐
│         Backend API                     │
└──────────────────┬──────────────────────┘
                   │ GET /api/users
                   ▼
┌─────────────────────────────────────────┐
│   Smart Component (Container)           │
│   ├─ Calls UserService.getUsers()      │
│   ├─ Manages users$ Observable         │
│   └─ Handles user events                │
└──────────────────┬──────────────────────┘
                   │ [user, isLoading]
                   ▼
┌─────────────────────────────────────────┐
│   Dumb Component (Presentational)       │
│   ├─ @Input user                        │
│   ├─ @Input isLoading                   │
│   └─ Displays data                      │
└──────────────────┬──────────────────────┘
                   │ User clicks button
                   ▼
┌─────────────────────────────────────────┐
│   Event Emitted                         │
│   @Output() delete.emit(userId)         │
└──────────────────┬──────────────────────┘
                   │ (delete)="onDelete($event)"
                   ▼
┌─────────────────────────────────────────┐
│   Smart Component Handles Event         │
│   ├─ Calls UserService.delete(userId)  │
│   ├─ Updates local state                │
│   └─ Triggers API call                  │
└──────────────────┬──────────────────────┘
                   │ DELETE /api/users/123
                   ▼
┌─────────────────────────────────────────┐
│         Backend API                     │
└─────────────────────────────────────────┘
```

### Flow Comparison

**❌ WRONG - Bidirectional:**

```typescript
// Component modifies parent data directly
@Component({
  template: `
    <input [(ngModel)]="user.name">  // Two-way binding!
  `
})
export class UserFormComponent {
  @Input() user: User;
  
  // User changes input → user.name changes
  // Parent has no control → hard to track changes
  // Multiple components can modify same object
}
```

**✅ RIGHT - Unidirectional:**

```typescript
// Parent owns data, child emits events
@Component({
  template: `
    <input 
      [value]="user.name"
      (change)="onNameChange($event)">
  `
})
export class UserFormComponent {
  @Input() user: User;
  @Output() nameChanged = new EventEmitter<string>();
  
  onNameChange(newName: string) {
    this.nameChanged.emit(newName);  // Emit, don't modify
  }
}

// Parent handles
@Component({
  template: `
    <app-user-form
      [user]="user"
      (nameChanged)="updateUser($event)">
    </app-user-form>
  `
})
export class UserListComponent {
  updateUser(newName: string) {
    // Parent controls the update
    this.userService.update({ name: newName });
  }
}
```

---

## State Management

### When to Use State Management

**Simple Rule:**

```
One Smart Component managing state → OK
Multiple Smart Components sharing state → Use State Management
```

### State Management Patterns

**Option 1: Service-Based (Recommended for simple apps)**

```typescript
// state.service.ts
import { Injectable } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class UserStateService {
  // Private state
  private usersSubject = new BehaviorSubject<User[]>([]);
  
  // Public observables (read-only)
  public users$ = this.usersSubject.asObservable();
  
  constructor(private api: ApiService) {}
  
  // Methods to modify state
  loadUsers() {
    this.api.getUsers().subscribe(users => {
      this.usersSubject.next(users);  // Update state
    });
  }
  
  updateUser(user: User) {
    this.api.updateUser(user).subscribe(updated => {
      const current = this.usersSubject.value;
      const index = current.findIndex(u => u.id === user.id);
      current[index] = updated;
      this.usersSubject.next([...current]);  // Immutable update
    });
  }
}

// Usage in component
@Component({
  template: `<div *ngFor="let user of userState.users$ | async">...</div>`
})
export class UserListComponent {
  constructor(public userState: UserStateService) {
    this.userState.loadUsers();
  }
}
```

**Option 2: Redux/NgRx (For complex apps)**

```typescript
// store/user.actions.ts
export const loadUsers = createAction('[Users] Load Users');
export const loadUsersSuccess = createAction(
  '[Users] Load Users Success',
  props<{ users: User[] }>()
);

// store/user.reducer.ts
const initialState: UserState = { users: [], loading: false };

export const userReducer = createReducer(
  initialState,
  on(loadUsers, state => ({ ...state, loading: true })),
  on(loadUsersSuccess, (state, { users }) => ({
    users,
    loading: false
  }))
);

// Usage
@Component({
  template: `<div *ngFor="let user of users$ | async">...</div>`
})
export class UserListComponent {
  users$ = this.store.select(state => state.users.users);
  
  constructor(private store: Store<AppState>) {
    this.store.dispatch(loadUsers());
  }
}
```

**Option 3: Reactive (RxJS only)**

```typescript
export class UserListComponent {
  // Compose multiple sources
  users$ = this.userService.users$.pipe(
    shareReplay(1)
  );
  
  isLoading$ = this.userService.loading$;
  
  selectedUser$ = this.route.params.pipe(
    switchMap(params => this.userService.getUser(params.id))
  );
  
  combined$ = combineLatest([
    this.users$,
    this.selectedUser$
  ]).pipe(
    map(([users, selected]) => ({ users, selected }))
  );
}
```

### State Management Guidelines

```
✅ Keep state in service
✅ Expose state as Observable
✅ Provide methods to update state
✅ Never expose state directly
✅ Always use immutable updates
❌ Store state in components
❌ Mutate state directly
❌ Share component state between features
```

---

## Service Layer

### Service Responsibilities

**Each service has ONE job:**

```
UserService → User-related API calls + user state
PostService → Post-related API calls + post state
AuthService → Authentication (tokens, login, logout)
NotificationService → User notifications (toasts, dialogs)
ThemeService → Theme management (dark/light mode)
```

### Example Service

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { BehaviorSubject, Observable } from 'rxjs';
import { tap, shareReplay } from 'rxjs/operators';

@Injectable({ providedIn: 'root' })
export class UserService {
  // Private state
  private usersSubject = new BehaviorSubject<User[]>([]);
  private loadingSubject = new BehaviorSubject<boolean>(false);
  
  // Public observables
  public users$ = this.usersSubject.asObservable();
  public loading$ = this.loadingSubject.asObservable();
  
  constructor(private http: HttpClient) {}
  
  // GET all users
  getUsers(): Observable<User[]> {
    this.loadingSubject.next(true);
    
    return this.http.get<User[]>('/api/users').pipe(
      tap(users => {
        this.usersSubject.next(users);
        this.loadingSubject.next(false);
      }),
      shareReplay(1)  // Cache result
    );
  }
  
  // GET single user
  getUser(id: number): Observable<User> {
    return this.http.get<User>(`/api/users/${id}`);
  }
  
  // POST create user
  create(user: Omit<User, 'id'>): Observable<User> {
    return this.http.post<User>('/api/users', user).pipe(
      tap(newUser => {
        const current = this.usersSubject.value;
        this.usersSubject.next([...current, newUser]);
      })
    );
  }
  
  // PUT update user
  update(user: User): Observable<User> {
    return this.http.put<User>(`/api/users/${user.id}`, user).pipe(
      tap(updated => {
        const current = this.usersSubject.value;
        const index = current.findIndex(u => u.id === user.id);
        current[index] = updated;
        this.usersSubject.next([...current]);
      })
    );
  }
  
  // DELETE user
  delete(id: number): Observable<void> {
    return this.http.delete<void>(`/api/users/${id}`).pipe(
      tap(() => {
        const current = this.usersSubject.value;
        this.usersSubject.next(current.filter(u => u.id !== id));
      })
    );
  }
}
```

### Service Guidelines

```
✅ One Observable per state item
✅ Private state, public observables
✅ Update state after API calls
✅ Use shareReplay() to cache
✅ Handle loading/error states
✅ Consistent error handling
❌ Return Subjects (expose internal state)
❌ Require calling code to know about state
❌ No side effects outside subscriptions
```

---

## API Integration

### HTTP Interceptor for Common Tasks

```typescript
import { Injectable } from '@angular/core';
import {
  HttpRequest,
  HttpHandler,
  HttpEvent,
  HttpInterceptor,
  HttpErrorResponse
} from '@angular/common/http';
import { Observable, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';

@Injectable()
export class ErrorInterceptor implements HttpInterceptor {
  constructor(private notificationService: NotificationService) {}
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    return next.handle(req).pipe(
      catchError((error: HttpErrorResponse) => {
        // Handle errors consistently
        let message = 'An error occurred';
        
        if (error.status === 401) {
          message = 'Please log in again';
          // Logout user
        } else if (error.status === 403) {
          message = 'You do not have permission';
        } else if (error.status === 404) {
          message = 'Resource not found';
        } else if (error.status >= 500) {
          message = 'Server error. Please try again later';
        }
        
        this.notificationService.error(message);
        return throwError(() => error);
      })
    );
  }
}
```

### Error Handling Pattern

```typescript
// ❌ WRONG
this.userService.getUsers().subscribe(
  users => this.users = users
  // What happens on error? Nothing!
);

// ✅ RIGHT
this.users$ = this.userService.getUsers().pipe(
  catchError(error => {
    this.notificationService.error('Failed to load users');
    return of([]);  // Return empty array
  })
);
```

---

## Common Violations

### Violation 1: Authorization in UI

**❌ WRONG:**
```typescript
if (this.userService.currentUser.role === 'admin') {
  showAdminMenu();
}
```

**✅ RIGHT:**
```typescript
// Backend returns canViewAdmin based on role
this.hasAdminAccess$ = this.userService.currentUser$.pipe(
  map(user => user.canViewAdmin)  // Backend decided
);
```

---

### Violation 2: Complex Calculations in UI

**❌ WRONG:**
```typescript
const total = this.cart.items.reduce((sum, item) => {
  const itemTotal = item.price * item.quantity;
  const discount = itemTotal > 100 ? itemTotal * 0.1 : 0;
  return sum + itemTotal - discount;
}, 0);
```

**✅ RIGHT:**
```typescript
// Backend calculates
const response = await this.api.post('/checkout/calculate', { items: this.cart });
this.total = response.total;  // Display backend result
this.discount = response.discount;
this.tax = response.tax;
```

---

### Violation 3: Validation Only in UI

**❌ WRONG:**
```typescript
onSubmit() {
  if (this.form.valid) {
    this.api.post('/users', this.form.value).subscribe(...);
  }
}
```

**✅ RIGHT:**
```typescript
onSubmit() {
  // UI validation for UX
  if (!this.form.valid) {
    this.showFormErrors();
    return;
  }
  
  // Backend also validates
  this.api.post('/users', this.form.value).subscribe(
    success => this.handleSuccess(),
    error => this.showServerErrors(error)  // Show backend errors
  );
}
```

---

### Violation 4: Two-Way Binding on Business Data

**❌ WRONG:**
```html
<input [(ngModel)]="user.email">
<!-- Direct modification, no validation -->
```

**✅ RIGHT:**
```typescript
// Component
@Component({
  template: `
    <input 
      [value]="user.email"
      (change)="onEmailChange($event)">
  `
})
export class UserFormComponent {
  @Input() user: User;
  @Output() emailChanged = new EventEmitter<string>();
  
  onEmailChange(newEmail: string) {
    this.emailChanged.emit(newEmail);
  }
}

// Parent
@Component({
  template: `
    <app-user-form
      [user]="user"
      (emailChanged)="updateEmail($event)">
    </app-user-form>
  `
})
export class UserListComponent {
  updateEmail(newEmail: string) {
    this.userService.update({ email: newEmail });
  }
}
```

---

### Violation 5: No Error Handling

**❌ WRONG:**
```typescript
this.userService.delete(userId).subscribe(
  () => this.refresh()
);
// No error handling!
```

**✅ RIGHT:**
```typescript
this.userService.delete(userId).subscribe(
  () => {
    this.notificationService.success('User deleted');
    this.refresh();
  },
  (error) => {
    this.notificationService.error('Failed to delete user');
    console.error('Delete error:', error);
  }
);
```

---

### Violation 6: Memory Leaks (No Unsubscribe)

**❌ WRONG:**
```typescript
ngOnInit() {
  this.userService.users$.subscribe(users => {
    this.users = users;  // Memory leak!
  });
}
```

**✅ RIGHT:**
```typescript
private destroy$ = new Subject<void>();

ngOnInit() {
  this.userService.users$
    .pipe(takeUntil(this.destroy$))
    .subscribe(users => {
      this.users = users;
    });
}

ngOnDestroy() {
  this.destroy$.next();
  this.destroy$.complete();
}
```

---

### Violation 7: Tight Coupling to Service

**❌ WRONG:**
```typescript
// Component depends on specific service implementation
@Component({
  template: `<div>{{ userService.users[0].name }}</div>`
})
export class HeaderComponent {
  constructor(public userService: UserService) {}
}
```

**✅ RIGHT:**
```typescript
// Component depends on interface
@Component({
  template: `<div>{{ (currentUser$ | async)?.name }}</div>`
})
export class HeaderComponent {
  currentUser$ = this.userService.currentUser$;
  
  constructor(private userService: UserService) {}
}
```

---

## Testing Strategy

### Testing Dumb Components (Easy)

```typescript
describe('UserCardComponent', () => {
  let component: UserCardComponent;
  let fixture: ComponentFixture<UserCardComponent>;
  
  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [UserCardComponent]
      // NO services!
    }).compileComponents();
    
    fixture = TestBed.createComponent(UserCardComponent);
    component = fixture.componentInstance;
  });
  
  it('should display user name', () => {
    component.user = { id: 1, name: 'John', email: 'john@example.com' };
    fixture.detectChanges();
    
    expect(fixture.nativeElement.textContent).toContain('John');
  });
  
  it('should emit delete event when delete clicked', () => {
    spyOn(component.delete, 'emit');
    component.user = { id: 1, name: 'John', email: 'john@example.com' };
    fixture.detectChanges();
    
    const deleteBtn = fixture.nativeElement.querySelector('.delete-btn');
    deleteBtn.click();
    
    expect(component.delete.emit).toHaveBeenCalledWith(1);
  });
});
```

### Testing Smart Components (Harder)

```typescript
describe('UsersListComponent', () => {
  let component: UsersListComponent;
  let fixture: ComponentFixture<UsersListComponent>;
  let userServiceMock: jasmine.SpyObj<UserService>;
  
  beforeEach(async () => {
    userServiceMock = jasmine.createSpyObj('UserService', [
      'getUsers',
      'delete'
    ]);
    userServiceMock.getUsers.and.returnValue(of([
      { id: 1, name: 'John' }
    ]));
    
    await TestBed.configureTestingModule({
      declarations: [UsersListComponent],
      providers: [
        { provide: UserService, useValue: userServiceMock }
      ]
    }).compileComponents();
    
    fixture = TestBed.createComponent(UsersListComponent);
    component = fixture.componentInstance;
  });
  
  it('should load users on init', () => {
    fixture.detectChanges();
    
    expect(userServiceMock.getUsers).toHaveBeenCalled();
  });
  
  it('should delete user when delete event emitted', () => {
    userServiceMock.delete.and.returnValue(of(void 0));
    fixture.detectChanges();
    
    component.onDeleteUser(1);
    
    expect(userServiceMock.delete).toHaveBeenCalledWith(1);
  });
});
```

---

## Performance Optimization

### Change Detection Strategy

```typescript
// ❌ Default (checks all components)
@Component({
  selector: 'app-user-card'
})
export class UserCardComponent { }

// ✅ OnPush (checks only when inputs change)
@Component({
  selector: 'app-user-card',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserCardComponent {
  @Input() user: User;  // Notify when this changes
}
```

### Lazy Loading Routes

```typescript
const routes: Routes = [
  {
    path: 'users',
    loadChildren: () => import('./users/users.module')
      .then(m => m.UsersModule)
  }
];
```

### Memoization with MemoizedSelector

```typescript
// Selector only recomputes if inputs change
export const selectUser = (id: number) => 
  createSelector(
    selectUsers,
    (users: User[]) => users.find(u => u.id === id)
  );
```

### Async Pipe (Auto-unsubscribe)

```typescript
// ❌ Manual subscription
users: User[];
ngOnInit() {
  this.userService.getUsers().subscribe(u => this.users = u);
}

// ✅ Async pipe
users$ = this.userService.getUsers();
// Template: *ngFor="let user of users$ | async"
// Auto-unsubscribes on destroy!
```

---

## Anti-Patterns

| Anti-Pattern | Wrong | Right |
|---|---|---|
| **God Component** | One component doing everything | Break into Smart/Dumb |
| **Template Logic** | Business logic in template | Move to component |
| **Hardcoded URLs** | URLs in components | Inject service |
| **Direct DOM Access** | Use `document.querySelector()` | Use `@ViewChild` |
| **Mixing concerns** | Component + HTTP + state | Separate into services |
| **No loading state** | No feedback on API call | Show spinner/message |
| **Ignoring errors** | No error handling | Show error message |
| **String IDs** | Use `string` for IDs | Use proper types |

---

## Frontend Checklist

**Before shipping frontend code:**

- [ ] No business logic in components
- [ ] No authorization checks in UI
- [ ] No authentication logic in UI
- [ ] No data validation rules in UI (only format)
- [ ] All Smart components have tests
- [ ] All Dumb components have tests
- [ ] No service dependencies in Dumb components
- [ ] No memory leaks (proper unsubscribe)
- [ ] Error handling on all API calls
- [ ] Loading states implemented
- [ ] No hardcoded URLs
- [ ] No console.log in production
- [ ] Responsive design tested
- [ ] Accessibility tested (keyboard, screen reader)
- [ ] Performance profiled
- [ ] Security headers present
- [ ] No sensitive data in localStorage
- [ ] No passwords/tokens in logs

---

## Quick Reference

**Always:**
- ✅ Keep business logic in backend
- ✅ Use Dumb components for presentation
- ✅ One Smart component per feature
- ✅ Emit events from Dumb to Smart
- ✅ Data flows down, events flow up
- ✅ Use RxJS Observables
- ✅ Handle errors explicitly
- ✅ Show loading states
- ✅ Test components thoroughly
- ✅ Unsubscribe from Observables

**Never:**
- ❌ Put authorization in UI
- ❌ Calculate prices in frontend
- ❌ Validate business rules in UI only
- ❌ Hardcode API URLs
- ❌ Use two-way binding on business data
- ❌ Inject services into Dumb components
- ❌ Trust client-side validation
- ❌ Ignore error responses
- ❌ Store sensitive data in UI
- ❌ Forget to unsubscribe

---

## Related Documents

- ARCHITECTURE_PRINCIPLES.md — Overall system architecture
- API_STANDARDS.md — Backend API contracts
- SECURITY_POLICY.md — Security requirements

---

**Last Updated:** [DATE]  
**Next Review:** [DATE]  
**Maintainer:** [TEAM/PERSON]
