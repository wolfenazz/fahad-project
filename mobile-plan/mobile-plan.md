# Mobile Plan - foodTracksBH (Advanced Development Plan)

## Overview
React Native (Expo) mobile app for food truck owners and regular users to discover events and manage registrations.

---

## Tech Stack & Key Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Framework | Expo SDK 51+ | Managed workflow, OTA updates |
| Navigation | expo-router (file-based) | Better DX, deep linking built-in |
| State | Zustand | Lightweight, no boilerplate |
| Forms | react-hook-form + zod | Type-safe validation |
| Storage | AsyncStorage + MMKV | MMKV for sensitive data |
| Maps | react-native-maps | Google/Apple Maps support |
| Images | expo-image-picker + expo-image | Optimized loading |

---

## Project Structure

```
mobile/
├── app/                          # expo-router routes
│   ├── (auth)/
│   │   ├── login.tsx
│   │   ├── register.tsx
│   │   └── role-select.tsx
│   ├── (tabs)/                   # Bottom tabs (role-based)
│   │   ├── _layout.tsx
│   │   ├── index.tsx             # Home/Explore
│   │   ├── events.tsx
│   │   ├── registrations.tsx     # Vendor only
│   │   └── profile.tsx
│   ├── event/
│   │   └── [id].tsx              # Event details
│   ├── truck/
│   │   └── [id].tsx              # Truck profile (public)
│   ├── _layout.tsx               # Root layout
│   └── +not-found.tsx
├── src/
│   ├── components/
│   │   ├── ui/                   # Base components
│   │   │   ├── Button.tsx
│   │   │   ├── Card.tsx
│   │   │   ├── Input.tsx
│   │   │   ├── Badge.tsx
│   │   │   ├── Avatar.tsx
│   │   │   ├── Skeleton.tsx
│   │   │   └── index.ts
│   │   ├── EventCard.tsx
│   │   ├── TruckCard.tsx
│   │   ├── RegistrationStatus.tsx
│   │   ├── MapPreview.tsx
│   │   └── StatusBadge.tsx
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   ├── useEvents.ts
│   │   ├── useRegistrations.ts
│   │   ├── useUser.ts
│   │   └── useLocation.ts
│   ├── stores/
│   │   ├── authStore.ts
│   │   └── userStore.ts
│   ├── services/
│   │   ├── firebase.ts
│   │   ├── auth.ts
│   │   ├── events.ts
│   │   ├── registrations.ts
│   │   └── storage.ts
│   ├── types/
│   │   ├── user.ts
│   │   ├── event.ts
│   │   └── registration.ts
│   ├── constants/
│   │   ├── colors.ts
│   │   ├── theme.ts
│   │   └── config.ts
│   └── utils/
│       ├── formatters.ts
│       ├── validators.ts
│       └── helpers.ts
├── app.json
├── eas.json
└── package.json
```

---

## Core Components

### UI Components (`src/components/ui/`)

**Button**
```typescript
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'outline' | 'ghost';
  size: 'sm' | 'md' | 'lg';
  loading?: boolean;
  disabled?: boolean;
  fullWidth?: boolean;
  leftIcon?: ReactNode;
  rightIcon?: ReactNode;
}
```

**Card**
```typescript
interface CardProps {
  variant?: 'elevated' | 'outlined' | 'filled';
  padding?: 'none' | 'sm' | 'md' | 'lg';
  onPress?: () => void;
}
```

**Input**
```typescript
interface InputProps {
  label?: string;
  error?: string;
  leftIcon?: ReactNode;
  rightIcon?: ReactNode;
  secureTextEntry?: boolean;
}
```

**Badge**
```typescript
interface BadgeProps {
  variant: 'pending' | 'accepted' | 'rejected' | 'default';
  size?: 'sm' | 'md';
}
```

**Skeleton**
```typescript
interface SkeletonProps {
  variant: 'text' | 'rectangular' | 'circular';
  width?: number | string;
  height?: number;
  borderRadius?: number;
}
```

### Feature Components

**EventCard** - Displays event preview in list
- Props: `event: Event`, `onPress`, `variant: 'user' | 'vendor'`
- Shows: poster, title, date, location, capacity indicator (vendor)

**TruckCard** - Displays truck in participant list
- Props: `truck: Truck`, `onPress`
- Shows: logo, name, cuisine type

**RegistrationStatus** - Status tracker for vendor registrations
- Props: `registration: Registration`
- Shows: status badge, submitted date, event info

**MapPreview** - Interactive map location display
- Props: `location: Location`, `interactive?: boolean`
- Shows: map with marker, address

---

## Data Models (TypeScript)

```typescript
// types/user.ts
type UserRole = 'user' | 'vendor';

interface User {
  uid: string;
  email: string;
  displayName?: string;
  role: UserRole;
  phone?: string;
  avatar?: string;
  createdAt: Date;
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
}

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
  createdAt: Date;
}

// types/registration.ts
type RegistrationStatus = 'pending' | 'accepted' | 'rejected';

interface Registration {
  id: string;
  eventId: string;
  eventTitle: string;
  vendorId: string;
  truckName: string;
  status: RegistrationStatus;
  submittedAt: Date;
  reviewedAt?: Date;
  notes?: string;
}
```

---

## Zustand Stores

### authStore
```typescript
interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  login: (email: string, password: string) => Promise<void>;
  register: (data: RegisterData) => Promise<void>;
  logout: () => Promise<void>;
  checkAuth: () => Promise<void>;
}
```

### userStore
```typescript
interface UserState {
  profile: User | Vendor | null;
  isVendor: boolean;
  updateProfile: (data: Partial<User>) => Promise<void>;
  updateTruckProfile: (data: Partial<Vendor>) => Promise<void>;
}
```

---

## Custom Hooks

| Hook | Purpose | Returns |
|------|---------|---------|
| `useAuth()` | Auth state & actions | `{ user, isAuthenticated, login, logout, ... }` |
| `useEvents()` | Fetch & filter events | `{ events, loading, refetch, filterByDate }` |
| `useEvent(id)` | Single event details | `{ event, loading, error }` |
| `useRegistrations()` | Vendor registrations | `{ registrations, pending, accepted, submit }` |
| `useLocation()` | Device location | `{ location, requestPermission }` |

---

## Screen Specifications

### 1. Splash Screen (`app/index.tsx`)
- Animated logo (Lottie or react-native-reanimated)
- Check auth state
- Redirect to: `(tabs)` if authenticated, `(auth)` if not

### 2. Login (`app/(auth)/login.tsx`)
- Email/password inputs
- "Forgot password" link
- Social login (optional: Google)
- Redirect to role-select if first login

### 3. Register (`app/(auth)/register.tsx`)
- Email, password, confirm password
- Display name
- Redirect to role-select on success

### 4. Role Select (`app/(auth)/role-select.tsx`)
- Two large cards: "I'm a Foodie" | "I'm a Vendor"
- Animated selection
- Set role in Firestore

### 5. Home/Explore (`app/(tabs)/index.tsx`)
**For Users:**
- Featured event banner (carousel)
- Upcoming events list (EventCard)
- Location filter button

**For Vendors:**
- Quick stats (pending registrations, upcoming events)
- Available events for registration
- CTA: "Browse All Events"

### 6. Events List (`app/(tabs)/events.tsx`)
- Calendar toggle (list/calendar view)
- Filter chips: date range, location
- Pull-to-refresh
- Infinite scroll pagination

### 7. Event Details (`app/event/[id].tsx`)
**Shared:**
- Hero image (poster)
- Title, description
- Date/time display
- Map location
- "View Participants" button

**Vendor-only:**
- Capacity indicator (X/Y spots)
- Fee display
- Requirements list
- Amenities list
- "Register" CTA button (if not registered)
- Registration status (if registered)

### 8. My Registrations (`app/(tabs)/registrations.tsx`) - Vendor only
- Tabs: Pending | Accepted | Past
- RegistrationStatus cards
- Cancel pending registration

### 9. Profile (`app/(tabs)/profile.tsx`)
**User:**
- Avatar, name, email
- Edit profile button
- Notification preferences
- Logout

**Vendor:**
- Truck profile card
- Edit truck profile
- View public profile link
- Logout

---

## Firebase Services Structure

```typescript
// services/firebase.ts
import { initializeApp } from 'firebase/app';
import { getAuth } from 'firebase/auth';
import { getFirestore } from 'firebase/firestore';
import { getStorage } from 'firebase/storage';

// services/auth.ts
export const signIn = (email, password) => { ... }
export const signUp = (email, password, displayName) => { ... }
export const signOut = () => { ... }
export const sendPasswordReset = (email) => { ... }
export const onAuthStateChange = (callback) => { ... }

// services/events.ts
export const getEvents = (filters?) => { ... }
export const getEventById = (id) => { ... }
export const getUpcomingEvents = () => { ... }
export const subscribeToEvent = (id, callback) => { ... }

// services/registrations.ts
export const getVendorRegistrations = (vendorId) => { ... }
export const submitRegistration = (eventId, vendorId) => { ... }
export const cancelRegistration = (registrationId) => { ... }
export const subscribeToRegistrations = (vendorId, callback) => { ... }

// services/storage.ts
export const uploadImage = (uri, path) => { ... }
export const getDownloadUrl = (path) => { ... }
```

---

## Error Handling Strategy

### Error Boundary
- Wrap root layout
- Show friendly error screen with retry button

### Form Errors
- react-hook-form + zod for validation
- Show inline errors below inputs

### API Errors
- Toast notifications (react-native-toast-message)
- Retry button for failed requests
- Offline detection with retry queue

### Loading States
- Skeleton loaders for lists
- Button loading spinners
- Pull-to-refresh indicator

---

## Development Tasks (Ordered by Dependency)

### Phase 1: Setup (Day 1)
- [ ] `npx create-expo-app mobile --template tabs`
- [ ] Install dependencies: `zustand react-hook-form zod react-native-maps expo-image-picker`
- [ ] Configure Firebase (create project, add config)
- [ ] Set up project structure (create folders)
- [ ] Create constants: `colors.ts`, `theme.ts`
- [ ] Set up ESLint + Prettier

### Phase 2: Foundation (Day 2-3)
- [ ] Build UI components: `Button`, `Input`, `Card`, `Badge`, `Skeleton`
- [ ] Create `firebase.ts` service
- [ ] Create `authStore` with Zustand
- [ ] Implement `useAuth` hook
- [ ] Set up expo-router layouts

### Phase 3: Authentication (Day 4-5)
- [ ] Build Splash screen with auth check
- [ ] Build Login screen
- [ ] Build Register screen
- [ ] Build Role-select screen
- [ ] Connect to Firebase Auth
- [ ] Test auth flow end-to-end

### Phase 4: Events Feature (Day 6-8)
- [ ] Create `types/event.ts`
- [ ] Build `events.ts` service
- [ ] Build `EventCard` component
- [ ] Build `MapPreview` component
- [ ] Build Home screen (user view)
- [ ] Build Events list screen
- [ ] Build Event details screen
- [ ] Implement pull-to-refresh

### Phase 5: Vendor Features (Day 9-11)
- [ ] Create `types/registration.ts`
- [ ] Build `registrations.ts` service
- [ ] Build `RegistrationStatus` component
- [ ] Build Vendor Home dashboard
- [ ] Build My Registrations screen
- [ ] Implement registration submission
- [ ] Build Truck Profile management

### Phase 6: Polish (Day 12-14)
- [ ] Add loading skeletons everywhere
- [ ] Add error boundaries
- [ ] Add toast notifications
- [ ] Implement offline indicator
- [ ] Add animations (layout transitions)
- [ ] Test all flows
- [ ] Fix bugs

### Phase 7: Deployment Prep (Day 15)
- [ ] Configure EAS Build
- [ ] Add app icons and splash
- [ ] Build test APK/IPA
- [ ] Internal testing
- [ ] Submit to stores

---

## Key Dependencies

```json
{
  "dependencies": {
    "expo": "~51.0.0",
    "expo-router": "~3.5.0",
    "expo-image-picker": "~15.0.0",
    "expo-location": "~17.0.0",
    "firebase": "^10.0.0",
    "zustand": "^4.5.0",
    "react-hook-form": "^7.50.0",
    "zod": "^3.22.0",
    "react-native-maps": "^1.10.0",
    "react-native-reanimated": "~3.10.0",
    "react-native-toast-message": "^2.2.0",
    "date-fns": "^3.0.0"
  },
  "devDependencies": {
    "@types/react": "~18.2.0",
    "typescript": "~5.3.0"
  }
}
```

---

## Environment Variables

Create `env.js` (not committed):
```javascript
export default {
  FIREBASE_API_KEY: "...",
  FIREBASE_AUTH_DOMAIN: "...",
  FIREBASE_PROJECT_ID: "...",
  FIREBASE_STORAGE_BUCKET: "...",
  FIREBASE_MESSAGING_SENDER_ID: "...",
  FIREBASE_APP_ID: "...",
};
```

---

## Testing Checklist

### Authentication
- [ ] Login with valid credentials
- [ ] Login with invalid credentials (error shown)
- [ ] Register new user
- [ ] Password reset flow
- [ ] Role selection persists
- [ ] Logout clears session

### Events
- [ ] Events list loads
- [ ] Pull-to-refresh works
- [ ] Event details display correctly
- [ ] Map shows correct location
- [ ] Filtering works

### Registrations (Vendor)
- [ ] Can submit registration
- [ ] Status updates reflected
- [ ] Cannot register twice for same event
- [ ] Can cancel pending registration

### Profile
- [ ] Profile data displays
- [ ] Can edit profile
- [ ] Truck profile updates (vendor)
- [ ] Logout works
