# WORKFLOW HỆ THỐNG

## 📑 Mục lục

- [1. WORKFLOW HỆ THỐNG](#1-workflow-hệ-thống)
- [2. USER JOURNEY](#2-user-journey)
- [3. SYSTEM FLOW & KIẾN TRÚC](#3-system-flow--kiến-trúc)
- [4. DEVELOPMENT SETUP & STRUCTURE](#4-development-setup--structure)
- [5. DATABASE SCHEMA](#5-database-schema)
- [6. SUPABASE EDGE FUNCTIONS](#6-supabase-edge-functions)

---

## 1. WORKFLOW HỆ THỐNG

### 1.1 Tổng quan hệ thống

Hệ thống là một nền tảng học tập sử dụng:

- **Next.js** cho toàn bộ frontend và backend nhẹ (server actions, route handlers)
- **Supabase** làm backend chính (auth, database Postgres, storage, vector store)
- **AI SDK** để xử lý AI (chat, tạo quiz, tạo khóa học, RAG)
- **Shadcn/UI** để tiêu chuẩn hóa UI
- **App Router + Server Components** để tối ưu hóa tốc độ tải và SEO

### 1.2 Luồng hoạt động tổng quát (High-level Workflow)

```
User → Next.js FE → Supabase Auth → Database
                          ↓
                    AI SDK / RAG
```

**Các nhóm workflow:**

#### 1.2.1 Authentication workflow

- User đăng nhập qua Supabase Auth
- Next.js nhận session qua cookies
- RLS trên Supabase Postgres kiểm soát dữ liệu

#### 1.2.2 Learning workflow

- User xem khóa học → modules → lessons
- Làm quiz → chấm điểm → lưu progress
- Trao đổi với AI theo context khóa học (RAG)

#### 1.2.3 Instructor workflow

- Tạo khóa học → tạo module → lesson
- Tải tài liệu lên Supabase Storage
- Tạo quiz thủ công hoặc AI tạo tự động
- Xem báo cáo học viên

#### 1.2.4 AI workflow

- Chat thông minh
- Sinh lộ trình học tập
- Sinh khóa học từ prompt
- Sinh quiz từ nội dung lesson
- RAG: Search nội dung văn bản → embedding → supabase vector

### 1.3 Key Components Overview

- **Next.js frontend**
- **Next.js backend** (server actions)
- **Supabase services** (DB, Auth, Storage, Functions, Vector)
- **AI Integration Layer**
- **UI Layer** (Shadcn)

### 1.4 Các tài nguyên chính

- Database (Postgres)
- Storage (assets + tài liệu)
- Embeddings (vector store)
- AI endpoints

### 1.5 Yêu cầu phi chức năng

- SEO + SSR (built-in từ Next.js)
- RLS bảo mật dữ liệu
- Real-time update (Supabase realtime)
- Khả năng mở rộng nhờ Supabase Edge Functions

---

## 2. USER JOURNEY

### 2.1 Student Journey

1. Đăng ký Supabase Auth
2. Onboarding → Kiểm tra năng lực (AI)
3. Gợi ý khóa học cá nhân hóa
4. Chọn khóa học → Học module → lesson
5. Làm quiz → xem kết quả chi tiết
6. Chat AI theo course context
7. Track progress → Earn certificates

### 2.2 Instructor Journey

1. Đăng nhập với role instructor
2. Tạo khóa học mới, upload cover, description
3. Tạo modules → lessons
4. Tải tài liệu lên Supabase Storage
5. Generate quiz bằng AI
6. Quản lý học viên → xem tiến độ → analytics

### 2.3 Admin Journey

1. Dashboard tổng quan → thống kê hệ thống
2. Quản lý users → reset role → ban
3. Quản lý courses → xem chất lượng & data
4. Kiểm soát tài nguyên → Storage → Vector DB
5. Điều chỉnh AI parameters (optional)

---

## 3. SYSTEM FLOW & KIẾN TRÚC

### 3.1 System Architecture Overview

**Version Next.js + Supabase (không microservices)**

```
┌───────────────────────────┐
│  Client Layer             │
│  - Browser (Next.js)      │
│  - Mobile PWA             │
└───────────────┬───────────┘
                │
        HTTPS / WebSocket
                │
┌───────────────▼──────────────┐
│        Next.js Layer          │
│ - App Router                  │
│ - Pages, Layouts              │
│ - Server Actions              │
│ - API Route Handlers          │
│ - Shadcn/UI                   │
│ - AI SDK calls                │
└───────────────┬──────────────┘
                │
     Supabase Client/Server SDK
                │
┌───────────────▼──────────────┐
│        Supabase Services      │
│ - Auth                        │
│ - Database (Postgres)         │
│ - Row Level Security          │
│ - Storage                     │
│ - Edge Functions (custom AI)  │
│ - Realtime                    │
│ - Vector Store (RAG)          │
└───────────────┬──────────────┘
                │
            AI Providers
         (Gemini / OpenAI)
```

### 3.2 Data Flow Architecture

#### 3.2.1 Authentication Flow

**Luồng đăng nhập/đăng ký:**

```
User → Next.js Login Page
    → Supabase Auth (email/password hoặc OAuth)
    → Generate Session Token
    → Store in Cookie (httpOnly)
    → Next.js Middleware verify session
    → RLS check permissions
    → Redirect to Dashboard
```

**Chức năng liên quan:**

- Đăng ký tài khoản mới
- Đăng nhập (email/password, Google, GitHub)
- Quên mật khẩu & reset
- Xác thực session cho mọi request
- Logout & clear session

---

#### 3.2.2 Assessment & Onboarding Flow

**Luồng đánh giá năng lực (AI-powered):**

```
1. User chọn lĩnh vực & mức độ
   → POST /api/v1/assessments/generate
   → Next.js Server Action
   → Edge Function: assessment_generate
   → AI (Gemini) sinh câu hỏi dựa trên courses
   → Save to ai_assessments table
   → Return questions[] to FE

2. User làm bài test
   → FE track thời gian & answers
   → POST /api/v1/assessments/submit
   → Edge Function: assessment_score
   → AI phân tích kết quả theo skill tags
   → Calculate score & level
   → Save to ai_results table
   → Return detailed feedback

3. AI tạo lộ trình cá nhân hóa
   → GET /api/v1/recommendations/from-assessment
   → AI phân tích lỗ hổng kiến thức
   → Query courses phù hợp từ DB
   → Rank theo độ ưu tiên
   → Save to ai_recommendations table
   → Return personalized learning path
```

**Tables sử dụng:** `ai_assessments`, `ai_results`, `ai_recommendations`, `courses`

---

#### 3.2.3 Course Discovery & Enrollment Flow

**Luồng tìm kiếm & đăng ký khóa học:**

```
1. Tìm kiếm khóa học
   → GET /api/v1/courses/search?q=python&level=beginner
   → Next.js API Route
   → Supabase query courses table
   → Apply filters (category, level, rating)
   → RLS check visibility
   → Return paginated results

2. Xem chi tiết khóa học
   → GET /api/v1/courses/{id}
   → Query courses + modules + lessons
   → Check enrollment status
   → Return full course structure

3. Đăng ký khóa học
   → POST /api/v1/enrollments
   → Validate: user not enrolled, course published
   → Insert into course_enrollments
   → Initialize lesson_progress records
   → Return enrollment_id
```

**Tables sử dụng:** `courses`, `modules`, `lessons`, `course_enrollments`, `lesson_progress`

---

#### 3.2.4 Learning & Progress Tracking Flow

**Luồng học tập & theo dõi tiến độ:**

```
1. Xem nội dung lesson
   → GET /api/v1/courses/{id}/modules/{moduleId}/lessons/{lessonId}
   → Check enrollment & prerequisites
   → Query lesson content + resources
   → Track view time
   → Return lesson data

2. Hoàn thành lesson
   → POST /api/v1/lessons/{id}/complete
   → Update lesson_progress (completed = true)
   → Calculate module progress
   → Calculate course progress
   → Unlock next lesson (if sequential)
   → Return updated progress

3. Làm quiz
   → GET /api/v1/quizzes/{id}
   → Return quiz questions
   → User submit answers
   → POST /api/v1/quizzes/{id}/attempt
   → Edge Function: quiz_submit
   → Calculate score & check pass (≥70% + mandatory questions)
   → Save to quiz_attempts
   → If pass: unlock next lesson
   → If fail: generate new quiz variant
   → Return results + feedback
```

**Tables sử dụng:** `lessons`, `lesson_progress`, `quiz`, `quiz_questions`, `quiz_attempts`

---

#### 3.2.5 AI Chat Flow (RAG)

**Luồng chat với AI có context khóa học:**

```
1. User gửi câu hỏi
   → POST /api/v1/chat/course/{courseId}
   → Next.js Server Action
   → Edge Function: chat_course

2. RAG Processing
   → Embed user question (AI SDK)
   → Vector search in Supabase Vector DB
   → Retrieve top K relevant lessons/modules
   → Build context from course content

3. AI Generate Response
   → Send context + question to AI (Gemini)
   → AI generate answer
   → Extract source references

4. Save & Return
   → Save to ai_chat_history + ai_messages
   → Return response + sources[] to FE
   → FE render with markdown + code highlight
```

**Tables sử dụng:** `ai_chat_history`, `ai_messages`, `courses`, `lessons` (vector embeddings)

---

#### 3.2.6 Quiz Generation Flow (AI)

**Luồng tạo quiz tự động:**

```
1. Request quiz generation
   → POST /api/v1/quizzes/generate
   → Input: lesson_id, difficulty, question_count
   → Edge Function: quiz_generate

2. AI Processing
   → Fetch lesson content
   → AI (Gemini) analyze content
   → Generate questions based on:
      - Learning outcomes
      - Key concepts
      - Difficulty level
   → Create multiple choice, fill-in-blank, drag-drop

3. Save & Return
   → Save to quiz + quiz_questions tables
   → Mark mandatory questions (điểm liệt)
   → Return quiz_id + questions[]
```

**Tables sử dụng:** `quiz`, `quiz_questions`, `lessons`

---

#### 3.2.7 Personal Course Creation Flow

**Luồng tạo khóa học cá nhân:**

```
Option 1: AI-Generated Course
   → POST /api/v1/courses/from-prompt
   → User input: natural language description
   → AI analyze prompt
   → Generate course structure:
      - Modules (ordered logically)
      - Lessons per module
      - Learning outcomes
      - Basic content
   → Save to personal_courses, personal_modules, personal_lessons
   → Return preview for user confirmation

Option 2: Manual Creation
   → POST /api/v1/courses/personal
   → User input: title, description, category, level
   → Create empty course (status: draft)
   → Return course_id
   → User manually add modules & lessons
```

**Tables sử dụng:** `personal_courses`, `personal_modules`, `personal_lessons`

---

#### 3.2.8 Instructor Class Management Flow

**Luồng quản lý lớp học (Instructor):**

```
1. Tạo lớp học
   → POST /api/v1/classes
   → Select base course
   → Input: name, description, start/end date, max students
   → Generate unique invite code (6-8 chars)
   → Save to classes table
   → Return class_id + invite_code

2. Student join class
   → POST /api/v1/classes/join
   → Input: invite_code
   → Validate: code exists, class not full, not expired
   → Insert into class_students
   → Initialize class_progress
   → Return success

3. Track student progress
   → GET /api/v1/classes/{id}/students/{studentId}
   → Query class_progress + lesson_progress + quiz_attempts
   → Calculate metrics:
      - Completion percentage
      - Average quiz score
      - Time spent
   → Return detailed analytics
```

**Tables sử dụng:** `classes`, `class_students`, `class_progress`, `courses`

---

#### 3.2.9 Admin Management Flow

**Luồng quản trị hệ thống:**

```
1. User Management
   → GET /api/v1/admin/users
   → Query users + profiles + user_roles
   → Apply filters & search
   → Return paginated user list

   → PUT /api/v1/admin/users/{id}/role
   → Validate new role
   → Update user_roles table
   → Check impact (classes, courses affected)
   → Return success

2. Course Management
   → GET /api/v1/admin/courses
   → Query all courses (public + personal)
   → Show analytics (enrollments, completion rate)
   → Admin can edit/delete any course

3. System Dashboard
   → GET /api/v1/admin/dashboard
   → Aggregate statistics:
      - Total users by role
      - Active courses & classes
      - System activity metrics
   → Return dashboard data
```

**Tables sử dụng:** `users`, `profiles`, `user_roles`, `courses`, `classes`

### 3.3 Architectural Layers

| Layer            | Công nghệ        | Trách nhiệm                  |
| ---------------- | ---------------- | ---------------------------- |
| **Presentation** | Next.js + Shadcn | UI, UX, navigation           |
| **Application**  | Server Actions   | Logic ứng dụng, orchestrator |
| **AI Layer**     | AI SDK           | chat, quiz, content gen      |
| **Data Layer**   | Supabase SDK     | CRUD, RLS, realtime          |
| **Database**     | Postgres         | Toàn bộ dữ liệu              |
| **Storage**      | Supabase Storage | Tài liệu, assets             |
| **Vector**       | Supabase Vector  | Embedding + semantic search  |

---

## 4. DEVELOPMENT SETUP & STRUCTURE

### 4.1 Development Requirements

**Frontend & Backend hợp nhất trong Next.js**

#### 4.1.1 Requirements

- Node.js 18+
- Next.js 15
- Supabase CLI
- AI SDK (OpenAI/Gemini)
- TailwindCSS + Shadcn/UI

### 4.2 Development Workflow

#### 4.2.1 Next.js Development Flow

1. `npx create-next-app`
2. `shadcn-ui init`
3. `supabase init`
4. Tạo database tables & RLS
5. Tạo UI page (App Router)
6. Tạo server actions cho business logic
7. Tích hợp AI SDK
8. Viết services (course, quiz, chat)
9. Testing → deploy lên Vercel

### 4.3 Project Folder Structure

```
project/
├── src/
│   ├── app/                        # Next.js App Router
│   │   ├── (auth)/                 # Login/Register
│   │   ├── dashboard/
│   │   ├── courses/
│   │   ├── classes/
│   │   ├── admin/
│   │   └── api/                    # Route handlers
│   ├── components/
│   │   ├── ui/                     # Shadcn components
│   │   └── common/
│   ├── lib/
│   │   ├── supabase.ts             # Client + server client
│   │   ├── ai/                     # AI SDK wrapper
│   │   ├── utils.ts
│   │   └── validators/
│   ├── services/
│   │   ├── course.service.ts
│   │   ├── quiz.service.ts
│   │   └── ai.service.ts
│   ├── types/
│   └── styles/
├── supabase/
│   ├── migrations/
│   ├── seed/
│   └── policies/
└── package.json
```

---

## 5. DATABASE SCHEMA

### 5.1 User & Roles

| Table        | Mô tả                                                |
| ------------ | ---------------------------------------------------- |
| `users`      | Thông tin người dùng cơ bản (sync với Supabase Auth) |
| `profiles`   | Thông tin profile mở rộng (avatar, bio, preferences) |
| `roles`      | Danh sách các role (student, instructor, admin)      |
| `user_roles` | Mapping user với roles (many-to-many)                |

### 5.2 Courses

| Table                | Mô tả                                                  |
| -------------------- | ------------------------------------------------------ |
| `courses`            | Thông tin khóa học (title, description, instructor_id) |
| `modules`            | Modules trong khóa học                                 |
| `lessons`            | Bài học trong module                                   |
| `resources`          | Tài liệu đính kèm (files, links, videos)               |
| `quiz`               | Bài quiz                                               |
| `quiz_questions`     | Câu hỏi trong quiz                                     |
| `quiz_attempts`      | Lịch sử làm quiz của học viên                          |
| `lesson_progress`    | Tiến độ học từng lesson                                |
| `course_enrollments` | Đăng ký khóa học của học viên                          |

### 5.3 Personal Courses

| Table              | Mô tả                          |
| ------------------ | ------------------------------ |
| `personal_courses` | Khóa học cá nhân do AI tạo     |
| `personal_modules` | Modules trong khóa học cá nhân |
| `personal_lessons` | Lessons trong khóa học cá nhân |

### 5.4 Instructor Class

| Table            | Mô tả                                   |
| ---------------- | --------------------------------------- |
| `classes`        | Lớp học do instructor quản lý           |
| `class_students` | Danh sách học viên trong lớp            |
| `class_progress` | Tiến độ học của từng học viên trong lớp |

### 5.5 AI

| Table                | Mô tả                           |
| -------------------- | ------------------------------- |
| `ai_assessments`     | Bài đánh giá năng lực do AI tạo |
| `ai_results`         | Kết quả đánh giá năng lực       |
| `ai_recommendations` | Gợi ý khóa học từ AI            |
| `ai_chat_history`    | Lịch sử chat sessions           |
| `ai_messages`        | Tin nhắn trong chat (user + AI) |

---

## 6. SUPABASE EDGE FUNCTIONS

### 6.1 Edge Functions Overview

| Function              | Vai trò                                | Input                       | Output                     |
| --------------------- | -------------------------------------- | --------------------------- | -------------------------- |
| `assessment_generate` | Sinh bộ câu hỏi đánh giá năng lực (AI) | user_id, topic, level       | assessment_id, questions[] |
| `assessment_score`    | Chấm điểm bài đánh giá                 | assessment_id, answers[]    | score, feedback, level     |
| `quiz_generate`       | Sinh quiz tự động theo lesson/module   | lesson_id/module_id, count  | quiz_id, questions[]       |
| `quiz_submit`         | Chấm điểm quiz và tính pass/fail       | quiz_id, answers[]          | score, passed, feedback    |
| `chat_course`         | Chat AI có context RAG từ khóa học     | course_id, message, history | response, sources[]        |
| `practice_generate`   | Sinh bài tập luyện tập                 | lesson_id, difficulty       | practice_id, exercises[]   |

### 6.2 Edge Function Details

#### 6.2.1 assessment_generate

```typescript
// Input
{
  user_id: string,
  topic: string,
  level?: 'beginner' | 'intermediate' | 'advanced'
}

// Output
{
  assessment_id: string,
  questions: [
    {
      id: string,
      question: string,
      options: string[],
      type: 'multiple_choice' | 'true_false'
    }
  ]
}
```

#### 6.2.2 assessment_score

```typescript
// Input
{
  assessment_id: string,
  answers: { question_id: string, answer: string }[]
}

// Output
{
  score: number,
  total: number,
  percentage: number,
  level: string,
  feedback: string,
  recommendations: string[]
}
```

#### 6.2.3 quiz_generate

```typescript
// Input
{
  lesson_id?: string,
  module_id?: string,
  question_count: number,
  difficulty?: 'easy' | 'medium' | 'hard'
}

// Output
{
  quiz_id: string,
  questions: [
    {
      id: string,
      question: string,
      options: string[],
      correct_answer: string,
      explanation: string
    }
  ]
}
```

#### 6.2.4 quiz_submit

```typescript
// Input
{
  quiz_id: string,
  user_id: string,
  answers: { question_id: string, answer: string }[]
}

// Output
{
  attempt_id: string,
  score: number,
  total: number,
  percentage: number,
  passed: boolean,
  feedback: string,
  results: [
    {
      question_id: string,
      correct: boolean,
      user_answer: string,
      correct_answer: string,
      explanation: string
    }
  ]
}
```

#### 6.2.5 chat_course

```typescript
// Input
{
  course_id: string,
  user_id: string,
  message: string,
  chat_history_id?: string
}

// Output
{
  response: string,
  sources: [
    {
      lesson_id: string,
      lesson_title: string,
      excerpt: string
    }
  ],
  chat_history_id: string,
  message_id: string
}
```

#### 6.2.6 practice_generate

```typescript
// Input
{
  lesson_id: string,
  difficulty: 'easy' | 'medium' | 'hard',
  exercise_count: number
}

// Output
{
  practice_id: string,
  exercises: [
    {
      id: string,
      type: 'coding' | 'problem_solving' | 'essay',
      question: string,
      hints: string[],
      solution?: string
    }
  ]
}
```

---

Kết thúc tài liệu SYSTEM.md
