# خطة الموقع - لوحة تحكم foodTracksBH للإدارة (خطة تطوير متقدمة)

## نظرة عامة
لوحة تحكم إدارية بـ Next.js لإدارة الفعاليات، تسجيلات البائعين، والمستخدمين. مبنية بـ App Router، Tailwind CSS، و Firebase.

---

## التقنيات والقرارات الأساسية

| القرار | الاختيار | السبب |
|--------|---------|-------|
| الإطار الأساسي | Next.js 14+ (App Router) | مكونات الخادم، البث |
| التنسيق | Tailwind CSS + shadcn/ui | تطوير سريع للواجهة |
| النماذج | react-hook-form + zod | تحقق آمن من الأنواع |
| إدارة الحالة | Zustand (العميل) + Server State | بدون تعقيدات |
| المصادقة | Firebase Auth + Middleware | حماية مسارات الإدارة |
| الجداول | TanStack Table | ترتيب، تصفية، ترقيم صفحات |
| التواريخ | date-fns | معالجة خفيفة للتواريخ |
| الإشعارات | sonner | إشعارات Toast |

---

## هيكل المشروع

```
website/
├── src/
│   ├── app/
│   │   ├── (auth)/
│   │   │   ├── login/
│   │   │   │   └── page.tsx
│   │   │   └── layout.tsx
│   │   ├── (dashboard)/
│   │   │   ├── page.tsx              # النظرة العامة
│   │   │   ├── events/
│   │   │   │   ├── page.tsx          # قائمة الفعاليات
│   │   │   │   ├── new/
│   │   │   │   │   └── page.tsx      # إنشاء فعالية
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx      # تعديل فعالية
│   │   │   ├── registrations/
│   │   │   │   └── page.tsx          # إدارة التسجيلات
│   │   │   ├── users/
│   │   │   │   ├── page.tsx          # قائمة المستخدمين
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx      # تفاصيل المستخدم
│   │   │   └── layout.tsx
│   │   ├── api/
│   │   │   ├── events/
│   │   │   │   ├── route.ts          # GET, POST
│   │   │   │   └── [id]/
│   │   │   │       └── route.ts      # GET, PUT, DELETE
│   │   │   ├── registrations/
│   │   │   │   ├── route.ts          # GET
│   │   │   │   └── [id]/
│   │   │   │       └── route.ts      # PUT (قبول/رفض)
│   │   │   └── stats/
│   │   │       └── route.ts          # إحصائيات لوحة التحكم
│   │   ├── layout.tsx
│   │   └── globals.css
│   ├── components/
│   │   ├── ui/                       # مكونات shadcn
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
│   │       ├── DataTable.tsx         # غلاف جدول عام
│   │       ├── PageHeader.tsx
│   │       ├── EmptyState.tsx
│   │       ├── LoadingSkeleton.tsx
│   │       └── ConfirmDialog.tsx
│   ├── lib/
│   │   ├── firebase.ts               # عميل Firebase
│   │   ├── firebase-admin.ts         # Admin SDK
│   │   ├── auth.ts                   # أدوات المصادقة
│   │   ├── utils.ts                  # دوال مساعدة
│   │   └── validations.ts            # مخططات Zod
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
├── middleware.ts                     # حماية المصادقة
├── tailwind.config.ts
├── next.config.js
└── package.json
```

---

## المكونات الأساسية

### مكونات التخطيط

**الشريط الجانبي (Sidebar)**
- الشعار
- روابط التنقل: لوحة التحكم، الفعاليات، التسجيلات، المستخدمين
- مؤشر الحالة النشطة (لمحة حمراء)
- زر تسجيل الخروج في الأسفل
- قابل للطي على الموبايل

**الترويسة (Header)**
- عنوان الصفحة
- مسار التنقل
- قائمة منسدلة للصورة الرمزية

**تنقل الموبايل (MobileNav)**
- قائمة همبرغر
- ورقة مع التنقل

### جداول البيانات

**DataTable** (عام)
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
- الأعمدة: الصورة، العنوان، التاريخ، الموقع، السعة، الحالة، الإجراءات
- الإجراءات: عرض، تعديل، حذف
- التصفية حسب الحالة (قادمة، جارية، منتهية)

**RegistrationsTable**
- الأعمدة: البائع، الفعالية، اسم الشاحنة، تاريخ الإرسال، الحالة، الإجراءات
- الإجراءات: قبول، رفض، عرض التفاصيل
- التصفية حسب الحالة (معلقة، مقبولة، مرفوضة)

**UsersTable**
- الأعمدة: الصورة، الاسم، البريد الإلكتروني، الدور، تاريخ الانضمام، الإجراءات
- الإجراءات: عرض الملف الشخصي
- التصفية حسب الدور (مستخدم، بائع)

### مكونات النماذج

**EventForm**
```typescript
interface EventFormProps {
  initialData?: Event;  // لوضع التعديل
  onSubmit: (data: EventFormData) => Promise<void>;
  isSubmitting?: boolean;
}

// الحقول:
// - title (نص، مطلوب)
// - description (منطقة نص)
// - posterImage (رفع ملف)
// - startDate, endDate (منتقيات تاريخ)
// - location (الإكمال التلقائي لـ Google Places + خريطة)
// - capacity (رقم)
// - fees (رقم)
// - requirements (إدخال نص متعدد)
// - amenities (تحديد متعدد: كهرباء، ماء، واي فاي)
```

**LocationPicker**
- حقل الإكمال التلقائي لـ Google Places
- معاينة الخريطة مع علامة قابلة للسحب
- يخزن: الاسم، العنوان، خط العرض، خط الطول

### البطاقات

**StatCard**
```typescript
interface StatCardProps {
  title: string;
  value: number | string;
  change?: number;  // نسبة التغيير
  icon: ReactNode;
  trend?: 'up' | 'down' | 'neutral';
}
```

---

## نماذج البيانات (TypeScript)

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

## مخططات التحقق (Zod)

```typescript
// lib/validations.ts
import { z } from 'zod';

export const eventSchema = z.object({
  title: z.string().min(3, 'يجب أن يكون العنوان 3 أحرف على الأقل'),
  description: z.string().min(10, 'يجب أن يكون الوصف 10 أحرف على الأقل'),
  posterImage: z.string().url('يجب أن يكون رابط صالح'),
  startDate: z.date(),
  endDate: z.date(),
  location: z.object({
    name: z.string(),
    address: z.string(),
    latitude: z.number(),
    longitude: z.number(),
  }),
  capacity: z.number().min(1, 'يجب أن تكون السعة 1 على الأقل'),
  fees: z.number().min(0, 'لا يمكن أن تكون الرسوم سالبة'),
  requirements: z.array(z.string()),
  amenities: z.array(z.string()),
}).refine(data => data.endDate > data.startDate, {
  message: 'يجب أن يكون تاريخ الانتهاء بعد تاريخ البدء',
  path: ['endDate'],
});

export const loginSchema = z.object({
  email: z.string().email('عنوان بريد إلكتروني غير صالح'),
  password: z.string().min(6, 'يجب أن تكون كلمة المرور 6 أحرف على الأقل'),
});
```

---

## مواصفات الصفحات

### 1. تسجيل الدخول (`/login`)
- نموذج البريد الإلكتروني/كلمة المرور
- خانة "تذكرني"
- إشعار خطأ عند فشل تسجيل الدخول
- التحويل إلى لوحة التحكم عند النجاح

### 2. نظرة عامة على لوحة التحكم (`/`)
**بطاقات الإحصائيات:**
- إجمالي الفعاليات النشطة
- إجمالي البائعين المسجلين
- التسجيلات المعلقة (قابلة للنقر، تربط بالتسجيلات)
- فعاليات هذا الشهر

**النشاط الأخير:**
- آخر 5 طلبات تسجيل
- إجراءات سريعة (قبول/رفض مضمن)

**إجراءات سريعة:**
- زر "إنشاء فعالية"
- زر "عرض التسجيلات المعلقة"

### 3. قائمة الفعاليات (`/events`)
- البحث بالعنوان
- تصفية الحالة (قادمة، جارية، منتهية)
- EventsTable
- زر "إنشاء فعالية" (أعلى اليمين)
- ترقيم الصفحات (10 في الصفحة)

### 4. إنشاء فعالية (`/events/new`)
- EventForm
- رفع الصورة مع معاينة
- منتقي نطاق التاريخ
- منتقي الموقع مع الخريطة
- المتطلبات كقائمة ديناميكية
- أزرار حفظ كمسودة / نشر
- رابط إلغاء

### 5. تعديل فعالية (`/events/[id]`)
- نفس الإنشاء، مملوء مسبقاً
- زر "حذف الفعالية" (مع تأكيد)
- يعرض عدد التسجيلات الحالية

### 6. التسجيلات (`/registrations`)
- علامات تبويب تصفية الحالة (الكل، معلقة، مقبولة، مرفوضة)
- البحث باسم البائع/الشاحنة
- RegistrationsTable
- أزرار قبول/رفض (المعلقة فقط)
- النقر على الصف لتوسيع التفاصيل

### 7. المستخدمون (`/users`)
- تصفية الدور (الكل، المستخدمون، البائعون)
- البحث بالاسم/البريد الإلكتروني
- UsersTable
- النقر لعرض الملف الشخصي

### 8. تفاصيل المستخدم (`/users/[id]`)
- بطاقة الملف الشخصي مع الصورة
- معلومات الاتصال
- للبائعين: تفاصيل الشاحنة، سجل التسجيلات

---

## مسارات API

### `/api/events`
```
GET    /api/events           → Event[] (مع ترقيم الصفحات)
POST   /api/events           → إنشاء فعالية
GET    /api/events/[id]      → Event
PUT    /api/events/[id]      → تحديث فعالية
DELETE /api/events/[id]      → حذف فعالية
```

### `/api/registrations`
```
GET  /api/registrations           → Registration[] (مع تصفيات)
PUT  /api/registrations/[id]      → تحديث الحالة (قبول/رفض)
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

## الوسيطة والمصادقة

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

## إعداد Firebase Admin

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

## مهام التطوير (مرتبة حسب الاعتمادية)

### المرحلة 1: الإعداد (اليوم 1)
- [ ] `npx create-next-app@latest website --typescript --tailwind --app`
- [ ] تثبيت: `zustand firebase react-hook-form zod @hookform/resolvers date-fns sonner @tanstack/react-table`
- [ ] تهيئة shadcn/ui: `npx shadcn-ui@latest init`
- [ ] إضافة مكونات shadcn: button, input, card, badge, dialog, table, tabs
- [ ] إعداد مشروع Firebase
- [ ] إنشاء `lib/firebase.ts` و `lib/firebase-admin.ts`
- [ ] تكوين متغيرات البيئة

### المرحلة 2: المصادقة والتخطيط (اليوم 2)
- [ ] إنشاء الوسيطة لحماية المسارات
- [ ] بناء صفحة تسجيل الدخول مع النموذج
- [ ] تنفيذ Firebase Auth (بريد إلكتروني/كلمة مرور)
- [ ] بناء مكون الشريط الجانبي
- [ ] بناء مكون الترويسة
- [ ] إنشاء تخطيط لوحة التحكم
- [ ] إضافة وظيفة تسجيل الخروج

### المرحلة 3: نظرة عامة لوحة التحكم (اليوم 3)
- [ ] إنشاء مكون `StatCard`
- [ ] بناء مسار `/api/stats`
- [ ] إنشاء صفحة لوحة التحكم مع الإحصائيات
- [ ] إضافة عنصر واجهة التسجيلات الأخيرة
- [ ] إضافة قسم الإجراءات السريعة

### المرحلة 4: إدارة الفعاليات (اليوم 4-6)
- [ ] إنشاء أنواع الفعاليات ومخطط التحقق
- [ ] بناء مكون `EventsTable`
- [ ] إنشاء صفحة قائمة الفعاليات
- [ ] بناء مكون `EventForm`
- [ ] إنشاء صفحة "فعالية جديدة"
- [ ] بناء `LocationPicker` مع Google Places
- [ ] تنفيذ رفع الصور (Firebase Storage)
- [ ] إنشاء صفحة "تعديل فعالية"
- [ ] إضافة وظيفة الحذف مع التأكيد

### المرحلة 5: إدارة التسجيلات (اليوم 7)
- [ ] إنشاء أنواع التسجيل
- [ ] بناء مكون `RegistrationsTable`
- [ ] إنشاء صفحة قائمة التسجيلات
- [ ] تنفيذ علامات تبويب تصفية الحالة
- [ ] إضافة إجراءات قبول/رفض
- [ ] إنشاء مكون `ConfirmDialog`
- [ ] إضافة إشعارات Toast للإجراءات

### المرحلة 6: دليل المستخدمين (اليوم 8)
- [ ] بناء مكون `UsersTable`
- [ ] إنشاء صفحة قائمة المستخدمين
- [ ] تنفيذ تصفية الدور
- [ ] إنشاء صفحة تفاصيل المستخدم
- [ ] عرض تفاصيل البائع والسجل

### المرحلة 7: التحسين (اليوم 9-10)
- [ ] إضافة هياكل التحميل
- [ ] إضافة حالات فارغة
- [ ] تنفيذ حدود الأخطاء
- [ ] إضافة إشعارات Toast في كل مكان
- [ ] اختبار التصميم المتجاوب
- [ ] إضافة اختصارات لوحة المفاتيح
- [ ] تحسين الصور

### المرحلة 8: النشر (اليوم 11)
- [ ] إعداد مشروع Vercel
- [ ] تكوين متغيرات البيئة
- [ ] اختبار بناء الإنتاج
- [ ] النشر

---

## التبعيات الأساسية

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

## متغيرات البيئة

```env
# عميل Firebase
NEXT_PUBLIC_FIREBASE_API_KEY=
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=
NEXT_PUBLIC_FIREBASE_PROJECT_ID=
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=
NEXT_PUBLIC_FIREBASE_APP_ID=

# Firebase Admin (الخادم فقط)
FIREBASE_PROJECT_ID=
FIREBASE_CLIENT_EMAIL=
FIREBASE_PRIVATE_KEY=

# Google Places (لمنتقي الموقع)
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=
```

---

## إرشادات واجهة/تجربة المستخدم

### الألوان (تكوين Tailwind)
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

### شارات الحالة
- **معلقة**: شارة صفراء (`bg-yellow-100 text-yellow-800`)
- **مقبولة**: شارة خضراء (`bg-green-100 text-green-800`)
- **مرفوضة**: شارة حمراء (`bg-red-100 text-red-800`)

### حالات التحميل
- الهيكل: مستطيلات رمادية نابضة
- الزر: دوار + نص "جاري التحميل..."
- الصفحة: هيكل صفحة كاملة

### حالات الخطأ
- النموذج: حدود حمراء + رسالة خطأ أسفل الحقل
- API: إشعار Toast مع رسالة الخطأ
- 404: رسم توضيحي ودود + زر "العودة"

---

## قائمة الاختبار

### المصادقة
- [ ] تسجيل الدخول ببيانات صحيحة
- [ ] تسجيل الدخول ببيانات خاطئة (ظهور الخطأ)
- [ ] المسارات المحمية تحول إلى تسجيل الدخول
- [ ] تسجيل الخروج يمسح الجلسة
- [ ] الجلسة تستمر عند التحديث

### الفعاليات
- [ ] تحميل قائمة الفعاليات مع ترقيم الصفحات
- [ ] البحث يصفي الفعاليات
- [ ] إنشاء فعالية ببيانات صحيحة
- [ ] التحقق من النموذج يظهر الأخطاء
- [ ] تعديل الفعالية يحدث البيانات
- [ ] حذف الفعالية مع التأكيد
- [ ] رفع الصور يعمل

### التسجيلات
- [ ] القائمة تظهر جميع التسجيلات
- [ ] تصفية الحالة تعمل
- [ ] القبول يحدث الحالة
- [ ] الرفض يحدث الحالة
- [ ] Toast يظهر عند الإجراء

### المستخدمون
- [ ] تحميل قائمة المستخدمين
- [ ] تصفية الدور تعمل
- [ ] صفحة تفاصيل المستخدم تظهر البيانات
- [ ] تفاصيل البائع تظهر معلومات الشاحنة
