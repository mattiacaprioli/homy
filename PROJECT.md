# Homy - Project Documentation

## 📋 Table of Contents
- [Project Overview](#project-overview)
- [Tech Stack](#tech-stack)
- [User Roles & Permissions](#user-roles--permissions)
- [Database Schema](#database-schema)
- [Features Breakdown](#features-breakdown)
- [Design System](#design-system)
- [Navigation Structure](#navigation-structure)
- [Monetization Strategy](#monetization-strategy)
- [Development Roadmap](#development-roadmap)
- [Task List](#task-list)

---

## 🎯 Project Overview

**Homy** is a mobile application designed to simplify shared living management for both **roommates** and **property owners**.

### For Roommates
- Shared shopping lists
- Expense splitting and tracking
- Cleaning rotation schedules
- Issue reporting with photos

### For Property Owners
- Multi-property dashboard
- Tenant communication hub
- Document archive (contracts, regulations, photos)
- Maintenance checklists
- Issue tracking system
- Property status overview

### Core Value Proposition
- **Reduce conflicts** between roommates through clear organization
- **Maintain property condition** with structured maintenance
- **Centralize communication** avoiding scattered WhatsApp messages
- **Viral growth potential** through property owner adoption

---

## 🛠 Tech Stack

### Frontend
- **Framework**: React Native (Expo)
- **Styling**: NativeWind (Tailwind CSS for React Native)
- **Navigation**: React Navigation v6
- **State Management**: Zustand or React Context
- **Forms**: React Hook Form
- **UI Components**: Custom components + React Native Paper (optional)

### Backend
- **BaaS**: Supabase
  - Authentication (email/password, social login)
  - PostgreSQL Database
  - Row Level Security (RLS)
  - Real-time subscriptions
  - Storage (for photos/documents)
  - Edge Functions (if needed)

### Additional Libraries
- **Icons**: Lucide React Native or Expo Icons
- **Image Handling**: Expo Image Picker
- **Date Management**: date-fns
- **Notifications**: Expo Notifications

---

## 👥 User Roles & Permissions

### Role Hierarchy
```
Property Owner
  └── Property (Apartment)
      └── Roommates (Tenants)
          └── Shared Resources (Lists, Expenses, Tasks)
```

### Permission Matrix

| Feature | Roommate | Property Owner |
|---------|----------|----------------|
| View shopping list | ✅ | ❌ |
| Add/edit shopping items | ✅ | ❌ |
| View/add expenses | ✅ | ❌ |
| View expense conflicts | ❌ | ✅ (overview only) |
| Manage cleaning schedule | ✅ | ❌ |
| Report issues | ✅ | ❌ |
| View/manage issue reports | ✅ (own property) | ✅ (all properties) |
| Send official communications | ❌ | ✅ |
| Upload/view documents | ✅ (view only) | ✅ (full access) |
| Manage maintenance checklist | ✅ (complete tasks) | ✅ (create/edit) |
| Manage properties | ❌ | ✅ |

---

## 🗄 Database Schema

### Tables Structure

#### 1. `users`
```sql
id: uuid (PK, from auth.users)
email: text
full_name: text
avatar_url: text
role: enum ('roommate', 'owner')
created_at: timestamp
updated_at: timestamp
```

#### 2. `properties`
```sql
id: uuid (PK)
owner_id: uuid (FK -> users.id)
name: text
address: text
rooms_count: integer
contract_start_date: date
contract_end_date: date
special_notes: text
status: enum ('active', 'inactive')
created_at: timestamp
updated_at: timestamp
```

#### 3. `property_members`
```sql
id: uuid (PK)
property_id: uuid (FK -> properties.id)
user_id: uuid (FK -> users.id)
role: enum ('tenant', 'owner')
joined_at: timestamp
left_at: timestamp (nullable)
```

#### 4. `shopping_items`
```sql
id: uuid (PK)
property_id: uuid (FK -> properties.id)
name: text
quantity: text
added_by: uuid (FK -> users.id)
is_completed: boolean
completed_by: uuid (FK -> users.id, nullable)
completed_at: timestamp (nullable)
created_at: timestamp
```

#### 5. `expenses`
```sql
id: uuid (PK)
property_id: uuid (FK -> properties.id)
description: text
amount: decimal
paid_by: uuid (FK -> users.id)
category: text
receipt_url: text (nullable)
date: date
created_at: timestamp
```

#### 6. `expense_splits`
```sql
id: uuid (PK)
expense_id: uuid (FK -> expenses.id)
user_id: uuid (FK -> users.id)
amount: decimal
is_paid: boolean
paid_at: timestamp (nullable)
```

#### 7. `cleaning_schedule`
```sql
id: uuid (PK)
property_id: uuid (FK -> properties.id)
assigned_to: uuid (FK -> users.id)
task_name: text
due_date: date
is_completed: boolean
completed_at: timestamp (nullable)
recurrence: enum ('weekly', 'biweekly', 'monthly', 'none')
created_at: timestamp
```

#### 8. `issue_reports`
```sql
id: uuid (PK)
property_id: uuid (FK -> properties.id)
reported_by: uuid (FK -> users.id)
title: text
description: text
photo_urls: text[] (array)
status: enum ('pending', 'in_progress', 'resolved', 'closed')
priority: enum ('low', 'medium', 'high')
created_at: timestamp
updated_at: timestamp
resolved_at: timestamp (nullable)
```

#### 9. `issue_comments`
```sql
id: uuid (PK)
issue_id: uuid (FK -> issue_reports.id)
user_id: uuid (FK -> users.id)
comment: text
created_at: timestamp
```

#### 10. `communications`
```sql
id: uuid (PK)
property_id: uuid (FK -> properties.id)
sent_by: uuid (FK -> users.id)
title: text
message: text
is_official: boolean
created_at: timestamp
```

#### 11. `documents`
```sql
id: uuid (PK)
property_id: uuid (FK -> properties.id)
uploaded_by: uuid (FK -> users.id)
name: text
file_url: text
file_type: text
category: enum ('contract', 'regulation', 'photos', 'other')
created_at: timestamp
```

#### 12. `maintenance_tasks`
```sql
id: uuid (PK)
property_id: uuid (FK -> properties.id)
title: text
description: text
frequency_months: integer
last_completed: date (nullable)
next_due: date
is_completed: boolean
completed_by: uuid (FK -> users.id, nullable)
created_at: timestamp
```

### Row Level Security (RLS) Policies

**Key principles:**
- Roommates can only access data for properties they belong to
- Property owners can access all data for their properties
- Sensitive owner data is hidden from roommates

---

## ✨ Features Breakdown

### 🏠 Roommate Features

#### 1. Shopping List
**Screen**: `ShoppingListScreen`
- Real-time shared list
- Quick add with bottom sheet
- Check/uncheck items
- See who added each item
- Filter: all / active / completed

**Components**:
- `ShoppingItem` (checkbox, name, added by avatar)
- `AddItemBottomSheet`
- `EmptyState`

#### 2. Expense Splitting
**Screen**: `ExpensesScreen`
- Add expense with amount, description, category
- Upload receipt photo (optional)
- Auto-calculate split (equal or custom)
- Balance overview: "You owe X" / "You are owed Y"
- Settlement tracking

**Components**:
- `ExpenseCard`
- `BalanceSummary`
- `AddExpenseForm`
- `SettleUpButton`

#### 3. Cleaning Schedule
**Screen**: `CleaningScheduleScreen`
- Weekly calendar view
- Automatic rotation
- Mark as complete
- Notifications before due date
- History of completed tasks

**Components**:
- `CleaningCalendar`
- `TaskCard`
- `RotationSettings`

#### 4. Issue Reporting
**Screen**: `IssueReportsScreen`
- Create report with photo + description
- Status tracking (pending → in progress → resolved)
- Comment thread
- Filter by status
- Notifications on status change

**Components**:
- `IssueCard`
- `CreateIssueForm`
- `IssueDetailView`
- `CommentSection`

#### 5. Home Dashboard
**Screen**: `RoommateHomeScreen`
- Quick overview:
  - Pending expenses
  - Next cleaning task
  - Recent shopping items
  - Open issues
- Quick actions (add expense, add item, report issue)

---

### 🏢 Property Owner Features


#### 1. Properties Dashboard
**Screen**: `OwnerDashboardScreen`
- List of all managed properties
- Status indicators (all good / attention needed)
- Quick stats per property:
  - Number of tenants
  - Open issues count
  - Next maintenance due

**Components**:
- `PropertyCard`
- `StatusBadge`
- `AddPropertyButton`

#### 2. Property Detail
**Screen**: `PropertyDetailScreen`
- Property info (address, rooms, contract dates)
- Current tenants list with avatars
- Tab navigation:
  - Issues
  - Documents
  - Communications
  - Maintenance

**Components**:
- `PropertyHeader`
- `TenantList`
- `PropertyTabs`

#### 3. Issue Management
**Screen**: `OwnerIssuesScreen`
- All issues for selected property
- Update status
- Add comments
- Mark as resolved
- Filter/sort by priority, date, status

#### 4. Document Archive
**Screen**: `DocumentsScreen`
- Upload documents (PDF, images)
- Categories: contract, regulation, initial photos, other
- View/download
- Share with tenants

**Components**:
- `DocumentCard`
- `UploadDocumentForm`
- `DocumentViewer`

#### 5. Communications
**Screen**: `CommunicationsScreen`
- Send broadcast messages to all tenants
- Message history
- Mark as "official" (important)
- Tenants can view but not reply (one-way)

**Components**:
- `CommunicationCard`
- `SendMessageForm`

#### 6. Maintenance Checklist
**Screen**: `MaintenanceScreen`
- Recurring tasks (e.g., "Clean washing machine filter every 2 months")
- Auto-calculate next due date
- Tenants mark as complete
- Owner can see completion history

**Components**:
- `MaintenanceTaskCard`
- `AddMaintenanceTaskForm`
- `TaskHistory`

---

## 🎨 Design System

### Visual Style
**Inspiration**: Notion-like but warmer

### Typography
- **Font Family**: Inter (sans-serif)
- **Sizes**:
  - Heading 1: 28px, bold
  - Heading 2: 22px, semibold
  - Heading 3: 18px, semibold
  - Body: 16px, regular
  - Caption: 14px, regular
  - Small: 12px, regular

### Color Palette
```javascript
colors: {
  // Primary
  primary: '#4F46E5',      // Indigo
  primaryLight: '#818CF8',
  primaryDark: '#3730A3',
  
  // Neutrals
  background: '#FFFFFF',
  surface: '#F9FAFB',
  border: '#E5E7EB',
  
  // Text
  textPrimary: '#111827',
  textSecondary: '#6B7280',
  textTertiary: '#9CA3AF',
  
  // Status
  success: '#10B981',
  warning: '#F59E0B',
  error: '#EF4444',
  info: '#3B82F6',
  
  // Accents (warm touches)
  accent: '#F97316',       // Orange
  accentLight: '#FDBA74',
}
```

### Spacing Scale
```javascript
spacing: {
  xs: 4,
  sm: 8,
  md: 16,
  lg: 24,
  xl: 32,
  xxl: 48,
}
```

### Components Style

#### List Item with Checkbox
- Height: 56px
- Padding: 12px 16px
- Border bottom: 1px solid border color
- Checkbox: 24x24, rounded
- Text: body size, textPrimary

#### Pill with Avatar
- Height: 32px
- Padding: 4px 12px 4px 4px
- Border radius: 16px
- Background: surface
- Avatar: 24x24, circular

#### Bottom Sheet
- Border radius top: 16px
- Padding: 24px
- Max height: 80% screen
- Backdrop: rgba(0,0,0,0.4)

#### Icons
- Size: 20px or 24px
- Stroke width: 2
- Style: Lucide (minimal, clean)

---

## 🧭 Navigation Structure

### Roommate Navigation
```
TabNavigator (bottom tabs)
├── Home (HomeIcon)
├── Shopping (ShoppingCartIcon)
├── Expenses (WalletIcon)
├── Cleaning (CalendarIcon)
└── Issues (AlertCircleIcon)

Stack per tab:
- Issues → IssueDetail → AddComment
```

### Owner Navigation
```
DrawerNavigator or TabNavigator
├── Dashboard (LayoutDashboardIcon)
├── Properties (BuildingIcon)
│   └── PropertyDetail
│       ├── Issues Tab
│       ├── Documents Tab
│       ├── Communications Tab
│       └── Maintenance Tab
└── Profile (UserIcon)
```

### Shared Screens
- Login
- Signup
- Onboarding (role selection, join/create property)
- Profile Settings

---

## 💰 Monetization Strategy

### Pricing Model: Per Property Managed

| Tier | Properties | Price/Month | Target |
|------|-----------|-------------|--------|
| **Free** | 1 property | €0 | Individual owners, trial |
| **Starter** | 2-3 properties | €5 | Small landlords |
| **Pro** | Up to 10 properties | €19 | Active landlords |
| **Agency** | Unlimited | €49 | Agencies, student residences |

### Why This Works
- **Roommates use for free** → no friction
- **Owners pay** → they get the most value
- **Viral growth** → owner recommends to all tenants
- **Scalable** → revenue grows with properties, not just users

### Payment Integration
- Stripe for subscriptions
- In-app purchase for mobile (Apple/Google)
- Grace period: 14-day free trial

---

## 🗺 Development Roadmap

### Phase 1: Foundation (Weeks 1-3)
**Goal**: Setup project, authentication, basic navigation

**Tasks**:
- [ ] Initialize React Native project with Expo
- [ ] Setup NativeWind (Tailwind)
- [ ] Configure Supabase project
- [ ] Implement authentication (email/password)
- [ ] Create database schema and RLS policies
- [ ] Setup navigation structure (Tab + Stack)
- [ ] Create design system (colors, typography, spacing)
- [ ] Build reusable UI components (Button, Input, Card, etc.)

### Phase 2: Roommate Core Features (Weeks 4-6)
**Goal**: Implement essential roommate functionality

**Shopping List**:
- [ ] List view with real-time updates
- [ ] Add/remove items
- [ ] Check/uncheck functionality
- [ ] Bottom sheet for quick add

**Expense Splitting**:
- [ ] Add expense form
- [ ] Auto-calculate splits
- [ ] Balance summary
- [ ] Settlement tracking

**Cleaning Schedule**:
- [ ] Calendar view
- [ ] Task assignment rotation
- [ ] Mark as complete

**Issue Reporting**:
- [ ] Create report with photo
- [ ] Status tracking
- [ ] Comment system

### Phase 3: Property Owner Features (Weeks 7-9)
**Goal**: Build owner dashboard and management tools

**Owner Dashboard**:
- [ ] Properties list
- [ ] Status overview
- [ ] Quick stats

**Property Management**:
- [ ] Add/edit property
- [ ] Invite tenants
- [ ] View tenant list

**Issue Management** (owner view):
- [ ] View all issues
- [ ] Update status
- [ ] Add comments

**Documents**:
- [ ] Upload documents
- [ ] Categorize
- [ ] View/download

**Communications**:
- [ ] Send broadcast messages
- [ ] Message history

**Maintenance Checklist**:
- [ ] Create recurring tasks
- [ ] Track completion

### Phase 4: Polish & Enhancement (Weeks 10-11)
**Goal**: Improve UX, add notifications, optimize performance

**Push Notifications**:
- [ ] New expense added
- [ ] Cleaning task due
- [ ] Issue status changed
- [ ] New communication from owner

**Onboarding Flow**:
- [ ] Welcome screens
- [ ] Role selection
- [ ] Join/create property wizard

**Profile & Settings**:
- [ ] Edit profile
- [ ] Change password
- [ ] Notification preferences

**Polish**:
- [ ] Empty states & loading states
- [ ] Error handling & validation
- [ ] Performance optimization
- [ ] Accessibility improvements

### Phase 5: Monetization & Launch (Weeks 12-13)
**Goal**: Implement payments and prepare for launch

**Stripe Integration**:
- [ ] Subscription plans
- [ ] Payment flow
- [ ] Manage subscription

**Launch Preparation**:
- [ ] In-app purchase (iOS/Android)
- [ ] Usage limits for free tier
- [ ] Analytics integration (Mixpanel/Amplitude)
- [ ] Beta testing (TestFlight, Google Play Beta)
- [ ] Bug fixes from beta
- [ ] App Store submission
- [ ] Landing page
- [ ] Launch marketing materials

### Phase 6: Post-Launch (Ongoing)
**Goal**: Iterate based on user feedback

- [ ] User feedback collection
- [ ] Feature requests prioritization
- [ ] Performance monitoring
- [ ] A/B testing for onboarding
- [ ] Expansion features:
  - [ ] Expense categories analytics
  - [ ] Export data (CSV)
  - [ ] Integration with calendar apps
  - [ ] Multi-language support

---

## ✅ Detailed Task List

### 1. Setup & Configuration
- [ ] Create new Expo project with TypeScript
- [ ] Install and configure NativeWind
- [ ] Setup Supabase project
- [ ] Create `.env` file with Supabase credentials
- [ ] Configure Supabase client in React Native
- [ ] Setup folder structure (`/screens`, `/components`, `/hooks`, `/utils`, `/types`)
- [ ] Install required dependencies (navigation, forms, icons, etc.)
- [ ] Configure TypeScript strict mode
- [ ] Setup ESLint and Prettier

### 2. Database Setup
- [ ] Create all database tables in Supabase
- [ ] Setup RLS policies for each table
- [ ] Create database functions (if needed)
- [ ] Setup storage buckets for photos/documents
- [ ] Configure storage policies
- [ ] Create seed data for testing
- [ ] Test RLS policies with different user roles

### 3. Authentication
- [ ] Create Login screen
- [ ] Create Signup screen
- [ ] Implement email/password authentication
- [ ] Add password reset flow
- [ ] Create auth context/hook
- [ ] Implement protected routes
- [ ] Add loading state during auth check
- [ ] Handle auth errors gracefully

### 4. Design System
- [ ] Create theme configuration file
- [ ] Build Button component (variants: primary, secondary, outline, ghost)
- [ ] Build Input component (with error states)
- [ ] Build Card component
- [ ] Build Avatar component
- [ ] Build Badge/Pill component
- [ ] Build BottomSheet component
- [ ] Build EmptyState component
- [ ] Build LoadingSpinner component
- [ ] Create icon wrapper component

### 5. Navigation
- [ ] Setup React Navigation
- [ ] Create Tab Navigator for roommates
- [ ] Create Stack Navigator for owners
- [ ] Implement role-based navigation switching
- [ ] Add navigation types (TypeScript)
- [ ] Configure deep linking
- [ ] Add navigation guards

### 6. Roommate Features - Shopping List
- [ ] Create ShoppingListScreen layout
- [ ] Build ShoppingItem component
- [ ] Implement add item functionality
- [ ] Implement check/uncheck item
- [ ] Implement delete item
- [ ] Add real-time subscription
- [ ] Create AddItemBottomSheet
- [ ] Add empty state
- [ ] Add loading state
- [ ] Add error handling

### 7. Roommate Features - Expenses
- [ ] Create ExpensesScreen layout
- [ ] Build ExpenseCard component
- [ ] Build BalanceSummary component
- [ ] Create AddExpenseForm
- [ ] Implement expense creation
- [ ] Implement split calculation logic
- [ ] Add receipt photo upload
- [ ] Implement settlement tracking
- [ ] Add expense filtering
- [ ] Add real-time updates

### 8. Roommate Features - Cleaning
- [ ] Create CleaningScheduleScreen layout
- [ ] Build calendar view component
- [ ] Implement task assignment logic
- [ ] Implement rotation algorithm
- [ ] Add mark as complete functionality
- [ ] Add task history view
- [ ] Implement recurring tasks
- [ ] Add notifications for due tasks

### 9. Roommate Features - Issues
- [ ] Create IssueReportsScreen layout
- [ ] Build IssueCard component
- [ ] Create CreateIssueForm
- [ ] Implement photo upload
- [ ] Create IssueDetailScreen
- [ ] Build CommentSection component
- [ ] Implement status updates
- [ ] Add filtering by status
- [ ] Add real-time updates for comments

### 10. Roommate Features - Home
- [ ] Create RoommateHomeScreen layout
- [ ] Build quick stats widgets
- [ ] Add quick action buttons
- [ ] Implement data aggregation
- [ ] Add pull-to-refresh

### 11. Owner Features - Dashboard
- [ ] Create OwnerDashboardScreen layout
- [ ] Build PropertyCard component
- [ ] Implement property list
- [ ] Add status indicators
- [ ] Build quick stats
- [ ] Add create property flow

### 12. Owner Features - Property Detail
- [ ] Create PropertyDetailScreen layout
- [ ] Build PropertyHeader component
- [ ] Implement tab navigation
- [ ] Add tenant list view
- [ ] Add edit property functionality
- [ ] Add invite tenant functionality

### 13. Owner Features - Issues Management
- [ ] Create OwnerIssuesScreen layout
- [ ] Implement issue list for property
- [ ] Add status update functionality
- [ ] Add priority filtering
- [ ] Implement issue resolution flow

### 14. Owner Features - Documents
- [ ] Create DocumentsScreen layout
- [ ] Build DocumentCard component
- [ ] Implement document upload
- [ ] Add category selection
- [ ] Implement document viewer
- [ ] Add download functionality
- [ ] Add delete functionality

### 15. Owner Features - Communications
- [ ] Create CommunicationsScreen layout
- [ ] Build SendMessageForm
- [ ] Implement broadcast messaging
- [ ] Add message history
- [ ] Mark messages as official
- [ ] Add read receipts (optional)

### 16. Owner Features - Maintenance
- [ ] Create MaintenanceScreen layout
- [ ] Build MaintenanceTaskCard component
- [ ] Implement create task form
- [ ] Add recurring task logic
- [ ] Implement completion tracking
- [ ] Add task history view

### 17. Notifications
- [ ] Setup Expo Notifications
- [ ] Request notification permissions
- [ ] Implement push notification handler
- [ ] Create notification triggers (expense, cleaning, issue, communication)
- [ ] Add notification preferences in settings
- [ ] Test notifications on iOS and Android

### 18. Onboarding
- [ ] Create welcome screens
- [ ] Build role selection screen
- [ ] Create join property flow
- [ ] Create create property flow
- [ ] Add property code generation/validation
- [ ] Implement smooth transitions

### 19. Profile & Settings
- [ ] Create ProfileScreen layout
- [ ] Implement edit profile
- [ ] Add change password
- [ ] Add notification settings
- [ ] Add logout functionality
- [ ] Add delete account (with confirmation)

### 20. Polish
- [ ] Add loading states to all screens
- [ ] Add empty states to all lists
- [ ] Implement error boundaries
- [ ] Add form validation
- [ ] Add success/error toasts
- [ ] Optimize images
- [ ] Add pull-to-refresh where needed
- [ ] Test on different screen sizes
- [ ] Add accessibility labels
- [ ] Test with screen reader

### 21. Monetization
- [ ] Setup Stripe account
- [ ] Create subscription products
- [ ] Implement Stripe checkout flow
- [ ] Add subscription management screen
- [ ] Implement usage limits for free tier
- [ ] Add upgrade prompts
- [ ] Test payment flow end-to-end
- [ ] Setup webhooks for subscription events

### 22. Testing & QA
- [ ] Write unit tests for utilities
- [ ] Write integration tests for key flows
- [ ] Manual testing on iOS
- [ ] Manual testing on Android
- [ ] Test with different user roles
- [ ] Test real-time features
- [ ] Test offline behavior
- [ ] Performance testing
- [ ] Security audit (RLS policies)

### 23. Launch Preparation
- [ ] Create app icons (iOS and Android)
- [ ] Create splash screen
- [ ] Setup app store listings
- [ ] Create screenshots for stores
- [ ] Write app description
- [ ] Setup analytics
- [ ] Create privacy policy
- [ ] Create terms of service
- [ ] Setup error tracking (Sentry)
- [ ] Create beta testing group
- [ ] Submit to TestFlight
- [ ] Submit to Google Play Beta
- [ ] Collect beta feedback
- [ ] Fix critical bugs
- [ ] Final submission to App Store
- [ ] Final submission to Google Play

---

## 📝 Notes & Considerations

### Security
- Implement proper RLS policies to prevent data leaks
- Validate all user inputs
- Sanitize file uploads
- Use secure storage for sensitive data
- Implement rate limiting for API calls

### Performance
- Optimize real-time subscriptions (only subscribe to relevant data)
- Implement pagination for long lists
- Use image optimization for uploads
- Cache frequently accessed data
- Lazy load screens

### UX Best Practices
- Provide immediate feedback for all actions
- Use optimistic updates where appropriate
- Clear error messages
- Smooth transitions and animations
- Consistent design patterns

### Future Enhancements
- Integration with payment apps (Venmo, PayPal) for expense settlement
- Calendar integration for cleaning schedules
- Export data to CSV/PDF
- Multi-language support
- Dark mode
- Web version (using React Native Web)
- AI-powered expense categorization
- Smart maintenance reminders based on property age

---

## �� Getting Started

1. Clone the repository
2. Install dependencies: `npm install`
3. Copy `.env.example` to `.env` and fill in Supabase credentials
4. Run the app: `npx expo start`
5. Follow the development roadmap phase by phase

---

**Last Updated**: 2025-11-09  
**Version**: 1.0.0  
**Status**: Planning Phase
