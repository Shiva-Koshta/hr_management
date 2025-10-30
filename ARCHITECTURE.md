# TalentFlow Architecture Documentation

## 📐 System Architecture

### Overview

TalentFlow follows a modern React architecture with offline-first capabilities, utilizing IndexedDB for persistent storage and MirageJS for API simulation.

```
┌─────────────────────────────────────────────────────────────────┐
│                    Browser Environment                           │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                   React Application                       │  │
│  │                                                            │  │
│  │  ┌────────────┐  ┌────────────┐  ┌─────────────────┐   │  │
│  │  │   Jobs     │  │ Candidates │  │  Assessments    │   │  │
│  │  │  Features  │  │  Features  │  │   Features      │   │  │
│  │  └─────┬──────┘  └─────┬──────┘  └────────┬────────┘   │  │
│  │        │                │                  │             │  │
│  │        └────────────────┴──────────────────┘             │  │
│  │                         │                                 │  │
│  │              ┌──────────▼──────────┐                     │  │
│  │              │   Redux Store       │                     │  │
│  │              │   (Global State)    │                     │  │
│  │              └──────────┬──────────┘                     │  │
│  │                         │                                 │  │
│  └─────────────────────────┼─────────────────────────────────┘  │
│                            │                                    │
│                 ┌──────────▼──────────┐                        │
│                 │   MirageJS Server   │                        │
│                 │   (Mock API Layer)  │                        │
│                 │  - Routes           │                        │
│                 │  - Latency (200-1200ms)                     │
│                 │  - Error Rate (5-10%)                       │
│                 └──────────┬──────────┘                        │
│                            │                                    │
│                 ┌──────────▼──────────┐                        │
│                 │    Dexie.js         │                        │
│                 │  (IndexedDB Wrapper)│                        │
│                 └──────────┬──────────┘                        │
│                            │                                    │
│                 ┌──────────▼──────────┐                        │
│                 │     IndexedDB       │                        │
│                 │  (Browser Storage)  │                        │
│                 │  - jobs             │                        │
│                 │  - candidates       │                        │
│                 │  - candidateTimeline│                        │
│                 │  - assessments      │                        │
│                 │  - assessmentResponses                      │
│                 │  - metadata         │                        │
│                 └─────────────────────┘                        │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

## 🗂 Data Model

### Database Schema (IndexedDB)

#### jobs
```typescript
{
  id: number (auto-increment),
  title: string,
  slug: string (unique),
  description: string,
  status: 'active' | 'archived',
  tags: string[],
  order: number,
  expirationDate: ISO8601 string | null,
  createdAt: ISO8601 string
}
```

#### candidates
```typescript
{
  id: number (auto-increment),
  name: string,
  email: string,
  jobId: number (foreign key → jobs.id),
  stage: 'applied' | 'screen' | 'tech' | 'offer' | 'hired' | 'rejected',
  followUp: boolean,
  createdAt: ISO8601 string
}
```

#### candidateTimeline
```typescript
{
  id: number (auto-increment),
  candidateId: number (foreign key → candidates.id),
  fromStage: string | null,
  toStage: string,
  timestamp: ISO8601 string,
  notes: string
}
```

#### assessments
```typescript
{
  id: number (auto-increment),
  jobId: number (foreign key → jobs.id),
  sections: Section[],
  updatedAt: ISO8601 string
}

interface Section {
  id: string (nanoid),
  title: string,
  description: string,
  questions: Question[]
}

interface Question {
  id: string (nanoid),
  type: 'short-text' | 'long-text' | 'single-choice' | 'multi-choice' | 'numeric' | 'file-upload',
  label: string,
  required: boolean,
  options?: string[],  // for choice questions
  maxLength?: number,  // for text questions
  min?: number,        // for numeric questions
  max?: number,        // for numeric questions
  conditional?: {
    questionId: string,
    value: string
  }
}
```

#### assessmentResponses
```typescript
{
  id: number (auto-increment),
  candidateId: number,
  jobId: number,
  responses: Record<string, any>,  // questionId → answer
  submittedAt: ISO8601 string
}
```

#### metadata
```typescript
{
  key: string (primary key),
  value: any
}
```

## 🔄 State Management

### Redux Store Structure

```typescript
{
  jobs: {
    items: Job[],
    currentJob: Job | null,
    filters: {
      search: string,
      status: string,
      tags: string[]
    },
    pagination: {
      page: number,
      pageSize: number,
      total: number,
      totalPages: number
    },
    sort: string,
    loading: boolean,
    error: string | null,
    reorderSnapshot: Job[] | null
  },
  
  candidates: {
    items: Candidate[],
    currentCandidate: Candidate | null,
    currentTimeline: TimelineEvent[],
    filters: {
      search: string,
      stage: string,
      followUpOnly: boolean
    },
    viewMode: 'list' | 'kanban',
    loading: boolean,
    error: string | null
  },
  
  assessments: {
    currentAssessment: Assessment | null,
    currentJobId: number | null,
    sections: Section[],
    previewMode: boolean,
    responses: Record<string, any>,
    candidateResponse: AssessmentResponse | null,
    loading: boolean,
    saving: boolean,
    error: string | null
  },
  
  ui: {
    modals: {
      jobForm: {
        isOpen: boolean,
        jobId: number | null
      }
    },
    toasts: Toast[],
    loading: {
      global: boolean
    }
  }
}
```

### State Update Flow

1. **User Action** → Component dispatches action
   ```javascript
   dispatch(fetchJobs({ search: '', status: 'active' }))
   ```

2. **Async Thunk** → Makes API request
   ```javascript
   // In jobsSlice.js
   export const fetchJobs = createAsyncThunk(
     'jobs/fetchJobs',
     async (params) => {
       const response = await fetch(`/api/jobs?${params}`);
       return await response.json();
     }
   );
   ```

3. **Reducer** → Updates state
   ```javascript
   .addCase(fetchJobs.fulfilled, (state, action) => {
     state.items = action.payload.data;
     state.pagination = action.payload.pagination;
   })
   ```

4. **Component Re-renders** → UI updates
   ```javascript
   const jobs = useSelector(state => state.jobs.items);
   ```

## 🎯 Key Design Patterns

### 1. Feature-Based Architecture

Each feature (jobs, candidates, assessments) is self-contained:
- Redux slice (state + actions)
- Components (UI)
- Types/interfaces (if using TypeScript)

**Benefits:**
- Easy to locate related code
- Simple to add/remove features
- Better code organization at scale

### 2. Optimistic Updates with Rollback

Used in job reordering to provide instant feedback:

```javascript
// Optimistic update
dispatch(optimisticReorder({ fromIndex, toIndex }));

// Try API call
try {
  await dispatch(reorderJobs({ fromOrder, toOrder })).unwrap();
  // Success - keep the optimistic update
} catch (error) {
  // Failure - rollback to previous state
  dispatch(rollbackReorder());
  showToast({ type: 'error', message: 'Reorder failed' });
}
```

### 3. Virtualization for Performance

For lists with 1000+ items, only render visible items:

```javascript
const rowVirtualizer = useVirtualizer({
  count: filteredCandidates.length,
  getScrollElement: () => parentRef.current,
  estimateSize: () => 80,  // Estimated row height
  overscan: 10,  // Render 10 extra items for smoother scrolling
});
```

### 4. Conditional Rendering in Forms

Assessment questions can depend on other questions:

```javascript
const shouldShowQuestion = (question) => {
  if (!question.conditional) return true;
  
  const conditionResponse = responses[question.conditional.questionId];
  return conditionResponse === question.conditional.value;
};
```

### 5. Debounced Search

Reduce API calls during typing:

```javascript
useEffect(() => {
  const timer = setTimeout(() => {
    dispatch(setFilters({ search: localSearch }));
  }, 300);  // Wait 300ms after user stops typing

  return () => clearTimeout(timer);
}, [localSearch]);
```

## 🔌 API Layer (MirageJS)

### Request/Response Flow

```
Component → Redux Thunk → fetch() → MirageJS Handler → Dexie → IndexedDB
                                          ↓
                                    Add Latency
                                          ↓
                                   Simulate Errors
                                          ↓
IndexedDB → Dexie → Response ← MirageJS ← fetch() ← Redux State Update
```

### Example Handler

```javascript
// GET /api/jobs
this.get('/jobs', async (schema, request) => {
  await delay();  // 200-1200ms random delay
  
  const { search, status, page, pageSize } = request.queryParams;
  
  let jobs = await dbHelpers.getAllJobs();
  
  // Apply filters
  if (search) {
    jobs = jobs.filter(job => 
      job.title.toLowerCase().includes(search.toLowerCase())
    );
  }
  
  // Pagination
  const start = (page - 1) * pageSize;
  const paginatedJobs = jobs.slice(start, start + pageSize);
  
  return {
    data: paginatedJobs,
    pagination: { page, pageSize, total: jobs.length }
  };
});
```

### Error Simulation

```javascript
const shouldFail = () => Math.random() < 0.075;  // 7.5% failure rate

if (shouldFail()) {
  return new Response(500, {}, { error: 'Internal server error' });
}
```

## 🎨 UI Component Hierarchy

```
App
├── Layout
│   ├── Navigation
│   └── Outlet (from React Router)
│       ├── JobsBoard
│       │   ├── JobCard (sortable, draggable)
│       │   └── JobFormModal
│       │
│       ├── JobDetail
│       │
│       ├── CandidatesList (virtualized)
│       │   └── CandidateCard
│       │
│       ├── CandidatesKanban
│       │   └── KanbanColumn (droppable)
│       │       └── CandidateCard (draggable)
│       │
│       ├── CandidateProfile
│       │   └── Timeline
│       │
│       ├── AssessmentBuilder
│       │   ├── QuestionEditor
│       │   └── AssessmentPreviewPane
│       │       └── QuestionRenderer
│       │
│       └── AssessmentPreview
│           └── QuestionRenderer
│
└── ToastContainer
    └── ToastItem
```

## 🔐 Security Considerations

### Current Implementation

- **No Authentication**: This is a demo app. All users see the same data.
- **Client-Side Only**: All logic runs in the browser.
- **No Sensitive Data**: Uses mock data only.

### Production Recommendations

1. **Authentication**
   - Implement JWT-based auth
   - Use OAuth providers (Google, Microsoft)
   - Store tokens securely

2. **Authorization**
   - Role-based access control (RBAC)
   - Restrict API endpoints by role
   - Validate permissions on every request

3. **Data Protection**
   - Encrypt sensitive data at rest
   - Use HTTPS for all communication
   - Implement CORS properly

4. **Input Validation**
   - Sanitize all user inputs
   - Validate on both client and server
   - Prevent SQL/NoSQL injection

## 📊 Performance Optimizations

### Current Optimizations

1. **Code Splitting**
   - React Router automatically splits by route
   - Large libraries (date-fns, @dnd-kit) only loaded when needed

2. **Virtualization**
   - Only renders visible rows in candidate list
   - Dramatically reduces DOM nodes (1000+ items → ~20 DOM nodes)

3. **Memoization**
   - `useMemo` for expensive computations (filtering candidates)
   - `useCallback` for stable function references

4. **Debouncing**
   - Search inputs wait 300ms before triggering updates
   - Reduces unnecessary re-renders and API calls

### Future Optimizations

1. **React.memo**
   - Wrap components to prevent unnecessary re-renders
   - Particularly useful for list items

2. **IndexedDB Queries**
   - Add indexes for frequently queried fields
   - Use compound indexes for complex queries

3. **Bundle Size**
   - Implement tree-shaking
   - Use dynamic imports for large components
   - Replace large libraries with smaller alternatives

## 🧪 Testing Strategy

### Current State
- No automated tests (time constraint for assignment)

### Recommended Testing Pyramid

```
     E2E Tests (10%)
    ──────────────
   Integration (30%)
  ────────────────────
 Unit Tests (60%)
───────────────────────
```

#### Unit Tests (Vitest)
- Redux reducers
- Utility functions
- Custom hooks
- Validation logic

#### Integration Tests (React Testing Library)
- Component interactions
- Form submissions
- Redux state updates
- API mocking

#### E2E Tests (Playwright)
- Critical user flows
- Cross-browser testing
- Mobile responsiveness

## 🚀 Scalability Considerations

### Current Limits

- **Candidates**: Virtualization handles 1000+ efficiently, could support 10,000+
- **Jobs**: Pagination supports unlimited with good UX up to ~1000
- **Assessments**: No practical limit on questions

### Scaling Beyond

1. **Backend Migration**
   - Replace MirageJS with real API
   - Move heavy computations server-side
   - Implement proper pagination/filtering

2. **Data Loading**
   - Implement infinite scroll for candidates
   - Lazy load job details
   - Cache frequently accessed data

3. **State Management**
   - Consider normalizing Redux state (relational data)
   - Implement entity adapters
   - Use selectors with reselect for memoization

## 📱 Responsive Design

### Breakpoints (Tailwind)

```css
sm: 640px   /* Small devices */
md: 768px   /* Tablets */
lg: 1024px  /* Laptops */
xl: 1280px  /* Desktops */
2xl: 1536px /* Large screens */
```

### Mobile Considerations

- Touch-friendly hit targets (min 44x44px)
- Simplified layouts on small screens
- Bottom navigation for mobile (not yet implemented)
- Swipe gestures for kanban (future enhancement)

---

This architecture document provides a comprehensive overview of TalentFlow's design and implementation. For specific implementation details, refer to the source code and inline comments.

