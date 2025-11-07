# Phase 1: 基础架构 - 任务清单

## 任务概述

**执行周期：** Week 1-3 (15个工作日)
**团队配置：** 1个全栈开发 + 1个DevOps
**工作量估算：** 90-110 人时  

---

## 任务列表

- [ ] 1. 项目初始化与配置
  - 创建 Next.js 15 项目，配置 TypeScript、ESLint、Prettier 和 Tailwind CSS
  - 设置项目目录结构
  - 配置环境变量和 Git hooks
  - _需求: Req 29_

- [ ] 1.1 创建 Next.js 15 项目
  - 使用 `create-next-app` 初始化项目，启用 TypeScript、Tailwind CSS 和 ESLint
  - 验证项目可以正常启动
  - _需求: Req 29.1_

- [ ] 1.2 配置 TypeScript 严格模式
  - 在 `tsconfig.json` 中启用严格类型检查
  - 配置路径别名 `@/` 指向项目根目录
  - _需求: Req 29.2_

- [ ] 1.3 配置代码质量工具
  - 安装并配置 ESLint 和 Prettier
  - 设置 Husky 和 lint-staged 进行提交前检查
  - _需求: Req 29.3_

- [ ] 1.4 配置 Tailwind CSS
  - 安装 `@tailwindcss/typography` 和 `@tailwindcss/forms`
  - 配置自定义颜色和字体
  - 创建全局样式文件
  - _需求: Req 29.4_

- [ ] 1.5 创建项目目录结构
  - 创建 `components/`、`lib/`、`types/` 等目录
  - 创建基础页面目录 `app/tianji/`、`app/diji/`、`app/renji/`
  - _需求: Req 28_

- [ ] 2. 数据库配置
  - 安装和配置 Prisma
  - 设计基础数据库 Schema
  - 配置 Neon 数据库连接
  - 运行数据库迁移
  - _需求: Req 33_

- [ ] 2.1 安装和初始化 Prisma
  - 安装 `prisma` 和 `@prisma/client`
  - 运行 `prisma init` 初始化配置
  - _需求: Req 33.1_

- [ ] 2.2 设计统一Content数据模型
  - 定义 `ContentType` 枚举（STATIC_PAGE, ARTICLE, TIANJI_PAGE等）
  - 创建 `Content` 主模型（替代Page/Article分离设计）
  - 创建 `ContentTranslation` 多语言翻译表
  - 更新 `Analytics` 模型关联到Content
  - 定义所有字段类型和关系
  - _需求: Req 48, Req 33.2_

- [ ] 2.3 配置 Neon 数据库连接
  - 在 Neon 创建数据库实例
  - 配置 `DATABASE_URL` 环境变量
  - 测试数据库连接
  - _需求: Req 33.1_

- [ ] 2.4 运行数据库迁移
  - 运行 `prisma migrate dev` 创建表结构
  - 运行 `prisma generate` 生成 Prisma Client
  - _需求: Req 33.2_

- [ ] 2.5 创建数据库工具函数
  - 创建 `lib/db.ts` 文件
  - 实现数据库连接单例
  - 实现健康检查函数
  - _需求: Req 33.4, Req 33.5_

- [ ] 2.6 数据模型评审与文档
  - 创建数据模型设计文档（包含ER图）
  - 进行团队评审确认Content模型设计合理性
  - 确认ContentType枚举值覆盖所有内容类型
  - 验证多语言翻译表设计满足需求
  - 编写数据模型使用示例代码
  - _需求: Req 48_

- [ ] 3. 基础组件开发
  - 创建 Layout 组件（Header、Footer、Navigation）
  - 创建基础 UI 组件（Button、Loading）
  - 创建 SEO 组件（StructuredData）
  - _需求: Req 28_

- [ ] 3.1 创建 Header 组件
  - 实现固定顶部导航栏
  - 添加品牌 Logo 和标题
  - 集成 Navigation 组件
  - _需求: Req 1_

- [ ] 3.2 创建 Navigation 组件
  - 实现导航菜单（天纪、地纪、人纪、文章、关于）
  - 添加响应式设计（移动端隐藏）
  - 添加悬停效果
  - _需求: Req 1, Req 28_

- [ ] 3.3 创建 Footer 组件
  - 实现页脚布局
  - 添加版权信息
  - _需求: Req 1_

- [ ] 3.4 创建基础 UI 组件
  - 创建 `Button` 组件（支持不同样式和大小）
  - 创建 `Loading` 组件（加载动画）
  - _需求: Req 26_

- [ ] 3.5 创建 SEO 组件
  - 创建 `StructuredData` 组件
  - 实现 WebSite Schema 生成
  - _需求: Req 47.5_

- [ ] 3.6 配置国际化(i18n)基础设施
  - 安装 `next-i18next` 依赖
  - 创建 `next-i18next.config.js` 配置文件
  - 配置支持的locale（zh-CN, zh-TW, en）
  - 创建翻译资源文件目录结构（public/locales/）
  - 为每个locale创建基础翻译文件（common.json）
  - 创建LanguageSwitcher组件
  - 配置Next.js路由支持locale前缀
  - 测试语言切换功能
  - _需求: Req 49_

- [ ] 4. 页面开发
  - 创建主页
  - 创建占位符页面（天纪、地纪、人纪）
  - 创建 404 页面
  - _需求: Req 1, Req 2, Req 3, Req 4, Req 28_

- [ ] 4.1 创建主页 (app/page.tsx)
  - 实现 Hero 区域（标题和描述）
  - 实现特色板块展示（天纪、地纪、人纪卡片）
  - 添加 SEO 元数据和结构化数据
  - _需求: Req 1, Req 47_

- [ ] 4.2 创建天纪页面 (app/tianji/page.tsx)
  - 创建占位符页面
  - 添加基本信息和导航
  - 添加 SEO 元数据
  - _需求: Req 2, Req 28, Req 47_

- [ ] 4.3 创建地纪页面 (app/diji/page.tsx)
  - 创建占位符页面
  - 添加基本信息和导航
  - 添加 SEO 元数据
  - _需求: Req 3, Req 28, Req 47_

- [ ] 4.4 创建人纪页面 (app/renji/page.tsx)
  - 创建占位符页面
  - 添加基本信息和导航
  - 添加 SEO 元数据
  - _需求: Req 4, Req 28, Req 47_

- [ ] 4.5 创建 404 页面 (app/not-found.tsx)
  - 实现友好的 404 页面
  - 添加返回主页链接
  - _需求: Req 28.5_

- [ ] 4.6 创建根布局 (app/layout.tsx)
  - 集成 Header 和 Footer
  - 添加 Google Analytics 脚本
  - 配置全局元数据
  - _需求: Req 1, Req 22, Req 47_

- [ ] 4.7 CMS选型与决策
  - 评估Strapi、Payload、自建方案的优劣
  - 对比各方案的功能、性能、成本、易用性
  - 进行团队讨论并达成共识
  - 创建CMS选型决策文档（docs/cms-decision.md）
  - 创建内容边界定义文档（docs/content-boundary.md）
  - 如果选择第三方CMS，完成基础安装和配置
  - 配置内容类型定义（对应Content模型的ContentType）
  - 测试API连接和基本功能
  - _需求: Req 51_

- [ ] 5. API 开发
  - 创建健康检查 API
  - _需求: Req 33.5_

- [ ] 5.1 创建健康检查 API (app/api/health/route.ts)
  - 实现 GET 端点
  - 返回系统状态和数据库健康状态
  - 添加错误处理
  - _需求: Req 33.5_

- [ ] 6. SEO 基础设施
  - 创建 robots.txt
  - 实现动态 sitemap.xml
  - 配置页面元数据
  - _需求: Req 47_

- [ ] 6.1 创建 robots.txt
  - 在 `public/robots.txt` 创建文件
  - 配置允许和禁止的路径
  - 添加 sitemap 链接
  - _需求: Req 47.3_

- [ ] 6.2 实现动态 sitemap (app/sitemap.ts)
  - 创建 sitemap 生成函数
  - 添加所有页面的 URL
  - 配置更新频率和优先级
  - _需求: Req 47.3_

- [ ] 6.3 配置页面元数据
  - 为每个页面添加唯一的 title 和 description
  - 确保所有页面都有 SEO 元数据
  - _需求: Req 47.1, Req 47.2_

- [ ] 7. 部署配置
  - 配置 Vercel 部署
  - 设置环境变量
  - 配置自定义域名（可选）
  - _需求: Req 35_

- [ ] 7.1 配置 Vercel 项目
  - 在 Vercel 创建新项目
  - 连接 GitHub 仓库
  - 配置构建设置
  - _需求: Req 35.1_

- [ ] 7.2 配置环境变量
  - 在 Vercel 添加 `DATABASE_URL`
  - 添加 `NEXT_PUBLIC_GA_ID`
  - 添加其他必要的环境变量
  - _需求: Req 29.5, Req 35_

- [ ] 7.3 配置自动部署
  - 设置 main 分支自动部署到生产环境
  - 设置 PR 自动生成预览链接
  - _需求: Req 35.1, Req 35.2_

- [ ]* 7.4 配置自定义域名
  - 在 Vercel 添加自定义域名
  - 配置 DNS 记录
  - 验证 SSL 证书
  - _需求: Req 35.6_

- [ ] 8. CI/CD 配置
  - 创建 GitHub Actions 工作流
  - 配置自动化测试
  - 配置代码检查
  - _需求: Req 37_

- [ ] 8.1 创建 CI 工作流 (.github/workflows/ci.yml)
  - 配置在 push 和 PR 时触发
  - 添加依赖安装步骤
  - 添加 lint 和 type-check 步骤
  - 添加构建步骤
  - _需求: Req 37.1, Req 37.2, Req 37.6_

- [ ]* 8.2 配置测试步骤
  - 添加单元测试运行步骤
  - 配置测试覆盖率报告
  - _需求: Req 37.1_

- [ ] 8.3 配置测试基础设施
  - 安装Jest和React Testing Library依赖
  - 创建jest.config.js配置文件
  - 配置测试环境（@testing-library/jest-dom）
  - 创建测试工具函数（test-utils.tsx）
  - 编写示例组件测试验证配置正确
  - 配置测试覆盖率阈值
  - 更新package.json添加测试脚本
  - _需求: Req 37_

- [ ] 9. 监控配置
  - 配置 Google Analytics 4
  - 配置 Google Search Console
  - _需求: Req 22, Req 50_

- [ ] 9.1 配置 Google Analytics 4
  - 创建 GA4 属性
  - 获取测量 ID
  - 在 `app/layout.tsx` 中集成 GA4 脚本
  - 验证数据收集
  - _需求: Req 22, Req 50.3_

- [ ] 9.2 配置 Google Search Console
  - 验证网站所有权
  - 提交 sitemap
  - 开始监控索引状态
  - _需求: Req 50.2_

- [ ] 10. 文档和验证
  - 编写 README.md
  - 创建 .env.example
  - 运行完整验收测试
  - _需求: 所有需求_

- [ ] 10.1 编写 README.md
  - 添加项目介绍
  - 添加快速开始指南
  - 添加部署说明
  - _需求: Req 29_

- [ ] 10.2 创建 .env.example
  - 列出所有必需的环境变量
  - 添加说明和示例值
  - _需求: Req 29.5_

- [ ] 10.3 运行验收测试
  - 验证所有页面可访问
  - 验证 API 端点正常工作
  - 验证 SEO 元素正确
  - 运行 Lighthouse 测试
  - 验证部署流程
  - _需求: 所有验收标准_

---

## 任务依赖关系

```
1. 项目初始化
   ↓
2. 数据库配置
   ↓
3. 基础组件开发
   ↓
4. 页面开发
   ↓
5. API 开发
   ↓
6. SEO 基础设施
   ↓
7. 部署配置 ← 8. CI/CD 配置
   ↓
9. 监控配置
   ↓
10. 文档和验证
```

---

## 验收检查清单

### 功能验收
- [ ] 可以访问主页并看到完整内容
- [ ] 可以访问天纪、地纪、人纪页面
- [ ] 导航菜单正常工作
- [ ] 404 页面正常显示
- [ ] /api/health 返回正确的健康状态

### 性能验收
- [ ] Lighthouse Performance > 90
- [ ] Lighthouse Accessibility > 95
- [ ] Lighthouse Best Practices > 95
- [ ] Lighthouse SEO > 95
- [ ] 页面加载时间 < 3秒

### SEO 验收
- [ ] 所有页面有唯一的 title
- [ ] 所有页面有 meta description
- [ ] robots.txt 可访问
- [ ] sitemap.xml 可访问
- [ ] 结构化数据正确

### 部署验收
- [ ] 推送到 main 分支自动部署
- [ ] PR 自动生成预览链接
- [ ] 部署在 5 分钟内完成
- [ ] GitHub Actions 检查通过

### 监控验收
- [ ] Google Analytics 收集数据
- [ ] Google Search Console 已验证
- [ ] 可以看到实时用户数据

---

**文档版本：** 1.0.0  
**创建日期：** 2024-11-07  
**最后更新：** 2024-11-07  
**文档状态：** ✅ 已完成
