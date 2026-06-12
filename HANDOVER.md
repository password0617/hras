## HRAS 全球劳务工供应商管理平台 — 项目交接文档

### 一、项目概述

HRAS 是一个面向集团 CHO（首席人力资源官）的全球劳务工供应商管理看板平台，用于供应商明细查看、定价分析、绩效考核、新供应商引入和合规预警。目前为单页应用（SPA），所有代码打包在一个 HTML 文件中。

**线上地址**：https://maofeifei146-max.github.io/hras/
**GitHub 仓库**：https://github.com/maofeifei146-max/hras
**本地部署目录**：/tmp/hras-deploy/
**源文件位置**：/Users/maofifi/.qoderworkcn/workspace/mq9c4tb22iw62uav/outputs/cho-dashboard.html

---

### 二、技术架构

**前端技术栈**：
- 纯 HTML + CSS + JavaScript（ES6+），无框架
- ECharts 5.5.0（CDN 加载）用于图表可视化
- 单文件架构：所有样式、结构、逻辑均在 `cho-dashboard.html` 中

**文件规模**：约 1955 行，约 98KB

**部署方式**：
- 本地 git 仓库 `/tmp/hras-deploy/` 推送至 GitHub `main` 分支
- GitHub Pages 自动从 `main` 分支的 `index.html` 部署
- 部署命令：`cp cho-dashboard.html /tmp/hras-deploy/index.html && cd /tmp/hras-deploy && git add . && git commit -m "msg" && git push origin main`

**认证**：使用 `gh` CLI（GitHub CLI）完成 GitHub 设备授权登录，二进制文件位于 `/tmp/gh`

---

### 三、页面模块结构

| 页面 | data-page | 说明 |
|------|-----------|------|
| 供应商看板 | kanban | 默认首页，结构化表格展示 BU/国家/地区/使用情况 |
| 供应商明细 | registry | 供应商列表，支持筛选（BU/国家/地区/状态/评级/搜索） |
| 供应商定价 | pricing | BU 平均 Markup 对比、供应商下钻排名、跨地区定价分析 |
| 供应商考核 | assessment | 考核指标雷达图、月度趋势、供应商搜索 |
| 引入供应商 | onboard | 新增供应商表单、批量导入（模拟） |
| 合规及预警 | compliance | 红/黄/蓝预警列表、级别筛选 |

---

### 四、数据维度说明

当前全局维度已统一为 **BU → 国家 → 地区** 三级结构：

**BU（Business Unit）**：
- FBU美洲、FBU欧洲、FBU亚太

**国家**（12 个）：
- 美国、加拿大、墨西哥、英国、捷克、法国、德国、波兰、意大利、西班牙、澳大利亚、日本、韩国

**地区**（22 个）：
- 美洲：加州区、新泽西区、亚特兰大区、芝加哥区、萨凡纳区、达拉斯区、迈阿密区、休斯顿区、诺福克区、西雅图区、加拿大区、墨西哥区
- 欧洲：英国区、捷克区、法国区、德国区、波兰区、意大利区、西班牙区
- 亚太：澳洲区、日本区、韩国区

**映射关系**（JS 中维护）：
```javascript
BU_MAP = { '美洲区': 'FBU美洲', '欧洲区': 'FBU欧洲', '亚太区': 'FBU亚太' }
COUNTRY_MAP = { '加州区': '美国', '新泽西区': '美国', ... }
AREAS_MAP = { 'FBU美洲': ['加州区', '新泽西区', ...], ... }
```

---

### 五、Mock 数据结构

共 41 条供应商记录，每条结构如下：
```javascript
{
  id: 'S000116',
  name: 'Citi Staffing',
  region: '美洲区',      // 原始大区字段（保留用于后处理）
  bu: 'FBU美洲',         // 后处理添加
  country: '美国',       // 后处理添加
  area: '加州区',
  type: '劳务',          // 劳务/卸柜/外包/保安
  status: '使用中',      // 使用中/停用/新增
  rate: 22.5,            // 结算时薪
  base: 16.5,            // 基本时薪
  markup: '36%',
  ratio: '29%',
  rating: 'B',           // A/B/C/D/—
  contact: 'Jakob Vicencio · 909-225-5015',
  startDate: '2022-03',
  cert: '劳务派遣证 / ISO9001',
  metrics: {...},        // 考核指标（delivery/quality/stability/safety/compliance）
  trend: [...],          // 月度趋势数据
  composite: 82.5        // 综合评分
}
```

数据来源：基于用户提供的 Excel 文件 `/Users/maofifi/Downloads/FBU交付前台各区劳务供应商更新表.xlsx` 提取的真实数据结构生成。

---

### 六、权限系统

- **CHO 权限**（默认开启）：显示全量数据
- **受限视图**（关闭）：结算时薪、Markup 等敏感字段显示 `***`
- 切换方式：页面右上角 toggle 开关
- 控制变量：`isCHO`，函数 `maskSensitive(val)` 处理脱敏

---

### 七、视觉风格

**CSS 变量**：
```css
--primary: #2E6BE5;
--primary-light: #DAE8FB;
--primary-dark: #1D4FA0;
--navy: #1E3D7A;
--navy-light: #2B5DAE;
--bg: #E8EEF6;
--sidebar-width: 180px;
--card: #FFFFFF;
--border: #E0E4EB;
```

**关键组件样式**：
- 统计卡片（stat-card）：蓝色背景 `#3A7BD5`，白色文字，flex 一行排列，带月份切换
- 侧边栏：深蓝渐变背景，白色文字，180px 宽度
- 看板表格：白色卡片 + 蓝色渐变标题栏，斑马纹行
- 合规预警卡片：彩色渐变（红/黄/蓝/绿）

---

### 八、关键函数索引

| 函数名 | 作用 |
|--------|------|
| `switchPage(page)` | 页面导航切换 |
| `renderCurrentPage()` | 根据当前页面调用对应渲染函数 |
| `renderKanban()` | 渲染供应商看板（4 张结构化表格） |
| `renderBUCards()` | 渲染供应商明细页的 BU 统计卡片 |
| `renderRegistryTable()` | 渲染供应商明细表格 |
| `getFilteredSuppliers()` | 获取筛选后的供应商列表 |
| `updateCascadingFilters()` | BU → 国家 → 地区级联筛选 |
| `openSupplierDrawer(idx)` | 打开供应商详情抽屉 |
| `renderPricingCharts()` | 渲染定价分析页图表 |
| `renderBUComparison()` | BU 平均 Markup 对比图 |
| `showBUDrillDown(bu)` | BU 下钻供应商排名 |
| `renderAssessment()` | 渲染考核页面 |
| `renderAlerts()` | 渲染合规预警列表 |
| `switchMonth(btn)` | 月份切换（1月-6月+累计） |
| `toggleRole()` | CHO/受限视图切换 |

---

### 九、已知问题与待优化项

1. **ECharts 内存管理**：定价页和考核页的图表实例通过 `pricingChartInstances` 对象管理，切换时已做 dispose 处理，但抽屉反复开关时仍需注意
2. **数据为 Mock**：41 条供应商数据为模拟数据，如需接入真实数据需要后端 API 或定期 Excel 导入
3. **单文件架构**：随着功能增加，单 HTML 文件会越来越大（目前约 98KB），后续可考虑拆分为模块化项目
4. **月份切换**：统计卡片的月份切换目前使用固定系数模拟月度变化（0.68-1.0），可改为真实月度数据
5. **合规预警**：预警数据为硬编码的 8 条，可改为动态生成
6. **响应式**：窄屏下统计卡片会缩小间距，但表格横向滚动体验可优化

---

### 十、建议的下一步开发方向

1. 接入真实数据源（Excel 定期导入或 API 对接）
2. 添加数据导出功能（Excel/PDF 导出）
3. 供应商详情页增加更多维度的趋势图表
4. 增加搜索和高级筛选功能
5. 添加用户登录和多角色权限管理
6. 移动端适配优化
7. 考虑迁移到前端框架（Vue/React）以便后续维护

---

### 十一、如何交接给另一个 AI Agent

将以下内容作为 Prompt 提供给新的 AI Agent：

```
我正在开发一个 HRAS 全球劳务工供应商管理平台项目。

项目文件：cho-dashboard.html（单文件 SPA，约 1955 行）
线上地址：https://maofeifei146-max.github.io/hras/
GitHub 仓库：maofeifei146-max/hras

技术栈：纯 HTML + CSS + JS + ECharts 5.5.0，无框架。

部署方式：将 cho-dashboard.html 复制为 /tmp/hras-deploy/index.html，然后 git push 到 GitHub main 分支即可自动部署到 GitHub Pages。

项目包含 6 个页面模块：供应商看板（默认首页）、供应商明细、供应商定价、供应商考核、引入供应商、合规及预警。数据维度为 BU（FBU美洲/欧洲/亚太）→ 国家（12个）→ 地区（22个）三级结构。Mock 数据共 41 条供应商记录。

请先阅读完整的项目交接文档（HANDOVER.md），然后阅读 cho-dashboard.html 源代码，了解当前状态后再开始新的开发任务。
```

将本交接文档（HANDOVER.md）和 cho-dashboard.html 一起提供给新的 AI Agent 即可。
