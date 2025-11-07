# Phase 2: 核心页面 - 设计文档

## 设计概述

**设计目标：** 构建完整的内容展示系统，提供优秀的用户体验和SEO性能
**技术栈：** Next.js 15 + TypeScript + Tailwind CSS + CMS（已在Phase 1选型） + MDX + next-i18next
**设计原则：** 内容为王、性能优先、SEO友好、响应式设计、多语言支持  

---

## 系统架构

### 整体架构图

```
用户浏览器
    ↓
Vercel CDN (全球边缘节点)
    ↓
Next.js 15 App (SSR/SSG/ISR)
    ↓
├── Strapi CMS API (内容管理)
├── MDX 内容处理
├── YouTube API (视频数据)
├── Prisma ORM
│   └── Neon PostgreSQL
└── 搜索引擎 (Algolia/MeiliSearch)
```

### 页面渲染策略

| 页面类型 | 渲染方式 | 理由 |
|---------|---------|------|
| **主页** | SSG | 内容相对固定，需要最快加载速度 |
| **专题页面** | SSG + ISR | 内容更新不频繁，使用ISR保持新鲜度 |
| **文章列表** | SSR | 需要实时显示最新文章 |
| **文章详情** | SSG + ISR | 内容固定，使用ISR处理更新 |
| **搜索结果** | CSR | 需要实时交互和过滤 |

---

## 页面结构设计

### 1. 主页设计

#### 页面布局
```
┌─────────────────────────────────────┐
│          Header (固定)               │
├─────────────────────────────────────┤
│                                     │
│          Hero Section               │
│     (标题 + 描述 + CTA按钮)          │
│                                     │
├─────────────────────────────────────┤
│                                     │
│      三大板块展示区                  │
│   ┌─────┐  ┌─────┐  ┌─────┐       │
│   │天纪 │  │地纪 │  │人纪 │       │
│   └─────┘  └─────┘  └─────┘       │
│                                     │
├─────────────────────────────────────┤
│                                     │
│      五大经典医著展示区              │
│   ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐      │
│   │针│ │伤│ │金│ │本│ │内│      │
│   └──┘ └──┘ └──┘ └──┘ └──┘      │
│                                     │
├─────────────────────────────────────┤
│                                     │
│      最新文章推荐                    │
│   ┌─────────────────────┐          │
│   │ 文章1               │          │
│   ├─────────────────────┤          │
│   │ 文章2               │          │
│   └─────────────────────┘          │
│                                     │
├─────────────────────────────────────┤
│          Footer                     │
└─────────────────────────────────────┘
```

#### 组件实现
```typescript
// app/page.tsx
import { Metadata } from 'next';
import { HeroSection } from '@/components/home/HeroSection';
import { ThreePillarsSection } from '@/components/home/ThreePillarsSection';
import { ClassicBooksSection } from '@/components/home/ClassicBooksSection';
import { LatestArticles } from '@/components/home/LatestArticles';
import { WebsiteSchema } from '@/components/seo/StructuredData';

export const metadata: Metadata = {
  title: '倪海厦内容网站 - 中医易学学习平台',
  description: '学习倪海厦老师的中医、易学、命理知识，包含天纪、地纪、人纪等经典内容',
  keywords: '倪海厦,中医,易学,命理,针灸,伤寒论,金匮要略',
};

export default async function HomePage() {
  const latestArticles = await getLatestArticles(6);
  
  return (
    <>
      <WebsiteSchema
        name="倪海厦内容网站"
        url="https://nihaixia.com"
        description="倪海厦中医易学内容学习平台"
      />
      
      <main>
        <HeroSection />
        <ThreePillarsSection />
        <ClassicBooksSection />
        <LatestArticles articles={latestArticles} />
      </main>
    </>
  );
}
```

### 2. 专题页面设计（天纪/地纪/人纪）

#### 页面布局
```
┌─────────────────────────────────────┐
│          Header                     │
├─────────────────────────────────────┤
│                                     │
│      专题 Hero Banner               │
│   (背景图 + 标题 + 简介)             │
│                                     │
├─────────────────────────────────────┤
│  Sidebar    │    Main Content       │
│  ┌────────┐ │  ┌──────────────┐   │
│  │导航菜单│ │  │  内容区域    │   │
│  │        │ │  │              │   │
│  │- 章节1 │ │  │  文字 + 图片 │   │
│  │- 章节2 │ │  │              │   │
│  │- 章节3 │ │  │  视频嵌入    │   │
│  └────────┘ │  └──────────────┘   │
│             │                      │
├─────────────────────────────────────┤
│          Footer                     │
└─────────────────────────────────────┘
```

#### 数据模型（基于Phase 1的统一Content模型）

```typescript
// types/content.ts
import { ContentType } from '@prisma/client';

// 从数据库获取的内容（使用统一Content模型）
export interface ContentWithTranslation {
  id: string;
  type: ContentType; // TIANJI_PAGE | DIJI_PAGE | RENJI_PAGE | ARTICLE | STATIC_PAGE
  slug: string;
  published: boolean;
  views: number;
  createdAt: Date;
  updatedAt: Date;

  // 当前locale的翻译内容
  translation: {
    locale: string;
    title: string;
    description: string;
    body: string; // MDX或HTML内容
  };

  // 所有可用翻译的locale列表
  availableLocales: string[];
}

// 专题页面特有的扩展数据（存储在body的frontmatter中）
export interface TopicPageData {
  heroImage?: string;
  sections?: Section[];
  videos?: Video[];
  relatedContentIds?: string[]; // 关联的其他Content的ID
}

export interface Section {
  id: string;
  title: string;
  slug: string;
  content: string;
  order: number;
}

export interface Video {
  id: string;
  title: string;
  youtubeId: string;
  thumbnail: string;
  duration: string;
  description: string;
}

// API响应类型示例
export interface ContentListResponse {
  contents: ContentWithTranslation[];
  total: number;
  page: number;
  pageSize: number;
}
```

**数据模型说明**：

1. **统一使用Content模型**：所有页面类型（专题页、文章、静态页）都使用同一个Content表
2. **ContentType区分**：通过type字段区分不同内容类型（TIANJI_PAGE, DIJI_PAGE, RENJI_PAGE等）
3. **多语言支持**：通过ContentTranslation关联表实现，API返回当前locale的翻译
4. **扩展数据**：专题页特有的数据（如sections, videos）存储在body的MDX frontmatter中
5. **关联内容**：使用relatedContentIds存储关联内容的ID，查询时再获取详情


### 3. 文章列表页面设计

#### 页面布局
```
┌─────────────────────────────────────┐
│          Header                     │
├─────────────────────────────────────┤
│                                     │
│      搜索和筛选区                    │
│   [搜索框] [分类] [标签] [排序]     │
│                                     │
├─────────────────────────────────────┤
│                                     │
│      文章列表                        │
│   ┌─────────────────────────────┐  │
│   │ [图片] 文章标题              │  │
│   │        文章摘要...           │  │
│   │        [分类] [日期] [阅读量]│  │
│   └─────────────────────────────┘  │
│   ┌─────────────────────────────┐  │
│   │ [图片] 文章标题              │  │
│   │        文章摘要...           │  │
│   └─────────────────────────────┘  │
│                                     │
│      [加载更多] 或 [分页]           │
│                                     │
├─────────────────────────────────────┤
│          Footer                     │
└─────────────────────────────────────┘
```

#### 组件实现
```typescript
// app/[locale]/articles/page.tsx (支持多语言路由)
import { Metadata } from 'next';
import { ContentList } from '@/components/content/ContentList';
import { ContentFilters } from '@/components/content/ContentFilters';
import { getContents } from '@/lib/api/content';
import { serverSideTranslations } from 'next-i18next/serverSideTranslations';

export const metadata: Metadata = {
  title: '文章列表 - 倪海厦内容网站',
  description: '浏览倪海厦老师的中医易学文章，学习传统文化知识',
};

interface PageProps {
  params: { locale: string };
  searchParams: {
    category?: string;
    tag?: string;
    search?: string;
    page?: string;
  };
}

export default async function ArticlesPage({ params, searchParams }: PageProps) {
  const page = parseInt(searchParams.page || '1');

  // 使用统一的getContents API，通过type参数过滤文章类型
  const { contents, total } = await getContents({
    type: 'ARTICLE', // 只获取文章类型的内容
    locale: params.locale,
    category: searchParams.category,
    tag: searchParams.tag,
    search: searchParams.search,
    page,
    limit: 12,
  });

  return (
    <main className="container mx-auto px-4 py-8">
      <h1 className="text-4xl font-bold mb-8">文章列表</h1>

      <ContentFilters
        currentCategory={searchParams.category}
        currentTag={searchParams.tag}
      />

      <ContentList
        contents={contents}
        total={total}
        currentPage={page}
      />
    </main>
  );
}
```

**重要变更说明**：

1. **路由支持多语言**：路径从`/articles`改为`/[locale]/articles`
2. **API调用统一**：使用`getContents()`替代`getArticles()`，通过`type: 'ARTICLE'`参数过滤
3. **组件命名统一**：`ArticleList`→`ContentList`，`ArticleFilters`→`ContentFilters`
4. **传递locale参数**：确保API返回对应语言的翻译内容

### 4. 文章详情页面设计

#### 页面布局
```
┌─────────────────────────────────────┐
│          Header                     │
├─────────────────────────────────────┤
│                                     │
│      文章标题                        │
│      [分类] [作者] [日期] [阅读量]  │
│                                     │
├─────────────────────────────────────┤
│  Sidebar    │    Article Content    │
│  ┌────────┐ │  ┌──────────────┐   │
│  │目录    │ │  │  正文内容    │   │
│  │        │ │  │              │   │
│  │- 标题1 │ │  │  段落 + 图片 │   │
│  │- 标题2 │ │  │              │   │
│  │- 标题3 │ │  │  代码块      │   │
│  └────────┘ │  │              │   │
│             │  │  引用块      │   │
│  ┌────────┐ │  └──────────────┘   │
│  │相关文章│ │                      │
│  └────────┘ │  [分享按钮]          │
│             │  [标签]              │
├─────────────────────────────────────┤
│          Footer                     │
└─────────────────────────────────────┘
```

#### MDX 内容处理
```typescript
// lib/mdx.ts
import { compileMDX } from 'next-mdx-remote/rsc';
import rehypeHighlight from 'rehype-highlight';
import rehypeSlug from 'rehype-slug';
import rehypeAutolinkHeadings from 'rehype-autolink-headings';
import remarkGfm from 'remark-gfm';

export async function processMDX(content: string) {
  const { content: processedContent, frontmatter } = await compileMDX({
    source: content,
    options: {
      parseFrontmatter: true,
      mdxOptions: {
        remarkPlugins: [remarkGfm],
        rehypePlugins: [
          rehypeHighlight,
          rehypeSlug,
          [rehypeAutolinkHeadings, { behavior: 'wrap' }],
        ],
      },
    },
  });
  
  return { content: processedContent, frontmatter };
}
```

---

## 组件库设计

### 1. 内容展示组件

#### Card 组件
```typescript
// components/ui/Card.tsx
interface CardProps {
  title: string;
  description: string;
  image?: string;
  href: string;
  category?: string;
  date?: Date;
  variant?: 'default' | 'featured' | 'compact';
}

export function Card({
  title,
  description,
  image,
  href,
  category,
  date,
  variant = 'default',
}: CardProps) {
  return (
    <Link
      href={href}
      className={cn(
        "group block overflow-hidden rounded-lg border border-gray-200 bg-white shadow-sm transition-all hover:shadow-md",
        variant === 'featured' && "md:col-span-2",
        variant === 'compact' && "flex flex-row"
      )}
    >
      {image && (
        <div className={cn(
          "relative overflow-hidden",
          variant === 'compact' ? "w-32 h-32" : "aspect-video"
        )}>
          <Image
            src={image}
            alt={title}
            fill
            className="object-cover transition-transform group-hover:scale-105"
          />
        </div>
      )}
      
      <div className="p-4">
        {category && (
          <span className="text-xs font-medium text-primary-600 uppercase">
            {category}
          </span>
        )}
        
        <h3 className={cn(
          "font-bold text-gray-900 group-hover:text-primary-600 transition-colors",
          variant === 'featured' ? "text-2xl mt-2" : "text-lg mt-1"
        )}>
          {title}
        </h3>
        
        <p className="mt-2 text-sm text-gray-600 line-clamp-2">
          {description}
        </p>
        
        {date && (
          <time className="mt-2 block text-xs text-gray-500">
            {formatDate(date)}
          </time>
        )}
      </div>
    </Link>
  );
}
```

#### VideoPlayer 组件
```typescript
// components/ui/VideoPlayer.tsx
'use client';

import { useState } from 'react';
import Image from 'next/image';

interface VideoPlayerProps {
  youtubeId: string;
  title: string;
  thumbnail?: string;
}

export function VideoPlayer({ youtubeId, title, thumbnail }: VideoPlayerProps) {
  const [isPlaying, setIsPlaying] = useState(false);
  
  const thumbnailUrl = thumbnail || `https://img.youtube.com/vi/${youtubeId}/maxresdefault.jpg`;
  
  if (!isPlaying) {
    return (
      <div className="relative aspect-video rounded-lg overflow-hidden cursor-pointer group">
        <Image
          src={thumbnailUrl}
          alt={title}
          fill
          className="object-cover"
        />
        <div className="absolute inset-0 bg-black/30 flex items-center justify-center group-hover:bg-black/40 transition-colors">
          <button
            onClick={() => setIsPlaying(true)}
            className="w-20 h-20 bg-red-600 rounded-full flex items-center justify-center hover:bg-red-700 transition-colors"
            aria-label="播放视频"
          >
            <svg className="w-8 h-8 text-white ml-1" fill="currentColor" viewBox="0 0 24 24">
              <path d="M8 5v14l11-7z" />
            </svg>
          </button>
        </div>
      </div>
    );
  }
  
  return (
    <div className="relative aspect-video rounded-lg overflow-hidden">
      <iframe
        src={`https://www.youtube.com/embed/${youtubeId}?autoplay=1`}
        title={title}
        allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
        allowFullScreen
        className="absolute inset-0 w-full h-full"
      />
    </div>
  );
}
```

### 2. 导航组件

#### Breadcrumb 组件
```typescript
// components/ui/Breadcrumb.tsx
interface BreadcrumbItem {
  label: string;
  href?: string;
}

interface BreadcrumbProps {
  items: BreadcrumbItem[];
}

export function Breadcrumb({ items }: BreadcrumbProps) {
  return (
    <nav aria-label="面包屑导航" className="mb-4">
      <ol className="flex items-center space-x-2 text-sm">
        {items.map((item, index) => (
          <li key={index} className="flex items-center">
            {index > 0 && (
              <svg className="w-4 h-4 mx-2 text-gray-400" fill="currentColor" viewBox="0 0 20 20">
                <path fillRule="evenodd" d="M7.293 14.707a1 1 0 010-1.414L10.586 10 7.293 6.707a1 1 0 011.414-1.414l4 4a1 1 0 010 1.414l-4 4a1 1 0 01-1.414 0z" clipRule="evenodd" />
              </svg>
            )}
            
            {item.href ? (
              <Link
                href={item.href}
                className="text-gray-600 hover:text-primary-600 transition-colors"
              >
                {item.label}
              </Link>
            ) : (
              <span className="text-gray-900 font-medium">{item.label}</span>
            )}
          </li>
        ))}
      </ol>
    </nav>
  );
}
```

#### TableOfContents 组件
```typescript
// components/ui/TableOfContents.tsx
'use client';

import { useEffect, useState } from 'react';

interface Heading {
  id: string;
  text: string;
  level: number;
}

export function TableOfContents() {
  const [headings, setHeadings] = useState<Heading[]>([]);
  const [activeId, setActiveId] = useState<string>('');
  
  useEffect(() => {
    const elements = Array.from(document.querySelectorAll('h2, h3, h4'));
    const headingData = elements.map((element) => ({
      id: element.id,
      text: element.textContent || '',
      level: parseInt(element.tagName.substring(1)),
    }));
    setHeadings(headingData);
    
    // 监听滚动，高亮当前标题
    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            setActiveId(entry.target.id);
          }
        });
      },
      { rootMargin: '-100px 0px -80% 0px' }
    );
    
    elements.forEach((element) => observer.observe(element));
    
    return () => observer.disconnect();
  }, []);
  
  return (
    <nav className="sticky top-24">
      <h3 className="text-sm font-bold text-gray-900 mb-4">目录</h3>
      <ul className="space-y-2 text-sm">
        {headings.map((heading) => (
          <li
            key={heading.id}
            style={{ paddingLeft: `${(heading.level - 2) * 12}px` }}
          >
            <a
              href={`#${heading.id}`}
              className={cn(
                "block py-1 transition-colors hover:text-primary-600",
                activeId === heading.id
                  ? "text-primary-600 font-medium"
                  : "text-gray-600"
              )}
            >
              {heading.text}
            </a>
          </li>
        ))}
      </ul>
    </nav>
  );
}
```

---

## 内容管理实现

> **注意**：CMS方案已在Phase 1选型完成。本章节说明如何使用选定的CMS（假设为Strapi，如选择其他方案请相应调整）。

### CMS Content Types 配置（基于统一Content模型）

**重要原则**：CMS中的内容类型需要与数据库中的Content模型保持一致。

#### 方案A：CMS直接使用数据库（推荐）

如果CMS（如Strapi）配置为直接使用Neon PostgreSQL数据库，则无需在CMS中重新定义Content Types，直接使用Phase 1定义的Prisma Schema即可。

#### 方案B：CMS作为独立数据源

如果CMS使用独立数据库，需要在CMS中定义对应的Content Type，然后通过同步脚本将数据同步到主数据库。

```javascript
// strapi/api/content/content-types/content/schema.json
{
  "kind": "collectionType",
  "collectionName": "contents",
  "info": {
    "singularName": "content",
    "pluralName": "contents",
    "displayName": "Content"
  },
  "options": {
    "draftAndPublish": true
  },
  "attributes": {
    "type": {
      "type": "enumeration",
      "enum": ["STATIC_PAGE", "ARTICLE", "TIANJI_PAGE", "DIJI_PAGE", "RENJI_PAGE"],
      "required": true
    },
    "slug": {
      "type": "uid",
      "required": true
    },
    "published": {
      "type": "boolean",
      "default": false
    },
    "views": {
      "type": "integer",
      "default": 0
    },
    "translations": {
      "type": "component",
      "repeatable": true,
      "component": "content.translation"
    }
  }
}

// strapi/components/content/translation.json
{
  "collectionName": "components_content_translations",
  "info": {
    "displayName": "Translation",
    "description": "Content translation for different locales"
  },
  "attributes": {
    "locale": {
      "type": "enumeration",
      "enum": ["zh-CN", "zh-TW", "en"],
      "required": true
    },
    "title": {
      "type": "string",
      "required": true
    },
    "description": {
      "type": "text"
    },
    "body": {
      "type": "richtext",
      "required": true
    }
  }
}
```

**优势对比**：

| 方案 | 优点 | 缺点 | 推荐度 |
|------|------|------|--------|
| **方案A（共享数据库）** | 无需同步、数据一致性高 | CMS配置相对复杂 | ⭐⭐⭐⭐⭐ |
| **方案B（独立数据库）** | CMS配置简单、独立性强 | 需要同步脚本、可能数据不一致 | ⭐⭐⭐ |

#### API 客户端
```typescript
// lib/strapi.ts
const STRAPI_URL = process.env.STRAPI_URL || 'http://localhost:1337';
const STRAPI_TOKEN = process.env.STRAPI_TOKEN;

export async function fetchAPI(path: string, options: RequestInit = {}) {
  const url = `${STRAPI_URL}/api${path}`;
  
  const response = await fetch(url, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      Authorization: `Bearer ${STRAPI_TOKEN}`,
      ...options.headers,
    },
  });
  
  if (!response.ok) {
    throw new Error(`API error: ${response.status}`);
  }
  
  return response.json();
}

// 统一的内容获取API（替代原getArticles）
export async function getContents(params: {
  type?: ContentType; // 内容类型过滤（ARTICLE, TIANJI_PAGE等）
  locale?: string;    // 语言过滤
  category?: string;
  tag?: string;
  search?: string;
  page?: number;
  limit?: number;
}) {
  const query = new URLSearchParams();

  // 按内容类型过滤
  if (params.type) {
    query.append('filters[type][$eq]', params.type);
  }

  // 按locale过滤翻译
  if (params.locale) {
    query.append('filters[translations][locale][$eq]', params.locale);
  }

  if (params.category) {
    query.append('filters[category][slug][$eq]', params.category);
  }

  if (params.tag) {
    query.append('filters[tags][slug][$eq]', params.tag);
  }

  if (params.search) {
    query.append('filters[$or][0][translations][title][$containsi]', params.search);
    query.append('filters[$or][1][translations][description][$containsi]', params.search);
  }

  query.append('pagination[page]', String(params.page || 1));
  query.append('pagination[pageSize]', String(params.limit || 12));
  query.append('populate[translations][filters][locale][$eq]', params.locale || 'zh-CN');
  query.append('sort', 'createdAt:desc');

  const data = await fetchAPI(`/contents?${query.toString()}`);

  return {
    contents: data.data.map(transformContentResponse),
    total: data.meta.pagination.total,
    pageCount: data.meta.pagination.pageCount,
  };
}

// 根据slug和locale获取单个内容
export async function getContentBySlug(slug: string, locale: string = 'zh-CN') {
  const data = await fetchAPI(
    `/contents?filters[slug][$eq]=${slug}&populate[translations][filters][locale][$eq]=${locale}`
  );

  if (!data.data || data.data.length === 0) {
    return null;
  }

  return transformContentResponse(data.data[0]);
}

// 转换CMS响应为统一格式
function transformContentResponse(cmsData: any): ContentWithTranslation {
  const translation = cmsData.attributes.translations?.[0];

  return {
    id: cmsData.id,
    type: cmsData.attributes.type,
    slug: cmsData.attributes.slug,
    published: cmsData.attributes.published,
    views: cmsData.attributes.views,
    createdAt: new Date(cmsData.attributes.createdAt),
    updatedAt: new Date(cmsData.attributes.updatedAt),
    translation: translation ? {
      locale: translation.locale,
      title: translation.title,
      description: translation.description,
      body: translation.body,
    } : null,
    availableLocales: cmsData.attributes.translations?.map((t: any) => t.locale) || [],
  };
}
```

---

## SEO 优化设计

### 1. 元数据生成（支持多语言）

```typescript
// lib/seo.ts
import { Metadata } from 'next';
import { ContentWithTranslation, ContentType } from '@/types/content';

export function generateContentMetadata(
  content: ContentWithTranslation,
  locale: string
): Metadata {
  const title = `${content.translation.title} - 倪海厦内容网站`;
  const description = content.translation.description || content.translation.body.substring(0, 160);

  // 根据内容类型生成URL路径
  const pathPrefix = getPathByContentType(content.type);
  const url = `https://nihaixia.com/${locale}/${pathPrefix}/${content.slug}`;
  const image = '/og-image.jpg'; // 可从frontmatter中提取

  return {
    title,
    description,
    openGraph: {
      title,
      description,
      url,
      type: content.type === 'ARTICLE' ? 'article' : 'website',
      publishedTime: content.createdAt.toISOString(),
      modifiedTime: content.updatedAt.toISOString(),
      images: [{ url: image }],
      locale: locale,
    },
    twitter: {
      card: 'summary_large_image',
      title,
      description,
      images: [image],
    },
    alternates: {
      canonical: url,
      languages: buildLanguageAlternates(content, pathPrefix),
    },
  };
}

// 根据ContentType获取URL路径前缀
function getPathByContentType(type: ContentType): string {
  const pathMap: Record<ContentType, string> = {
    'ARTICLE': 'articles',
    'TIANJI_PAGE': 'tianji',
    'DIJI_PAGE': 'diji',
    'RENJI_PAGE': 'renji',
    'STATIC_PAGE': 'pages',
  };
  return pathMap[type] || 'contents';
}

// 构建多语言hreflang标签
function buildLanguageAlternates(
  content: ContentWithTranslation,
  pathPrefix: string
) {
  const baseUrl = 'https://nihaixia.com';
  const alternates: Record<string, string> = {};

  content.availableLocales.forEach(locale => {
    alternates[locale] = `${baseUrl}/${locale}/${pathPrefix}/${content.slug}`;
  });

  return alternates;
}
```

### 2. 结构化数据（基于Content模型）

```typescript
// components/seo/ContentSchema.tsx
interface ContentSchemaProps {
  content: ContentWithTranslation;
}

export function ContentSchema({ content }: ContentSchemaProps) {
  // 根据内容类型选择Schema类型
  const schemaType = content.type === 'ARTICLE' ? 'Article' : 'WebPage';

  const schema = {
    '@context': 'https://schema.org',
    '@type': schemaType,
    headline: content.translation.title,
    description: content.translation.description,
    inLanguage: content.translation.locale,
    datePublished: content.createdAt.toISOString(),
    dateModified: content.updatedAt.toISOString(),
    author: {
      '@type': 'Person',
      name: '倪海厦',
    },
    publisher: {
      '@type': 'Organization',
      name: '倪海厦内容网站',
      logo: {
        '@type': 'ImageObject',
        url: 'https://nihaixia.com/logo.png',
      },
    },
    mainEntityOfPage: {
      '@type': 'WebPage',
      '@id': `https://nihaixia.com/${content.translation.locale}/${getPathByContentType(content.type)}/${content.slug}`,
    },
  };
  
  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  );
}
```

### 3. pSEO 页面生成

```typescript
// lib/pseo.ts
export async function generatePSEOPages() {
  const keywords = [
    '中医基础理论',
    '针灸穴位',
    '伤寒论方剂',
    '金匮要略',
    '本草药性',
    // ... 更多关键词
  ];
  
  const templates = {
    guide: (keyword: string) => ({
      title: `${keyword}完整指南 - 倪海厦内容网站`,
      description: `学习${keyword}的完整指南，包含理论、实践和案例分析`,
      content: generateGuideContent(keyword),
    }),
    tutorial: (keyword: string) => ({
      title: `${keyword}教程 - 从入门到精通`,
      description: `${keyword}的系统教程，适合初学者和进阶学习者`,
      content: generateTutorialContent(keyword),
    }),
  };
  
  const pages = [];
  
  for (const keyword of keywords) {
    pages.push({
      slug: `guide-${slugify(keyword)}`,
      ...templates.guide(keyword),
    });
    
    pages.push({
      slug: `tutorial-${slugify(keyword)}`,
      ...templates.tutorial(keyword),
    });
  }
  
  return pages;
}
```

---

## 性能优化设计

### 1. 图片优化

```typescript
// components/ui/OptimizedImage.tsx
import Image from 'next/image';

interface OptimizedImageProps {
  src: string;
  alt: string;
  width?: number;
  height?: number;
  priority?: boolean;
  className?: string;
}

export function OptimizedImage({
  src,
  alt,
  width,
  height,
  priority = false,
  className,
}: OptimizedImageProps) {
  return (
    <Image
      src={src}
      alt={alt}
      width={width}
      height={height}
      priority={priority}
      className={className}
      sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
      quality={85}
      placeholder="blur"
      blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZJRg..."
    />
  );
}
```

### 2. 代码分割

```typescript
// 动态导入重型组件
import dynamic from 'next/dynamic';

const VideoPlayer = dynamic(() => import('@/components/ui/VideoPlayer'), {
  loading: () => <div className="aspect-video bg-gray-200 animate-pulse" />,
  ssr: false,
});

const CommentSection = dynamic(() => import('@/components/comments/CommentSection'), {
  loading: () => <div>加载评论中...</div>,
  ssr: false,
});
```

### 3. 缓存策略

```typescript
// lib/cache.ts
import { unstable_cache } from 'next/cache';

export const getCachedArticles = unstable_cache(
  async (category?: string) => {
    return await getArticles({ category });
  },
  ['articles'],
  {
    revalidate: 3600, // 1小时
    tags: ['articles'],
  }
);

export const getCachedArticle = unstable_cache(
  async (slug: string) => {
    return await getArticleBySlug(slug);
  },
  ['article'],
  {
    revalidate: 3600,
    tags: ['article'],
  }
);
```

---

## 响应式设计

### Tailwind 断点配置

```javascript
// tailwind.config.js
module.exports = {
  theme: {
    screens: {
      'xs': '475px',
      'sm': '640px',
      'md': '768px',
      'lg': '1024px',
      'xl': '1280px',
      '2xl': '1536px',
    },
  },
};
```

### 响应式组件示例

```typescript
// components/layout/ResponsiveGrid.tsx
export function ResponsiveGrid({ children }: { children: React.ReactNode }) {
  return (
    <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
      {children}
    </div>
  );
}
```

---

**文档版本：** 1.0.0  
**创建日期：** 2024-11-07  
**最后更新：** 2024-11-07  
**文档状态：** ✅ 已完成
