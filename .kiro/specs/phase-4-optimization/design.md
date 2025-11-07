# Phase 4: 优化与测试 - 设计文档

## 设计概述

**设计目标：** 建立生产级别的质量保证体系，优化性能，加强安全，完善监控  
**技术栈：** Vitest + Playwright + Sentry + Vercel Analytics + Google Analytics 4  
**设计原则：** 性能第一、安全至上、可观测性、持续改进  

---

## 系统架构

### 监控和测试架构图

```
应用层
    ↓
├── 性能监控
│   ├── Vercel Analytics (Real User Monitoring)
│   ├── Web Vitals API
│   └── Custom Performance Metrics
│
├── 错误追踪
│   ├── Sentry (Error Tracking)
│   └── Custom Error Logging
│
├── 用户分析
│   ├── Google Analytics 4
│   └── Custom Event Tracking
│
├── 测试系统
│   ├── Vitest (Unit Tests)
│   ├── Playwright (E2E Tests)
│   └── Jest (Component Tests)
│
└── 安全防护
    ├── Rate Limiting
    ├── CSRF Protection
    ├── XSS Prevention
    └── SQL Injection Prevention
```

---

## 性能优化设计

### 1. Core Web Vitals 优化

#### LCP (Largest Contentful Paint) 优化

```typescript
// 优化策略
1. 图片优化
   - 使用 Next.js Image 组件
   - 转换为 WebP 格式
   - 实施响应式图片
   - 添加优先级加载

2. 字体优化
   - 使用 next/font 优化字体加载
   - 预加载关键字体
   - 使用 font-display: swap

3. 服务器响应优化
   - 使用 CDN
   - 实施缓存策略
   - 优化数据库查询
```

```typescript
// next.config.js
const nextConfig = {
  images: {
    formats: ['image/webp', 'image/avif'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    minimumCacheTTL: 60,
  },
  
  // 字体优化
  optimizeFonts: true,
  
  // 压缩
  compress: true,
  
  // 实验性功能
  experimental: {
    optimizeCss: true,
    optimizePackageImports: ['lucide-react', '@radix-ui/react-icons'],
  },
};
```

#### FID (First Input Delay) 优化

```typescript
// 代码分割策略
// app/articles/[slug]/page.tsx
import dynamic from 'next/dynamic';

// 延迟加载非关键组件
const CommentSection = dynamic(() => import('@/components/comments/CommentSection'), {
  loading: () => <CommentSkeleton />,
  ssr: false,
});

const ShareButtons = dynamic(() => import('@/components/share/ShareButtons'), {
  ssr: false,
});

const RelatedArticles = dynamic(() => import('@/components/articles/RelatedArticles'), {
  loading: () => <ArticlesSkeleton />,
});
```

#### CLS (Cumulative Layout Shift) 优化

```typescript
// 防止布局偏移
// components/ui/OptimizedImage.tsx
export function OptimizedImage({ src, alt, width, height }: ImageProps) {
  return (
    <div style={{ aspectRatio: `${width}/${height}` }}>
      <Image
        src={src}
        alt={alt}
        width={width}
        height={height}
        placeholder="blur"
        blurDataURL={generateBlurDataURL(width, height)}
        style={{ width: '100%', height: 'auto' }}
      />
    </div>
  );
}

// 为动态内容预留空间
export function CommentSkeleton() {
  return (
    <div className="space-y-4">
      {[1, 2, 3].map((i) => (
        <div key={i} className="flex space-x-4">
          <div className="w-10 h-10 bg-gray-200 rounded-full animate-pulse" />
          <div className="flex-1 space-y-2">
            <div className="h-4 bg-gray-200 rounded animate-pulse w-1/4" />
            <div className="h-16 bg-gray-200 rounded animate-pulse" />
          </div>
        </div>
      ))}
    </div>
  );
}
```

### 2. 缓存策略

```typescript
// lib/cache.ts
import { unstable_cache } from 'next/cache';

// 页面级缓存
export const revalidate = 3600; // 1小时

// 数据缓存
export const getCachedData = unstable_cache(
  async (key: string) => {
    return await fetchData(key);
  },
  ['data-cache'],
  {
    revalidate: 3600,
    tags: ['data'],
  }
);

// API 路由缓存
// app/api/articles/route.ts
export const dynamic = 'force-static';
export const revalidate = 3600;

export async function GET() {
  const articles = await getCachedArticles();
  
  return NextResponse.json(articles, {
    headers: {
      'Cache-Control': 'public, s-maxage=3600, stale-while-revalidate=86400',
    },
  });
}
```

### 3. 性能监控实现

```typescript
// lib/performance.ts
import { onCLS, onFID, onLCP, onFCP, onTTFB } from 'web-vitals';

export function initPerformanceMonitoring() {
  onCLS(sendToAnalytics);
  onFID(sendToAnalytics);
  onLCP(sendToAnalytics);
  onFCP(sendToAnalytics);
  onTTFB(sendToAnalytics);
}

function sendToAnalytics(metric: Metric) {
  // 发送到 Google Analytics
  if (window.gtag) {
    window.gtag('event', metric.name, {
      value: Math.round(metric.name === 'CLS' ? metric.value * 1000 : metric.value),
      event_category: 'Web Vitals',
      event_label: metric.id,
      non_interaction: true,
    });
  }
  
  // 发送到自定义分析端点
  fetch('/api/analytics/performance', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      name: metric.name,
      value: metric.value,
      id: metric.id,
      navigationType: metric.navigationType,
    }),
  });
}

// app/layout.tsx
'use client';

import { useEffect } from 'react';
import { initPerformanceMonitoring } from '@/lib/performance';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  useEffect(() => {
    initPerformanceMonitoring();
  }, []);
  
  return (
    <html>
      <body>{children}</body>
    </html>
  );
}
```

---

## 测试系统设计

### 1. 单元测试配置

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: ['./test/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'test/',
        '**/*.config.ts',
        '**/*.d.ts',
      ],
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 80,
        statements: 80,
      },
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './'),
    },
  },
});
```

```typescript
// test/setup.ts
import '@testing-library/jest-dom';
import { cleanup } from '@testing-library/react';
import { afterEach, vi } from 'vitest';

// 清理
afterEach(() => {
  cleanup();
});

// Mock Next.js router
vi.mock('next/navigation', () => ({
  useRouter: () => ({
    push: vi.fn(),
    replace: vi.fn(),
    prefetch: vi.fn(),
  }),
  usePathname: () => '/',
  useSearchParams: () => new URLSearchParams(),
}));

// Mock Next.js image
vi.mock('next/image', () => ({
  default: (props: any) => {
    return <img {...props} />;
  },
}));
```

### 2. 单元测试示例

```typescript
// components/ui/Button.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './Button';

describe('Button', () => {
  it('renders correctly', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });
  
  it('handles click events', () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    
    fireEvent.click(screen.getByText('Click me'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
  
  it('can be disabled', () => {
    const handleClick = vi.fn();
    render(<Button disabled onClick={handleClick}>Click me</Button>);
    
    const button = screen.getByText('Click me');
    expect(button).toBeDisabled();
    
    fireEvent.click(button);
    expect(handleClick).not.toHaveBeenCalled();
  });
  
  it('renders different variants', () => {
    const { rerender } = render(<Button variant="primary">Primary</Button>);
    expect(screen.getByText('Primary')).toHaveClass('bg-primary-600');
    
    rerender(<Button variant="secondary">Secondary</Button>);
    expect(screen.getByText('Secondary')).toHaveClass('bg-secondary-600');
  });
});
```

```typescript
// lib/api-client.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { apiClient } from './api-client';

describe('API Client', () => {
  beforeEach(() => {
    global.fetch = vi.fn();
  });
  
  it('makes GET requests', async () => {
    const mockData = { id: 1, title: 'Test' };
    (global.fetch as any).mockResolvedValueOnce({
      ok: true,
      json: async () => mockData,
    });
    
    const result = await apiClient.get('/api/test');
    
    expect(global.fetch).toHaveBeenCalledWith('/api/test', {
      method: 'GET',
      headers: { 'Content-Type': 'application/json' },
    });
    expect(result).toEqual(mockData);
  });
  
  it('handles errors', async () => {
    (global.fetch as any).mockResolvedValueOnce({
      ok: false,
      status: 404,
      json: async () => ({ error: 'Not found' }),
    });
    
    await expect(apiClient.get('/api/test')).rejects.toThrow('Not found');
  });
  
  it('retries on failure', async () => {
    (global.fetch as any)
      .mockRejectedValueOnce(new Error('Network error'))
      .mockRejectedValueOnce(new Error('Network error'))
      .mockResolvedValueOnce({
        ok: true,
        json: async () => ({ success: true }),
      });
    
    const result = await apiClient.get('/api/test');
    
    expect(global.fetch).toHaveBeenCalledTimes(3);
    expect(result).toEqual({ success: true });
  });
});
```

### 3. E2E 测试配置

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html'],
    ['json', { outputFile: 'test-results/results.json' }],
  ],
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
  ],
  
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

### 4. E2E 测试示例

```typescript
// e2e/auth.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test('user can sign in with Google', async ({ page }) => {
    await page.goto('/');
    
    // 点击登录按钮
    await page.click('text=登录');
    
    // 选择 Google 登录
    await page.click('text=使用 Google 登录');
    
    // 等待跳转到 Google 授权页面
    await page.waitForURL(/accounts\.google\.com/);
    
    // 在测试环境中，我们使用 mock 的 OAuth 流程
    // 实际生产中需要配置测试账号
  });
  
  test('signed in user can see profile', async ({ page, context }) => {
    // 设置已登录的 cookie
    await context.addCookies([
      {
        name: 'next-auth.session-token',
        value: 'test-session-token',
        domain: 'localhost',
        path: '/',
      },
    ]);
    
    await page.goto('/');
    
    // 验证用户信息显示
    await expect(page.locator('[data-testid="user-avatar"]')).toBeVisible();
    await expect(page.locator('text=测试用户')).toBeVisible();
  });
});
```

```typescript
// e2e/comments.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Comments', () => {
  test.beforeEach(async ({ page, context }) => {
    // 设置已登录状态
    await context.addCookies([
      {
        name: 'next-auth.session-token',
        value: 'test-session-token',
        domain: 'localhost',
        path: '/',
      },
    ]);
  });
  
  test('user can post a comment', async ({ page }) => {
    await page.goto('/articles/test-article');
    
    // 滚动到评论区
    await page.locator('[data-testid="comment-section"]').scrollIntoViewIfNeeded();
    
    // 输入评论
    await page.fill('[data-testid="comment-input"]', '这是一条测试评论');
    
    // 提交评论
    await page.click('text=发表评论');
    
    // 验证评论提交成功
    await expect(page.locator('text=等待审核')).toBeVisible();
  });
  
  test('user can reply to a comment', async ({ page }) => {
    await page.goto('/articles/test-article');
    
    // 点击回复按钮
    await page.click('[data-testid="comment-item"]:first-child >> text=回复');
    
    // 输入回复
    await page.fill('[data-testid="reply-input"]', '这是一条回复');
    
    // 提交回复
    await page.click('text=发表评论');
    
    // 验证回复提交成功
    await expect(page.locator('text=等待审核')).toBeVisible();
  });
  
  test('user can like a comment', async ({ page }) => {
    await page.goto('/articles/test-article');
    
    // 获取初始点赞数
    const likeButton = page.locator('[data-testid="like-button"]:first-child');
    const initialLikes = await likeButton.locator('text=/\\d+/').textContent();
    
    // 点击点赞
    await likeButton.click();
    
    // 验证点赞数增加
    await expect(likeButton.locator('text=/\\d+/')).not.toHaveText(initialLikes!);
  });
});
```

### 5. 多语言功能测试设计

```typescript
// e2e/i18n.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Multi-language Functionality', () => {
  test('content displays correctly in all locales', async ({ page }) => {
    // 测试简体中文
    await page.goto('/zh-CN');
    await expect(page).toHaveTitle(/泥嗨侠/);
    await expect(page.locator('nav')).toContainText('天纪');
    await expect(page.locator('nav')).toContainText('地纪');
    await expect(page.locator('nav')).toContainText('人纪');

    // 测试繁体中文
    await page.goto('/zh-TW');
    await expect(page).toHaveTitle(/泥嗨俠/);
    await expect(page.locator('nav')).toContainText('天紀');

    // 测试英文
    await page.goto('/en');
    await expect(page).toHaveTitle(/Nihaixia/);
    await expect(page.locator('nav')).toContainText('Tianji');
  });

  test('language switcher works correctly', async ({ page }) => {
    await page.goto('/zh-CN');

    // 点击语言切换器
    await page.click('[data-testid="language-switcher"]');

    // 切换到英文
    await page.click('text=English');

    // 验证URL和内容更新
    await expect(page).toHaveURL(/\/en/);
    await expect(page.locator('nav')).toContainText('Tianji');
  });

  test('user interaction components support multi-language', async ({ page }) => {
    // 测试登录按钮多语言
    await page.goto('/zh-CN');
    await expect(page.locator('[data-testid="sign-in-button"]')).toContainText('登录');

    await page.goto('/en');
    await expect(page.locator('[data-testid="sign-in-button"]')).toContainText('Sign In');

    // 测试评论区多语言
    await page.goto('/zh-CN/articles/test');
    await expect(page.locator('[data-testid="comments-section"]')).toContainText('评论');

    await page.goto('/en/articles/test');
    await expect(page.locator('[data-testid="comments-section"]')).toContainText('Comments');
  });

  test('hreflang tags are correctly configured', async ({ page }) => {
    await page.goto('/zh-CN/articles/test');

    // 验证hreflang标签
    const hreflangs = await page.locator('link[rel="alternate"]').all();
    expect(hreflangs.length).toBeGreaterThanOrEqual(3);

    // 检查所有locale都有hreflang
    const hrefs = await Promise.all(hreflangs.map(el => el.getAttribute('href')));
    expect(hrefs.some(href => href?.includes('/zh-CN/'))).toBeTruthy();
    expect(hrefs.some(href => href?.includes('/zh-TW/'))).toBeTruthy();
    expect(hrefs.some(href => href?.includes('/en/'))).toBeTruthy();
  });

  test('all translation keys have values', async ({ page }) => {
    // 访问不同页面检查是否有缺失的翻译
    const locales = ['zh-CN', 'zh-TW', 'en'];
    const pages = ['/', '/tianji', '/diji', '/renji', '/articles'];

    for (const locale of locales) {
      for (const pagePath of pages) {
        await page.goto(`/${locale}${pagePath}`);

        // 检查是否有翻译key未被替换（如 "common.title" 这样的原始key）
        const bodyText = await page.locator('body').textContent();
        expect(bodyText).not.toMatch(/[a-z]+\.[a-z]+/); // 不应包含点号分隔的key
      }
    }
  });
});
```

```typescript
// test/i18n-utils.test.ts
import { describe, it, expect } from 'vitest';
import { getTranslations, validateTranslations } from '@/lib/i18n-utils';

describe('I18n Utilities', () => {
  it('loads all translation files', async () => {
    const locales = ['zh-CN', 'zh-TW', 'en'];
    const namespaces = ['common', 'user-interaction', 'content'];

    for (const locale of locales) {
      for (const ns of namespaces) {
        const translations = await getTranslations(locale, ns);
        expect(translations).toBeDefined();
        expect(Object.keys(translations).length).toBeGreaterThan(0);
      }
    }
  });

  it('validates translation completeness', () => {
    const result = validateTranslations();

    // 所有locale应该有相同的key
    expect(result.missingKeys).toEqual([]);
    expect(result.extraKeys).toEqual([]);
  });

  it('generates correct locale routes', () => {
    const routes = generateLocaleRoutes('/articles/test');

    expect(routes).toEqual({
      'zh-CN': '/zh-CN/articles/test',
      'zh-TW': '/zh-TW/articles/test',
      'en': '/en/articles/test',
    });
  });
});
```

### 6. Content模型完整性测试设计

```typescript
// test/content-model.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import { prisma } from '@/lib/db';
import { ContentType } from '@prisma/client';

describe('Content Model Integrity', () => {
  beforeEach(async () => {
    // 清理测试数据
    await prisma.comment.deleteMany();
    await prisma.like.deleteMany();
    await prisma.contentTranslation.deleteMany();
    await prisma.content.deleteMany();
  });

  it('creates content with all ContentTypes', async () => {
    const contentTypes: ContentType[] = [
      'STATIC_PAGE',
      'ARTICLE',
      'TIANJI_PAGE',
      'DIJI_PAGE',
      'RENJI_PAGE',
    ];

    for (const type of contentTypes) {
      const content = await prisma.content.create({
        data: {
          type,
          slug: `test-${type.toLowerCase()}`,
          title: `Test ${type}`,
          published: true,
        },
      });

      expect(content.type).toBe(type);
      expect(content.slug).toBe(`test-${type.toLowerCase()}`);
    }
  });

  it('creates content with translations for all locales', async () => {
    const content = await prisma.content.create({
      data: {
        type: 'ARTICLE',
        slug: 'test-article',
        title: 'Test Article',
        published: true,
        translations: {
          create: [
            {
              locale: 'zh-CN',
              title: '测试文章',
              description: '这是一篇测试文章',
              body: '文章内容',
            },
            {
              locale: 'zh-TW',
              title: '測試文章',
              description: '這是一篇測試文章',
              body: '文章內容',
            },
            {
              locale: 'en',
              title: 'Test Article',
              description: 'This is a test article',
              body: 'Article content',
            },
          ],
        },
      },
      include: {
        translations: true,
      },
    });

    expect(content.translations).toHaveLength(3);
    expect(content.translations.map(t => t.locale)).toEqual(['zh-CN', 'zh-TW', 'en']);
  });

  it('comments associate correctly with content via contentId', async () => {
    // 创建不同类型的Content
    const article = await prisma.content.create({
      data: { type: 'ARTICLE', slug: 'article-1', title: 'Article 1', published: true },
    });

    const tianjiPage = await prisma.content.create({
      data: { type: 'TIANJI_PAGE', slug: 'tianji-1', title: 'Tianji 1', published: true },
    });

    // 创建用户（用于评论）
    const user = await prisma.user.create({
      data: { email: 'test@example.com', name: 'Test User' },
    });

    // 对不同类型的Content创建评论
    const articleComment = await prisma.comment.create({
      data: {
        content: '文章评论',
        contentId: article.id,
        userId: user.id,
        status: 'APPROVED',
      },
      include: { content: true },
    });

    const tianjiComment = await prisma.comment.create({
      data: {
        content: '天纪页面评论',
        contentId: tianjiPage.id,
        userId: user.id,
        status: 'APPROVED',
      },
      include: { content: true },
    });

    expect(articleComment.contentId).toBe(article.id);
    expect(articleComment.content.type).toBe('ARTICLE');

    expect(tianjiComment.contentId).toBe(tianjiPage.id);
    expect(tianjiComment.content.type).toBe('TIANJI_PAGE');
  });

  it('likes associate correctly with content via contentId', async () => {
    const content = await prisma.content.create({
      data: { type: 'ARTICLE', slug: 'article-1', title: 'Article 1', published: true },
    });

    const user = await prisma.user.create({
      data: { email: 'test@example.com', name: 'Test User' },
    });

    const like = await prisma.like.create({
      data: {
        contentId: content.id,
        userId: user.id,
      },
      include: { content: true },
    });

    expect(like.contentId).toBe(content.id);
    expect(like.content?.type).toBe('ARTICLE');
  });

  it('queries content by ContentType', async () => {
    // 创建不同类型的Content
    await prisma.content.createMany({
      data: [
        { type: 'ARTICLE', slug: 'article-1', title: 'Article 1', published: true },
        { type: 'ARTICLE', slug: 'article-2', title: 'Article 2', published: true },
        { type: 'TIANJI_PAGE', slug: 'tianji-1', title: 'Tianji 1', published: true },
        { type: 'DIJI_PAGE', slug: 'diji-1', title: 'Diji 1', published: true },
      ],
    });

    const articles = await prisma.content.findMany({
      where: { type: 'ARTICLE', published: true },
    });

    const tianjiPages = await prisma.content.findMany({
      where: { type: 'TIANJI_PAGE', published: true },
    });

    expect(articles).toHaveLength(2);
    expect(tianjiPages).toHaveLength(1);
    expect(articles.every(a => a.type === 'ARTICLE')).toBeTruthy();
    expect(tianjiPages.every(p => p.type === 'TIANJI_PAGE')).toBeTruthy();
  });

  it('getContents API supports all ContentTypes', async () => {
    await prisma.content.createMany({
      data: [
        { type: 'ARTICLE', slug: 'article-1', title: 'Article 1', published: true },
        { type: 'TIANJI_PAGE', slug: 'tianji-1', title: 'Tianji 1', published: true },
      ],
    });

    // 测试API
    const articlesResponse = await fetch('/api/contents?type=ARTICLE');
    const articlesData = await articlesResponse.json();
    expect(articlesData.contents.every((c: any) => c.type === 'ARTICLE')).toBeTruthy();

    const tianjiResponse = await fetch('/api/contents?type=TIANJI_PAGE');
    const tianjiData = await tianjiResponse.json();
    expect(tianjiData.contents.every((c: any) => c.type === 'TIANJI_PAGE')).toBeTruthy();
  });

  it('validates content translation completeness', async () => {
    const content = await prisma.content.create({
      data: {
        type: 'ARTICLE',
        slug: 'incomplete-article',
        title: 'Incomplete Article',
        published: true,
        translations: {
          create: [
            {
              locale: 'zh-CN',
              title: '不完整文章',
              description: '缺少其他语言',
              body: '内容',
            },
          ],
        },
      },
      include: { translations: true },
    });

    // 验证翻译完整性
    const requiredLocales = ['zh-CN', 'zh-TW', 'en'];
    const existingLocales = content.translations.map(t => t.locale);
    const missingLocales = requiredLocales.filter(l => !existingLocales.includes(l));

    expect(missingLocales).toEqual(['zh-TW', 'en']);
    expect(missingLocales.length).toBeGreaterThan(0); // 表明翻译不完整
  });
});
```

```typescript
// e2e/content-model.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Content Model E2E Tests', () => {
  test('all content types are accessible', async ({ page }) => {
    // 测试ARTICLE类型
    await page.goto('/zh-CN/articles');
    await expect(page.locator('[data-content-type="ARTICLE"]')).toHaveCount(10);

    // 测试TIANJI_PAGE类型
    await page.goto('/zh-CN/tianji');
    const tianjiContent = page.locator('[data-content-type="TIANJI_PAGE"]');
    await expect(tianjiContent).toBeVisible();

    // 测试DIJI_PAGE类型
    await page.goto('/zh-CN/diji');
    const dijiContent = page.locator('[data-content-type="DIJI_PAGE"]');
    await expect(dijiContent).toBeVisible();

    // 测试RENJI_PAGE类型
    await page.goto('/zh-CN/renji');
    const renjiContent = page.locator('[data-content-type="RENJI_PAGE"]');
    await expect(renjiContent).toBeVisible();
  });

  test('comments work on all content types', async ({ page, context }) => {
    // 登录
    await context.addCookies([
      { name: 'next-auth.session-token', value: 'test-token', domain: 'localhost', path: '/' },
    ]);

    const contentTypes = [
      { url: '/zh-CN/articles/test-article', type: 'ARTICLE' },
      { url: '/zh-CN/tianji', type: 'TIANJI_PAGE' },
      { url: '/zh-CN/diji', type: 'DIJI_PAGE' },
      { url: '/zh-CN/renji', type: 'RENJI_PAGE' },
    ];

    for (const { url, type } of contentTypes) {
      await page.goto(url);

      // 找到评论区
      const commentSection = page.locator('[data-testid="comment-section"]');
      await expect(commentSection).toBeVisible();

      // 输入评论
      await page.fill('[data-testid="comment-input"]', `测试 ${type} 的评论`);
      await page.click('text=发表评论');

      // 验证评论提交
      await expect(page.locator('text=等待审核')).toBeVisible();
    }
  });

  test('content with all translations displays correctly', async ({ page }) => {
    const testSlug = 'multilingual-content';

    // 访问不同语言版本
    await page.goto(`/zh-CN/articles/${testSlug}`);
    const zhTitle = await page.locator('h1').textContent();

    await page.goto(`/zh-TW/articles/${testSlug}`);
    const twTitle = await page.locator('h1').textContent();

    await page.goto(`/en/articles/${testSlug}`);
    const enTitle = await page.locator('h1').textContent();

    // 验证标题不同（表明翻译生效）
    expect(zhTitle).not.toBe(enTitle);
    expect(twTitle).not.toBe(enTitle);
  });
});
```

---

## 安全防护设计

### 1. API 限流实现

```typescript
// lib/rate-limit.ts
import { LRUCache } from 'lru-cache';

type RateLimitOptions = {
  interval: number;
  uniqueTokenPerInterval: number;
};

export function rateLimit(options: RateLimitOptions) {
  const tokenCache = new LRUCache({
    max: options.uniqueTokenPerInterval || 500,
    ttl: options.interval || 60000,
  });
  
  return {
    check: (limit: number, token: string) =>
      new Promise<void>((resolve, reject) => {
        const tokenCount = (tokenCache.get(token) as number[]) || [0];
        if (tokenCount[0] === 0) {
          tokenCache.set(token, tokenCount);
        }
        tokenCount[0] += 1;
        
        const currentUsage = tokenCount[0];
        const isRateLimited = currentUsage >= limit;
        
        return isRateLimited ? reject() : resolve();
      }),
  };
}

// 使用示例
// app/api/comments/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { rateLimit } from '@/lib/rate-limit';

const limiter = rateLimit({
  interval: 60 * 1000, // 1分钟
  uniqueTokenPerInterval: 500,
});

export async function POST(request: NextRequest) {
  try {
    const ip = request.ip || request.headers.get('x-forwarded-for') || 'unknown';
    
    await limiter.check(10, ip); // 每分钟最多10次请求
    
    // 处理请求...
  } catch {
    return NextResponse.json(
      { error: '请求过于频繁，请稍后再试' },
      { status: 429 }
    );
  }
}
```

### 2. CSRF 防护

```typescript
// lib/csrf.ts
import { randomBytes } from 'crypto';

export function generateCSRFToken(): string {
  return randomBytes(32).toString('hex');
}

export function verifyCSRFToken(token: string, sessionToken: string): boolean {
  return token === sessionToken;
}

// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // 对于修改数据的请求，验证 CSRF token
  if (['POST', 'PUT', 'DELETE', 'PATCH'].includes(request.method)) {
    const csrfToken = request.headers.get('x-csrf-token');
    const sessionToken = request.cookies.get('csrf-token')?.value;
    
    if (!csrfToken || !sessionToken || csrfToken !== sessionToken) {
      return NextResponse.json(
        { error: 'Invalid CSRF token' },
        { status: 403 }
      );
    }
  }
  
  return NextResponse.next();
}
```

### 3. XSS 防护

```typescript
// lib/sanitize.ts
import DOMPurify from 'isomorphic-dompurify';

export function sanitizeHTML(dirty: string): string {
  return DOMPurify.sanitize(dirty, {
    ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'u', 'a', 'ul', 'ol', 'li'],
    ALLOWED_ATTR: ['href', 'target', 'rel'],
  });
}

// 使用示例
// components/comments/CommentItem.tsx
import { sanitizeHTML } from '@/lib/sanitize';

export function CommentItem({ comment }: { comment: Comment }) {
  return (
    <div
      dangerouslySetInnerHTML={{
        __html: sanitizeHTML(comment.content),
      }}
    />
  );
}
```

---

## 错误追踪设计

### Sentry 集成

```typescript
// lib/sentry.ts
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 1.0,
  
  beforeSend(event, hint) {
    // 过滤敏感信息
    if (event.request) {
      delete event.request.cookies;
      delete event.request.headers;
    }
    
    return event;
  },
  
  integrations: [
    new Sentry.BrowserTracing(),
    new Sentry.Replay({
      maskAllText: true,
      blockAllMedia: true,
    }),
  ],
});

// 自定义错误处理
export function captureException(error: Error, context?: Record<string, any>) {
  Sentry.captureException(error, {
    extra: context,
  });
}

export function captureMessage(message: string, level: Sentry.SeverityLevel = 'info') {
  Sentry.captureMessage(message, level);
}
```

---

## AIO 优化设计

### FAQ Schema 实现

```typescript
// components/seo/FAQSchema.tsx
interface FAQItem {
  question: string;
  answer: string;
}

interface FAQSchemaProps {
  faqs: FAQItem[];
}

export function FAQSchema({ faqs }: FAQSchemaProps) {
  const schema = {
    '@context': 'https://schema.org',
    '@type': 'FAQPage',
    mainEntity: faqs.map((faq) => ({
      '@type': 'Question',
      name: faq.question,
      acceptedAnswer: {
        '@type': 'Answer',
        text: faq.answer,
      },
    })),
  };
  
  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  );
}

// 使用示例
// app/articles/[slug]/page.tsx
export default function ArticlePage({ article }: { article: Article }) {
  const faqs = extractFAQs(article.content);
  
  return (
    <>
      <FAQSchema faqs={faqs} />
      {/* 页面内容 */}
    </>
  );
}
```

---

**文档版本：** 1.1.0
**创建日期：** 2024-11-07
**最后更新：** 2025-01-07
**文档状态：** ✅ 已完成
**更新说明：**
- 添加多语言功能测试设计（Section 5）
- 添加Content模型完整性测试设计（Section 6）
- 包含E2E测试和单元测试示例
- 覆盖所有locale（zh-CN, zh-TW, en）和所有ContentType的测试
