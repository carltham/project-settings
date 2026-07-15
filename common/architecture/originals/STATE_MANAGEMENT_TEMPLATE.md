# State Management Architecture

**Project:** [PROJECT_NAME]  
**Framework:** [Angular/React/Vue/Svelte]  
**State Solution:** [Redux/MobX/Vuex/Context/Signals/Service-Based]  
**Last Updated:** [DATE]  
**Next Review:** [DATE]

---

## Overview

This document defines state management patterns for [PROJECT_NAME]. Choosing the right state management strategy is critical for scalability and maintainability.

**Core Philosophy:** Centralize shared state, keep local state local.

---

## Table of Contents

1. [When to Use State Management](#when-to-use-state-management)
2. [State Architecture Patterns](#state-architecture-patterns)
3. [Service-Based State](#service-based-state)
4. [Redux/NgRx Pattern](#reduxngrx-pattern)
5. [MobX Pattern](#mobx-pattern)
6. [Context API Pattern](#context-api-pattern)
7. [Vue Composition API](#vue-composition-api)
8. [State Updates & Immutability](#state-updates--immutability)
9. [Performance Optimization](#performance-optimization)
10. [Testing Strategy](#testing-strategy)

---

## When to Use State Management

### Decision Tree

```
Does your app have shared state?

├─ NO → Local state only (component state)
│       ✅ Use @State() in components
│       ✅ Pass via @Input/@Output
│       ✅ Simple form states
│
└─ YES → Is state complex?
         ├─ SIMPLE (1-2 data sources)
         │  ✅ Service-based state
         │  ├─ UserService.currentUser$
         │  ├─ ThemeService.theme$
         │  └─ AuthService.isLoggedIn$
         │
         └─ COMPLEX (many interactions)
            ├─ Multiple data sources?
            ├─ Undo/redo needed?
            ├─ Time-travel debugging needed?
            └─ if YES to any → Use Redux/NgRx/Vuex
               if NO → Use Reactive (RxJS/Signals)
```

### State Classification

| State Type | Example | Where to Store |
|---|---|---|
| **Local** | Form input, expanded/collapsed UI | Component |
| **Shared** | Current user, theme, auth token | Service Observable |
| **Global** | App state, user settings, permissions | State management lib |
| **Transient** | Loading indicators, temporary modals | Local component |

**Example:**

```typescript
// LOCAL STATE - Keep in component
@Component({
  selector: 'app-search-form'
})
export class SearchFormComponent {
  searchText = '';  // Just local form state
  
  onSearch() {
    this.appService.search(this.searchText);  // Send to app
  }
}

// SHARED STATE - Service Observable
@Injectable({ providedIn: 'root' })
export class AppService {
  searchResults$ = new BehaviorSubject<Result[]>([]);
  
  search(query: string) {
    this.api.search(query).subscribe(results => {
      this.searchResults$.next(results);  // Update shared state
    });
  }
}

// GLOBAL STATE - Redux/Store
// If: Multiple components need search history, undo, filtering
// Then: Use Redux to manage all search-related state
```

---

## State Architecture Patterns

### Architecture Levels

```
┌─────────────────────────────────────────┐
│         Application State (Global)      │
│  ┌─────────────────────────────────────┐│
│  │    Feature State (Shared)           ││
│  │  ┌─────────────────────────────────┐││
│  │  │  Component State (Local)        │││
│  │  │  - Form inputs                  │││
│  │  │  - UI toggles                   │││
│  │  │  - Temporary changes            │││
│  │  └─────────────────────────────────┘││
│  │                                     ││
│  │  - User data                       ││
│  │  - Feature flags                   ││
│  │  - Cached API responses            ││
│  └─────────────────────────────────────┘│
│                                         │
│  - Current user                        │
│  - Authentication state                │
│  - Routing state                       │
│  - Global settings                     │
└─────────────────────────────────────────┘
```

### State Flow Pattern

```
User Interaction
    ↓
Action Creator
    ↓
Action (plain object/event)
    ↓
Middleware (logging, effects)
    ↓
Reducer/Handler
    ↓
State Update
    ↓
Selectors
    ↓
Components (via Observable/Store)
    ↓
Template Render
```

---

## Service-Based State

### When to Use

**Best for:**
- Simple apps (CRUD operations)
- Few data sources
- Limited state interactions
- Small to medium teams

**Example: Simple app**

```
┌──────────────────────────┐
│  UserService             │
│  ├─ users$ Observable    │
│  ├─ currentUser$         │
│  ├─ loading$             │
│  ├─ error$               │
│  └─ Methods:             │
│     ├─ getUsers()        │
│     ├─ createUser()      │
│     ├─ updateUser()      │
│     └─ deleteUser()      │
└──────────────────────────┘
         ↓
   ┌─────────────┐
   │ Components  │
   │ Subscribe   │
   │ & Display   │
   └─────────────┘
```

### Implementation

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { BehaviorSubject, Observable } from 'rxjs';
import { tap, shareReplay, catchError } from 'rxjs/operators';
import { of } from 'rxjs';

interface User {
  id: number;
  name: string;
  email: string;
}

interface UserState {
  users: User[];
  currentUser: User | null;
  loading: boolean;
  error: string | null;
}

@Injectable({ providedIn: 'root' })
export class UserService {
  // Private state subjects
  private usersSubject = new BehaviorSubject<User[]>([]);
  private currentUserSubject = new BehaviorSubject<User | null>(null);
  private loadingSubject = new BehaviorSubject<boolean>(false);
  private errorSubject = new BehaviorSubject<string | null>(null);
  
  // Public observables
  users$ = this.usersSubject.asObservable();
  currentUser$ = this.currentUserSubject.asObservable();
  loading$ = this.loadingSubject.asObservable();
  error$ = this.errorSubject.asObservable();
  
  constructor(private http: HttpClient) {}
  
  // QUERY - GET all users
  getUsers(): Observable<User[]> {
    this.loadingSubject.next(true);
    this.errorSubject.next(null);
    
    return this.http.get<User[]>('/api/users').pipe(
      tap(users => {
        this.usersSubject.next(users);
        this.loadingSubject.next(false);
      }),
      catchError(error => {
        this.errorSubject.next(error.message);
        this.loadingSubject.next(false);
        return of([]);
      }),
      shareReplay(1)
    );
  }
  
  // QUERY - GET current user
  getCurrentUser(id: number): Observable<User | null> {
    return this.http.get<User>(`/api/users/${id}`).pipe(
      tap(user => this.currentUserSubject.next(user)),
      catchError(error => {
        this.errorSubject.next(error.message);
        return of(null);
      })
    );
  }
  
  // COMMAND - CREATE user
  createUser(user: Omit<User, 'id'>): Observable<User> {
    return this.http.post<User>('/api/users', user).pipe(
      tap(newUser => {
        const current = this.usersSubject.value;
        this.usersSubject.next([...current, newUser]);
      }),
      catchError(error => {
        this.errorSubject.next(error.message);
        throw error;
      })
    );
  }
  
  // COMMAND - UPDATE user
  updateUser(user: User): Observable<User> {
    return this.http.put<User>(`/api/users/${user.id}`, user).pipe(
      tap(updated => {
        const current = this.usersSubject.value;
        const index = current.findIndex(u => u.id === user.id);
        if (index !== -1) {
          current[index] = updated;
          this.usersSubject.next([...current]);
        }
        
        if (this.currentUserSubject.value?.id === user.id) {
          this.currentUserSubject.next(updated);
        }
      }),
      catchError(error => {
        this.errorSubject.next(error.message);
        throw error;
      })
    );
  }
  
  // COMMAND - DELETE user
  deleteUser(id: number): Observable<void> {
    return this.http.delete<void>(`/api/users/${id}`).pipe(
      tap(() => {
        const current = this.usersSubject.value;
        this.usersSubject.next(current.filter(u => u.id !== id));
      }),
      catchError(error => {
        this.errorSubject.next(error.message);
        throw error;
      })
    );
  }
  
  // SIDE EFFECT - Clear state
  clearState(): void {
    this.usersSubject.next([]);
    this.currentUserSubject.next(null);
    this.loadingSubject.next(false);
    this.errorSubject.next(null);
  }
}
```

### Usage in Components

```typescript
@Component({
  selector: 'app-users-list',
  template: `
    <!-- Loading -->
    <div *ngIf="loading$ | async">Loading...</div>
    
    <!-- Error -->
    <div *ngIf="error$ | async as error" class="error">
      {{ error }}
    </div>
    
    <!-- List -->
    <div *ngFor="let user of users$ | async">
      {{ user.name }}
      <button (click)="selectUser(user)">Select</button>
    </div>
  `
})
export class UsersListComponent {
  users$ = this.userService.users$;
  loading$ = this.userService.loading$;
  error$ = this.userService.error$;
  
  constructor(private userService: UserService) {}
  
  ngOnInit() {
    this.userService.getUsers().subscribe();
  }
  
  selectUser(user: User) {
    this.userService.updateCurrentUser(user);
  }
}
```

### Pros & Cons

**Pros:**
- ✅ Simple to implement
- ✅ Easy to understand
- ✅ Low boilerplate
- ✅ Good for small apps

**Cons:**
- ❌ Not suitable for complex state
- ❌ Hard to track state changes
- ❌ No time-travel debugging
- ❌ Can become messy with many services

---

## Redux/NgRx Pattern

### When to Use

**Best for:**
- Complex apps (many interactions)
- Multiple interconnected data sources
- Time-travel debugging required
- Large teams

### Core Concepts

**1. Actions** - What happened

```typescript
export const loadUsers = createAction('[Users] Load Users');

export const loadUsersSuccess = createAction(
  '[Users] Load Users Success',
  props<{ users: User[] }>()
);

export const loadUsersError = createAction(
  '[Users] Load Users Error',
  props<{ error: string }>()
);
```

**2. Reducer** - How state changes

```typescript
interface UsersState {
  users: User[];
  loading: boolean;
  error: string | null;
}

const initialState: UsersState = {
  users: [],
  loading: false,
  error: null
};

export const usersReducer = createReducer(
  initialState,
  on(loadUsers, state => ({
    ...state,
    loading: true,
    error: null
  })),
  on(loadUsersSuccess, (state, { users }) => ({
    ...state,
    users,
    loading: false
  })),
  on(loadUsersError, (state, { error }) => ({
    ...state,
    error,
    loading: false
  }))
);
```

**3. Effects** - Side effects (API calls)

```typescript
@Injectable()
export class UsersEffects {
  loadUsers$ = createEffect(() =>
    this.actions$.pipe(
      ofType(loadUsers),
      switchMap(() =>
        this.userService.getUsers().pipe(
          map(users => loadUsersSuccess({ users })),
          catchError(error => of(loadUsersError({ error })))
        )
      )
    )
  );
  
  constructor(
    private actions$: Actions,
    private userService: UserService
  ) {}
}
```

**4. Selectors** - Query state

```typescript
export const selectUsers = (state: AppState) => state.users.users;
export const selectLoading = (state: AppState) => state.users.loading;
export const selectError = (state: AppState) => state.users.error;

// Memoized selector (only recomputes if input changes)
export const selectUserById = (id: number) =>
  createSelector(
    selectUsers,
    (users: User[]) => users.find(u => u.id === id)
  );
```

**5. Store** - Central state container

```typescript
@Component({
  template: `
    <div *ngIf="loading$ | async">Loading...</div>
    <div *ngFor="let user of users$ | async">
      {{ user.name }}
    </div>
  `
})
export class UsersListComponent {
  users$ = this.store.select(selectUsers);
  loading$ = this.store.select(selectLoading);
  
  constructor(private store: Store<AppState>) {}
  
  ngOnInit() {
    this.store.dispatch(loadUsers());
  }
}
```

### Redux Flow

```
Component
    ↓
Dispatch Action
    │
    ├→ Middleware (logging, effects)
    │       ↓
    │   API Call
    │       ↓
    │   Dispatch Success/Error
    │
    ↓
Reducer
    ↓
State Updated
    ↓
Selectors
    ↓
Component (via async pipe)
    ↓
Template Rendered
```

### Pros & Cons

**Pros:**
- ✅ Predictable state management
- ✅ Time-travel debugging
- ✅ Clear data flow
- ✅ Easy to test
- ✅ Works at scale

**Cons:**
- ❌ High boilerplate (actions, reducers, selectors)
- ❌ Steep learning curve
- ❌ Overkill for simple apps
- ❌ More code to maintain

---

## MobX Pattern

### When to Use

**Alternative to Redux with:**
- Less boilerplate
- Reactive declarations
- Simpler mental model

### Implementation

```typescript
import { makeObservable, observable, action } from 'mobx';

class UserStore {
  users: User[] = [];
  loading = false;
  error: string | null = null;
  
  constructor(private api: ApiService) {
    makeObservable(this, {
      users: observable,
      loading: observable,
      error: observable,
      loadUsers: action,
      addUser: action
    });
  }
  
  loadUsers = async () => {
    this.loading = true;
    try {
      this.users = await this.api.getUsers();
    } catch (error) {
      this.error = error.message;
    } finally {
      this.loading = false;
    }
  }
  
  addUser = (user: User) => {
    this.users.push(user);
  }
}

// React component
const UsersList = observer(({ store }: { store: UserStore }) => (
  <div>
    {store.loading && <div>Loading...</div>}
    {store.users.map(user => (
      <div key={user.id}>{user.name}</div>
    ))}
  </div>
));
```

### Pros & Cons

**Pros:**
- ✅ Less boilerplate than Redux
- ✅ Simple to learn
- ✅ Feels like normal JavaScript
- ✅ Reactive by default

**Cons:**
- ❌ Less predictable (mutation-based)
- ❌ Harder to debug (implicit dependencies)
- ❌ Less tooling support
- ❌ Smaller community

---

## Context API Pattern

### When to Use

**React apps with:**
- Simple shared state
- Avoid Redux complexity
- Small to medium apps

### Implementation

```typescript
// Create context
const UserContext = createContext<{
  users: User[];
  currentUser: User | null;
  loading: boolean;
  getUsers: () => void;
} | undefined>(undefined);

// Provider component
export function UserProvider({ children }) {
  const [users, setUsers] = useState<User[]>([]);
  const [currentUser, setCurrentUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(false);
  
  const getUsers = async () => {
    setLoading(true);
    try {
      const data = await fetch('/api/users').then(r => r.json());
      setUsers(data);
    } finally {
      setLoading(false);
    }
  };
  
  const value = { users, currentUser, loading, getUsers };
  
  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}

// Custom hook
export function useUsers() {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUsers must be used within UserProvider');
  }
  return context;
}

// Usage in component
function UsersList() {
  const { users, loading, getUsers } = useUsers();
  
  useEffect(() => {
    getUsers();
  }, []);
  
  return (
    <div>
      {loading && <div>Loading...</div>}
      {users.map(user => <div key={user.id}>{user.name}</div>)}
    </div>
  );
}
```

### Pros & Cons

**Pros:**
- ✅ Built into React
- ✅ Simple for basic state
- ✅ No external dependencies
- ✅ Good for theme/auth

**Cons:**
- ❌ Performance issues with large state trees
- ❌ No built-in dev tools
- ❌ Boilerplate for reducers
- ❌ Not ideal for complex logic

---

## Vue Composition API

### When to Use

**Vue 3+ apps with:**
- Composition API
- Modern, reactive state management

### Implementation

```typescript
import { ref, computed, reactive } from 'vue';

export function useUsers() {
  const users = ref<User[]>([]);
  const loading = ref(false);
  const error = ref<string | null>(null);
  
  // Computed property
  const userCount = computed(() => users.value.length);
  
  // Methods
  const getUsers = async () => {
    loading.value = true;
    try {
      const response = await fetch('/api/users');
      users.value = await response.json();
    } catch (err) {
      error.value = err.message;
    } finally {
      loading.value = false;
    }
  };
  
  // Expose publicly
  return {
    users,
    loading,
    error,
    userCount,
    getUsers
  };
}

// In component
<template>
  <div>
    <div v-if="loading">Loading...</div>
    <div v-for="user in users" :key="user.id">
      {{ user.name }}
    </div>
    <p>Total: {{ userCount }}</p>
  </div>
</template>

<script setup>
import { useUsers } from './useUsers';

const { users, loading, userCount, getUsers } = useUsers();

onMounted(() => {
  getUsers();
});
</script>
```

### Pros & Cons

**Pros:**
- ✅ Built into Vue 3
- ✅ Composable and reusable
- ✅ Reactive by default
- ✅ Simple and intuitive

**Cons:**
- ❌ Requires Vue 3+
- ❌ Less structure than Pinia
- ❌ Manual dependency injection
- ❌ Can get messy with many composables

---

## State Updates & Immutability

### Why Immutability Matters

```typescript
// ❌ WRONG - Mutation
const state = { users: [{ id: 1, name: 'John' }] };
state.users[0].name = 'Jane';  // Mutates original
// React/Vue might not detect change!

// ✅ RIGHT - Immutable update
const state = { users: [{ id: 1, name: 'John' }] };
const newState = {
  users: state.users.map(u =>
    u.id === 1 ? { ...u, name: 'Jane' } : u
  )
};
```

### Update Patterns

**Object Update:**

```typescript
// ❌ WRONG
state.user.name = 'Jane';

// ✅ RIGHT
return {
  ...state,
  user: {
    ...state.user,
    name: 'Jane'
  }
};
```

**Array Update:**

```typescript
// ❌ WRONG
state.users[0].name = 'Jane';
state.users.push(newUser);

// ✅ RIGHT - Using map
return {
  ...state,
  users: state.users.map(u =>
    u.id === 1 ? { ...u, name: 'Jane' } : u
  )
};

// ✅ RIGHT - Using spread
return {
  ...state,
  users: [...state.users, newUser]
};

// ✅ RIGHT - Using filter
return {
  ...state,
  users: state.users.filter(u => u.id !== id)
};
```

**Nested Update:**

```typescript
// ❌ WRONG - Mutates nested
state.user.profile.avatar = 'url';

// ✅ RIGHT - Immutable all the way
return {
  ...state,
  user: {
    ...state.user,
    profile: {
      ...state.user.profile,
      avatar: 'url'
    }
  }
};

// ✅ RIGHT - Using library
import produce from 'immer';

return produce(state, draft => {
  draft.user.profile.avatar = 'url';  // Looks mutable, actually immutable
});
```

---

## Performance Optimization

### Memoization

```typescript
// ❌ WRONG - Recalculates every time
export const selectActiveUsers = (state: AppState) =>
  state.users.filter(u => u.active);

// ✅ RIGHT - Memoized (only recalculates if users change)
export const selectActiveUsers = createSelector(
  selectUsers,
  (users: User[]) => users.filter(u => u.active)
);
```

### Change Detection

```typescript
// ❌ WRONG - Updates entire state (triggers re-render)
this.store.dispatch(setLoading(true));
this.store.dispatch(setUsers(users));
this.store.dispatch(setError(null));

// ✅ RIGHT - Single update
this.store.dispatch(loadUsersSuccess({ users }));
```

### Lazy State Modules

```typescript
// Load state only when needed
const routes = [
  {
    path: 'users',
    loadChildren: () => import('./users/users.module').then(m => {
      m.UsersModule.registerStoreModule();  // Register state
      return m.UsersModule;
    })
  }
];
```

---

## Testing Strategy

### Testing Service-Based State

```typescript
describe('UserService', () => {
  let service: UserService;
  let httpMock: HttpTestingController;
  
  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [UserService],
      imports: [HttpClientTestingModule]
    });
    service = TestBed.inject(UserService);
    httpMock = TestBed.inject(HttpTestingController);
  });
  
  it('should load users', (done) => {
    const mockUsers = [{ id: 1, name: 'John' }];
    
    service.getUsers().subscribe(users => {
      expect(users).toEqual(mockUsers);
      done();
    });
    
    const req = httpMock.expectOne('/api/users');
    req.flush(mockUsers);
  });
  
  it('should update users subject', (done) => {
    const mockUsers = [{ id: 1, name: 'John' }];
    
    service.users$.subscribe(users => {
      expect(users).toEqual(mockUsers);
      done();
    });
    
    service.getUsers().subscribe();
    
    const req = httpMock.expectOne('/api/users');
    req.flush(mockUsers);
  });
});
```

### Testing Redux State

```typescript
describe('UsersReducer', () => {
  it('should handle loadUsersSuccess', () => {
    const mockUsers = [{ id: 1, name: 'John' }];
    const action = loadUsersSuccess({ users: mockUsers });
    
    const state = usersReducer(initialState, action);
    
    expect(state.users).toEqual(mockUsers);
    expect(state.loading).toBe(false);
  });
});

describe('Users Effects', () => {
  let effects: UsersEffects;
  let actions$: Observable<any>;
  let userService: jasmine.SpyObj<UserService>;
  
  beforeEach(() => {
    const spy = jasmine.createSpyObj('UserService', ['getUsers']);
    
    TestBed.configureTestingModule({
      providers: [
        UsersEffects,
        provideMockActions(() => actions$),
        { provide: UserService, useValue: spy }
      ]
    });
    
    effects = TestBed.inject(UsersEffects);
    userService = TestBed.inject(UserService) as jasmine.SpyObj<UserService>;
  });
  
  it('should load users', () => {
    const mockUsers = [{ id: 1, name: 'John' }];
    userService.getUsers.and.returnValue(of(mockUsers));
    
    actions$ = hot('-a', { a: loadUsers() });
    
    const completion = cold('--(b|)', {
      b: loadUsersSuccess({ users: mockUsers })
    });
    
    expect(effects.loadUsers$).toBeObservable(completion);
  });
});
```

---

## State Management Checklist

**Before implementing state management:**

- [ ] Do you actually need it? (not just "feels right")
- [ ] Is state shared between multiple components?
- [ ] Is state complex with many interactions?
- [ ] Do you need time-travel debugging?
- [ ] Team comfortable with chosen solution?
- [ ] Type safety (TypeScript) configured?
- [ ] Dev tools/extensions installed?
- [ ] Performance profiled?
- [ ] Immutability enforced?
- [ ] Tests written for state logic?

---

## Quick Reference

**Choose based on complexity:**

| Complexity | Solution |
|---|---|
| None | Local component state only |
| Low (1-3 shared data sources) | Service-based Observable |
| Medium (multiple interactions) | Redux/NgRx or MobX |
| High (complex logic, many sources) | Redux/NgRx + Effects |

**Always:**
- ✅ Keep state normalized
- ✅ Use immutable updates
- ✅ Memoize selectors
- ✅ Type your state (TypeScript)
- ✅ Test state logic
- ✅ Use dev tools
- ✅ Document state shape

**Never:**
- ❌ Store sensitive data (passwords, tokens)
- ❌ Store local UI state in global store
- ❌ Mutate state directly
- ❌ Put business logic in state (use services)
- ❌ Skip testing state
- ❌ Store API responses without normalization

---

## Related Documents

- ARCHITECTURE_PRINCIPLES.md — Overall system architecture
- FRONTEND_ARCHITECTURE.md — Frontend patterns
- API_STANDARDS.md — Backend API contracts

---

**Last Updated:** [DATE]  
**Next Review:** [DATE]  
**Maintainer:** [TEAM/PERSON]
