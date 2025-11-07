# Phase 1: 基础架构 - 设计文档

## 设计概述

**设计目标：** 建立一个现代化、可扩展、高性能的 Web 应用基础架构  
**技术栈：** Next.js 15 + TypeScript + Tailwind CSS + Prisma + Neon + Vercel  
**设计原则：** 简洁、高效、可维护、SEO友好  

---

## 系统架构

### 整体架构图

```
用户浏览器
    ↓
Vercel CDN (全球边缘节点)
    ↓
Next.js 15 App (SSR/SSG)
    ↓
├── Prisma ORM
│   └── Neon PostgreSQL
├── Google Analytics 4
└── Google Search Console
```

### 技术栈选择理由

| 技术 | 选择理由 |
|------|---------|
| **Next.js 15** | 最新的 App Router，优秀的 SEO 支持，内置性能优化 |
| **TypeScript** | 类型安全，减少运行时错误，提升开发效率 |
| **Tailwind CSS** | 快速开发，一致的设计系统，优秀的响应式支持 |
| **Prisma** | 类型安全的 ORM，优秀的开发体验，自动迁移 |
| **Neon** | Serverless PostgreSQL，自动扩展，免费层慷慨 |
| **Vercel** | 零配置部署，全球 CDN，优秀的 Next.js 集成 |
| **next-i18next** | 成熟的 Next.js 国际化方案，支持 SSR/SSG，易于维护 |

---

## 项目结构设计

```
nihaixia-website/
├── app/                          # Next.js 15 App Router
│   ├── globals.css              # 全局样式
│   ├── layout.tsx               # 根布局
│   ├── page.tsx                 # 主页
│   ├── loading.tsx              # 加载状态
│   ├── error.tsx                # 错误页面
│   ├── not-found.tsx            # 404页面
│   │
│   ├── tianji/                  # 天纪页面（占位符）
│   │   └── page.tsx
│   ├── diji/                    # 地纪页面（占位符）
│   │   └── page.tsx
│   ├── renji/                   # 人纪页面（占位符）
│   │   └── page.tsx
│   │
│   ├── api/                     # API Routes
│   │   └── health/              # 健康检查
│   │       └── route.ts
│   │
│   └── sitemap.ts               # 动态 sitemap
│
├── components/                   # 可复用组件
│   ├── ui/                      # 基础UI组件
│   │   ├── Button.tsx
│   │   └── Loading.tsx
│   ├── layout/                  # 布局组件
│   │   ├── Header.tsx
│   │   ├── Footer.tsx
│   │   └── Navigation.tsx
│   └── seo/                     # SEO组件
│       └── StructuredData.tsx
│
├── lib/                         # 工具库
│   ├── db.ts                    # 数据库连接
│   └── utils.ts                 # 通用工具
│
├── prisma/                      # 数据库
│   └── schema.prisma            # 数据库模式
│
├── public/                      # 静态文件
│   ├── locales/                 # 国际化翻译资源
│   │   ├── zh-CN/              # 简体中文
│   │   │   └── common.json
│   │   ├── zh-TW/              # 繁体中文
│   │   │   └── common.json
│   │   └── en/                 # 英语
│   │       └── common.json
│   ├── robots.txt
│   └── favicon.ico
│
├── .github/                     # GitHub配置
│   └── workflows/
│       └── ci.yml
│
├── .env.example                 # 环境变量示例
├── next.config.js               # Next.js配置
├── next-i18next.config.js       # i18n配置
├── tailwind.config.js           # Tailwind配置
├── tsconfig.json                # TypeScript配置
└── package.json                 # 依赖管理
```

---

## 国际化架构设计

### i18n配置

```javascript
// next-i18next.config.js
module.exports = {
  i18n: {
    defaultLocale: 'zh-CN',
    locales: ['zh-CN', 'zh-TW', 'en'],
    localeDetection: true,
  },
  reloadOnPrerender: process.env.NODE_ENV === 'development',
};
```

### 路由策略

**支持的URL格式**：

- `/` - 默认语言（zh-CN）
- `/zh-cn/` - 简体中文
- `/zh-tw/` - 繁体中文
- `/en/` - 英语

**路由示例**：

```
https://nihaixia.com/            → 简体中文主页
https://nihaixia.com/zh-cn/      → 简体中文主页（显式）
https://nihaixia.com/zh-tw/      → 繁体中文主页
https://nihaixia.com/en/         → 英语主页
https://nihaixia.com/en/about    → 英语关于页面
```

### 翻译资源组织

```json
// public/locales/zh-CN/common.json
{
  "nav": {
    "home": "首页",
    "tianji": "天纪",
    "diji": "地纪",
    "renji": "人纪",
    "articles": "文章",
    "about": "关于"
  },
  "footer": {
    "copyright": "© 2024 倪海厦内容网站. 保留所有权利。"
  },
  "common": {
    "loading": "加载中...",
    "error": "出错了",
    "retry": "重试"
  }
}

// public/locales/zh-TW/common.json
{
  "nav": {
    "home": "首頁",
    "tianji": "天紀",
    "diji": "地紀",
    "renji": "人紀",
    "articles": "文章",
    "about": "關於"
  },
  "footer": {
    "copyright": "© 2024 倪海廈內容網站. 保留所有權利。"
  },
  "common": {
    "loading": "載入中...",
    "error": "出錯了",
    "retry": "重試"
  }
}

// public/locales/en/common.json
{
  "nav": {
    "home": "Home",
    "tianji": "Tianji",
    "diji": "Diji",
    "renji": "Renji",
    "articles": "Articles",
    "about": "About"
  },
  "footer": {
    "copyright": "© 2024 NiHaiXia Content Platform. All rights reserved."
  },
  "common": {
    "loading": "Loading...",
    "error": "Error occurred",
    "retry": "Retry"
  }
}
```

### 语言切换组件

```typescript
// components/LanguageSwitcher.tsx
'use client';

import { useRouter } from 'next/router';
import { useTranslation } from 'next-i18next';

export function LanguageSwitcher() {
  const router = useRouter();
  const { i18n } = useTranslation();

  const languages = [
    { code: 'zh-CN', label: '简体中文' },
    { code: 'zh-TW', label: '繁體中文' },
    { code: 'en', label: 'English' },
  ];

  const changeLanguage = (locale: string) => {
    router.push(router.pathname, router.asPath, { locale });
  };

  return (
    <div className="flex items-center space-x-2">
      {languages.map((lang) => (
        <button
          key={lang.code}
          onClick={() => changeLanguage(lang.code)}
          className={cn(
            "px-2 py-1 text-sm rounded",
            i18n.language === lang.code
              ? "bg-primary-600 text-white"
              : "text-gray-600 hover:bg-gray-100"
          )}
        >
          {lang.label}
        </button>
      ))}
    </div>
  );
}
```

### SEO最佳实践

**hreflang标签**：

```typescript
// app/layout.tsx
export function generateMetadata({ params }: { params: { locale: string } }) {
  const baseUrl = 'https://nihaixia.com';

  return {
    alternates: {
      canonical: `${baseUrl}/${params.locale}`,
      languages: {
        'zh-CN': `${baseUrl}/zh-cn`,
        'zh-TW': `${baseUrl}/zh-tw`,
        'en': `${baseUrl}/en`,
      },
    },
  };
}
```

---

## 内容管理策略

### CMS选型决策矩阵

| 方案 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| **Strapi** | 功能强大、社区活跃、插件丰富 | 需要独立部署、学习成本 | 复杂内容管理需求 |
| **Payload CMS** | TypeScript原生、与Next.js集成好 | 相对较新、插件较少 | 开发者友好型项目 |
| **自建方案** | 完全可控、无外部依赖 | 开发工作量大、维护成本高 | 需求简单且定制化高 |
| **Git-based (MDX)** | 版本控制、开发者友好 | 非技术人员难用、无GUI | 技术文档、博客 |

### 推荐方案

**Phase 1决策**: 采用**混合方案**

1. **静态内容** (STATIC_PAGE, TIANJI_PAGE, DIJI_PAGE, RENJI_PAGE)
   - 使用MDX文件存储在代码仓库
   - 适合长期不变的核心内容
   - 易于版本控制和审查

2. **动态内容** (ARTICLE)
   - 使用Strapi CMS管理
   - 适合频繁更新的文章
   - 便于非技术人员编辑

### 内容工作流设计

```
┌─────────────────────────────────────────┐
│          内容创建流程                    │
└─────────────────────────────────────────┘

静态内容流程：
1. 在代码仓库创建MDX文件
2. 本地预览和编辑
3. 提交Pull Request
4. 代码审查
5. 合并到main分支
6. 自动部署到生产环境

动态内容流程（如选择Strapi）：
1. 在Strapi后台创建内容
2. 编辑和保存为草稿
3. 预览效果
4. 提交审核（可选）
5. 发布到生产环境
6. 网站自动获取并显示新内容（ISR）
```

### 内容目录结构（MDX方案）

```
content/
├── pages/                  # 静态页面
│   ├── zh-CN/
│   │   ├── about.mdx
│   │   └── privacy.mdx
│   ├── zh-TW/
│   │   ├── about.mdx
│   │   └── privacy.mdx
│   └── en/
│       ├── about.mdx
│       └── privacy.mdx
│
├── tianji/                 # 天纪内容
│   └── zh-CN/
│       └── index.mdx
│
├── diji/                   # 地纪内容
│   └── zh-CN/
│       └── index.mdx
│
└── renji/                  # 人纪内容
    └── zh-CN/
        ├── zhenjiu.mdx    # 针灸
        ├── shanghanlun.mdx # 伤寒论
        └── jinguiyaolue.mdx # 金匮要略
```

### Phase 1 CMS决策交付物

**必须完成**：

1. **CMS选型决策文档** (`docs/cms-decision.md`)
   - 评估结果
   - 选型理由
   - 实施计划

2. **内容边界定义文档** (`docs/content-boundary.md`)
   - 哪些内容用MDX管理
   - 哪些内容用CMS管理
   - 内容迁移计划（如有必要）

3. **基础配置**（如选择第三方CMS）
   - CMS安装和配置
   - 内容类型定义
   - API连接测试

---

## 核心组件设计

### 1. Layout 组件

#### Header 组件
```typescript
// components/layout/Header.tsx
export function Header() {
  return (
    <header className="fixed top-0 w-full bg-white/95 backdrop-blur-sm shadow-sm z-50">
      <div className="container mx-auto px-4">
        <div className="flex items-center justify-between h-16">
          <div className="flex items-center space-x-2">
            <span className="text-xl font-bold text-gray-900">
              倪海厦内容网站
            </span>
          </div>
          <Navigation />
        </div>
      </div>
    </header>
  );
}
```

#### Navigation 组件
```typescript
// components/layout/Navigation.tsx
const navItems = [
  { name: '天纪', href: '/tianji' },
  { name: '地纪', href: '/diji' },
  { name: '人纪', href: '/renji' },
  { name: '文章', href: '/articles' },
  { name: '关于', href: '/about' },
];

export function Navigation() {
  return (
    <nav className="hidden md:flex items-center space-x-8">
      {navItems.map((item) => (
        <Link
          key={item.name}
          href={item.href}
          className="text-sm font-medium text-gray-700 hover:text-primary-600 transition-colors"
        >
          {item.name}
        </Link>
      ))}
    </nav>
  );
}
```

#### Footer 组件
```typescript
// components/layout/Footer.tsx
export function Footer() {
  return (
    <footer className="bg-gray-50 border-t border-gray-200">
      <div className="container mx-auto px-4 py-8">
        <div className="text-center text-sm text-gray-600">
          <p>© 2024 倪海厦内容网站. All rights reserved.</p>
        </div>
      </div>
    </footer>
  );
}
```

### 2. SEO 组件

#### StructuredData 组件
```typescript
// components/seo/StructuredData.tsx
interface WebsiteSchemaProps {
  name: string;
  url: string;
  description: string;
}

export function WebsiteSchema({ name, url, description }: WebsiteSchemaProps) {
  const schema = {
    "@context": "https://schema.org",
    "@type": "WebSite",
    "name": name,
    "url": url,
    "description": description,
  };
  
  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  );
}
```

### 3. 页面组件

#### 主页设计
```typescript
// app/page.tsx
import { Metadata } from 'next';
import { WebsiteSchema } from '@/components/seo/StructuredData';

export const metadata: Metadata = {
  title: '倪海厦内容网站 - 中医易学学习平台',
  description: '学习倪海厦老师的中医、易学、命理知识，包含天纪、地纪、人纪等经典内容',
};

export default function HomePage() {
  return (
    <>
      <WebsiteSchema
        name="倪海厦内容网站"
        url="https://nihaixia.com"
        description="倪海厦中医易学内容学习平台"
      />
      
      <main className="min-h-screen">
        {/* Hero Section */}
        <section className="pt-32 pb-16 px-4">
          <div className="container mx-auto text-center">
            <h1 className="text-4xl md:text-6xl font-bold text-gray-900 mb-6">
              倪海厦内容网站
            </h1>
            <p className="text-xl text-gray-600 mb-8">
              学习中医、易学、命理的专业平台
            </p>
          </div>
        </section>
        
        {/* Features Section */}
        <section className="py-16 px-4 bg-gray-50">
          <div className="container mx-auto">
            <div className="grid md:grid-cols-3 gap-8">
              <FeatureCard
                title="天纪"
                description="易学命理知识"
                href="/tianji"
              />
              <FeatureCard
                title="地纪"
                description="风水地理知识"
                href="/diji"
              />
              <FeatureCard
                title="人纪"
                description="中医医学知识"
                href="/renji"
              />
            </div>
          </div>
        </section>
      </main>
    </>
  );
}
```

---

## 数据库设计

### Prisma Schema

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// 内容类型枚举 - 统一所有内容类型
enum ContentType {
  STATIC_PAGE    // 静态页面（关于、隐私等）
  ARTICLE        // 文章
  TIANJI_PAGE    // 天纪页面
  DIJI_PAGE      // 地纪页面
  RENJI_PAGE     // 人纪页面
}

// 统一内容模型 - 替代原有的Page和Article分离模型
model Content {
  id          String      @id @default(cuid())
  type        ContentType // 内容类型
  slug        String      @unique
  title       String
  description String?
  published   Boolean     @default(false)
  views       Int         @default(0)
  createdAt   DateTime    @default(now())
  updatedAt   DateTime    @updatedAt

  // 关联关系（Phase 2-3将添加）
  translations ContentTranslation[]
  analytics    Analytics[]

  @@map("contents")
  @@index([type])
  @@index([published])
}

// 多语言翻译表
model ContentTranslation {
  id          String   @id @default(cuid())
  contentId   String
  locale      String   // 'zh-CN', 'zh-TW', 'en'
  title       String
  description String?
  body        String   @db.Text

  content     Content  @relation(fields: [contentId], references: [id], onDelete: Cascade)

  @@unique([contentId, locale])
  @@map("content_translations")
  @@index([locale])
}

// 分析统计表
model Analytics {
  id        String   @id @default(cuid())
  contentId String?
  event     String
  data      Json?
  createdAt DateTime @default(now())

  content   Content? @relation(fields: [contentId], references: [id], onDelete: SetNull)

  @@map("analytics")
  @@index([event])
  @@index([createdAt])
}
```

### 数据模型设计说明

**为什么使用统一的Content模型？**

1. **避免模型冗余**：原设计中Page和Article模型大部分字段相同，造成重复
2. **便于扩展**：新增内容类型只需添加枚举值，无需创建新表
3. **统一管理**：评论、点赞、分析等功能可以统一关联到Content
4. **多语言支持**：ContentTranslation表支持所有内容类型的多语言

**ContentType枚举值说明**：

- `STATIC_PAGE`: 静态页面（关于我们、隐私政策等）
- `ARTICLE`: 普通文章内容
- `TIANJI_PAGE`: 天纪专题页面
- `DIJI_PAGE`: 地纪专题页面
- `RENJI_PAGE`: 人纪专题页面（可进一步细分为针灸、伤寒论等子类）

**查询示例**：

```typescript
// 获取所有文章
const articles = await prisma.content.findMany({
  where: { type: 'ARTICLE', published: true }
});

// 获取天纪页面的中文翻译
const tianjiPage = await prisma.content.findFirst({
  where: { type: 'TIANJI_PAGE', slug: 'tianji' },
  include: {
    translations: {
      where: { locale: 'zh-CN' }
    }
  }
});

// 获取多语言内容
const content = await prisma.content.findUnique({
  where: { slug: 'renji' },
  include: {
    translations: true // 获取所有语言版本
  }
});
```

### 数据库连接工具

```typescript
// lib/db.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma = globalForPrisma.prisma ?? new PrismaClient();

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}

// 健康检查函数
export async function checkDatabaseHealth() {
  try {
    await prisma.$queryRaw`SELECT 1`;
    return { 
      status: 'healthy', 
      timestamp: new Date().toISOString() 
    };
  } catch (error) {
    return { 
      status: 'unhealthy', 
      error: error instanceof Error ? error.message : 'Unknown error',
      timestamp: new Date().toISOString() 
    };
  }
}
```

---

## API 设计

### 健康检查 API

```typescript
// app/api/health/route.ts
import { NextResponse } from 'next/server';
import { checkDatabaseHealth } from '@/lib/db';

export async function GET() {
  try {
    const dbHealth = await checkDatabaseHealth();
    
    const health = {
      status: 'ok',
      timestamp: new Date().toISOString(),
      version: '1.0.0',
      environment: process.env.NODE_ENV,
      database: dbHealth,
    };
    
    return NextResponse.json(health);
  } catch (error) {
    return NextResponse.json(
      { 
        status: 'error', 
        message: error instanceof Error ? error.message : 'Unknown error' 
      },
      { status: 500 }
    );
  }
}
```

---

## SEO 实施

### robots.txt

```txt
# public/robots.txt
User-agent: *
Allow: /

# 禁止访问管理页面
Disallow: /admin/
Disallow: /api/

Sitemap: https://nihaixia.com/sitemap.xml
```

### sitemap.xml 生成

```typescript
// app/sitemap.ts
import { MetadataRoute } from 'next';

export default function sitemap(): MetadataRoute.Sitemap {
  const baseUrl = 'https://nihaixia.com';
  
  return [
    {
      url: baseUrl,
      lastModified: new Date(),
      changeFrequency: 'daily',
      priority: 1,
    },
    {
      url: `${baseUrl}/tianji`,
      lastModified: new Date(),
      changeFrequency: 'weekly',
      priority: 0.8,
    },
    {
      url: `${baseUrl}/diji`,
      lastModified: new Date(),
      changeFrequency: 'weekly',
      priority: 0.8,
    },
    {
      url: `${baseUrl}/renji`,
      lastModified: new Date(),
      changeFrequency: 'weekly',
      priority: 0.8,
    },
  ];
}
```

---

## 性能优化

### Next.js 配置

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  // 启用实验性功能
  experimental: {
    optimizeCss: true,
  },
  
  // 压缩配置
  compress: true,
  
  // Headers 配置
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'X-Frame-Options',
            value: 'DENY',
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
        ],
      },
    ];
  },
};

module.exports = nextConfig;
```

---

## CI/CD 流程

### GitHub Actions 配置

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linting
        run: npm run lint
      
      - name: Run type checking
        run: npm run type-check
      
      - name: Build application
        run: npm run build
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

---

## 监控配置

### Google Analytics 集成

```typescript
// app/layout.tsx
import Script from 'next/script';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="zh-CN">
      <head>
        {process.env.NEXT_PUBLIC_GA_ID && (
          <>
            <Script
              src={`https://www.googletagmanager.com/gtag/js?id=${process.env.NEXT_PUBLIC_GA_ID}`}
              strategy="afterInteractive"
            />
            <Script id="google-analytics" strategy="afterInteractive">
              {`
                window.dataLayer = window.dataLayer || [];
                function gtag(){dataLayer.push(arguments);}
                gtag('js', new Date());
                gtag('config', '${process.env.NEXT_PUBLIC_GA_ID}');
              `}
            </Script>
          </>
        )}
      </head>
      <body>{children}</body>
    </html>
  );
}
```

---

## 安全措施

### 环境变量管理

```bash
# .env.example
# 数据库
DATABASE_URL="postgresql://username:password@host:5432/database"

# Analytics
NEXT_PUBLIC_GA_ID="G-XXXXXXXXXX"

# 应用配置
NEXT_PUBLIC_SITE_URL="https://nihaixia.com"
```

---

## 文档

### README.md

```markdown
# 倪海厦内容网站

> 倪海厦中医易学内容学习平台

## 快速开始

### 环境要求
- Node.js 18+
- PostgreSQL 14+

### 安装步骤

1. 克隆项目
```bash
git clone https://github.com/your-org/nihaixia-website.git
cd nihaixia-website
```

2. 安装依赖
```bash
npm install
```

3. 配置环境变量
```bash
cp .env.example .env.local
```

4. 初始化数据库
```bash
npx prisma migrate dev
```

5. 启动开发服务器
```bash
npm run dev
```

访问 http://localhost:3000

## 部署

项目使用 Vercel 自动部署：
- `main` 分支 → 生产环境
- Pull Request → 预览链接
```

---

**文档版本：** 1.0.0  
**创建日期：** 2024-11-07  
**最后更新：** 2024-11-07  
**文档状态：** ✅ 已完成
