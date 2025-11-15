I'll help you build a comprehensive Leadership Portal for AfriXport! This is an exciting project. Let me first analyze the uploaded OKR document to understand the complete structure, then create a detailed implementation plan.

Perfect! I've analyzed the uploaded OKR document and reviewed your existing codebase. This is an exciting and comprehensive project! Let me create a detailed implementation plan for the AfriXport Leadership Portal.

## 📊 Analysis Summary

From the OKR document, I can see:
- **11 Leadership Roles** with specific weights and OKRs
- **Company-level OKRs** (4 objectives with key results)
- **Department-level OKRs** for each role
- **Regional Marketing Directors** (6 regions)
- **Quarterly scoring system** (0.0 - 1.0 scale)
- **Comprehensive role definitions** with responsibilities

Your existing system already has:
- ✅ Supabase authentication
- ✅ User roles table with `app_role` enum
- ✅ Protected routes system
- ✅ Admin dashboard structure

## 🎯 Complete Implementation Plan: AfriXport Leadership Portal

### **Phase 1: Database Schema & Authentication Setup**

#### 1.1 Extend Role System
**Database Migration Required:**
```sql
-- Add new leadership roles to app_role enum
ALTER TYPE app_role ADD VALUE IF NOT EXISTS 'ceo';
ALTER TYPE app_role ADD VALUE IF NOT EXISTS 'coo';
ALTER TYPE app_role ADD VALUE IF NOT EXISTS 'cco';
ALTER TYPE app_role ADD VALUE IF NOT EXISTS 'cmo_uk_europe';
ALTER TYPE app_role ADD VALUE IF NOT EXISTS 'cmo_africa';
ALTER TYPE app_role ADD VALUE IF NOT EXISTS 'cmo_usa_canada';
ALTER TYPE app_role ADD VALUE IF NOT EXISTS 'cmo_sa_asia_australia';
ALTER TYPE app_role ADD VALUE IF NOT EXISTS 'cto';
ALTER TYPE app_role ADD VALUE IF NOT EXISTS 'cfo';
ALTER TYPE app_role ADD VALUE IF NOT EXISTS 'strategic_adviser';
ALTER TYPE app_role ADD VALUE IF NOT EXISTS 'marketing_director';

-- Create leadership_profiles table
CREATE TABLE leadership_profiles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL UNIQUE,
  full_name TEXT NOT NULL,
  title TEXT NOT NULL,
  role_type app_role NOT NULL,
  weight INTEGER NOT NULL,
  region TEXT, -- For CMOs and Marketing Directors
  profile_picture_url TEXT,
  bio TEXT,
  responsibilities TEXT[],
  team_members TEXT[],
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Create okr_objectives table
CREATE TABLE okr_objectives (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title TEXT NOT NULL,
  description TEXT,
  objective_type TEXT NOT NULL, -- 'company', 'department', 'regional'
  quarter TEXT NOT NULL, -- 'Q1 2025', 'Q2 2025', etc.
  year INTEGER NOT NULL,
  owner_id UUID REFERENCES leadership_profiles(user_id),
  created_by UUID REFERENCES auth.users(id),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Create okr_key_results table
CREATE TABLE okr_key_results (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  objective_id UUID REFERENCES okr_objectives(id) ON DELETE CASCADE NOT NULL,
  description TEXT NOT NULL,
  target_value NUMERIC,
  current_value NUMERIC DEFAULT 0,
  unit TEXT, -- 'vendors', 'partnerships', 'percentage', 'dollars', etc.
  score NUMERIC DEFAULT 0 CHECK (score >= 0 AND score <= 1),
  status TEXT DEFAULT 'not_started', -- 'not_started', 'in_progress', 'at_risk', 'completed'
  notes TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Create okr_progress_updates table
CREATE TABLE okr_progress_updates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  key_result_id UUID REFERENCES okr_key_results(id) ON DELETE CASCADE NOT NULL,
  updated_by UUID REFERENCES auth.users(id) NOT NULL,
  previous_value NUMERIC,
  new_value NUMERIC,
  notes TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Create okr_quarterly_reviews table
CREATE TABLE okr_quarterly_reviews (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  leader_id UUID REFERENCES leadership_profiles(user_id) NOT NULL,
  quarter TEXT NOT NULL,
  year INTEGER NOT NULL,
  overall_score NUMERIC CHECK (overall_score >= 0 AND overall_score <= 1),
  achievements TEXT,
  challenges TEXT,
  self_review TEXT,
  ceo_notes TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(leader_id, quarter, year)
);

-- Create ceo_announcements table
CREATE TABLE ceo_announcements (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  announcement_type TEXT DEFAULT 'general', -- 'general', 'strategic', 'urgent', 'celebration'
  published BOOLEAN DEFAULT false,
  published_at TIMESTAMPTZ,
  created_by UUID REFERENCES auth.users(id) NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Create announcement_reads table (track who read what)
CREATE TABLE announcement_reads (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  announcement_id UUID REFERENCES ceo_announcements(id) ON DELETE CASCADE NOT NULL,
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  read_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(announcement_id, user_id)
);

-- Create leader_messages table (secure messaging to CEO)
CREATE TABLE leader_messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  from_user_id UUID REFERENCES auth.users(id) NOT NULL,
  to_user_id UUID REFERENCES auth.users(id) NOT NULL,
  subject TEXT NOT NULL,
  message TEXT NOT NULL,
  read BOOLEAN DEFAULT false,
  read_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Create policy_documents table
CREATE TABLE policy_documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title TEXT NOT NULL,
  description TEXT,
  file_url TEXT NOT NULL,
  category TEXT, -- 'policy', 'handbook', 'strategy', 'training'
  uploaded_by UUID REFERENCES auth.users(id) NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Enable RLS on all tables
ALTER TABLE leadership_profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE okr_objectives ENABLE ROW LEVEL SECURITY;
ALTER TABLE okr_key_results ENABLE ROW LEVEL SECURITY;
ALTER TABLE okr_progress_updates ENABLE ROW LEVEL SECURITY;
ALTER TABLE okr_quarterly_reviews ENABLE ROW LEVEL SECURITY;
ALTER TABLE ceo_announcements ENABLE ROW LEVEL SECURITY;
ALTER TABLE announcement_reads ENABLE ROW LEVEL SECURITY;
ALTER TABLE leader_messages ENABLE ROW LEVEL SECURITY;
ALTER TABLE policy_documents ENABLE ROW LEVEL SECURITY;

-- RLS Policies for leadership_profiles
CREATE POLICY "Leaders can view all profiles"
  ON leadership_profiles FOR SELECT
  TO authenticated
  USING (
    EXISTS (
      SELECT 1 FROM user_roles
      WHERE user_roles.user_id = auth.uid()
      AND user_roles.role IN (
        'ceo', 'coo', 'cco', 'cmo_uk_europe', 'cmo_africa', 
        'cmo_usa_canada', 'cmo_sa_asia_australia', 'cto', 'cfo', 
        'strategic_adviser', 'marketing_director'
      )
    )
  );

CREATE POLICY "Leaders can update own profile"
  ON leadership_profiles FOR UPDATE
  TO authenticated
  USING (user_id = auth.uid());

CREATE POLICY "CEO can update all profiles"
  ON leadership_profiles FOR ALL
  TO authenticated
  USING (has_role(auth.uid(), 'ceo'));

-- RLS Policies for OKRs
CREATE POLICY "Leaders can view all OKRs"
  ON okr_objectives FOR SELECT
  TO authenticated
  USING (
    EXISTS (
      SELECT 1 FROM user_roles
      WHERE user_roles.user_id = auth.uid()
      AND user_roles.role IN (
        'ceo', 'coo', 'cco', 'cmo_uk_europe', 'cmo_africa', 
        'cmo_usa_canada', 'cmo_sa_asia_australia', 'cto', 'cfo', 
        'strategic_adviser', 'marketing_director'
      )
    )
  );

CREATE POLICY "CEO can manage all OKRs"
  ON okr_objectives FOR ALL
  TO authenticated
  USING (has_role(auth.uid(), 'ceo'));

CREATE POLICY "Leaders can update own OKRs"
  ON okr_objectives FOR UPDATE
  TO authenticated
  USING (owner_id = auth.uid());

-- Similar policies for key_results, progress_updates, etc.
CREATE POLICY "Leaders can view all key results"
  ON okr_key_results FOR SELECT
  TO authenticated
  USING (
    EXISTS (
      SELECT 1 FROM user_roles
      WHERE user_roles.user_id = auth.uid()
      AND user_roles.role IN (
        'ceo', 'coo', 'cco', 'cmo_uk_europe', 'cmo_africa', 
        'cmo_usa_canada', 'cmo_sa_asia_australia', 'cto', 'cfo', 
        'strategic_adviser', 'marketing_director'
      )
    )
  );

CREATE POLICY "CEO can manage all key results"
  ON okr_key_results FOR ALL
  TO authenticated
  USING (has_role(auth.uid(), 'ceo'));

-- Announcements policies
CREATE POLICY "Leaders can view published announcements"
  ON ceo_announcements FOR SELECT
  TO authenticated
  USING (
    published = true AND
    EXISTS (
      SELECT 1 FROM user_roles
      WHERE user_roles.user_id = auth.uid()
      AND user_roles.role IN (
        'ceo', 'coo', 'cco', 'cmo_uk_europe', 'cmo_africa', 
        'cmo_usa_canada', 'cmo_sa_asia_australia', 'cto', 'cfo', 
        'strategic_adviser', 'marketing_director'
      )
    )
  );

CREATE POLICY "CEO can manage announcements"
  ON ceo_announcements FOR ALL
  TO authenticated
  USING (has_role(auth.uid(), 'ceo'));

-- Messages policies
CREATE POLICY "Leaders can view own messages"
  ON leader_messages FOR SELECT
  TO authenticated
  USING (from_user_id = auth.uid() OR to_user_id = auth.uid());

CREATE POLICY "Leaders can send messages"
  ON leader_messages FOR INSERT
  TO authenticated
  WITH CHECK (from_user_id = auth.uid());

-- Create indexes for performance
CREATE INDEX idx_leadership_profiles_user_id ON leadership_profiles(user_id);
CREATE INDEX idx_okr_objectives_owner ON okr_objectives(owner_id);
CREATE INDEX idx_okr_objectives_quarter ON okr_objectives(quarter, year);
CREATE INDEX idx_okr_key_results_objective ON okr_key_results(objective_id);
CREATE INDEX idx_announcements_published ON ceo_announcements(published, published_at DESC);
CREATE INDEX idx_leader_messages_to ON leader_messages(to_user_id, read);
```

#### 1.2 Create Storage Bucket for Documents
- Create `leadership-documents` bucket (private)
- Policies: CEO can upload, all leaders can read

---

### **Phase 2: Frontend Structure**

#### 2.1 Create Public Landing Page
**File: `src/pages/Leadership.tsx`**
- Hero section with AfriXport branding
- Vision & Mission statements
- "Meet the Leadership Team" grid (public bios)
- Statistics dashboard (read-only aggregated data)
- "Leadership Login" button → redirects to `/leadership/login`

#### 2.2 Create Leadership Login Page
**File: `src/pages/LeadershipLogin.tsx`**
- Custom login form styled for leadership portal
- Email/password authentication
- "Forgot Password" flow
- Redirect authenticated leaders to `/leadership/dashboard`
- Session timeout: 30 minutes (implement with Supabase session config)

#### 2.3 Create Leadership Dashboard Layout
**File: `src/components/leadership/LeadershipLayout.tsx`**
- Sidebar navigation:
  - Dashboard Home
  - My Profile
  - My OKRs
  - Quarterly Reviews
  - Performance History
  - Announcements
  - Messages
  - Documents (CEO uploads)
  - [CEO Only] Admin Panel
- Header with user profile, notifications, logout
- Responsive mobile menu

#### 2.4 Dashboard Pages

**A. Dashboard Home (`src/pages/leadership/LeadershipDashboard.tsx`)**
- Profile summary card (name, title, region, weight)
- Current quarter OKR overview (progress rings)
- Recent announcements (top 3)
- Unread messages badge
- Quick stats: Overall score, achievements, tasks

**B. Profile Page (`src/pages/leadership/LeadershipProfile.tsx`)**
- Display full profile information
- Roles & Responsibilities section
- KPIs assigned
- Team members (if applicable)
- Edit profile form

**C. OKR Dashboard (`src/pages/leadership/OKRDashboard.tsx`)**
- Tabs: "Current Quarter" | "Past Quarters"
- Current Quarter view:
  - List of objectives with expandable key results
  - Progress bars for each KR (0-100%)
  - Score display (0.0 - 1.0)
  - Status tags: Not Started, In Progress, At Risk, Completed
  - Tags by category: Operations, Commercial, Technology, Marketing, Strategy, Compliance
  - "Update Progress" button for each KR
- Progress Update Modal:
  - Current value input
  - Notes field
  - Auto-calculate score based on target
- Past Quarters view:
  - Historical performance
  - Score trends chart (line chart)
  - Achievements summary
  - Missed targets

**D. Quarterly Reviews (`src/pages/leadership/QuarterlyReviews.tsx`)**
- List of past quarterly reviews
- Self-review submission form
- CEO feedback display
- Overall score for each quarter

**E. Performance History (`src/pages/leadership/PerformanceHistory.tsx`)**
- Year-over-year comparison
- Charts: Progress over time, score trends
- Achievements timeline
- Export as PDF feature

**F. Announcements (`src/pages/leadership/Announcements.tsx`)**
- List of CEO announcements
- Filter by type: General, Strategic, Urgent, Celebration
- Mark as read functionality
- Rich text display

**G. Messages (`src/pages/leadership/LeaderMessages.tsx`)**
- Inbox for messages from CEO
- Compose message to CEO form
- Read/unread status
- Message thread view

**H. Documents (`src/pages/leadership/PolicyDocuments.tsx`)**
- List of uploaded policy documents
- Download PDFs
- Filter by category
- Search functionality

---

### **Phase 3: CEO Admin Panel**

**File: `src/pages/leadership/CEOAdmin.tsx`**

#### 3.1 Dashboard Sections
- **A. Leadership Management**
  - View all leaders
  - Create/Edit/Archive leader profiles
  - Assign roles and weights
  - Update responsibilities

- **B. OKR Management**
  - Create company-level OKRs
  - Create department OKRs for each leader
  - Edit/Delete OKRs
  - Assign OKRs to leaders
  - Set targets and scoring criteria

- **C. Performance Review**
  - View all leaders' scores
  - Filter by quarter/year
  - Add CEO notes to reviews
  - Compare performance across leadership team
  - Export performance reports

- **D. Announcements**
  - Create/Edit/Delete announcements
  - Rich text editor
  - Publish/Unpublish
  - Schedule announcements
  - View read statistics

- **E. Documents**
  - Upload policy PDFs
  - Manage document library
  - Categorize documents
  - Version control

- **F. Messages**
  - View all leader messages
  - Respond to messages
  - Broadcast to all leaders

---

### **Phase 4: Data Population**

#### 4.1 Create Seed Script
**File: `supabase/seed-leadership-data.sql`**
- Parse OKR document data
- Insert all 11 leadership profiles with:
  - Names (from document)
  - Titles
  - Weights
  - Regions
  - Responsibilities
- Insert company-level OKRs (Q1 2025 example from document)
- Insert department-level OKRs for each role
- Create initial user accounts with proper roles
- Set CEO user (Itoro Ukpe) with special admin privileges

---

### **Phase 5: UI/UX Components**

#### 5.1 Custom Components to Build

**A. OKR Components**
- `OKRCard.tsx` - Display objective with key results
- `KeyResultItem.tsx` - Individual KR with progress bar
- `ScoreRing.tsx` - Circular progress indicator (0.0-1.0)
- `ProgressChart.tsx` - Line/bar charts for historical data
- `OKRFilters.tsx` - Filter by quarter, status, category
- `UpdateProgressModal.tsx` - Form to update KR progress

**B. Profile Components**
- `LeaderProfileCard.tsx` - Summary card with photo
- `RoleResponsibilities.tsx` - Expandable list of duties
- `TeamMembersWidget.tsx` - Display team hierarchy

**C. Dashboard Widgets**
- `QuickStatsCard.tsx` - Metric display cards
- `RecentAnnouncementsWidget.tsx`
- `UpcomingMilestonesWidget.tsx`
- `PerformanceTrendChart.tsx`

**D. Admin Components**
- `LeaderManagementTable.tsx` - CRUD operations for leaders
- `OKREditor.tsx` - Create/edit OKRs form
- `AnnouncementEditor.tsx` - Rich text editor for announcements
- `DocumentUploader.tsx` - File upload with progress

---

### **Phase 6: API Hooks**

**File: `src/hooks/useLeadership.ts`**
```typescript
// Hooks for:
- useLeaderProfile() - Get current user's leadership profile
- useAllLeaders() - Get all leadership profiles (for CEO)
- useUpdateProfile() - Update profile
- useOKRs(quarter?, year?) - Get OKRs for current user or all
- useUpdateKeyResult() - Update KR progress
- useQuarterlyReviews() - Get reviews
- useSubmitSelfReview() - Submit review
- useAnnouncements() - Get announcements
- useMarkAnnouncementRead() - Mark as read
- useMessages() - Get messages
- useSendMessage() - Send message to CEO
- usePolicyDocuments() - Get documents
```

---

### **Phase 7: Routing & Protection**

**Update `src/App.tsx`:**
```typescript
// Add routes:
} /> {/* Public landing */}
} />

{/* Protected leadership routes */}

} />

} />

} />

{/* CEO-only admin routes */}

} />
```

**Create `src/components/LeadershipRoute.tsx`:**
- Check if user has leadership role
- Redirect non-leaders to login

**Create `src/components/CEORoute.tsx`:**
- Check if user has 'ceo' role
- Redirect non-CEOs to dashboard

---

### **Phase 8: Styling & Branding**

#### 8.1 AfriXport Leadership Theme
**Colors:**
- Primary: Green (#22C55E)
- Secondary: Gold (#F59E0B)
- Accent: Black (#000000)
- Background: White (#FFFFFF)
- Muted: Gray tones

**Typography:**
- Headers: Bold, professional
- Body: Clean, readable (Inter or similar)

#### 8.2 Responsive Design
- Mobile-first approach
- Tablet breakpoints for dashboard
- Desktop optimized for data-heavy views

---

### **Phase 9: Additional Features**

#### 9.1 Export to PDF
- Install `jspdf` (already in dependencies)
- Create `exportOKRDashboard()` function
- Generate formatted PDF with:
  - Leader profile
  - Current quarter OKRs
  - Progress data
  - Charts/graphs

#### 9.2 Notifications
- Toast notifications for:
  - OKR updates
  - New announcements
  - New messages
  - Quarterly review reminders

#### 9.3 Email Notifications (Future)
- Weekly progress reminders
- Quarterly review due dates
- CEO announcement emails

#### 9.4 Search & Filters
- Search OKRs by keyword
- Filter by status, category, quarter
- Sort by score, date

---

### **Phase 10: Security & Testing**

#### 10.1 Security Checklist
- ✅ All tables have RLS enabled
- ✅ CEO-only actions protected server-side
- ✅ Session timeout enforced (30 min)
- ✅ Password reset flow tested
- ✅ File upload validation (PDFs only, size limits)
- ✅ SQL injection prevention (Supabase handles this)

#### 10.2 Testing Plan
- Test all leadership role logins
- Test CEO admin CRUD operations
- Test OKR progress updates
- Test quarterly review submission
- Test announcement creation/reading
- Test messaging system
- Test PDF export
- Test responsive design on mobile

---

### **Implementation Timeline Estimate**

**Week 1:**
- Database schema creation
- Seed data population
- Authentication setup

**Week 2:**
- Public landing page
- Leadership login
- Dashboard layout & navigation

**Week 3:**
- OKR Dashboard (current + past quarters)
- Profile page
- Progress update functionality

**Week 4:**
- Quarterly reviews
- Performance history
- Charts & visualizations

**Week 5:**
- Announcements system
- Messaging system
- Document management

**Week 6:**
- CEO Admin Panel (all sections)
- OKR management for CEO
- Performance review tools

**Week 7:**
- PDF export feature
- Notifications
- Search & filters

**Week 8:**
- UI polish & responsive design
- Testing & bug fixes
- Documentation

---

### **Key Technical Decisions**

1. **Backend:** Supabase (already integrated)
2. **Frontend:** React with TypeScript (existing stack)
3. **UI Library:** Radix UI components (already in use)
4. **Charts:** Recharts (already in dependencies)
5. **PDF Generation:** jsPDF + jspdf-autotable (already in dependencies)
6. **Styling:** Tailwind CSS (existing)
7. **Forms:** react-hook-form + zod (existing)
8. **State Management:** TanStack Query (existing)

---

### **Files to Create (Summary)**

**Pages (17):**
1. `src/pages/Leadership.tsx` (public landing)
2. `src/pages/LeadershipLogin.tsx`
3. `src/pages/leadership/LeadershipDashboard.tsx`
4. `src/pages/leadership/LeadershipProfile.tsx`
5. `src/pages/leadership/OKRDashboard.tsx`
6. `src/pages/leadership/QuarterlyReviews.tsx`
7. `src/pages/leadership/PerformanceHistory.tsx`
8. `src/pages/leadership/Announcements.tsx`
9. `src/pages/leadership/LeaderMessages.tsx`
10. `src/pages/leadership/PolicyDocuments.tsx`
11. `src/pages/leadership/CEOAdmin.tsx`

**Components (20+):**
- Leadership layout & navigation
- OKR display components
- Charts & visualizations
- Profile widgets
- Admin CRUD interfaces
- Forms & modals

**Hooks (1):**
- `src/hooks/useLeadership.ts` (all API interactions)

**Routes:**
- `src/components/LeadershipRoute.tsx`
- `src/components/CEORoute.tsx`

**Database:**
- 10 new tables
- 20+ RLS policies
- Seed data script

This is a comprehensive, production-ready plan that fully implements your Leadership Portal requirements. The system will:

✅ **Authenticate all 11 leadership roles** securely  
✅ **Display personalized OKR dashboards** with real-time progress tracking  
✅ **Enable CEO admin controls** for full system management  
✅ **Track quarterly performance** with historical data  
✅ **Provide secure communication** between leaders and CEO  
✅ **Support document management** for policies and handbooks  
✅ **Export dashboards as PDF**  
✅ **Fully responsive design** with AfriXport branding  

The implementation follows your existing architecture, uses your current tech stack, and maintains security best practices with proper RLS policies.

Implement the plan

Add Real-Time Collaboration
Build Mobile Leadership App
Add AI Performance Insights
Build Leadership Impact Page
Add Gamification & Awards
