# Phase 3: 用户交互 - 需求文档

## 阶段概述

**阶段名称：** Phase 3 - 用户交互  
**实施周期：** Week 7-9 (3周)  
**团队规模：** 2-3 人（1个全栈开发 + 1个前端开发 + 1个UI/UX设计师）  
**预算估算：** $6,000 - $9,000  

### 🎯 阶段目标

实现用户认证、评论、分享等交互功能，让网站从静态展示变成活跃的社区平台，提升用户参与度和留存率。

### 📋 核心价值

1. **用户参与**：用户可以登录、评论、互动
2. **社区建设**：形成学习交流的社区氛围
3. **内容传播**：通过分享功能扩大影响力
4. **用户留存**：通过互动功能提升用户粘性
5. **转化优化**：优化用户转化路径

### 🎨 预期成果

**用户视角：**
- 可以通过 Google 账号快速登录
- 可以在文章下方发表评论和回复
- 可以点赞评论和文章
- 可以分享内容到社交平台
- 可以收藏喜欢的文章

**开发者视角：**
- 完整的用户认证系统（NextAuth.js + JWT）
- 评论系统和审核机制（基于统一Content模型）
- 状态管理方案（Zustand + SWR）
- API 集成完善（支持多语言内容）
- 用户交互界面的多语言支持（基于Phase 1的i18n配置）

**运营视角：**
- 可以审核用户评论
- 可以查看用户互动数据
- 可以管理用户权限

---

## 从主需求文档提取的相关需求

### Req 34: 用户认证与授权
**优先级：** P0  
**用户故事：** 作为用户，我希望能够登录网站，以便参与评论和互动

**验收标准：**
1. WHEN User点击登录按钮，THE System SHALL显示登录选项（Google OAuth）
2. WHEN User选择Google登录，THE System SHALL跳转到Google授权页面
3. WHEN User授权成功，THE System SHALL创建或更新用户账号
4. WHEN User登录成功，THE System SHALL在导航栏显示用户头像和名称
5. WHEN User点击头像，THE System SHALL显示用户菜单（个人中心、退出登录）
6. WHEN User退出登录，THE System SHALL清除会话并返回未登录状态
7. WHEN System验证用户，THE System SHALL使用JWT token进行身份验证

### Req 15: 用户评论与互动
**优先级：** P0  
**用户故事：** 作为学习者，我希望能够在文章下方评论和讨论，以便与其他学习者交流

**验收标准：**
1. WHEN User查看文章，THE System SHALL在文章底部显示评论区
2. WHEN User已登录，THE System SHALL显示评论输入框
3. WHEN User未登录，THE System SHALL显示"登录后评论"提示
4. WHEN User提交评论，THE System SHALL保存评论并显示"等待审核"状态
5. WHEN User回复评论，THE System SHALL创建嵌套回复
6. WHEN User点赞评论，THE System SHALL增加点赞数并记录用户点赞
7. WHEN Admin审核评论，THE System SHALL更新评论状态为"已发布"或"已拒绝"

### Req 16: 社交分享功能
**优先级：** P1  
**用户故事：** 作为用户，我希望能够分享文章到社交平台，以便与朋友分享学习内容

**验收标准：**
1. WHEN User查看文章，THE System SHALL显示分享按钮
2. WHEN User点击微信分享，THE System SHALL显示二维码供扫描分享
3. WHEN User点击微博分享，THE System SHALL打开微博分享窗口
4. WHEN User点击Twitter分享，THE System SHALL打开Twitter分享窗口
5. WHEN User点击复制链接，THE System SHALL复制文章URL到剪贴板
6. WHEN User分享成功，THE System SHALL记录分享事件
7. WHEN System生成分享链接，THE System SHALL包含UTM参数用于追踪

### Req 25: 状态管理与数据流
**优先级：** P1  
**用户故事：** 作为开发者，我希望有清晰的状态管理方案，以便管理复杂的应用状态

**验收标准：**
1. WHEN Application初始化，THE System SHALL使用Zustand管理全局状态
2. WHEN User登录，THE System SHALL更新用户状态到全局store
3. WHEN Component需要用户数据，THE System SHALL从store获取而不是prop drilling
4. WHEN API请求数据，THE System SHALL使用SWR进行数据获取和缓存
5. WHEN数据更新，THE System SHALL自动重新验证和更新UI
6. WHEN网络断开，THE System SHALL使用缓存数据并显示离线提示
7. WHEN状态变化，THE System SHALL只重新渲染相关组件

### Req 26: UI组件库架构
**优先级：** P1  
**用户故事：** 作为开发者，我希望有完善的UI组件库，以便快速构建一致的用户界面

**验收标准：**
1. WHEN Developer创建新页面，THE System SHALL提供可复用的UI组件
2. WHEN Component渲染，THE System SHALL支持不同的变体和尺寸
3. WHEN Component交互，THE System SHALL提供一致的交互反馈
4. WHEN Component使用，THE System SHALL提供TypeScript类型支持
5. WHEN Component样式，THE System SHALL使用Tailwind CSS类名
6. WHEN Component测试，THE System SHALL提供单元测试
7. WHEN Component文档，THE System SHALL提供使用示例和API文档

### Req 27: API集成与网络请求
**优先级：** P1  
**用户故事：** 作为开发者，我希望有统一的API请求方案，以便处理网络请求和错误

**验收标准：**
1. WHEN Application发起API请求，THE System SHALL使用统一的API客户端
2. WHEN API请求失败，THE System SHALL自动重试最多3次
3. WHEN API返回错误，THE System SHALL显示友好的错误提示
4. WHEN API请求中，THE System SHALL显示加载状态
5. WHEN API响应慢，THE System SHALL在3秒后显示超时提示
6. WHEN API需要认证，THE System SHALL自动添加认证token
7. WHEN Token过期，THE System SHALL自动刷新token或跳转登录

### Req 30: 数据模型与类型定义
**优先级：** P1  
**用户故事：** 作为开发者，我希望有完整的类型定义，以便避免类型错误和提升开发效率

**验收标准：**
1. WHEN Developer定义数据模型，THE System SHALL使用TypeScript interface
2. WHEN API返回数据，THE System SHALL验证数据类型
3. WHEN Component接收props，THE System SHALL有完整的类型定义
4. WHEN Function处理数据，THE System SHALL有输入输出类型
5. WHEN编译代码，THE System SHALL通过TypeScript类型检查
6. WHEN IDE提示，THE System SHALL提供准确的类型提示
7. WHEN重构代码，THE System SHALL通过类型系统发现潜在问题

### Req 43: Last-Click模型与转化优化
**优先级：** P1
**用户故事：** 作为运营人员，我希望优化用户转化路径，以便提升关键指标

**验收标准：**
1. WHEN User访问页面，THE System SHALL记录用户来源和渠道
2. WHEN User完成关键操作，THE System SHALL归因到最后点击的渠道
3. WHEN User浏览内容，THE System SHALL优化CTA按钮的位置和文案
4. WHEN User犹豫，THE System SHALL在适当时机显示引导提示
5. WHEN User离开，THE System SHALL记录退出页面和原因
6. WHEN分析转化，THE System SHALL提供转化漏斗报告
7. WHEN优化转化，THE System SHALL进行A/B测试验证效果

### Req 52: 用户交互的多语言支持
**优先级：** P1
**用户故事：** 作为国际用户，我希望用户交互界面（登录、评论、分享等）也支持多语言，以便更好地使用这些功能

**验收标准：**
1. WHEN User切换语言，THE System SHALL显示对应语言的用户交互界面文本
2. WHEN User登录，THE System SHALL根据用户选择的locale显示登录界面
3. WHEN User查看评论，THE System SHALL显示多语言的评论提示和按钮
4. WHEN User分享内容，THE System SHALL根据locale生成对应语言的分享文案
5. WHEN User查看错误提示，THE System SHALL显示多语言的错误信息
6. WHEN User使用表单，THE System SHALL显示多语言的验证提示
7. WHEN Developer添加新交互文本，THE System SHALL为所有支持的locale（zh-CN, zh-TW, en）提供翻译

### Req 53: 评论与Content模型集成
**优先级：** P0
**用户故事：** 作为开发者，我希望评论系统与统一Content模型集成，以便支持所有类型的内容评论

**验收标准：**
1. WHEN User评论内容，THE System SHALL通过contentId关联到Content表
2. WHEN User查看不同类型内容，THE System SHALL显示该Content的评论（无论是ARTICLE、TIANJI_PAGE等）
3. WHEN System存储评论，THE System SHALL使用Comment模型关联到Content
4. WHEN User切换语言，THE System SHALL评论内容保持不变（评论本身不翻译）
5. WHEN System统计评论，THE System SHALL支持按ContentType分组统计
6. WHEN Admin管理评论，THE System SHALL可以查看所有Content类型的评论
7. WHEN API返回评论，THE System SHALL包含关联的Content信息（type, slug等）

---

## 可预览验证的验收标准

### 🔐 用户认证验证

**验证方式：** 实际操作测试

- [ ] **AC-1.1** 点击"登录"按钮显示登录选项
- [ ] **AC-1.2** 选择 Google 登录跳转到授权页面
- [ ] **AC-1.3** 授权成功后返回网站并显示用户信息
- [ ] **AC-1.4** 导航栏显示用户头像和名称
- [ ] **AC-1.5** 点击头像显示用户菜单
- [ ] **AC-1.6** 点击"退出登录"成功退出
- [ ] **AC-1.7** 刷新页面后登录状态保持

### 💬 评论功能验证

**验证方式：** 实际操作测试

- [ ] **AC-2.1** 文章底部显示评论区
- [ ] **AC-2.2** 登录后显示评论输入框
- [ ] **AC-2.3** 未登录显示"登录后评论"提示
- [ ] **AC-2.4** 提交评论后显示"等待审核"
- [ ] **AC-2.5** 可以回复其他人的评论
- [ ] **AC-2.6** 可以点赞评论
- [ ] **AC-2.7** 管理员可以审核评论

### 📤 分享功能验证

**验证方式：** 实际操作测试

- [ ] **AC-3.1** 文章页面显示分享按钮
- [ ] **AC-3.2** 点击微信分享显示二维码
- [ ] **AC-3.3** 点击微博分享打开分享窗口
- [ ] **AC-3.4** 点击 Twitter 分享打开分享窗口
- [ ] **AC-3.5** 点击复制链接成功复制
- [ ] **AC-3.6** 分享链接包含 UTM 参数
- [ ] **AC-3.7** 分享事件被正确记录

### 🎨 UI 组件验证

**验证方式：** 视觉检查和交互测试

- [ ] **AC-4.1** 所有按钮有一致的样式和交互
- [ ] **AC-4.2** 表单输入有验证和错误提示
- [ ] **AC-4.3** 加载状态有统一的显示方式
- [ ] **AC-4.4** 模态框和弹窗正常工作
- [ ] **AC-4.5** Toast 通知正常显示
- [ ] **AC-4.6** 所有组件在移动端正常显示

### 🔄 状态管理验证

**验证方式：** 开发者工具检查

- [ ] **AC-5.1** 用户状态在全局 store 中正确管理
- [ ] **AC-5.2** 数据请求使用 SWR 缓存
- [ ] **AC-5.3** 状态变化只触发相关组件更新
- [ ] **AC-5.4** 离线时使用缓存数据
- [ ] **AC-5.5** 网络恢复后自动同步数据

### 🌐 API 集成验证

**验证方式：** 网络请求监控

- [ ] **AC-6.1** API 请求使用统一的客户端
- [ ] **AC-6.2** 请求失败自动重试
- [ ] **AC-6.3** 错误有友好的提示
- [ ] **AC-6.4** 请求中显示加载状态
- [ ] **AC-6.5** 认证 token 自动添加
- [ ] **AC-6.6** Token 过期自动刷新

### 🌍 多语言用户交互验证

**验证方式：** 实际操作测试

- [ ] **AC-7.1** 切换语言后登录界面显示对应语言
- [ ] **AC-7.2** 评论区的提示和按钮支持多语言
- [ ] **AC-7.3** 分享功能生成多语言文案
- [ ] **AC-7.4** 表单验证错误显示多语言提示
- [ ] **AC-7.5** 所有用户交互文本都有zh-CN、zh-TW、en翻译

### 📝 评论与Content模型集成验证

**验证方式：** 数据库和功能测试

- [ ] **AC-8.1** 评论通过contentId正确关联到Content
- [ ] **AC-8.2** 文章、天纪、地纪、人纪页面都可以评论
- [ ] **AC-8.3** 可以按ContentType统计评论数
- [ ] **AC-8.4** API返回评论时包含Content的type和slug信息
- [ ] **AC-8.5** 切换语言不影响评论内容显示

---

## 依赖关系

### 前置依赖
- ✅ Phase 1: 基础架构已完成
- ✅ Phase 2: 核心页面已完成
- ✅ 数据库 Schema 已建立

### 外部依赖
- 🔄 Google OAuth 应用配置
- 🔄 社交平台开发者账号
- 🔄 评论审核流程定义

### 后续阶段依赖
- Phase 4 依赖本阶段的用户数据和互动数据
- Phase 5 依赖本阶段的用户系统

---

## 风险与假设

### 🚨 主要风险

1. **技术风险**
   - OAuth 集成可能遇到配置问题
   - 评论系统可能有性能问题
   - 状态管理可能变得复杂

2. **用户风险**
   - 用户可能不愿意登录
   - 评论质量可能参差不齐
   - 垃圾评论需要处理

3. **运营风险**
   - 评论审核工作量可能很大
   - 用户互动可能不活跃
   - 需要社区运营支持

### 💡 关键假设

1. **技术假设**
   - NextAuth.js 满足认证需求
   - Zustand + SWR 满足状态管理需求
   - 评论系统性能可以接受

2. **用户假设**
   - 用户愿意使用 Google 账号登录
   - 用户愿意参与评论和互动
   - 用户会分享有价值的内容

3. **运营假设**
   - 有人力进行评论审核
   - 可以建立社区规范
   - 可以引导用户参与

---

## 成功指标

### 📈 量化指标

| 指标类别 | 指标名称 | 目标值 | 测量方式 |
|---------|---------|--------|----------|
| **用户** | 注册用户数 | 50+ 用户 | 数据库统计 |
| **互动** | 日均评论数 | 10+ 条 | 数据库统计 |
| **互动** | 评论回复率 | > 30% | 数据库统计 |
| **分享** | 日均分享次数 | 20+ 次 | Analytics |
| **留存** | 7日留存率 | > 20% | Analytics |
| **转化** | 登录转化率 | > 10% | Analytics |
| **多语言** | 多语言用户占比 | > 15% | 数据库统计 |
| **测试** | 交互功能测试覆盖率 | ≥ 80% | 测试报告 |

### 🎯 定性指标

- **用户体验**：登录和评论流程流畅
- **社区氛围**：评论质量高，讨论有价值
- **内容传播**：分享功能被积极使用
- **运营效率**：评论审核流程高效

---

## 向 Phase 4 的过渡

### Phase 3 完成后的交付物

1. **功能交付**
   - 完整的用户认证系统
   - 评论和互动功能
   - 社交分享功能

2. **技术交付**
   - 状态管理方案
   - API 集成方案
   - UI 组件库

3. **数据交付**
   - 用户数据
   - 互动数据
   - 分享数据

### 过渡准备

- ✅ 用户系统稳定运行
- ✅ 评论功能正常工作
- ✅ 互动数据开始积累
- 🔄 准备性能优化
- 🔄 准备测试策略
- 🔄 准备监控方案

---

**文档版本：** 1.0.0  
**创建日期：** 2024-11-07  
**最后更新：** 2024-11-07  
**文档状态：** ✅ 已完成  

**相关文档：**
- ../nihaixia-website/requirements.md - 主需求文档
- ../phase-1-foundation/requirements.md - Phase 1 需求
- ../phase-2-core-pages/requirements.md - Phase 2 需求
- design.md - Phase 3 详细设计
- tasks.md - Phase 3 实施任务
