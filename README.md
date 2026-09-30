# 品牌优质 UI 与排版设计灵感全景库
> Brand Design & Typography Inspiration Archive

基于 Google Sheets 云端数据源驱动的高品质前端设计与排版灵感索引库。支持时间流、便当盒画廊、高密度列表与 6 大旗舰品牌深度设计系统拆解。

---

## ✨ 核心特性

- 🔄 **云端数据实时直连**：网页通过 Google Sheets GViz API 异步拉取最新数据。只要 Google 表格更新，线上网页刷新即可秒级呈现最新收录案例。
- 🛡️ **双轨可靠性兜底**：离线或弱网环境下自动切换至内置的高品质静态快照数据（已收录 940+ 案例），确保零白屏。
- ⏱️ **时间流时间轴分类**：按收录日期倒序自动分组，配备吸顶式日期快速锚点导航。
- 🎨 **多维视图自由切换**：支持时间流（Timeline）、便当盒画廊（Bento Grid）、紧凑表格（Table）以及品牌深度拆解（Deep Dives）。
- 🔍 **毫秒级模糊检索**：按键盘 `/` 即可随时聚焦搜索栏，支持品牌名、视觉亮点、出处平台即时过滤。

---

## 🚀 GitHub Pages 一键部署指南

### 第一步：在 GitHub 上创建仓库
1. 登录 [GitHub](https://github.com/)，点击右上角的 **+** 号选择 **New repository**。
2. 仓库名称填入例如：`brand-ui-archive`。
3. 权限设置为 **Public**（公开），其他选项保持默认，点击 **Create repository**。

### 第二步：上传文件
1. 将本项目中的三个文件上传至仓库根目录：
   - `index.html`（核心页面）
   - `.nojekyll`（确保静态资源正常解析）
   - `README.md`
2. 点击 **Commit changes** 确认提交。

### 第三步：开启 GitHub Pages
1. 点击仓库顶部的 **Settings** 选项卡。
2. 在左侧菜单中找到 **Pages**（位于 Code and automation 下方）。
3. 在 **Build and deployment -> Branch** 中：
   - 分支选择：`main`
   - 文件夹保持：`/ (root)`
   - 点击 **Save**。
4. 等待 1~2 分钟，页面上方会显示绿色提示并给出您的专属线上网址：
   `https://<你的GitHub用户名>.github.io/brand-ui-archive/`

---

## ⚙️ Google Sheets 表格权限配置（确保实时同步生效）

为了让 GitHub Pages 上的网页能够跨域拉取您 Google Sheets 的最新更新，请确保：
1. 打开您的 Google 表格：[品牌优质UI设计资源库](https://docs.google.com/spreadsheets/d/1I9WU29mrIeYRibcf00Hy5JsBKoebkiHouW5Yxk7NJcg/edit)
2. 点击右上角 **“共享 (Share)”**。
3. 在“常规使用权”中，将权限修改为 **“任何知道链接的人 (Anyone with the link)” -> “查看者 (Viewer)”**。
