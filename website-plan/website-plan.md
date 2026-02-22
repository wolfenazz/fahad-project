# Website Plan - foodTracksBH Admin Dashboard (Advanced Development Plan)

## Overview
Next.js admin dashboard for managing events, vendor registrations, and users. Built with App Router, Tailwind CSS, and Firebase.

---

## Tech Stack & Key Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Framework | Next.js 14+ (App Router) | Server components, streaming |
| Styling | Tailwind CSS + shadcn/ui | Rapid UI development |
| Forms | react-hook-form + zod | Type-safe validation |
| State | Zustand (client) + Server State | Minimal boilerplate |
| Auth | Firebase Auth + Middleware | Secure admin routes |
| Tables | TanStack Table | Sorting, filtering, pagination |
| Date | date-fns | Lightweight date handling |
| Notifications | sonner | Toast notifications |

---

## Project Structure

```
website/
├── src/
│   ├── app/
│   │   ├── (auth)/
│   │   │   ├── login/
│   │   │   │   └── page.tsx
│   │   │   └── layout.tsx
│   │   ├── (dashboard)/
│   │   │   ├── page.tsx              # Overview
│   │   │   ├── events/
│   │   │   │   ├── page.tsx          # Events list
│   │   │   │   ├── new/
│   │   │   │   │   └── page.tsx      # Create event
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx      # Edit event
│   │   │   ├── registrations/
│   │   │   │   └── page.tsx          # Registration management
│   │   │   ├── users/
│   │   │   │   ├── page.tsx          # Users list
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx      # User details
│   │   │   └── layout.tsx
│   │   ├── api/
│   │   │   ├── events/
│   │   │   │   ├── route.ts          # GET, POST
│   │   │   │   └── [id]/
│   │   │   │       └── route.ts      # GET, PUT, DELETE
│   │   │   ├── registrations/
│   │   │   │   ├── route.ts          # GET
│   │   │   │   └── [id]/
│   │   │   │       └── route.ts      # PUT (accept/reject)
│   │   │   └── stats/
│   │   │       └── route.ts          # Dashboard stats
│   │   ├── layout.tsx
│   │   └── globals.css
│   ├── components/
│   │   ├── ui/                       # shadcn components
│   │   │   ├── button.tsx
│   │   │   ├── input.tsx
│   │   │   ├── card.tsx
│   │   │   ├── badge.tsx
│   │   │   ├── dialog.tsx
│   │   │   ├── dropdown-menu.tsx
│   │   │   ├── table.tsx
│   │   │   ├── tabs.tsx
│   │   │   └── ...
│   │   ├── layout/
│   │   │   ├── Sidebar.tsx
│   │   │   ├── Header.tsx
│   │   │   └── MobileNav.tsx
│   │   ├── forms/
│   │   │   ├── EventForm.tsx
│   │   │   └── LocationPicker.tsx
│   │   ├── tables/
│   │   │   ├── EventsTable.tsx
│   │   │   ├── RegistrationsTable.tsx
│   │   │   └── UsersTable.tsx
│   │   ├── cards/
│   │   │   ├── StatCard.tsx
│   │   │   ├── EventCard.tsx
│   │   │   └── RegistrationCard.tsx
│   │   └── shared/
│   │       ├── DataTable.tsx         # Generic table wrapper
│   │       ├── PageHeader.tsx
│   │       ├── EmptyState.tsx
│   │       ├── LoadingSkeleton.tsx
│   │       └── ConfirmDialog.tsx
│   ├── lib/
│   │   ├── firebase.ts               # Firebase client
│   │   ├── firebase-admin.ts         # Admin SDK
│   │   ├── auth.ts                   # Auth utilities
│   │   ├── utils.ts                  # Helper functions
│   │   └── validations.ts            # Zod schemas
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   ├── useEvents.ts
│   │   ├── useRegistrations.ts
│   │   └── useStats.ts
│   ├── types/
│   │   ├── user.ts
│   │   ├── event.ts
│   │   └── registration.ts
│   └── constants/
│       └── colors.ts
├── public/
├── middleware.ts                     # Auth protection
├── tailwind.config.ts
├── next.config.js
└── package.json
```

---

## Core Components

### Layout Components

**Sidebar**
- Logo
- Navigation links: Dashboard, Events, Registrations, Users
- Active state indicator (red accent)
- Logout button at bottom
- Collapsible on mobile

**Header**
- Page title
- Breadcrumbs
- User avatar dropdown

**MobileNav**
- Hamburger menu
- Sheet with navigation

### Data Tables

**DataTable** (Generic)
```typescript
interface DataTableProps<T> {
  data: T[];
  columns: ColumnDef<T>[];
  searchKey?: string;
  searchPlaceholder?: string;
  statusFilter?: { options: string[]; defaultValue: string };
  pagination?: boolean;
}
```

**EventsTable**
- Columns: Image, Title, Date, Location, Capacity, Status, Actions
- Actions: View, Edit, Delete
- Filter by status (upcoming, ongoing, ended)

**RegistrationsTable**
- Columns: Vendor, Event, Truck Name, Submitted, Status, Actions
- Actions: Accept, Reject, View Details
- Filter by status (pending, accepted, rejected)

**UsersTable**
- Columns: Avatar, Name, Email, Role, Joined, Actions
- Actions: View Profile
- Filter by role (user, vendor)

### Form Components

**EventForm**
```typescript
interface EventFormProps {
  initialData?: Event;  // For edit mode
  onSubmit: (data: EventFormData) => Promise<void>;
  isSubmitting?: boolean;
}

// Fields:
// - title (text, required)
// - description (textarea)
// - posterImage (file upload)
// - startDate, endDate (date pickers)
// - location (Google Places autocomplete + map)
// - capacity (number)
// - fees (number)
// - requirements (multi-text input)
// - amenities (multi-select: electricity, water, wifi)
```

**LocationPicker**
- Google Places autocomplete input
- Map preview with draggable marker
- Stores: name, address, lat, lng

### Cards

**StatCard**
```typescript
interface StatCardProps {
  title: string;
  value: number | string;
  change?: number;  // Percentage change
  icon: ReactNode;
  trend?: 'up' | 'down' | 'neutral';
}
```

---

## Data Models (TypeScript)

```typescript
// types/event.ts
interface Location {
  name: string;
  address: string;
  latitude: number;
  longitude: number;
}

interface Event {
  id: string;
  title: string;
  description: string;
  posterImage: string;
  startDate: Date;
  endDate: Date;
  location: Location;
  capacity: number;
  currentRegistrations: number;
  fees: number;
  requirements: string[];
  amenities: string[];
  status: 'upcoming' | 'ongoing' | 'ended';
  createdBy: string;
  createdAt: Date;
  updatedAt: Date;
}

// types/registration.ts
type RegistrationStatus = 'pending' | 'accepted' | 'rejected';

interface Registration {
  id: string;
  eventId: string;
  eventTitle: string;
  eventStartDate: Date;
  vendorId: string;
  vendorName: string;
  vendorEmail: string;
  truckName: string;
  truckLogo?: string;
  status: RegistrationStatus;
  submittedAt: Date;
  reviewedAt?: Date;
  reviewedBy?: string;
  notes?: string;
}

// types/user.ts
type UserRole = 'user' | 'vendor' | 'admin';

interface User {
  uid: string;
  email: string;
  displayName?: string;
  role: UserRole;
  phone?: string;
  avatar?: string;
  createdAt: Date;
  lastLogin?: Date;
}

interface Vendor extends User {
  role: 'vendor';
  truckName: string;
  truckLogo?: string;
  truckDescription?: string;
  truckCuisine?: string;
  menuLink?: string;
  instagram?: string;
  isVerified: boolean;
  totalRegistrations: number;
  acceptedRegistrations: number;
}
```

---

## Validation Schemas (Zod)

```typescript
// lib/validations.ts
import { z } from 'zod';

export const eventSchema = z.object({
  title: z.string().min(3, 'Title must be at least 3 characters'),
  description: z.string().min(10, 'Description must be at least 10 characters'),
  posterImage: z.string().url('Must be a valid URL'),
  startDate: z.date(),
  endDate: z.date(),
  location: z.object({
    name: z.string(),
    address: z.string(),
    latitude: z.number(),
    longitude: z.number(),
  }),
  capacity: z.number().min(1, 'Capacity must be at least 1'),
  fees: z.number().min(0, 'Fees cannot be negative'),
  requirements: z.array(z.string()),
  amenities: z.array(z.string()),
}).refine(data => data.endDate > data.startDate, {
  message: 'End date must be after start date',
  path: ['endDate'],
});

export const loginSchema = z.object({
  email: z.string().email('Invalid email address'),
  password: z.string().min(6, 'Password must be at least 6 characters'),
});
```

---

## Page Specifications

### 1. Login (`/login`)
- Email/password form
- "Remember me" checkbox
- Error toast on failed login
- Redirect to dashboard on success

### 2. Dashboard Overview (`/`)
**Stat Cards:**
- Total Active Events
- Total Registered Vendors
- Pending Registrations (clickable, links to registrations)
- This Month's Events

**Recent Activity:**
- Last 5 registration requests
- Quick actions (Accept/Reject inline)

**Quick Actions:**
- "Create Event" button
- "View Pending Registrations" button

### 3. Events List (`/events`)
- Search by title
- Status filter (upcoming, ongoing, ended)
- EventsTable
- "Create Event" button (top right)
- Pagination (10 per page)

### 4. Create Event (`/events/new`)
- EventForm
- Image upload with preview
- Date range picker
- Location picker with map
- Requirements as dynamic list
- Save as draft / Publish buttons
- Cancel link

### 5. Edit Event (`/events/[id]`)
- Same as Create, pre-filled
- "Delete Event" button (with confirmation)
- Shows current registrations count

### 6. Registrations (`/registrations`)
- Status filter tabs (All, Pending, Accepted, Rejected)
- Search by vendor/truck name
- RegistrationsTable
- Accept/Reject buttons (pending only)
- Click row to expand details

### 7. Users (`/users`)
- Role filter (All, Users, Vendors)
- Search by name/email
- UsersTable
- Click to view profile

### 8. User Details (`/users/[id]`)
- Profile card with avatar
- Contact info
- For vendors: truck details, registration history

---

## API Routes

### `/api/events`
```
GET    /api/events           → Event[] (with pagination)
POST   /api/events           → Create event
GET    /api/events/[id]      → Event
PUT    /api/events/[id]      → Update event
DELETE /api/events/[id]      → Delete event
```

### `/api/registrations`
```
GET  /api/registrations           → Registration[] (with filters)
PUT  /api/registrations/[id]      → Update status (accept/reject)
```

### `/api/stats`
```
GET  /api/stats  → { activeEvents, totalVendors, pendingRegistrations, monthlyEvents }
```

### `/api/upload`
```
POST /api/upload  → { url: string }  (Firebase Storage)
```

---

## Middleware & Auth

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export async function middleware(request: NextRequest) {
  const session = request.cookies.get('session');
  const isAdmin = await verifyAdminSession(session);

  if (!isAdmin && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico|login).*)'],
};
```

---

## Firebase Admin Setup

```typescript
// lib/firebase-admin.ts
import { initializeApp, getApps, cert } from 'firebase-admin/app';
import { getAuth } from 'firebase-admin/auth';
import { getFirestore } from 'firebase-admin/firestore';

if (!getApps().length) {
  initializeApp({
    credential: cert({
      projectId: process.env.FIREBASE_PROJECT_ID,
      clientEmail: process.env.FIREBASE_CLIENT_EMAIL,
      privateKey: process.env.FIREBASE_PRIVATE_KEY?.replace(/\\n/g, '\n'),
    }),
  });
}

export const adminAuth = getAuth();
export const adminDb = getFirestore();
```

---

## Development Tasks (Ordered by Dependency)

### Phase 1: Setup (Day 1)
- [ ] `npx create-next-app@latest website --typescript --tailwind --app`
- [ ] Install: `zustand firebase react-hook-form zod @hookform/resolvers date-fns sonner @tanstack/react-table`
- [ ] Initialize shadcn/ui: `npx shadcn-ui@latest init`
- [ ] Add shadcn components: button, input, card, badge, dialog, table, tabs
- [ ] Set up Firebase project
- [ ] Create `lib/firebase.ts` and `lib/firebase-admin.ts`
- [ ] Configure environment variables

### Phase 2: Auth & Layout (Day 2)
- [ ] Create middleware for route protection
- [ ] Build Login page with form
- [ ] Implement Firebase Auth (email/password)
- [ ] Build Sidebar component
- [ ] Build Header component
- [ ] Create dashboard layout
- [ ] Add logout functionality

### Phase 3: Dashboard Overview (Day 3)
- [ ] Create `StatCard` component
- [ ] Build `/api/stats` route
- [ ] Create dashboard page with stats
- [ ] Add recent registrations widget
- [ ] Add quick actions section

### Phase 4: Events Management (Day 4-6)
- [ ] Create event types and validation schema
- [ ] Build `EventsTable` component
- [ ] Create events list page
- [ ] Build `EventForm` component
- [ ] Create "New Event" page
- [ ] Build `LocationPicker` with Google Places
- [ ] Implement image upload (Firebase Storage)
- [ ] Create "Edit Event" page
- [ ] Add delete functionality with confirmation

### Phase 5: Registrations Management (Day 7)
- [ ] Create registration types
- [ ] Build `RegistrationsTable` component
- [ ] Create registrations list page
- [ ] Implement status filter tabs
- [ ] Add accept/reject actions
- [ ] Create `ConfirmDialog` component
- [ ] Add toast notifications for actions

### Phase 6: Users Directory (Day 8)
- [ ] Build `UsersTable` component
- [ ] Create users list page
- [ ] Implement role filter
- [ ] Create user details page
- [ ] Show vendor details and history

### Phase 7: Polish (Day 9-10)
- [ ] Add loading skeletons
- [ ] Add empty states
- [ ] Implement error boundaries
- [ ] Add toast notifications everywhere
- [ ] Responsive design testing
- [ ] Add keyboard shortcuts
- [ ] Optimize images

### Phase 8: Deployment (Day 11)
- [ ] Set up Vercel project
- [ ] Configure environment variables
- [ ] Test production build
- [ ] Deploy

---

## Key Dependencies

```json
{
  "dependencies": {
    "next": "^14.0.0",
    "react": "^18.2.0",
    "firebase": "^10.0.0",
    "firebase-admin": "^12.0.0",
    "zustand": "^4.5.0",
    "react-hook-form": "^7.50.0",
    "zod": "^3.22.0",
    "@hookform/resolvers": "^3.3.0",
    "@tanstack/react-table": "^8.11.0",
    "date-fns": "^3.0.0",
    "sonner": "^1.3.0",
    "lucide-react": "^0.300.0",
    "tailwind-merge": "^2.2.0",
    "clsx": "^2.1.0"
  },
  "devDependencies": {
    "typescript": "^5.3.0",
    "tailwindcss": "^3.4.0",
    "@types/node": "^20.0.0",
    "@types/react": "^18.2.0"
  }
}
```

---

## Environment Variables

```env
# Firebase Client
NEXT_PUBLIC_FIREBASE_API_KEY=
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=
NEXT_PUBLIC_FIREBASE_PROJECT_ID=
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=
NEXT_PUBLIC_FIREBASE_APP_ID=

# Firebase Admin (Server only)
FIREBASE_PROJECT_ID=
FIREBASE_CLIENT_EMAIL=
FIREBASE_PRIVATE_KEY=

# Google Places (for location picker)
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=
```

---

## UI/UX Guidelines

### Colors (Tailwind Config)
```typescript
colors: {
  primary: {
    DEFAULT: '#df160b',
    dark: '#b31209',
    light: '#ff1f12',
  },
  success: '#10b981',
  warning: '#f59e0b',
  danger: '#ef4444',
}
```

### Status Badges
- **Pending**: Yellow badge (`bg-yellow-100 text-yellow-800`)
- **Accepted**: Green badge (`bg-green-100 text-green-800`)
- **Rejected**: Red badge (`bg-red-100 text-red-800`)

### Loading States
- Skeleton: Gray pulsing rectangles
- Button: Spinner + "Loading..." text
- Page: Full-page skeleton

### Error States
- Form: Red border + error message below input
- API: Toast notification with error message
- 404: Friendly illustration + "Go back" button

---

## Testing Checklist

### Authentication
- [ ] Login with valid credentials
- [ ] Login with invalid credentials (error shown)
- [ ] Protected routes redirect to login
- [ ] Logout clears session
- [ ] Session persists on refresh

### Events
- [ ] Events list loads with pagination
- [ ] Search filters events
- [ ] Create event with valid data
- [ ] Form validation shows errors
- [ ] Edit event updates data
- [ ] Delete event with confirmation
- [ ] Image upload works

### Registrations
- [ ] List shows all registrations
- [ ] Status filter works
- [ ] Accept updates status
- [ ] Reject updates status
- [ ] Toast shows on action

### Users
- [ ] Users list loads
- [ ] Role filter works
- [ ] User details page shows data
- [ ] Vendor details show truck info
