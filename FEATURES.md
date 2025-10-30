# TalentFlow - Feature Documentation

This document provides a detailed walkthrough of all features implemented in TalentFlow.

## 📋 Table of Contents

1. [Jobs Management](#jobs-management)
2. [Candidates Management](#candidates-management)
3. [Assessments](#assessments)
4. [Additional Features](#additional-features)

---

## 1. Jobs Management

### 1.1 Jobs Board

**Route:** `/jobs`

#### Features:
- **Grid View**: Jobs displayed in a responsive grid (1-3 columns based on screen size)
- **Pagination**: Server-like pagination with configurable page size
- **Search**: Real-time search by job title with debouncing
- **Filters**: 
  - Status (Active/Archived)
  - Tags (multiple selection)
- **Sorting**: By order, title, or creation date

#### Implementation Details:
```javascript
// Location: src/features/jobs/JobsBoard.jsx
- Uses Redux for state management
- Debounced search (300ms delay)
- Supports 25+ jobs with pagination
```

### 1.2 Create/Edit Job

**Component:** Job Form Modal

#### Fields:
- **Title** (required): Job position title
- **Slug** (required, unique): URL-friendly identifier (auto-generated from title)
- **Description**: Detailed job description
- **Status**: Active or Archived
- **Tags**: Multiple tag selection from predefined list
- **Expiration Date**: Optional date when job should auto-archive

#### Validation:
- Title is required
- Slug must be unique across all jobs
- Slug must be URL-friendly (lowercase, hyphens)
- Expiration date must be in the future

#### Implementation Details:
```javascript
// Location: src/features/jobs/JobFormModal.jsx
- Opens from Jobs Board via "Create Job" button or "Edit" on job card
- Real-time slug generation from title
- Duplicate slug detection
- Auto-archive on expiration date
```

### 1.3 Drag & Drop Reordering

**Feature:** Reorder jobs via drag and drop

#### Behavior:
1. Drag any job card to reorder
2. UI updates immediately (optimistic update)
3. API call made in background
4. On failure (10% simulation), rollback to original order
5. Toast notification shows success/failure

#### Implementation Details:
```javascript
// Location: src/features/jobs/JobsBoard.jsx
- Uses @dnd-kit/sortable
- Optimistic updates with rollback
- Error simulation for testing
- Smooth animations
```

### 1.4 Job Detail Page

**Route:** `/jobs/:jobId`

#### Displays:
- Full job information
- List of candidates who applied
- Link to assessment builder
- Statistics (total applicants, in progress, hired)

#### Quick Actions:
- Edit job
- View/create assessment
- View all candidates

#### Implementation Details:
```javascript
// Location: src/features/jobs/JobDetail.jsx
- Deep linking support
- Real-time candidate count
- Assessment status check
```

### 1.5 Job Expiration

**Feature:** Automatic archiving of expired jobs

#### Behavior:
- Jobs with expiration dates are automatically archived when date passes
- Warning badge shown for jobs expiring within 7 days
- Visual indicator for expired jobs
- Check performed on each jobs fetch

#### Implementation Details:
```javascript
// Location: src/features/jobs/JobCard.jsx, src/mocks/server.js
- Client-side check with date-fns
- Server-side auto-archiving
- Warning badges for expiring soon
```

---

## 2. Candidates Management

### 2.1 Candidates List (Virtualized)

**Route:** `/candidates`

#### Features:
- **Virtualized Scrolling**: Efficiently renders 1000+ candidates
- **Search**: Real-time search by name or email (client-side)
- **Filters**:
  - Stage (Applied, Screen, Tech, Offer, Hired, Rejected)
  - Follow-up only toggle
- **View Modes**: List or Kanban

#### Performance:
- Only renders ~20 visible items at a time
- Handles 1000+ candidates smoothly
- Smooth scrolling with overscan

#### Implementation Details:
```javascript
// Location: src/features/candidates/CandidatesList.jsx
- Uses @tanstack/react-virtual
- Debounced search (300ms)
- Follow-up flag icons
- Click to view candidate profile
```

### 2.2 Kanban Board

**Route:** `/candidates` (Kanban view mode)

#### Features:
- **6 Columns**: Applied → Screen → Tech → Offer → Hired/Rejected
- **Drag & Drop**: Move candidates between stages
- **Visual Feedback**: Highlight drop zones
- **Auto Timeline**: Creates timeline entry on stage change

#### Behavior:
1. Drag candidate card to new stage column
2. Drop to update stage
3. API call updates candidate
4. Timeline entry created automatically
5. Toast notification on success/failure

#### Implementation Details:
```javascript
// Location: src/features/candidates/CandidatesKanban.jsx
- Uses @dnd-kit/core with droppable columns
- Touch-friendly drag
- Candidate count per column
- DragOverlay for smooth experience
```

### 2.3 Candidate Profile

**Route:** `/candidates/:id`

#### Displays:
- **Basic Info**: Name, email, current stage
- **Job Applied**: Link to job posting
- **Assessment Status**: Completed or Not Started (with date)
- **Follow-up Flag**: Toggle on/off
- **Timeline**: Complete history of stage changes

#### Timeline Features:
- Shows all stage transitions
- Displays notes for each transition
- Timestamps for each event
- Visual flow with connecting lines

#### Implementation Details:
```javascript
// Location: src/features/candidates/CandidateProfile.jsx
- Fetches candidate and timeline data
- Stage change dropdown
- Follow-up toggle
- Assessment completion check from IndexedDB
```

### 2.4 Follow-up Flag

**Feature:** Mark candidates for follow-up

#### Behavior:
- Toggle flag on candidate profile
- Icon shown on candidate list
- Filter to show only follow-up candidates
- Persists to IndexedDB

#### Use Case:
HR can mark candidates who need attention (e.g., waiting for interview scheduling, need to send offer, follow up on references)

#### Implementation Details:
```javascript
// Location: src/features/candidates/CandidateProfile.jsx
- Simple boolean flag
- Visual indicator (flag icon)
- Filter option in candidates list
```

---

## 3. Assessments

### 3.1 Assessment Builder

**Route:** `/assessments/:jobId`

#### Features:
- **Two-Pane Layout**: Builder (left) + Live Preview (right)
- **Sections**: Organize questions into sections
- **Multiple Question Types**:
  - Short Text (with max length)
  - Long Text (textarea with max length)
  - Single Choice (radio buttons)
  - Multiple Choice (checkboxes)
  - Numeric (with min/max validation)
  - File Upload (UI stub)
- **Conditional Logic**: Show questions based on other answers
- **Real-time Preview**: See changes immediately
- **Persistent Storage**: Auto-saves to IndexedDB

#### Section Management:
- Add/remove sections
- Edit section title and description
- Reorder sections (future enhancement)

#### Question Management:
- Add/remove questions
- Edit question properties
- Set validation rules
- Configure conditional display

#### Implementation Details:
```javascript
// Location: src/features/assessments/AssessmentBuilder.jsx
- Redux state for builder
- Question editor component for each type
- Live preview pane
- Save to IndexedDB via MirageJS
```

### 3.2 Question Types

#### Short Text
- Single-line text input
- Optional max length validation
- Required field option

#### Long Text
- Multi-line textarea
- Optional max length validation
- Required field option

#### Single Choice
- Radio button group
- Add/remove options dynamically
- Required field option

#### Multiple Choice
- Checkbox group
- Add/remove options dynamically
- Stores array of selected values
- Required field option

#### Numeric
- Number input
- Optional min/max range validation
- Required field option

#### File Upload
- File input (stub - UI only)
- Shows selected file name and size
- Note: Files not actually uploaded (front-end only app)

### 3.3 Conditional Questions

**Feature:** Show/hide questions based on other answers

#### How it Works:
1. Select a question to depend on
2. Specify the value that triggers display
3. Question only shows when condition is met

#### Example:
```
Q1: "Have you worked in a similar role before?" (Yes/No)
Q2: "Describe your experience" (Conditional on Q1 === "Yes")
```

#### Implementation Details:
```javascript
// Location: src/features/assessments/QuestionEditor.jsx
- Dropdown to select parent question
- Text input for matching value
- Live preview updates based on responses
```

### 3.4 Assessment Preview

**Route:** `/assessments/:jobId/preview`

#### Features:
- **Full-Screen Form**: See assessment as candidates will
- **Real-time Validation**: Immediate feedback on errors
- **Conditional Rendering**: Questions appear/disappear based on answers
- **Submit**: Save responses to IndexedDB

#### Validation:
- Required field checking
- Max length enforcement
- Numeric range validation
- Error messages below fields

#### Implementation Details:
```javascript
// Location: src/features/assessments/AssessmentPreview.jsx
- Uses QuestionRenderer for consistency
- Full form validation on submit
- Stores responses with candidate ID
- Links back to builder
```

### 3.5 Assessment Completion Tracking

**Feature:** Track which candidates completed assessments

#### Displays:
- On candidate profile: Completed ✓ or Not Started
- Completion date when available
- Link to view responses (future enhancement)

#### Implementation Details:
```javascript
// Location: src/features/candidates/CandidateProfile.jsx
- Checks assessmentResponses table in IndexedDB
- Shows completion status
- Could be extended to show responses
```

---

## 4. Additional Features

### 4.1 Toast Notifications

**Feature:** User feedback for actions

#### Types:
- Success (green)
- Error (red)
- Warning (yellow)
- Info (blue)

#### Behavior:
- Auto-dismiss after 3 seconds
- Manual close button
- Stacks multiple toasts
- Smooth animations

#### Implementation Details:
```javascript
// Location: src/components/Toast.jsx
- Redux state management
- Headless UI transitions
- Position: top-right
```

### 4.2 Loading States

**Feature:** Visual feedback during async operations

#### Implementations:
- Full-page loading spinner
- Button loading states (with spinner)
- Skeleton loaders (future enhancement)

#### Implementation Details:
```javascript
// Location: src/components/LoadingSpinner.jsx
- Three sizes: sm, md, lg
- Animated SVG spinner
- Used throughout app
```

### 4.3 Empty States

**Feature:** Helpful messages when no data exists

#### Displays:
- Icon
- Title
- Description
- Call-to-action button

#### Examples:
- "No jobs found" with "Create Job" button
- "No candidates found" with filter suggestions

#### Implementation Details:
```javascript
// Location: src/components/EmptyState.jsx
- Reusable component
- Customizable icon, text, and action
```

### 4.4 Responsive Design

**Feature:** Works on all screen sizes

#### Breakpoints:
- Mobile: < 640px (1 column)
- Tablet: 640-1024px (2 columns)
- Desktop: > 1024px (3 columns)

#### Adaptations:
- Collapsible sidebar (future enhancement)
- Stacked layouts on mobile
- Touch-friendly targets
- Responsive grids

#### Implementation Details:
```javascript
// Tailwind CSS classes throughout
- Uses sm:, md:, lg: prefixes
- Grid auto-adapts
- Touch-friendly drag on mobile
```

### 4.5 Deep Linking

**Feature:** Direct URLs to specific resources

#### Supported Routes:
- `/jobs/:jobId` - Direct link to job
- `/candidates/:id` - Direct link to candidate
- `/assessments/:jobId` - Direct link to assessment builder

#### Benefits:
- Share specific jobs/candidates
- Bookmark frequently accessed pages
- Better navigation UX

### 4.6 Data Persistence

**Feature:** All data persists across page refreshes

#### Implementation:
- IndexedDB stores all data
- Auto-seed on first load
- MirageJS reads from IndexedDB
- No data loss on refresh

#### Data Tables:
- jobs
- candidates
- candidateTimeline
- assessments
- assessmentResponses
- metadata

---

## Feature Completion Checklist

### Core Requirements

#### Jobs
- [x] Create, edit, archive jobs
- [x] Server-like pagination & filtering
- [x] Drag-and-drop reordering
- [x] Optimistic updates with rollback
- [x] Deep linking
- [x] Validation (title, unique slug)

#### Candidates
- [x] Virtualized list (1000+ candidates)
- [x] Client-side search (name/email)
- [x] Server-like filter (stage)
- [x] Candidate profile with timeline
- [x] Kanban board with drag-drop
- [x] Stage transitions create timeline entries
- [x] Notes on timeline entries

#### Assessments
- [x] Assessment builder per job
- [x] Add sections and questions
- [x] 6 question types
- [x] Live preview pane
- [x] Form runtime with validation
- [x] Conditional questions
- [x] Persist to IndexedDB

### Additional Features
- [x] Job expiration dates
- [x] Auto-archive expired jobs
- [x] Expiring soon indicators
- [x] Candidate follow-up flag
- [x] Follow-up filter
- [x] Assessment completion status
- [x] Display completion on profile

### Data & API
- [x] MirageJS mock API
- [x] IndexedDB persistence
- [x] 25 jobs seeded
- [x] 1000 candidates seeded
- [x] 5 assessments with 10+ questions
- [x] Artificial latency (200-1200ms)
- [x] 5-10% error rate on writes
- [x] Restore from IndexedDB on refresh

### UI/UX
- [x] Modern, beautiful UI
- [x] Responsive design
- [x] Loading states
- [x] Error states
- [x] Toast notifications
- [x] Empty states
- [x] Smooth animations

---

## Usage Tips

1. **Testing Reorder Failures**: Drag jobs multiple times. ~10% of reorders will fail and rollback.

2. **Testing Virtualization**: Scroll through 1000 candidates smoothly in the list view.

3. **Testing Conditional Questions**: In assessment builder, create a Yes/No question, then add a follow-up question that only shows when "Yes" is selected.

4. **Testing Follow-up**: Mark several candidates for follow-up, then use the filter to see only those candidates.

5. **Testing Assessment Completion**: Submit an assessment in preview mode, then check the candidate profile to see completion status.

For more information, see [README.md](README.md) and [ARCHITECTURE.md](ARCHITECTURE.md).

