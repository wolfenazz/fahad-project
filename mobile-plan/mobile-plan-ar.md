# خطة تطبيق الموبايل - foodTracksBH (خطة تطوير متقدمة)

## نظرة عامة
تطبيق موبايل باستخدام React Native (Expo) ل أصحاب شاحنات الطعام والمستخدمين العاديين لاكتشاف الفعاليات وإدارة التسجيلات.

---

## التقنيات والقرارات الأساسية

| القرار | الاختيار | السبب |
|--------|---------|-------|
| الإطار الأساسي | Expo SDK 51+ | سير عمل مُدار، تحديثات OTA |
| التنقل | expo-router (قائم على الملفات) | تجربة مطور أفضل، روابط عميقة مدمجة |
| إدارة الحالة | Zustand | خفيف الوزن، بدون تعقيدات |
| النماذج | react-hook-form + zod | تحقق آمن من الأنواع |
| التخزين | AsyncStorage + MMKV | MMKV للبيانات الحساسة |
| الخرائط | react-native-maps | دعم Google/Apple Maps |
| الصور | expo-image-picker + expo-image | تحميل محسّن |

---

## هيكل المشروع

```
mobile/
├── app/                          # مسارات expo-router
│   ├── (auth)/
│   │   ├── login.tsx
│   │   ├── register.tsx
│   │   └── role-select.tsx
│   ├── (tabs)/                   # التبويبات السفلية (حسب الدور)
│   │   ├── _layout.tsx
│   │   ├── index.tsx             # الرئيسية/استكشف
│   │   ├── events.tsx
│   │   ├── registrations.tsx     # البائعين فقط
│   │   └── profile.tsx
│   ├── event/
│   │   └── [id].tsx              # تفاصيل الفعالية
│   ├── truck/
│   │   └── [id].tsx              # ملف الشاحنة (عام)
│   ├── _layout.tsx               # التخطيط الرئيسي
│   └── +not-found.tsx
├── src/
│   ├── components/
│   │   ├── ui/                   # المكونات الأساسية
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

## المكونات الأساسية

### مكونات الواجهة (`src/components/ui/`)

**زر (Button)**
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

**بطاقة (Card)**
```typescript
interface CardProps {
  variant?: 'elevated' | 'outlined' | 'filled';
  padding?: 'none' | 'sm' | 'md' | 'lg';
  onPress?: () => void;
}
```

**حقل إدخال (Input)**
```typescript
interface InputProps {
  label?: string;
  error?: string;
  leftIcon?: ReactNode;
  rightIcon?: ReactNode;
  secureTextEntry?: boolean;
}
```

**شارة (Badge)**
```typescript
interface BadgeProps {
  variant: 'pending' | 'accepted' | 'rejected' | 'default';
  size?: 'sm' | 'md';
}
```

**هيكل التحميل (Skeleton)**
```typescript
interface SkeletonProps {
  variant: 'text' | 'rectangular' | 'circular';
  width?: number | string;
  height?: number;
  borderRadius?: number;
}
```

### مكونات الميزات

**EventCard** - يعرض معاينة الفعالية في القائمة
- الخصائص: `event: Event`, `onPress`, `variant: 'user' | 'vendor'`
- يعرض: الملصق، العنوان، التاريخ، الموقع، مؤشر السعة (للبائعين)

**TruckCard** - يعرض الشاحنة في قائمة المشاركين
- الخصائص: `truck: Truck`, `onPress`
- يعرض: الشعار، الاسم، نوع المطبخ

**RegistrationStatus** - متتبع حالة تسجيلات البائعين
- الخصائص: `registration: Registration`
- يعرض: شارة الحالة، تاريخ الإرسال، معلومات الفعالية

**MapPreview** - عرض موقع الخريطة التفاعلي
- الخصائص: `location: Location`, `interactive?: boolean`
- يعرض: خريطة مع علامة، العنوان

---

## نماذج البيانات (TypeScript)

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

## مخازن Zustand

### authStore (مخزن المصادقة)
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

### userStore (مخزن المستخدم)
```typescript
interface UserState {
  profile: User | Vendor | null;
  isVendor: boolean;
  updateProfile: (data: Partial<User>) => Promise<void>;
  updateTruckProfile: (data: Partial<Vendor>) => Promise<void>;
}
```

---

## الخطافات المخصصة

| الخطاف | الغرض | القيم المرجعة |
|--------|-------|---------------|
| `useAuth()` | حالة وإجراءات المصادقة | `{ user, isAuthenticated, login, logout, ... }` |
| `useEvents()` | جلب وتصفية الفعاليات | `{ events, loading, refetch, filterByDate }` |
| `useEvent(id)` | تفاصيل فعالية واحدة | `{ event, loading, error }` |
| `useRegistrations()` | تسجيلات البائع | `{ registrations, pending, accepted, submit }` |
| `useLocation()` | موقع الجهاز | `{ location, requestPermission }` |

---

## مواصفات الشاشات

### 1. شاشة البداية (`app/index.tsx`)
- شعار متحرك (Lottie أو react-native-reanimated)
- التحقق من حالة المصادقة
- التحويل إلى: `(tabs)` إذا كان مصادق عليه، `(auth)` إذا لم يكن

### 2. تسجيل الدخول (`app/(auth)/login.tsx`)
- حقول البريد الإلكتروني/كلمة المرور
- رابط "نسيت كلمة المرور"
- تسجيل الدخول الاجتماعي (اختياري: Google)
- التحويل إلى اختيار الدور إذا كان أول تسجيل دخول

### 3. التسجيل (`app/(auth)/register.tsx`)
- البريد الإلكتروني، كلمة المرور، تأكيد كلمة المرور
- الاسم المعروض
- التحويل إلى اختيار الدور عند النجاح

### 4. اختيار الدور (`app/(auth)/role-select.tsx`)
- بطاقتان كبيرتان: "أنا زبون" | "أنا صاحب شاحنة"
- اختيار متحرك
- تعيين الدور في Firestore

### 5. الرئيسية/استكشف (`app/(tabs)/index.tsx`)
**للمستخدمين:**
- بانر الفعالية المميزة (carousel)
- قائمة الفعاليات القادمة (EventCard)
- زر تصفية الموقع

**للبائعين:**
- إحصائيات سريعة (التسجيلات المعلقة، الفعاليات القادمة)
- الفعاليات المتاحة للتسجيل
- زر: "تصفح جميع الفعاليات"

### 6. قائمة الفعاليات (`app/(tabs)/events.tsx`)
- تبديل التقويم (عرض القائمة/التقويم)
- شرائح التصفية: نطاق التاريخ، الموقع
- السحب للتحديث
- تمرير لا نهائي مع ترقيم الصفحات

### 7. تفاصيل الفعالية (`app/event/[id].tsx`)
**مشترك:**
- صورة البطل (الملصق)
- العنوان، الوصف
- عرض التاريخ/الوقت
- موقع الخريطة
- زر "عرض المشاركين"

**للبائعين فقط:**
- مؤشر السعة (X/Y مكان)
- عرض الرسوم
- قائمة المتطلبات
- قائمة المرافق
- زر "تسجيل" (إذا لم يكن مسجلاً)
- حالة التسجيل (إذا كان مسجلاً)

### 8. تسجيلاتي (`app/(tabs)/registrations.tsx`) - البائعين فقط
- علامات تبويب: معلقة | مقبولة | سابقة
- بطاقات RegistrationStatus
- إلغاء التسجيل المعلق

### 9. الملف الشخصي (`app/(tabs)/profile.tsx`)
**المستخدم:**
- الصورة الرمزية، الاسم، البريد الإلكتروني
- زر تعديل الملف الشخصي
- تفضيلات الإشعارات
- تسجيل الخروج

**البائع:**
- بطاقة ملف الشاحنة
- تعديل ملف الشاحنة
- رابط عرض الملف العام
- تسجيل الخروج

---

## هيكل خدمات Firebase

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

## استراتيجية معالجة الأخطاء

### حدود الخطأ (Error Boundary)
- تغليف التخطيط الرئيسي
- عرض شاشة خطأ ودودة مع زر إعادة المحاولة

### أخطاء النماذج
- react-hook-form + zod للتحقق
- عرض الأخطاء المضمنة أسفل الحقول

### أخطاء API
- إشعارات Toast (react-native-toast-message)
- زر إعادة المحاولة للطلبات الفاشلة
- كشف عدم الاتصال مع قائمة إعادة المحاولة

### حالات التحميل
- هياكل التحميل للقوائم
- دوائر تحميل الأزرار
- مؤشر السحب للتحديث

---

## مهام التطوير (مرتبة حسب الاعتمادية)

### المرحلة 1: الإعداد (اليوم 1)
- [ ] `npx create-expo-app mobile --template tabs`
- [ ] تثبيت التبعيات: `zustand react-hook-form zod react-native-maps expo-image-picker`
- [ ] تكوين Firebase (إنشاء المشروع، إضافة التكوين)
- [ ] إعداد هيكل المشروع (إنشاء المجلدات)
- [ ] إنشاء الثوابت: `colors.ts`, `theme.ts`
- [ ] إعداد ESLint + Prettier

### المرحلة 2: الأساسيات (اليوم 2-3)
- [ ] بناء مكونات الواجهة: `Button`, `Input`, `Card`, `Badge`, `Skeleton`
- [ ] إنشاء خدمة `firebase.ts`
- [ ] إنشاء `authStore` مع Zustand
- [ ] تنفيذ خطاف `useAuth`
- [ ] إعداد تخطيطات expo-router

### المرحلة 3: المصادقة (اليوم 4-5)
- [ ] بناء شاشة البداية مع التحقق من المصادقة
- [ ] بناء شاشة تسجيل الدخول
- [ ] بناء شاشة التسجيل
- [ ] بناء شاشة اختيار الدور
- [ ] الاتصال بـ Firebase Auth
- [ ] اختبار تدفق المصادقة من البداية للنهاية

### المرحلة 4: ميزة الفعاليات (اليوم 6-8)
- [ ] إنشاء `types/event.ts`
- [ ] بناء خدمة `events.ts`
- [ ] بناء مكون `EventCard`
- [ ] بناء مكون `MapPreview`
- [ ] بناء الشاشة الرئيسية (عرض المستخدم)
- [ ] بناء شاشة قائمة الفعاليات
- [ ] بناء شاشة تفاصيل الفعالية
- [ ] تنفيذ السحب للتحديث

### المرحلة 5: ميزات البائعين (اليوم 9-11)
- [ ] إنشاء `types/registration.ts`
- [ ] بناء خدمة `registrations.ts`
- [ ] بناء مكون `RegistrationStatus`
- [ ] بناء لوحة تحكم البائع الرئيسية
- [ ] بناء شاشة تسجيلاتي
- [ ] تنفيذ تقديم التسجيل
- [ ] بناء إدارة ملف الشاحنة

### المرحلة 6: التحسين (اليوم 12-14)
- [ ] إضافة هياكل التحميل في كل مكان
- [ ] إضافة حدود الأخطاء
- [ ] إضافة إشعارات Toast
- [ ] تنفيذ مؤشر عدم الاتصال
- [ ] إضافة الرسوم المتحركة (انتقالات التخطيط)
- [ ] اختبار جميع التدفقات
- [ ] إصلاح الأخطاء

### المرحلة 7: التحضير للنشر (اليوم 15)
- [ ] تكوين EAS Build
- [ ] إضافة أيقونات التطبيق وشاشة البداية
- [ ] بناء APK/IPA اختباري
- [ ] الاختبار الداخلي
- [ ] الإرسال إلى المتاجر

---

## التبعيات الأساسية

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

## متغيرات البيئة

إنشاء `env.js` (غير ملتزم):
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

## قائمة الاختبار

### المصادقة
- [ ] تسجيل الدخول ببيانات صحيحة
- [ ] تسجيل الدخول ببيانات خاطئة (ظهور الخطأ)
- [ ] تسجيل مستخدم جديد
- [ ] تدفق إعادة تعيين كلمة المرور
- [ ] اختيار الدور يستمر
- [ ] تسجيل الخروج يمسح الجلسة

### الفعاليات
- [ ] تحميل قائمة الفعاليات
- [ ] السحب للتحديث يعمل
- [ ] عرض تفاصيل الفعالية بشكل صحيح
- [ ] الخريطة تظهر الموقع الصحيح
- [ ] التصفية تعمل

### التسجيلات (البائع)
- [ ] يمكن تقديم التسجيل
- [ ] تحديثات الحالة تظهر
- [ ] لا يمكن التسجيل مرتين لنفس الفعالية
- [ ] يمكن إلغاء التسجيل المعلق

### الملف الشخصي
- [ ] عرض بيانات الملف الشخصي
- [ ] يمكن تعديل الملف الشخصي
- [ ] تحديث ملف الشاحنة (البائع)
- [ ] تسجيل الخروج يعمل
