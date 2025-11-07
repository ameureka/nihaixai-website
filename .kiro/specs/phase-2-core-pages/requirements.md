# Phase 2: 核心页面 - 需求文档

## 阶段概述

**阶段名称：** Phase 2 - 核心页面  
**实施周期：** Week 3-6 (4周)  
**团队规模：** 3-4 人（2个前端开发 + 1个内容编辑 + 1个设计师）  
**预算估算：** $8,000 - $12,000  

### 🎯 阶段目标

完成所有核心内容页面的开发，用户可以浏览完整的网站内容，包括天纪、地纪、人纪、五大经典医著、易学命理等专题页面。

### 📋 核心价值

1. **内容完整性**：用户可以访问所有核心内容页面
2. **用户体验**：提供流畅的浏览和导航体验
3. **SEO优化**：每个页面都经过SEO优化
4. **响应式设计**：在所有设备上都能良好显示
5. **性能优化**：快速加载和流畅交互

### 🎨 预期成果

**用户视角：**
- 可以浏览天纪、地纪、人纪的完整内容
- 可以查看五大经典医著的专题页面
- 可以浏览文章列表和详情
- 可以观看 YouTube 视频
- 在手机和平板上都能正常使用

**开发者视角：**
- 完整的页面组件库
- 可复用的内容展示组件
- 使用 CMS 系统管理内容（已在Phase 1选型完成）
- pSEO 页面生成系统
- 基于统一Content模型的API设计

**运营视角：**
- 所有页面都被搜索引擎索引
- 关键词排名开始提升
- 用户停留时间增加

---

## 从主需求文档提取的相关需求

### Req 1: 主页展示与导航
**优先级：** P0  
**用户故事：** 作为访问者，我希望看到一个专业的主页，以便快速了解网站内容和导航到感兴趣的板块

**验收标准：**
1. WHEN User访问主页，THE System SHALL显示Hero区域，包含网站标题和简介
2. WHEN User查看主页，THE System SHALL显示三大板块（天纪、地纪、人纪）的入口卡片
3. WHEN User点击板块卡片，THE System SHALL导航到对应的专题页面
4. WHEN User滚动页面，THE System SHALL显示特色内容推荐
5. WHEN User查看导航栏，THE System SHALL显示所有主要栏目链接
6. WHEN User在移动设备访问，THE System SHALL显示响应式布局
7. WHEN搜索引擎抓取主页，THE System SHALL提供完整的SEO元数据

### Req 2: 天纪内容展示
**优先级：** P0  
**用户故事：** 作为易学爱好者，我希望能够浏览天纪（易学命理）的内容，以便学习相关知识

**验收标准：**
1. WHEN User访问/tianji，THE System SHALL显示天纪专题页面
2. WHEN User查看页面，THE System SHALL显示易学命理的介绍和核心内容
3. WHEN User浏览内容，THE System SHALL提供清晰的内容分类和导航
4. WHEN User点击内容项，THE System SHALL显示详细内容
5. WHEN User查看相关视频，THE System SHALL嵌入YouTube播放器
6. WHEN User在移动设备访问，THE System SHALL自动适配屏幕尺寸
7. WHEN搜索引擎索引，THE System SHALL提供优化的SEO标签

### Req 3: 地纪内容展示
**优先级：** P0  
**用户故事：** 作为风水爱好者，我希望能够浏览地纪（风水地理）的内容，以便学习相关知识

**验收标准：**
1. WHEN User访问/diji，THE System SHALL显示地纪专题页面
2. WHEN User查看页面，THE System SHALL显示风水地理的介绍和核心内容
3. WHEN User浏览内容，THE System SHALL提供清晰的内容分类和导航
4. WHEN User点击内容项，THE System SHALL显示详细内容
5. WHEN User查看相关视频，THE System SHALL嵌入YouTube播放器
6. WHEN User在移动设备访问，THE System SHALL自动适配屏幕尺寸
7. WHEN搜索引擎索引，THE System SHALL提供优化的SEO标签

### Req 4: 人纪内容展示
**优先级：** P0  
**用户故事：** 作为中医学习者，我希望能够浏览人纪（中医医学）的内容，以便系统学习中医知识

**验收标准：**
1. WHEN User访问/renji，THE System SHALL显示人纪专题页面
2. WHEN User查看页面，THE System SHALL显示中医医学的介绍和核心内容
3. WHEN User浏览内容，THE System SHALL提供清晰的内容分类（针灸、伤寒、金匮、神农本草经、黄帝内经）
4. WHEN User点击内容项，THE System SHALL显示详细内容
5. WHEN User查看相关视频，THE System SHALL嵌入YouTube播放器
6. WHEN User在移动设备访问，THE System SHALL自动适配屏幕尺寸
7. WHEN搜索引擎索引，THE System SHALL提供优化的SEO标签

### Req 5-9: 五大经典医著专题页面
**优先级：** P0  
**用户故事：** 作为中医学习者，我希望能够深入学习五大经典医著，以便系统掌握中医理论

**验收标准：**
1. WHEN User访问针灸大成页面，THE System SHALL显示针灸相关的完整内容
2. WHEN User访问伤寒论页面，THE System SHALL显示伤寒论的完整内容
3. WHEN User访问金匮要略页面，THE System SHALL显示金匮要略的完整内容
4. WHEN User访问神农本草经页面，THE System SHALL显示本草经的完整内容
5. WHEN User访问黄帝内经页面，THE System SHALL显示内经的完整内容
6. WHEN User浏览任一专题，THE System SHALL提供章节导航和内容检索
7. WHEN搜索引擎索引，THE System SHALL为每个专题提供独特的SEO优化

### Req 10: 易学命理专题展示
**优先级：** P1  
**用户故事：** 作为易学爱好者，我希望能够学习易学命理知识，以便理解天人合一的思想

**验收标准：**
1. WHEN User访问易学命理页面，THE System SHALL显示易学命理的系统介绍
2. WHEN User浏览内容，THE System SHALL提供八字、紫微斗数等分类
3. WHEN User查看案例，THE System SHALL显示实际命理分析案例
4. WHEN User观看视频，THE System SHALL嵌入相关教学视频
5. WHEN User在移动设备访问，THE System SHALL保持良好的阅读体验
6. WHEN搜索引擎索引，THE System SHALL提供针对性的SEO优化
7. WHEN User分享内容，THE System SHALL提供社交分享功能

### Req 11: 文章列表与检索
**优先级：** P0  
**用户故事：** 作为学习者，我希望能够浏览和搜索文章，以便找到我需要的学习资料

**验收标准：**
1. WHEN User访问文章列表页，THE System SHALL显示所有文章的列表
2. WHEN User浏览列表，THE System SHALL显示文章标题、摘要、分类和发布日期
3. WHEN User使用搜索功能，THE System SHALL根据关键词过滤文章
4. WHEN User使用分类筛选，THE System SHALL显示对应分类的文章
5. WHEN User点击文章，THE System SHALL导航到文章详情页
6. WHEN User翻页，THE System SHALL加载更多文章（分页或无限滚动）
7. WHEN搜索引擎索引，THE System SHALL为每篇文章提供独特的URL和元数据

### Req 13: YouTube频道链接与视频展示
**优先级：** P1  
**用户故事：** 作为学习者，我希望能够观看倪海厦老师的教学视频，以便更好地理解内容

**验收标准：**
1. WHEN User访问包含视频的页面，THE System SHALL嵌入YouTube播放器
2. WHEN User点击播放，THE System SHALL在页面内播放视频
3. WHEN User查看视频列表，THE System SHALL显示视频缩略图、标题和时长
4. WHEN User点击YouTube频道链接，THE System SHALL在新标签页打开频道
5. WHEN User在移动设备观看，THE System SHALL提供响应式视频播放器
6. WHEN页面加载，THE System SHALL延迟加载视频以优化性能
7. WHEN搜索引擎索引，THE System SHALL提供VideoObject结构化数据

### Req 14: 响应式设计
**优先级：** P0  
**用户故事：** 作为移动设备用户，我希望网站在手机和平板上都能正常使用，以便随时随地学习

**验收标准：**
1. WHEN User在手机访问（< 768px），THE System SHALL显示移动端优化布局
2. WHEN User在平板访问（768px-1024px），THE System SHALL显示平板优化布局
3. WHEN User在桌面访问（> 1024px），THE System SHALL显示桌面完整布局
4. WHEN User旋转设备，THE System SHALL自动调整布局
5. WHEN User在触摸设备操作，THE System SHALL提供适当的触摸目标大小
6. WHEN User缩放页面，THE System SHALL保持可读性和可用性
7. WHEN测试响应式，THE System SHALL在所有断点都通过可用性测试

### Req 17: SEO优化
**优先级：** P0  
**用户故事：** 作为网站运营者，我希望网站在搜索引擎中有良好的排名，以便吸引更多用户

**验收标准：**
1. WHEN搜索引擎访问任何页面，THE System SHALL提供唯一且描述性的title标签
2. WHEN搜索引擎索引页面，THE System SHALL提供吸引人的meta description
3. WHEN搜索引擎分析内容，THE System SHALL使用语义化的HTML标签
4. WHEN搜索引擎评估页面，THE System SHALL提供适当的heading层级（h1-h6）
5. WHEN搜索引擎抓取图片，THE System SHALL为所有图片提供alt属性
6. WHEN搜索引擎识别内容，THE System SHALL提供Schema.org结构化数据
7. WHEN搜索引擎评估性能，THE System SHALL确保Core Web Vitals达标

### Req 18: 网站性能优化
**优先级：** P0  
**用户故事：** 作为用户，我希望网站加载快速，以便获得流畅的浏览体验

**验收标准：**
1. WHEN User访问任何页面，THE System SHALL在3秒内完成首次内容绘制（FCP）
2. WHEN User交互，THE System SHALL在100ms内响应
3. WHEN User滚动页面，THE System SHALL保持60fps的流畅度
4. WHEN User加载图片，THE System SHALL使用懒加载和现代图片格式（WebP）
5. WHEN User访问页面，THE System SHALL使用代码分割减少初始加载大小
6. WHEN User重复访问，THE System SHALL利用浏览器缓存
7. WHEN测试性能，THE System SHALL在Lighthouse中获得90+的性能分数

### Req 41: pSEO程序化SEO实施
**优先级：** P1  
**用户故事：** 作为SEO专员，我希望能够批量生成优化的页面，以便覆盖更多长尾关键词

**验收标准：**
1. WHEN System生成pSEO页面，THE System SHALL使用模板和数据自动创建页面
2. WHEN System创建页面，THE System SHALL为每个页面生成唯一的title和description
3. WHEN System生成内容，THE System SHALL确保内容质量和相关性
4. WHEN User访问pSEO页面，THE System SHALL提供与手动页面相同的用户体验
5. WHEN搜索引擎索引，THE System SHALL避免重复内容问题
6. WHEN System更新数据，THE System SHALL自动更新相关pSEO页面
7. WHEN监控效果，THE System SHALL跟踪pSEO页面的排名和流量

### Req 42: 关键词策略三层架构
**优先级：** P1  
**用户故事：** 作为SEO专员，我希望有系统的关键词布局，以便最大化搜索流量

**验收标准：**
1. WHEN规划关键词，THE System SHALL定义头部关键词（高竞争、高流量）
2. WHEN规划关键词，THE System SHALL定义腰部关键词（中等竞争、稳定流量）
3. WHEN规划关键词，THE System SHALL定义长尾关键词（低竞争、精准流量）
4. WHEN创建页面，THE System SHALL根据关键词层级分配页面类型
5. WHEN优化内容，THE System SHALL在标题、描述、正文中自然使用关键词
6. WHEN建立内链，THE System SHALL使用关键词作为锚文本
7. WHEN监控效果，THE System SHALL跟踪各层级关键词的排名表现

### Req 45: 内容策略与生产流程
**优先级：** P1  
**用户故事：** 作为内容编辑，我希望有清晰的内容生产流程，以便高效创作高质量内容

**验收标准：**
1. WHEN规划内容，THE System SHALL基于关键词研究确定内容主题
2. WHEN创作内容，THE System SHALL遵循SEO写作最佳实践
3. WHEN编辑内容，THE System SHALL确保内容质量和准确性
4. WHEN发布内容，THE System SHALL通过CMS系统管理
5. WHEN优化内容，THE System SHALL定期更新和改进现有内容
6. WHEN监控效果，THE System SHALL跟踪内容的表现指标
7. WHEN复用内容，THE System SHALL将内容改编为不同格式

---

## 可预览验证的验收标准

### 🌐 页面可访问性验证

**验证方式：** 直接访问页面URL

- [ ] **AC-1.1** 访问主页能看到完整的Hero区域和三大板块
- [ ] **AC-1.2** 访问 /tianji 能看到天纪专题页面
- [ ] **AC-1.3** 访问 /diji 能看到地纪专题页面
- [ ] **AC-1.4** 访问 /renji 能看到人纪专题页面
- [ ] **AC-1.5** 访问 /renji/zhenjiu 能看到针灸大成页面
- [ ] **AC-1.6** 访问 /articles 能看到文章列表
- [ ] **AC-1.7** 访问 /articles/[slug] 能看到文章详情

### 📱 响应式设计验证

**验证方式：** 在不同设备上测试

- [ ] **AC-2.1** 在 iPhone (375px) 上所有页面正常显示
- [ ] **AC-2.2** 在 iPad (768px) 上所有页面正常显示
- [ ] **AC-2.3** 在桌面 (1920px) 上所有页面正常显示
- [ ] **AC-2.4** 导航菜单在移动端显示汉堡菜单
- [ ] **AC-2.5** 图片在所有设备上正确缩放
- [ ] **AC-2.6** 文字在所有设备上可读

### 🎥 视频功能验证

**验证方式：** 测试视频播放

- [ ] **AC-3.1** YouTube 视频可以正常嵌入
- [ ] **AC-3.2** 点击播放按钮视频开始播放
- [ ] **AC-3.3** 视频在移动设备上正常播放
- [ ] **AC-3.4** 视频列表显示缩略图和标题
- [ ] **AC-3.5** 点击频道链接在新标签页打开

### 🔍 SEO 验证

**验证方式：** 查看页面源码和SEO工具

- [ ] **AC-4.1** 每个页面有唯一的 title 标签
- [ ] **AC-4.2** 每个页面有描述性的 meta description
- [ ] **AC-4.3** 页面使用语义化的 HTML 标签
- [ ] **AC-4.4** 所有图片有 alt 属性
- [ ] **AC-4.5** 页面包含适当的结构化数据
- [ ] **AC-4.6** sitemap.xml 包含所有页面

### ⚡ 性能验证

**验证方式：** Lighthouse 和 PageSpeed Insights

- [ ] **AC-5.1** 所有页面 Lighthouse Performance > 90
- [ ] **AC-5.2** 所有页面 FCP < 1.5s
- [ ] **AC-5.3** 所有页面 LCP < 2.5s
- [ ] **AC-5.4** 图片使用 WebP 格式
- [ ] **AC-5.5** 使用代码分割和懒加载
- [ ] **AC-5.6** 静态资源正确缓存

### 📝 内容质量验证

**验证方式：** 人工审核

- [ ] **AC-6.1** 所有页面内容完整且准确
- [ ] **AC-6.2** 文章格式统一且易读
- [ ] **AC-6.3** 专题页面结构清晰
- [ ] **AC-6.4** 内容分类合理
- [ ] **AC-6.5** 内链布局合理
- [ ] **AC-6.6** 无错别字和格式错误

---

## 依赖关系

### 前置依赖
- ✅ Phase 1: 基础架构已完成
- ✅ 项目结构和部署流程就绪
- ✅ SEO 基础设施就绪

### 外部依赖
- 🔄 内容素材准备完成
- 🔄 设计稿和原型完成
- 🔄 YouTube 视频链接整理完成
- 🔄 CMS 系统选型和配置

### 后续阶段依赖
- Phase 3 依赖本阶段的页面结构和组件
- Phase 4 依赖本阶段的内容和SEO优化
- Phase 5 依赖本阶段的页面模板

---

## 风险与假设

### 🚨 主要风险

1. **内容风险**
   - 内容素材可能不完整
   - 内容质量可能需要多次审核
   - 多语言翻译质量需要专业审核

2. **技术风险**
   - CMS 使用和内容迁移可能需要时间（CMS已在Phase 1选型）
   - 性能优化可能需要额外时间
   - 响应式设计可能需要多次调整
   - 统一Content模型在复杂场景下的适配

3. **时间风险**
   - 页面数量多，开发时间可能超出预期
   - 内容审核可能延迟进度
   - SEO 优化需要持续调整

### 💡 关键假设

1. **内容假设**
   - 所有内容素材在开发前准备完成
   - 内容质量符合发布标准
   - 有专人负责内容审核

2. **技术假设**
   - 选择的 CMS 系统满足需求
   - 性能优化目标可以达成
   - 响应式设计在所有设备上都能良好工作

3. **资源假设**
   - 有足够的开发人员
   - 设计师可以及时提供支持
   - 内容编辑可以配合开发进度

---

## 成功指标

### 📈 量化指标

| 指标类别 | 指标名称 | 目标值 | 测量方式 |
|---------|---------|--------|----------|
| **页面** | 完成页面数量 | 20+ 页面 | 人工统计 |
| **性能** | Lighthouse 分数 | > 90 | Lighthouse CI |
| **SEO** | 页面索引数量 | 20+ 页面 | Google Search Console |
| **流量** | 日访问量 | 100+ UV | Google Analytics |
| **用户** | 平均停留时间 | > 2分钟 | Google Analytics |
| **内容** | 文章数量 | 10+ 篇 | CMS 统计 |

### 🎯 定性指标

- **用户体验**：用户可以轻松找到和浏览内容
- **内容质量**：内容准确、完整、易懂
- **视觉设计**：页面美观、专业、一致
- **SEO效果**：搜索引擎正确索引和排名

---

## 向 Phase 3 的过渡

### Phase 2 完成后的交付物

1. **页面交付**
   - 所有核心页面完成
   - 页面组件库完善
   - 响应式设计验证通过

2. **内容交付**
   - 初始内容发布
   - CMS 系统配置完成
   - 内容生产流程建立

3. **SEO交付**
   - 所有页面 SEO 优化完成
   - pSEO 系统搭建完成
   - 关键词布局完成

### 过渡准备

- ✅ 所有核心页面稳定运行
- ✅ 内容质量达标
- ✅ SEO 基础优化完成
- 🔄 准备用户交互功能
- 🔄 准备评论和分享功能
- 🔄 准备用户认证系统

---

**文档版本：** 1.0.0  
**创建日期：** 2024-11-07  
**最后更新：** 2024-11-07  
**文档状态：** ✅ 已完成  

**相关文档：**
- ../nihaixia-website/requirements.md - 主需求文档
- ../phase-1-foundation/requirements.md - Phase 1 需求
- design.md - Phase 2 详细设计
- tasks.md - Phase 2 实施任务
