# 🎓 AI Learning Platform

> Nền tảng học tập thông minh sử dụng AI, được xây dựng với Next.js và Supabase

[![Next.js](https://img.shields.io/badge/Next.js-15-black?style=flat-square&logo=next.js)](https://nextjs.org/)
[![Supabase](https://img.shields.io/badge/Supabase-Backend-green?style=flat-square&logo=supabase)](https://supabase.com/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0-blue?style=flat-square&logo=typescript)](https://www.typescriptlang.org/)
[![TailwindCSS](https://img.shields.io/badge/TailwindCSS-3.0-38bdf8?style=flat-square&logo=tailwind-css)](https://tailwindcss.com/)

---

## 📖 Giới thiệu

**AI Learning Platform** là một hệ thống quản lý học tập (LMS) hiện đại, tích hợp AI để cá nhân hóa trải nghiệm học tập. Nền tảng cung cấp:

- 🤖 **AI-Powered Learning**: Chat thông minh, tạo quiz tự động, gợi ý khóa học cá nhân hóa
- 📚 **Course Management**: Quản lý khóa học, modules, lessons với giao diện trực quan
- 🎯 **Interactive Quizzes**: Hệ thống quiz tương tác với chấm điểm tự động
- 📊 **Progress Tracking**: Theo dõi tiến độ học tập chi tiết
- 🔐 **Secure Authentication**: Xác thực an toàn với Supabase Auth
- 💬 **RAG-based Chat**: Trò chuyện với AI dựa trên nội dung khóa học

---

## 🏗️ Kiến trúc hệ thống

```
┌─────────────────────────────┐
│   Client (Browser/PWA)      │
│   - Next.js App Router      │
│   - Shadcn/UI Components    │
└──────────────┬──────────────┘
               │
       ┌───────▼────────┐
       │   Next.js      │
       │   - Server     │
       │   - Actions    │
       │   - API Routes │
       └───────┬────────┘
               │
       ┌───────▼────────┐
       │   Supabase     │
       │   - Auth       │
       │   - Database   │
       │   - Storage    │
       │   - Vector DB  │
       └───────┬────────┘
               │
       ┌───────▼────────┐
       │  AI Providers  │
       │  (Gemini/GPT)  │
       └────────────────┘
```

---

## ✨ Tính năng chính

### 👨‍🎓 Dành cho Học viên

- ✅ Đăng ký và đăng nhập an toàn
- ✅ Kiểm tra năng lực với AI
- ✅ Gợi ý khóa học cá nhân hóa
- ✅ Học tập theo modules và lessons
- ✅ Làm quiz và xem kết quả chi tiết
- ✅ Chat với AI theo context khóa học
- ✅ Theo dõi tiến độ và nhận chứng chỉ

### 👨‍🏫 Dành cho Giảng viên

- ✅ Tạo và quản lý khóa học
- ✅ Upload tài liệu lên cloud storage
- ✅ Tạo quiz thủ công hoặc AI tự động
- ✅ Xem báo cáo và analytics học viên
- ✅ Quản lý modules và lessons

### 👨‍💼 Dành cho Admin

- ✅ Dashboard tổng quan hệ thống
- ✅ Quản lý users và phân quyền
- ✅ Kiểm soát chất lượng khóa học
- ✅ Quản lý tài nguyên (Storage, Vector DB)

---

## 🛠️ Tech Stack

### Frontend

- **Framework**: Next.js 15 (App Router)
- **UI Library**: Shadcn/UI + TailwindCSS
- **Language**: TypeScript
- **State Management**: React Server Components

### Backend

- **Platform**: Supabase
- **Database**: PostgreSQL (với RLS)
- **Authentication**: Supabase Auth
- **Storage**: Supabase Storage
- **Vector Store**: Supabase Vector (pgvector)
- **Edge Functions**: Supabase Functions

### AI Integration

- **SDK**: AI SDK (Vercel)
- **Providers**: Google Gemini / OpenAI
- **Features**: Chat, Quiz Generation, RAG

---

## 🚀 Bắt đầu

### Yêu cầu hệ thống

- Node.js 18+
- npm/yarn/pnpm
- Supabase CLI (optional)

### Cài đặt

```bash
# Clone repository
git clone https://github.com/yourusername/ai-learning.git
cd ai-learning

# Cài đặt dependencies
npm install

# Cấu hình environment variables
cp .env.example .env.local
# Chỉnh sửa .env.local với thông tin Supabase và AI API keys

# Chạy development server
npm run dev
```

Mở [http://localhost:3000](http://localhost:3000) để xem ứng dụng.

### Environment Variables

```env
# Supabase
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_service_role_key

# AI Provider
GOOGLE_GENERATIVE_AI_API_KEY=your_gemini_api_key
# hoặc
OPENAI_API_KEY=your_openai_api_key
```

---

## 📁 Cấu trúc dự án

```
project/
├── src/
│   ├── app/                    # Next.js App Router
│   │   ├── (auth)/            # Authentication pages
│   │   ├── dashboard/         # Dashboard
│   │   ├── courses/           # Course pages
│   │   ├── classes/           # Class management
│   │   ├── admin/             # Admin panel
│   │   └── api/               # API routes
│   ├── components/
│   │   ├── ui/                # Shadcn components
│   │   └── common/            # Shared components
│   ├── lib/
│   │   ├── supabase.ts        # Supabase clients
│   │   ├── ai/                # AI SDK wrapper
│   │   └── utils.ts           # Utilities
│   ├── services/              # Business logic
│   │   ├── course.service.ts
│   │   ├── quiz.service.ts
│   │   └── ai.service.ts
│   └── types/                 # TypeScript types
├── supabase/
│   ├── migrations/            # Database migrations
│   ├── seed/                  # Seed data
│   └── policies/              # RLS policies
└── package.json
```

---

## 📚 Tài liệu

- [📄 System Workflow](./Doc/SYSTEM.md) - Chi tiết về workflow và kiến trúc hệ thống

---

## 🤝 Đóng góp

Mọi đóng góp đều được chào đón! Vui lòng:

1. Fork repository
2. Tạo branch mới (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Mở Pull Request

---

## 📝 License

Dự án này được phân phối dưới giấy phép MIT. Xem file `LICENSE` để biết thêm chi tiết.

---

## 👥 Tác giả

**Your Name**

- GitHub: [@yourusername](https://github.com/yourusername)
- Email: your.email@example.com

---

## 🙏 Acknowledgments

- [Next.js](https://nextjs.org/) - The React Framework
- [Supabase](https://supabase.com/) - Open Source Firebase Alternative
- [Shadcn/UI](https://ui.shadcn.com/) - Beautiful UI Components
- [Vercel AI SDK](https://sdk.vercel.ai/) - AI Integration Made Easy

---

<div align="center">
  <strong>Made with ❤️ for better learning experiences</strong>
</div>
