# foodTracksBH

A dual-platform application ecosystem connecting food truck owners, event organizers, and food enthusiasts in Bahrain.

## Overview

**foodTracksBH** addresses the critical gap in Bahrain's food truck industry by providing a centralized system for event discovery, registration, and management.

### Problem Solved
- Food truck owners struggle to discover and register for local events
- Event organizers lack efficient tools to manage vendor registrations
- Public users have no centralized way to discover food truck events

## Platform Components

### Mobile Application (React Native + Expo)
For food truck owners and public users:
- Discover and browse food truck events
- Register for events (vendors)
- View participating food trucks
- Manage truck profiles

### Web Dashboard (Next.js)
For event organizers and administrators:
- Create and manage events
- Review and process vendor registrations
- User management
- Analytics dashboard

### Backend (Firebase)
- Authentication (Email/Password, Phone, Social)
- Firestore database
- Firebase Storage for images
- Cloud Functions for backend logic
- Push notifications

## Tech Stack

| Component | Technology |
|-----------|------------|
| Mobile | React Native, Expo SDK 51+, expo-router |
| Web | Next.js 14+, Tailwind CSS, shadcn/ui |
| Backend | Firebase (Auth, Firestore, Storage, Functions) |
| State Management | Zustand |
| Forms | react-hook-form + zod |
| Maps | react-native-maps, Google Maps API |

## Project Structure

```
fahad-project/
├── GeneralDocument.md          # Comprehensive project documentation
├── GeneralDocumetAR.md         # Arabic documentation
├── website-plan/
│   ├── website-plan.md         # Web dashboard development plan
│   └── website-plan-ar.md      # Arabic web plan
├── mobile-plan/
│   ├── mobile-plan.md          # Mobile app development plan
│   └── mobile-plan-ar.md       # Arabic mobile plan
└── design/                     # Design assets
```

## User Roles

| Role | Access | Description |
|------|--------|-------------|
| Admin | Web Dashboard | Full system management, analytics, user management |
| Food Truck Owner | Mobile App + Web | Manage truck profile, register for events, track registrations |
| Regular User | Mobile App | Browse events, favorite trucks, leave reviews |

## Key Features

### For Food Truck Owners
- Browse upcoming events with capacity and fee details
- Submit event registrations
- Track registration status
- Manage truck profile and menu

### For Event Organizers
- Create and publish events
- Set capacity, fees, and requirements
- Review and accept/reject vendor applications
- View participant list

### For Public Users
- Discover food truck events nearby
- View participating vendors
- Favorite events and trucks
- Leave reviews and ratings

## Getting Started

### Prerequisites
- Node.js 18+
- npm or yarn
- Firebase account
- Expo CLI (for mobile)

### Mobile App Setup
```bash
cd mobile
npm install
npx expo start
```

### Web Dashboard Setup
```bash
cd website
npm install
npm run dev
```

### Environment Variables
```env
# Firebase Client
NEXT_PUBLIC_FIREBASE_API_KEY=
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=
NEXT_PUBLIC_FIREBASE_PROJECT_ID=
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=
NEXT_PUBLIC_FIREBASE_APP_ID=

# Firebase Admin (Server)
FIREBASE_PROJECT_ID=
FIREBASE_CLIENT_EMAIL=
FIREBASE_PRIVATE_KEY=

# Google Maps
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=
```

## Development Timeline

| Phase | Duration | Deliverables |
|-------|----------|--------------|
| Planning & Design | 2 weeks | Documentation, UI/UX mockups, DB schema |
| Backend Development | 3 weeks | Firebase setup, API, Auth system |
| Web Dashboard | 4 weeks | Admin dashboard, event management |
| Mobile Application | 6 weeks | Mobile app, auth, event browsing |
| Testing & Deployment | 3 weeks | Testing, bug fixes, store deployment |

## Documentation

- [General Documentation](./GeneralDocument.md) - Complete project documentation
- [Website Plan](./website-plan/website-plan.md) - Web dashboard development details
- [Mobile Plan](./mobile-plan/mobile-plan.md) - Mobile app development details

## License

This project is proprietary and confidential.
