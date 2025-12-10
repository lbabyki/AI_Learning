# WORKFLOW HỆ THỐNG

## 📑 Mục lục

| Phần  | Nội dung                          | Link                                                   |
| ----- | --------------------------------- | ------------------------------------------------------ |
| **1** | **WORKFLOW HỆ THỐNG**             | [↓](#1-workflow-hệ-thống)                              |
| 1.1   | Tổng quan hệ thống                | [↓](#11-tổng-quan-hệ-thống)                            |
| 1.2   | Luồng hoạt động tổng quát         | [↓](#12-luồng-hoạt-động-tổng-quát-high-level-workflow) |
| 1.2.1 | Authentication workflow           | [↓](#121-authentication-workflow)                      |
| 1.2.2 | Learning workflow                 | [↓](#122-learning-workflow)                            |
| 1.2.3 | Instructor workflow               | [↓](#123-instructor-workflow)                          |
| 1.2.4 | AI workflow                       | [↓](#124-ai-workflow)                                  |
| 1.3   | Key Components Overview           | [↓](#13-key-components-overview)                       |
| 1.4   | Các tài nguyên chính              | [↓](#14-các-tài-nguyên-chính)                          |
| 1.5   | Yêu cầu phi chức năng             | [↓](#15-yêu-cầu-phi-chức-năng)                         |
| **2** | **USER JOURNEY**                  | [↓](#2-user-journey)                                   |
| 2.1   | Student Journey                   | [↓](#21-student-journey)                               |
| 2.2   | Instructor Journey                | [↓](#22-instructor-journey)                            |
| 2.3   | Admin Journey                     | [↓](#23-admin-journey)                                 |
| **3** | **SYSTEM FLOW & KIẾN TRÚC**       | [↓](#3-system-flow--kiến-trúc)                         |
| 3.1   | System Architecture Overview      | [↓](#31-system-architecture-overview)                  |
| 3.2   | Data Flow Architecture            | [↓](#32-data-flow-architecture)                        |
| 3.2.1 | Authentication Flow               | [↓](#321-authentication-flow)                          |
| 3.2.2 | Course Learning Flow              | [↓](#322-course-learning-flow)                         |
| 3.2.3 | AI Chat Flow (RAG)                | [↓](#323-ai-chat-flow-rag)                             |
| 3.2.4 | Quiz Flow                         | [↓](#324-quiz-flow)                                    |
| 3.3   | Architectural Layers              | [↓](#33-architectural-layers)                          |
| **4** | **DEVELOPMENT SETUP & STRUCTURE** | [↓](#4-development-setup--structure)                   |
| 4.1   | Development Requirements          | [↓](#41-development-requirements)                      |
| 4.2   | Development Workflow              | [↓](#42-development-workflow)                          |
| 4.3   | Project Folder Structure          | [↓](#43-project-folder-structure)                      |

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

```
User → Next.js → Supabase Auth → Session → RLS → Data
```

#### 3.2.2 Course Learning Flow

```
User → Next.js page → Supabase DB → RLS filter → return course
User → finish lesson → update progress
```

#### 3.2.3 AI Chat Flow (RAG)

```
User question → embedding → vector search → retrieve top K
→ AI SDK generate answer → save history → FE render
```

#### 3.2.4 Quiz Flow

```
User → request quiz
→ Next.js → Edge Function quiz_generate
→ AI SDK sinh câu hỏi
→ Save quiz vào DB → FE render
```

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

Kết thúc tài liệu SYSTEM.md
