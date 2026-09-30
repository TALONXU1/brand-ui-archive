# Apache Spark 视觉设计资源清洗与社交图谱富化提示词库 (Spark Prompts Suite)

本提示词库专为运行在 Apache Spark (PySpark / Spark SQL / Databricks) 上的设计资源大数据流水线设计。面向 1,000 ~ 100,000+ 品牌与视觉传达案例，提供工业级 ETL 清洗、社交媒体实体挖掘、设计质量量化评分 (DQS) 及反向发现能力。

---

## 模块一：原始数据质量清洗与去重 (ETL Cleaning & Normalization)

> **适用场景**：将 Google Sheets、爬虫队列或开源 CSV 中的脏数据转为标准 Parquet/Delta Lake 结构化表。

```markdown
你是一名资深大数据架构师与 PySpark 专家。请编写一段工业级 PySpark 批处理作业，对传入的设计资源原始 DataFrame 进行清洗与规范化：

【输入 Schema】
- 标题 (string), 分类 (string), 标签 (string), 描述 (string), 链接 (string), 图片 (string), 日期 (string)

【清洗与转换规则】
1. **URL 标准化与绝对去重 (Canonicalization)**：
   - 编写 PySpark UDF，去除 URL 中的协议前缀 (http://, https://)、www 前缀以及末尾斜杠；
   - 剔除所有 UTM 追踪参数（如 `utm_source`, `utm_medium`, `fbclid` 等）；
   - 基于标准化后的 `canonical_url` 执行 `dropDuplicates`，消除跨周期与跨分类重复录入的脏数据。
2. **多字节字符流与截断修复 (UTF-8 Sanitization)**：
   - 检查分类字段中因字节切片截断造成的残缺标签（如 `品牌视` -> `品牌视觉`, `牌官网` -> `品牌官网`, `空艺术` -> `空间艺术`）；
   - 对齐同义词 taxonomy：将 `Typography`、`Typewolf (Top 30 Favorite)`、`Fonts In Use` 统一归入 `排版字体` 门类。
3. **视觉物料与占位符标注 (Dummy Image Filtering)**：
   - 检测 `图片` 列中是否包含通配占位路径（如 `/assets/images/og.jpg`, `placeholder.png`, 或空字符串）；
   - 新增布尔标记列 `is_dummy_cover`，用于下游自动化替换为真实视口截图服务（mshots）。

【输出格式】
- 输出至 Parquet 格式，分区键按 `clean_category`，并打印出原始总数、去重后条目数及清洗统计报告。
```

---

## 模块二：社交媒体图谱富化与分布式挖掘 (Social Graph Enrichment)

> **适用场景**：利用 Spark 分布式并发优势，批量解析目标网站外链，穿透实体孤岛，建立“设计案例 - 创作者 - 社交资产”映射。

```markdown
你是一名高级数据采集与网络图谱分析工程师。请编写一个基于 PySpark 的分布式外部社交图谱挖掘任务：

【任务目标】
针对已去重的设计案例 URL 集合，分布式抓取每个网站的前端 HTML，提取其中的核心创作者身份与社交媒体连接。

【执行规范与网络容错】
1. **采用 mapPartitions 高并发处理**：
   - 严禁对每条记录单次触发 HTTP；必须使用 `rdd.mapPartitions`，在每个 Spark Worker 节点内部复用 `requests.Session` 或 `aiohttp` 异步并发事件循环，批量请求网站 HTML。
   - 必须设置严格的超时阈值（connect=2.5s, read=3.5s），捕获 DNS 解析失败、SSL 证书过期及 404/503 状态码。
2. **多层元数据穿透与正则提取**：
   - 解析 `<head>` 标签中的 OpenGraph 和 Twitter Card：
     - `meta[name="twitter:creator"]` -> 提取主创推特 Handle；
     - `meta[property="og:image"]` -> 提取真实高保真封面图；
   - 遍历 `<a>` 标签的外链 `href`：
     - **Are.na**：匹配 `are.na/channel/` 或 `are.na/user/`，提取所属先锋设计频道；
     - **Twitter / X**：提取官方主页 handle（过滤掉 share/intent 等无效操作链接）；
     - **Instagram**：提取机构官方主页；
     - **Read.cv / Layers**：提取主创履历个人页；
     - **GitHub**：针对创意编程（Creative Coding / WebGL）项目，提取仓库路径。
3. **输出富化 Schema**：
   - 返回 `StructType`，包含：`target_url`, `status_code`, `twitter_handle`, `instagram_handle`, `arena_urls`, `readcv_user`, `github_repo`。
```

---

## 模块三：设计质量量化评分模型 (Design Quality Score, DQS)

> **适用场景**：通过社交图谱反哺算法，自动量化设计水平，置顶前沿先锋案例，过滤粗劣模板站。

```markdown
你是一名视觉传达专家兼算法工程师。请在 PySpark DataFrame 上实现一个“设计质量量化评分模型 (Design Quality Score, DQS)”，取值区间为 [0.0, 100.0]：

【打分维度与权重逻辑】
1. **Are.na 策展加权 (权重: 35%)**：
   - Are.na 是国际当代视觉传达与独立出版领域的核心品味社区。若案例所属域名出现在 Are.na 公开 Channel 中，赋予最高权重加分；若出现 3 个以上不同 Channel 关联，给满 35 分。
2. **创作者社交真实性与同行认可度 (权重: 25%)**：
   - 成功关联真实设计师/机构 Twitter、Instagram 或 Read.cv 官方账号得 15 分；
   - 若创作者为已收录顶级机构库（如 Pentagram, Studio Dumbar, Collins, Dinamo 等）成员，加 10 分。
3. **高保真视觉物料完整度 (权重: 20%)**：
   - 具备独立定制的 OpenGraph/CDN 封面图且非占位图得 20 分；缺失或为 dummy 图得 0 分。
4. **域名存活心跳与响应性 (权重: 10%)**：
   - 探测返回 HTTP 200 且未发生可疑重定向（如域名过期后重定向至广告/博彩页）得 10 分；超时或死链扣至 0 分。
5. **视觉设计核心门类加权 (权重: 10%)**：
   - 属于 `品牌视觉`, `排版字体`, `空间艺术` 等专业美学门类得 10 分。

【交付要求】
- 编写 Spark SQL / UDF 函数计算 `quality_score`；
- 输出全量质量分布统计直方图（分位数 25%, 50%, 75%, 90%）；
- 筛选前 20% 高分标杆案例生成 `curated_hall_of_fame` 视图。
```

---

## 模块四：社交自繁殖与增量发现闭环 (Continuous Discovery Loop)

> **适用场景**：利用已收录的顶级设计师社交圈层，自动发现最新未上线项目，实现资源库从 1,000 到 10,000 的自增长。

```markdown
你是一名擅长社交媒体网络挖掘与流计算的工程师。请设计并编写一套基于 Spark 的“创作者驱动自繁殖发现流水线”：

【核心逻辑】
现已有 500+ 个已认证的高质量视觉设计师与工作室社交账号（Twitter/X, Are.na, Read.cv）。请建立持续发现任务：

1. **社交动态摄取与链接挖掘**：
   - 监听或定期拉取这些优质创作者发布的公开动态（Tweets / Posts / Are.na Blocks）；
   - 从推文正文与附件中提取包含 `http(s)://` 的外链；
   - 排除社交内链（如 youtube.com, bit.ly 垃圾跳板）与通用内容平台（如 medium, notion.site）。
2. **反向实体关联与去重入库**：
   - 检查新抓取的外链是否已存在于现有的 1,000 个案例库中；
   - 若为新域名，自动将推文作者标记为该站点的【主创设计机构/作者】；
   - 调用 Spark 模块二与模块三自动完成截图、分类判定与 DQS 评分；
3. **高分新案例自动化发布**：
   - 对 DQS >= 70 的新案例，自动生成符合展示规范的 JSON 记录，并触发静态页面构建任务更新 GitHub Pages。
```
