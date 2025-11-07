# Phase 5: 扩展功能 - 设计文档

## 设计概述

**设计目标：** 完成内容翻译、完善合规、建立品牌、优化成本
**技术栈：** 基于 Phase 1 已完成的 next-i18next + CMS，添加翻译工作流管理
**设计原则：** 可扩展、合规、高效、可持续

**说明：** i18n 基础架构（next-i18next）和 CMS 已在 Phase 1 完成，本阶段聚焦于内容翻译工作流和运营功能。  

---

## 系统架构

### 整体架构图

```
应用层（基于 Phase 1 已完成的架构）
    ↓
├── 内容翻译管理（新增）
│   ├── 翻译工作流
│   ├── 翻译进度仪表板
│   ├── 翻译质量审核
│   └── ContentTranslation 管理
│
├── 法律文档（新增）
│   ├── 隐私政策（多语言）
│   ├── 服务条款（多语言）
│   └── 免责声明（多语言）
│
├── 关于页面（新增）
│   ├── 倪海厦生平
│   ├── 时间线组件
│   ├── 学术贡献
│   ├── 学生评价
│   └── Person Schema
│
├── 备份系统（新增）
│   ├── 数据库备份
│   ├── 文件备份
│   └── 备份验证
│
├── 成本监控（新增）
│   ├── Vercel 使用量
│   ├── Neon 使用量
│   └── 第三方服务费用
│
└── 外链管理（新增）
    ├── 外链追踪
    ├── 流量分析
    └── 效果评估
```

---

## 内容翻译管理设计

**说明：** i18n 基础架构（next-i18next、语言切换等）已在 Phase 1 完成，本阶段聚焦于内容翻译工作流。

### 1. 翻译进度仪表板

```typescript
// app/admin/translations/page.tsx
import { prisma } from '@/lib/db';
import { ContentType } from '@prisma/client';

interface TranslationStats {
  contentType: ContentType;
  locale: string;
  total: number;
  translated: number;
  percentage: number;
}

async function getTranslationStats(): Promise<TranslationStats[]> {
  const contentTypes: ContentType[] = [
    'STATIC_PAGE',
    'ARTICLE',
    'TIANJI_PAGE',
    'DIJI_PAGE',
    'RENJI_PAGE',
  ];
  const locales = ['zh-TW', 'en']; // 简体中文是原文，只统计翻译

  const stats: TranslationStats[] = [];

  for (const contentType of contentTypes) {
    // 获取该类型的所有内容
    const totalContent = await prisma.content.count({
      where: { type: contentType, published: true },
    });

    for (const locale of locales) {
      // 获取已翻译的内容数量
      const translatedContent = await prisma.contentTranslation.count({
        where: {
          locale,
          content: {
            type: contentType,
            published: true,
          },
        },
      });

      stats.push({
        contentType,
        locale,
        total: totalContent,
        translated: translatedContent,
        percentage: totalContent > 0 ? (translatedContent / totalContent) * 100 : 0,
      });
    }
  }

  return stats;
}

export default async function TranslationsPage() {
  const stats = await getTranslationStats();

  // 按 ContentType 分组
  const groupedStats = stats.reduce((acc, stat) => {
    if (!acc[stat.contentType]) {
      acc[stat.contentType] = [];
    }
    acc[stat.contentType].push(stat);
    return acc;
  }, {} as Record<ContentType, TranslationStats[]>);

  return (
    <main className="container mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold mb-8">翻译进度仪表板</h1>

      {/* 总体进度 */}
      <div className="grid grid-cols-1 md:grid-cols-2 gap-6 mb-8">
        <div className="bg-white rounded-lg shadow p-6">
          <h3 className="text-lg font-semibold mb-2">繁体中文翻译进度</h3>
          <div className="text-4xl font-bold text-primary-600">
            {calculateOverallProgress(stats, 'zh-TW')}%
          </div>
        </div>
        <div className="bg-white rounded-lg shadow p-6">
          <h3 className="text-lg font-semibold mb-2">英文翻译进度</h3>
          <div className="text-4xl font-bold text-primary-600">
            {calculateOverallProgress(stats, 'en')}%
          </div>
        </div>
      </div>

      {/* 详细进度 */}
      <div className="bg-white rounded-lg shadow overflow-hidden">
        <table className="min-w-full divide-y divide-gray-200">
          <thead className="bg-gray-50">
            <tr>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                内容类型
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                语言
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                进度
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                已翻译/总数
              </th>
            </tr>
          </thead>
          <tbody className="bg-white divide-y divide-gray-200">
            {Object.entries(groupedStats).map(([contentType, typeStats]) =>
              typeStats.map((stat, index) => (
                <tr key={`${contentType}-${stat.locale}`}>
                  {index === 0 && (
                    <td
                      className="px-6 py-4 whitespace-nowrap font-medium text-gray-900"
                      rowSpan={typeStats.length}
                    >
                      {formatContentType(contentType as ContentType)}
                    </td>
                  )}
                  <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                    {formatLocale(stat.locale)}
                  </td>
                  <td className="px-6 py-4 whitespace-nowrap">
                    <div className="flex items-center">
                      <div className="w-full bg-gray-200 rounded-full h-2 mr-2">
                        <div
                          className="bg-primary-600 h-2 rounded-full"
                          style={{ width: `${stat.percentage}%` }}
                        />
                      </div>
                      <span className="text-sm font-medium text-gray-700">
                        {stat.percentage.toFixed(0)}%
                      </span>
                    </div>
                  </td>
                  <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                    {stat.translated} / {stat.total}
                  </td>
                </tr>
              ))
            )}
          </tbody>
        </table>
      </div>
    </main>
  );
}

function formatContentType(type: ContentType): string {
  const map: Record<ContentType, string> = {
    STATIC_PAGE: '静态页面',
    ARTICLE: '文章',
    TIANJI_PAGE: '天纪页面',
    DIJI_PAGE: '地纪页面',
    RENJI_PAGE: '人纪页面',
  };
  return map[type] || type;
}

function formatLocale(locale: string): string {
  const map: Record<string, string> = {
    'zh-TW': '繁体中文',
    'en': '英文',
  };
  return map[locale] || locale;
}

function calculateOverallProgress(stats: TranslationStats[], locale: string): number {
  const localeStats = stats.filter(s => s.locale === locale);
  const totalContent = localeStats.reduce((sum, s) => sum + s.total, 0);
  const translatedContent = localeStats.reduce((sum, s) => sum + s.translated, 0);
  return totalContent > 0 ? Math.round((translatedContent / totalContent) * 100) : 0;
}
```

### 2. 待翻译内容列表

```typescript
// app/admin/translations/pending/page.tsx
import { prisma } from '@/lib/db';
import { ContentType } from '@prisma/client';

interface PendingTranslation {
  id: string;
  type: ContentType;
  slug: string;
  title: string;
  missingLocales: string[];
}

async function getPendingTranslations(
  contentType?: ContentType,
  locale?: string
): Promise<PendingTranslation[]> {
  // 获取所有已发布的内容
  const contents = await prisma.content.findMany({
    where: contentType ? { type: contentType, published: true } : { published: true },
    include: {
      translations: true,
    },
  });

  // 筛选出缺少翻译的内容
  const pendingTranslations: PendingTranslation[] = [];
  const requiredLocales = ['zh-TW', 'en'];

  for (const content of contents) {
    const existingLocales = content.translations.map(t => t.locale);
    const missingLocales = requiredLocales.filter(
      l => !existingLocales.includes(l)
    );

    // 如果指定了 locale，只返回缺少该 locale 的内容
    if (locale && !missingLocales.includes(locale)) {
      continue;
    }

    if (missingLocales.length > 0) {
      // 获取 zh-CN 翻译作为标题
      const zhCNTranslation = content.translations.find(t => t.locale === 'zh-CN');

      pendingTranslations.push({
        id: content.id,
        type: content.type,
        slug: content.slug,
        title: zhCNTranslation?.title || content.title,
        missingLocales: locale ? [locale] : missingLocales,
      });
    }
  }

  return pendingTranslations;
}

export default async function PendingTranslationsPage({
  searchParams,
}: {
  searchParams: { type?: ContentType; locale?: string };
}) {
  const pendingTranslations = await getPendingTranslations(
    searchParams.type,
    searchParams.locale
  );

  return (
    <main className="container mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold mb-8">待翻译内容</h1>

      {/* 筛选器 */}
      <div className="mb-6 flex gap-4">
        <select className="px-4 py-2 border rounded-lg">
          <option value="">所有内容类型</option>
          <option value="TIANJI_PAGE">天纪页面</option>
          <option value="DIJI_PAGE">地纪页面</option>
          <option value="RENJI_PAGE">人纪页面</option>
          <option value="ARTICLE">文章</option>
          <option value="STATIC_PAGE">静态页面</option>
        </select>

        <select className="px-4 py-2 border rounded-lg">
          <option value="">所有语言</option>
          <option value="zh-TW">繁体中文</option>
          <option value="en">英文</option>
        </select>
      </div>

      {/* 待翻译列表 */}
      <div className="bg-white rounded-lg shadow overflow-hidden">
        <table className="min-w-full divide-y divide-gray-200">
          <thead className="bg-gray-50">
            <tr>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                内容类型
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                标题
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                缺少翻译
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                操作
              </th>
            </tr>
          </thead>
          <tbody className="bg-white divide-y divide-gray-200">
            {pendingTranslations.map((item) => (
              <tr key={item.id}>
                <td className="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">
                  {formatContentType(item.type)}
                </td>
                <td className="px-6 py-4 text-sm text-gray-900">
                  {item.title}
                </td>
                <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                  {item.missingLocales.map(l => formatLocale(l)).join(', ')}
                </td>
                <td className="px-6 py-4 whitespace-nowrap text-sm font-medium">
                  <a
                    href={`/admin/translations/translate/${item.id}`}
                    className="text-primary-600 hover:text-primary-900"
                  >
                    开始翻译
                  </a>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </main>
  );
}
```

### 3. 翻译编辑器

```typescript
// app/admin/translations/translate/[contentId]/page.tsx
import { prisma } from '@/lib/db';
import { TranslationEditor } from '@/components/admin/TranslationEditor';

export default async function TranslatePage({
  params,
  searchParams,
}: {
  params: { contentId: string };
  searchParams: { locale: string };
}) {
  const content = await prisma.content.findUnique({
    where: { id: params.contentId },
    include: {
      translations: true,
    },
  });

  if (!content) {
    return <div>内容不存在</div>;
  }

  // 获取原文（zh-CN）
  const sourceTranslation = content.translations.find(t => t.locale === 'zh-CN');

  // 获取目标语言的翻译（如果存在）
  const targetTranslation = content.translations.find(
    t => t.locale === searchParams.locale
  );

  return (
    <main className="container mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold mb-8">
        翻译内容 - {formatLocale(searchParams.locale)}
      </h1>

      <TranslationEditor
        contentId={content.id}
        sourceLocale="zh-CN"
        targetLocale={searchParams.locale}
        sourceTranslation={sourceTranslation}
        targetTranslation={targetTranslation}
      />
    </main>
  );
}
```

```typescript
// components/admin/TranslationEditor.tsx
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';

interface TranslationEditorProps {
  contentId: string;
  sourceLocale: string;
  targetLocale: string;
  sourceTranslation: any;
  targetTranslation?: any;
}

export function TranslationEditor({
  contentId,
  sourceLocale,
  targetLocale,
  sourceTranslation,
  targetTranslation,
}: TranslationEditorProps) {
  const router = useRouter();
  const [title, setTitle] = useState(targetTranslation?.title || '');
  const [excerpt, setExcerpt] = useState(targetTranslation?.excerpt || '');
  const [content, setContent] = useState(targetTranslation?.content || '');
  const [saving, setSaving] = useState(false);

  const handleSave = async () => {
    setSaving(true);

    try {
      const response = await fetch('/api/admin/translations', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          contentId,
          locale: targetLocale,
          title,
          excerpt,
          content,
        }),
      });

      if (response.ok) {
        alert('翻译已保存');
        router.push('/admin/translations/pending');
      } else {
        alert('保存失败');
      }
    } catch (error) {
      alert('保存失败');
    } finally {
      setSaving(false);
    }
  };

  return (
    <div className="grid grid-cols-2 gap-8">
      {/* 原文 */}
      <div className="bg-gray-50 p-6 rounded-lg">
        <h2 className="text-xl font-bold mb-4">
          原文（{formatLocale(sourceLocale)}）
        </h2>

        <div className="mb-4">
          <label className="block text-sm font-medium text-gray-700 mb-2">
            标题
          </label>
          <div className="p-3 bg-white rounded border">
            {sourceTranslation?.title}
          </div>
        </div>

        <div className="mb-4">
          <label className="block text-sm font-medium text-gray-700 mb-2">
            摘要
          </label>
          <div className="p-3 bg-white rounded border">
            {sourceTranslation?.excerpt}
          </div>
        </div>

        <div>
          <label className="block text-sm font-medium text-gray-700 mb-2">
            内容
          </label>
          <div className="p-3 bg-white rounded border max-h-96 overflow-y-auto">
            <div
              dangerouslySetInnerHTML={{ __html: sourceTranslation?.content || '' }}
            />
          </div>
        </div>
      </div>

      {/* 译文 */}
      <div>
        <h2 className="text-xl font-bold mb-4">
          译文（{formatLocale(targetLocale)}）
        </h2>

        <div className="mb-4">
          <label className="block text-sm font-medium text-gray-700 mb-2">
            标题
          </label>
          <input
            type="text"
            value={title}
            onChange={(e) => setTitle(e.target.value)}
            className="w-full px-3 py-2 border rounded-lg"
            placeholder="输入翻译后的标题"
          />
        </div>

        <div className="mb-4">
          <label className="block text-sm font-medium text-gray-700 mb-2">
            摘要
          </label>
          <textarea
            value={excerpt}
            onChange={(e) => setExcerpt(e.target.value)}
            className="w-full px-3 py-2 border rounded-lg"
            rows={3}
            placeholder="输入翻译后的摘要"
          />
        </div>

        <div className="mb-6">
          <label className="block text-sm font-medium text-gray-700 mb-2">
            内容
          </label>
          <textarea
            value={content}
            onChange={(e) => setContent(e.target.value)}
            className="w-full px-3 py-2 border rounded-lg font-mono text-sm"
            rows={20}
            placeholder="输入翻译后的内容（支持 HTML）"
          />
        </div>

        <div className="flex gap-4">
          <button
            onClick={handleSave}
            disabled={saving}
            className="px-6 py-2 bg-primary-600 text-white rounded-lg hover:bg-primary-700 disabled:opacity-50"
          >
            {saving ? '保存中...' : '保存翻译'}
          </button>

          <button
            onClick={() => router.back()}
            className="px-6 py-2 bg-gray-200 text-gray-700 rounded-lg hover:bg-gray-300"
          >
            取消
          </button>
        </div>
      </div>
    </div>
  );
}
```

### 4. 翻译 API 端点

```typescript
// app/api/admin/translations/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { prisma } from '@/lib/db';
import { getServerSession } from 'next-auth';
import { authOptions } from '@/lib/auth';

export async function POST(request: NextRequest) {
  const session = await getServerSession(authOptions);

  // 验证管理员权限
  if (!session || session.user.role !== 'ADMIN') {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const { contentId, locale, title, excerpt, content } = await request.json();

  try {
    // 创建或更新翻译
    const translation = await prisma.contentTranslation.upsert({
      where: {
        contentId_locale: {
          contentId,
          locale,
        },
      },
      update: {
        title,
        excerpt,
        content,
      },
      create: {
        contentId,
        locale,
        title,
        excerpt,
        content,
      },
    });

    return NextResponse.json(translation);
  } catch (error) {
    console.error('Translation save error:', error);
    return NextResponse.json(
      { error: 'Failed to save translation' },
      { status: 500 }
    );
  }
}
```

---

## 法律文档设计

**说明：** 法律文档需要支持多语言（zh-CN, zh-TW, en），使用已完成的 i18n 基础架构。

### 1. 隐私政策页面（多语言支持）

```typescript
// app/[locale]/privacy/page.tsx
import { Metadata } from 'next';
import { prisma } from '@/lib/db';

interface PrivacyPageProps {
  params: { locale: string };
}

export async function generateMetadata({
  params,
}: PrivacyPageProps): Promise<Metadata> {
  return {
    title: getTitle(params.locale),
    robots: 'noindex, nofollow',
  };
}

function getTitle(locale: string): string {
  const titles: Record<string, string> = {
    'zh-CN': '隐私政策 - 泥嗨侠',
    'zh-TW': '隱私政策 - 泥嗨俠',
    'en': 'Privacy Policy - Nihaixia',
  };
  return titles[locale] || titles['zh-CN'];
}

export default async function PrivacyPage({ params }: PrivacyPageProps) {
  // 从数据库获取隐私政策内容（作为 Content 的一种类型）
  const privacyContent = await prisma.content.findFirst({
    where: {
      type: 'STATIC_PAGE',
      slug: 'privacy',
      published: true,
    },
    include: {
      translations: {
        where: { locale: params.locale },
      },
    },
  });

  const translation = privacyContent?.translations[0];

  if (!translation) {
    return <div>内容未找到</div>;
  }

  return (
    <main className="container mx-auto px-4 py-12 max-w-4xl">
      <h1 className="text-4xl font-bold mb-4">{translation.title}</h1>
      <p className="text-gray-600 mb-8">
        {getLastUpdatedText(params.locale)}: {formatDate(privacyContent.updatedAt, params.locale)}
      </p>

      <div
        className="prose prose-lg max-w-none"
        dangerouslySetInnerHTML={{ __html: translation.content }}
      />
    </main>
  );
}

function getLastUpdatedText(locale: string): string {
  const texts: Record<string, string> = {
    'zh-CN': '最后更新',
    'zh-TW': '最後更新',
    'en': 'Last Updated',
  };
  return texts[locale] || texts['zh-CN'];
}

function formatDate(date: Date, locale: string): string {
  return new Intl.DateTimeFormat(locale, {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  }).format(date);
}
```

### 2. 服务条款页面（多语言支持）

```typescript
// app/[locale]/terms/page.tsx
import { Metadata } from 'next';
import { prisma } from '@/lib/db';

interface TermsPageProps {
  params: { locale: string };
}

export async function generateMetadata({
  params,
}: TermsPageProps): Promise<Metadata> {
  const titles: Record<string, string> = {
    'zh-CN': '服务条款 - 泥嗨侠',
    'zh-TW': '服務條款 - 泥嗨俠',
    'en': 'Terms of Service - Nihaixia',
  };

  return {
    title: titles[params.locale] || titles['zh-CN'],
    robots: 'noindex, nofollow',
  };
}

export default async function TermsPage({ params }: TermsPageProps) {
  const termsContent = await prisma.content.findFirst({
    where: {
      type: 'STATIC_PAGE',
      slug: 'terms',
      published: true,
    },
    include: {
      translations: {
        where: { locale: params.locale },
      },
    },
  });

  const translation = termsContent?.translations[0];

  if (!translation) {
    return <div>内容未找到</div>;
  }

  return (
    <main className="container mx-auto px-4 py-12 max-w-4xl">
      <h1 className="text-4xl font-bold mb-4">{translation.title}</h1>
      <p className="text-gray-600 mb-8">
        {getLastUpdatedText(params.locale)}: {formatDate(termsContent.updatedAt, params.locale)}
      </p>

      <div
        className="prose prose-lg max-w-none"
        dangerouslySetInnerHTML={{ __html: translation.content }}
      />
    </main>
  );
}
```

### 3. 免责声明页面（多语言支持）

```typescript
// app/[locale]/disclaimer/page.tsx
import { Metadata } from 'next';
import { prisma } from '@/lib/db';

interface DisclaimerPageProps {
  params: { locale: string };
}

export async function generateMetadata({
  params,
}: DisclaimerPageProps): Promise<Metadata> {
  const titles: Record<string, string> = {
    'zh-CN': '免责声明 - 泥嗨侠',
    'zh-TW': '免責聲明 - 泥嗨俠',
    'en': 'Disclaimer - Nihaixia',
  };

  return {
    title: titles[params.locale] || titles['zh-CN'],
    robots: 'noindex, nofollow',
  };
}

export default async function DisclaimerPage({ params }: DisclaimerPageProps) {
  const disclaimerContent = await prisma.content.findFirst({
    where: {
      type: 'STATIC_PAGE',
      slug: 'disclaimer',
      published: true,
    },
    include: {
      translations: {
        where: { locale: params.locale },
      },
    },
  });

  const translation = disclaimerContent?.translations[0];

  if (!translation) {
    return <div>内容未找到</div>;
  }

  return (
    <main className="container mx-auto px-4 py-12 max-w-4xl">
      <h1 className="text-4xl font-bold mb-4">{translation.title}</h1>
      <p className="text-gray-600 mb-8">
        {getLastUpdatedText(params.locale)}: {formatDate(disclaimerContent.updatedAt, params.locale)}
      </p>

      <div
        className="prose prose-lg max-w-none"
        dangerouslySetInnerHTML={{ __html: translation.content }}
      />
    </main>
  );
}
```

### 4. 法律文档内容结构

法律文档作为 `STATIC_PAGE` 类型的 Content 存储在数据库中，需要提供三种语言版本：

**隐私政策内容大纲：**
1. 信息收集（收集的个人信息类型）
2. 信息使用（如何使用收集的信息）
3. 信息共享（与第三方共享的情况）
4. 数据安全（保护措施）
5. 用户权利（访问、更正、删除权利）
6. Cookie 使用（Cookie 类型和用途）
7. 政策更新（更新通知机制）
8. 联系方式（隐私问题联系方式）

**服务条款内容大纲：**
1. 服务说明（网站提供的服务）
2. 用户责任（用户行为规范）
3. 知识产权（内容版权声明）
4. 免责条款（责任限制）
5. 服务变更（服务修改和终止）
6. 争议解决（法律适用和管辖）
7. 条款更新（更新通知机制）
8. 联系方式（法律问题联系方式）

**免责声明内容大纲：**
1. 医疗建议免责（内容仅供参考，不构成医疗建议）
2. 内容准确性（不保证内容绝对准确）
3. 外部链接（对第三方网站不承担责任）
4. 使用风险（用户自行承担使用风险）
5. 不保证条款（对服务可用性不作保证）
6. 责任限制（损害赔偿限制）

---

## 关于页面设计

**说明：** 关于页面需要支持多语言，内容通过 ContentTranslation 提供不同语言版本。

### 页面布局（多语言支持）

```typescript
// app/[locale]/about/page.tsx
import { Metadata } from 'next';
import { prisma } from '@/lib/db';
import { Timeline } from '@/components/about/Timeline';
import { Achievements } from '@/components/about/Achievements';
import { Testimonials } from '@/components/about/Testimonials';
import { PersonSchema } from '@/components/seo/PersonSchema';

interface AboutPageProps {
  params: { locale: string };
}

export async function generateMetadata({
  params,
}: AboutPageProps): Promise<Metadata> {
  const titles: Record<string, string> = {
    'zh-CN': '关于倪海厦 - 泥嗨侠',
    'zh-TW': '關於倪海廈 - 泥嗨俠',
    'en': 'About Ni Haixia - Nihaixia',
  };

  const descriptions: Record<string, string> = {
    'zh-CN': '了解倪海厦老师的生平、学术贡献和对中医易学的影响',
    'zh-TW': '了解倪海廈老師的生平、學術貢獻和對中醫易學的影響',
    'en': 'Learn about Master Ni Haixia\'s life, academic contributions, and influence on Traditional Chinese Medicine and I Ching',
  };

  return {
    title: titles[params.locale] || titles['zh-CN'],
    description: descriptions[params.locale] || descriptions['zh-CN'],
  };
}

export default async function AboutPage({ params }: AboutPageProps) {
  // 从数据库获取关于页面内容
  const aboutContent = await prisma.content.findFirst({
    where: {
      type: 'STATIC_PAGE',
      slug: 'about',
      published: true,
    },
    include: {
      translations: {
        where: { locale: params.locale },
      },
    },
  });

  const translation = aboutContent?.translations[0];

  if (!translation) {
    return <div>内容未找到</div>;
  }

  // Person Schema 数据
  const personNames: Record<string, string> = {
    'zh-CN': '倪海厦',
    'zh-TW': '倪海廈',
    'en': 'Ni Haixia',
  };

  const personDescriptions: Record<string, string> = {
    'zh-CN': '著名中医师、易学家、教育家',
    'zh-TW': '著名中醫師、易學家、教育家',
    'en': 'Renowned Traditional Chinese Medicine Practitioner, I Ching Scholar, and Educator',
  };

  return (
    <>
      <PersonSchema
        name={personNames[params.locale] || personNames['zh-CN']}
        description={personDescriptions[params.locale] || personDescriptions['zh-CN']}
        birthDate="1954-01-01"
        deathDate="2012-01-31"
        nationality={params.locale === 'en' ? 'Chinese' : '中国'}
      />

      <main>
        {/* Hero Section */}
        <section className="relative h-96 bg-gradient-to-r from-primary-600 to-primary-800">
          <div className="container mx-auto px-4 h-full flex items-center">
            <div className="text-white">
              <h1 className="text-5xl font-bold mb-4">{translation.title}</h1>
              <p className="text-xl">{translation.excerpt}</p>
            </div>
          </div>
        </section>

        {/* Biography - 从 ContentTranslation 渲染 */}
        <section className="py-16 bg-white">
          <div className="container mx-auto px-4 max-w-4xl">
            <div
              className="prose prose-lg max-w-none"
              dangerouslySetInnerHTML={{ __html: translation.content }}
            />
          </div>
        </section>

        {/* Timeline */}
        <section className="py-16 bg-gray-50">
          <div className="container mx-auto px-4">
            <h2 className="text-3xl font-bold mb-12 text-center">
              {getSectionTitle('timeline', params.locale)}
            </h2>
            <Timeline locale={params.locale} />
          </div>
        </section>

        {/* Achievements */}
        <section className="py-16 bg-white">
          <div className="container mx-auto px-4">
            <h2 className="text-3xl font-bold mb-12 text-center">
              {getSectionTitle('achievements', params.locale)}
            </h2>
            <Achievements locale={params.locale} />
          </div>
        </section>

        {/* Testimonials */}
        <section className="py-16 bg-gray-50">
          <div className="container mx-auto px-4">
            <h2 className="text-3xl font-bold mb-12 text-center">
              {getSectionTitle('testimonials', params.locale)}
            </h2>
            <Testimonials locale={params.locale} />
          </div>
        </section>
      </main>
    </>
  );
}

function getSectionTitle(section: string, locale: string): string {
  const titles: Record<string, Record<string, string>> = {
    timeline: {
      'zh-CN': '重要事件',
      'zh-TW': '重要事件',
      'en': 'Timeline',
    },
    achievements: {
      'zh-CN': '学术贡献',
      'zh-TW': '學術貢獻',
      'en': 'Academic Achievements',
    },
    testimonials: {
      'zh-CN': '学生评价',
      'zh-TW': '學生評價',
      'en': 'Testimonials',
    },
  };

  return titles[section]?.[locale] || titles[section]?.['zh-CN'] || '';
}
```

### Timeline 组件（多语言支持）

```typescript
// components/about/Timeline.tsx
interface TimelineProps {
  locale: string;
}

interface TimelineEvent {
  year: string;
  title: Record<string, string>;
  description: Record<string, string>;
}

const events: TimelineEvent[] = [
  {
    year: '1954',
    title: {
      'zh-CN': '出生',
      'zh-TW': '出生',
      'en': 'Born',
    },
    description: {
      'zh-CN': '出生于台湾',
      'zh-TW': '出生於台灣',
      'en': 'Born in Taiwan',
    },
  },
  {
    year: '1976',
    title: {
      'zh-CN': '开始学习中医',
      'zh-TW': '開始學習中醫',
      'en': 'Began Studying TCM',
    },
    description: {
      'zh-CN': '师从多位名师，系统学习中医理论',
      'zh-TW': '師從多位名師，系統學習中醫理論',
      'en': 'Studied Traditional Chinese Medicine theory under renowned masters',
    },
  },
  {
    year: '1985',
    title: {
      'zh-CN': '创办诊所',
      'zh-TW': '創辦診所',
      'en': 'Founded Clinic',
    },
    description: {
      'zh-CN': '在美国创办中医诊所，开始临床实践',
      'zh-TW': '在美國創辦中醫診所，開始臨床實踐',
      'en': 'Founded a TCM clinic in the United States, began clinical practice',
    },
  },
  {
    year: '2000',
    title: {
      'zh-CN': '开始教学',
      'zh-TW': '開始教學',
      'en': 'Began Teaching',
    },
    description: {
      'zh-CN': '创办汉唐中医学院，培养中医人才',
      'zh-TW': '創辦漢唐中醫學院，培養中醫人才',
      'en': 'Founded Hantang College of Traditional Chinese Medicine, trained TCM practitioners',
    },
  },
  {
    year: '2012',
    title: {
      'zh-CN': '逝世',
      'zh-TW': '逝世',
      'en': 'Passed Away',
    },
    description: {
      'zh-CN': '因病逝世，享年58岁',
      'zh-TW': '因病逝世，享年58歲',
      'en': 'Passed away at age 58',
    },
  },
];

export function Timeline({ locale }: TimelineProps) {
  return (
    <div className="max-w-4xl mx-auto">
      {events.map((event, index) => (
        <div key={index} className="flex mb-8">
          <div className="flex flex-col items-center mr-4">
            <div className="w-12 h-12 bg-primary-600 rounded-full flex items-center justify-center text-white font-bold">
              {event.year}
            </div>
            {index < events.length - 1 && (
              <div className="w-0.5 h-full bg-primary-200 mt-2" />
            )}
          </div>
          <div className="flex-1 pb-8">
            <h3 className="text-xl font-bold mb-2">
              {event.title[locale] || event.title['zh-CN']}
            </h3>
            <p className="text-gray-600">
              {event.description[locale] || event.description['zh-CN']}
            </p>
          </div>
        </div>
      ))}
    </div>
  );
}
```

---

## 备份系统设计

### 1. 数据库备份脚本

```typescript
// scripts/backup-database.ts
import { exec } from 'child_process';
import { promisify } from 'util';
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';
import { createReadStream } from 'fs';
import { createGzip } from 'zlib';
import { pipeline } from 'stream/promises';

const execAsync = promisify(exec);

const s3Client = new S3Client({
  region: process.env.AWS_REGION!,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID!,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY!,
  },
});

async function backupDatabase() {
  const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
  const filename = `backup-${timestamp}.sql`;
  const gzipFilename = `${filename}.gz`;
  
  try {
    // 1. 导出数据库
    console.log('Exporting database...');
    const { stdout, stderr } = await execAsync(
      `pg_dump ${process.env.DATABASE_URL} > ${filename}`
    );
    
    if (stderr) {
      console.error('Export error:', stderr);
    }
    
    // 2. 压缩备份文件
    console.log('Compressing backup...');
    await pipeline(
      createReadStream(filename),
      createGzip(),
      createWriteStream(gzipFilename)
    );
    
    // 3. 上传到 S3
    console.log('Uploading to S3...');
    await s3Client.send(
      new PutObjectCommand({
        Bucket: process.env.BACKUP_BUCKET!,
        Key: `database/${gzipFilename}`,
        Body: createReadStream(gzipFilename),
      })
    );
    
    // 4. 验证备份
    console.log('Verifying backup...');
    const verified = await verifyBackup(gzipFilename);
    
    if (verified) {
      console.log('Backup completed successfully!');
      
      // 5. 清理本地文件
      await execAsync(`rm ${filename} ${gzipFilename}`);
      
      // 6. 记录备份
      await recordBackup({
        filename: gzipFilename,
        size: (await stat(gzipFilename)).size,
        timestamp: new Date(),
        verified: true,
      });
    } else {
      throw new Error('Backup verification failed');
    }
  } catch (error) {
    console.error('Backup failed:', error);
    
    // 发送告警
    await sendAlert({
      type: 'backup_failed',
      error: error.message,
      timestamp: new Date(),
    });
    
    throw error;
  }
}

async function verifyBackup(filename: string): Promise<boolean> {
  try {
    // 简单验证：检查文件大小
    const stats = await stat(filename);
    return stats.size > 1000; // 至少 1KB
  } catch (error) {
    return false;
  }
}

// 定时任务配置（使用 Vercel Cron 或 GitHub Actions）
// .github/workflows/backup.yml
```

### 2. 备份管理界面

```typescript
// app/admin/backups/page.tsx
import { prisma } from '@/lib/db';

export default async function BackupsPage() {
  const backups = await prisma.backup.findMany({
    orderBy: { createdAt: 'desc' },
    take: 50,
  });
  
  return (
    <main className="container mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold mb-8">数据库备份</h1>
      
      <div className="bg-white rounded-lg shadow overflow-hidden">
        <table className="min-w-full divide-y divide-gray-200">
          <thead className="bg-gray-50">
            <tr>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                文件名
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                大小
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                时间
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                状态
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                操作
              </th>
            </tr>
          </thead>
          <tbody className="bg-white divide-y divide-gray-200">
            {backups.map((backup) => (
              <tr key={backup.id}>
                <td className="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">
                  {backup.filename}
                </td>
                <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                  {formatBytes(backup.size)}
                </td>
                <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                  {formatDate(backup.createdAt)}
                </td>
                <td className="px-6 py-4 whitespace-nowrap">
                  <span className={cn(
                    "px-2 inline-flex text-xs leading-5 font-semibold rounded-full",
                    backup.verified
                      ? "bg-green-100 text-green-800"
                      : "bg-red-100 text-red-800"
                  )}>
                    {backup.verified ? '已验证' : '未验证'}
                  </span>
                </td>
                <td className="px-6 py-4 whitespace-nowrap text-sm font-medium">
                  <button className="text-primary-600 hover:text-primary-900 mr-4">
                    下载
                  </button>
                  <button className="text-red-600 hover:text-red-900">
                    删除
                  </button>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </main>
  );
}
```

---

## 成本监控设计

### 成本监控仪表板

```typescript
// app/admin/costs/page.tsx
import { getCostData } from '@/lib/costs';

export default async function CostsPage() {
  const costs = await getCostData();
  
  return (
    <main className="container mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold mb-8">成本监控</h1>
      
      {/* 总览卡片 */}
      <div className="grid grid-cols-1 md:grid-cols-4 gap-6 mb-8">
        <CostCard
          title="本月总成本"
          value={`$${costs.total.toFixed(2)}`}
          change={costs.change}
        />
        <CostCard
          title="Vercel"
          value={`$${costs.vercel.toFixed(2)}`}
          subtitle="托管和部署"
        />
        <CostCard
          title="Neon"
          value={`$${costs.neon.toFixed(2)}`}
          subtitle="数据库"
        />
        <CostCard
          title="其他服务"
          value={`$${costs.others.toFixed(2)}`}
          subtitle="第三方服务"
        />
      </div>
      
      {/* 成本趋势图 */}
      <div className="bg-white rounded-lg shadow p-6 mb-8">
        <h2 className="text-xl font-bold mb-4">成本趋势</h2>
        <CostChart data={costs.history} />
      </div>
      
      {/* 成本明细 */}
      <div className="bg-white rounded-lg shadow p-6">
        <h2 className="text-xl font-bold mb-4">成本明细</h2>
        <CostTable data={costs.breakdown} />
      </div>
    </main>
  );
}
```

---

**文档版本：** 1.1.0
**创建日期：** 2024-11-07
**最后更新：** 2025-01-07
**文档状态：** ✅ 已完成
