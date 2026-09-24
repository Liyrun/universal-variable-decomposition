---
name: universal-literature-variable-decomposition
description: >
  面向任意研究主题、任意学科和任意类型研究文献的通用变量拆解 Skill。
  将文献中的概念、变量、测度、数据、模型、机制、识别策略和可迁移性
  转化为统一、可比较、可复现的结构化记录，并强制输出固定 Schema 的 Excel 工作簿。
  适用于单篇文献拆解、多篇文献汇总、候选变量池构建、变量测度复刻、
  概念迁移、研究设计比较与后续数据任务设计。
metadata:
  version: "2.2"
  language: zh-CN
  output_schema_version: UVD-2.2
---

# Universal Literature Variable Decomposition Skill

## 1. Skill 目标

本 Skill 不绑定任何具体研究主题、行业、数据类型、理论流派或变量类别。

它的任务是把文献中的“概念—变量—测度—数据—关系—模型—证据”拆成标准化研究设计单元，使来自不同主题、不同学科、不同方法的文献能够进入同一套比较框架。

每次执行必须回答：

1. 文献研究的核心问题是什么？
2. 文献中出现了哪些关键概念和变量？
3. 每个变量对应什么理论构念，边界是什么？
4. 作者实际上如何操作化和测量该变量？
5. 原始数据、样本、时间窗口、算法、量表、分类或计算规则是什么？
6. 变量之间建立了什么关系，理论机制如何展开？
7. 采用了什么模型、识别策略和稳健性设计？
8. 哪些内容是原文明确事实，哪些是推断，哪些是迁移建议？
9. 该变量或测度能否复现、迁移或纳入新的研究设计？
10. 与其他候选变量相比，它提供了什么新增信息，同时存在哪些风险？

核心原则：

> 先忠实还原原文，再标准化拆解；先界定构念，再解释指标；先记录证据，再进行推断；先说明可比维度，再给迁移建议。

## 2. 适用范围

本 Skill 可用于任意主题，例如但不限于公司治理、战略管理、组织行为、人力资源、创新、知识管理、数字化、供应链、运营、金融、会计、经济学、公共政策、环境、能源、市场营销等。

可处理定量实证、实验、问卷与量表、档案数据、混合方法、定性研究、理论/概念论文、元分析和测度开发论文。

若文献没有经验变量，对“变量测度、复现”字段填“**不适用**”，但仍拆解概念、理论关系和研究设计。不得为了填表而虚构变量。

## 3. 任务模式

执行前识别任务模式，但不得因为模式不同改变 Excel Schema。

```text
Mode A：前置文献初筛
Mode B：单篇正式变量拆解
Mode C：多篇文献批量拆解
Mode D：候选变量比较与去重
Mode E：概念迁移与自有变量构造
Mode F：测度复刻与数据任务设计
Mode G：研究设计 / 识别策略比较
```

同一任务可以同时包含多个 Mode。

## 4. 输入参数与数据源配置

本 Skill 必须将“从哪里读取材料”和“如何拆解材料”分离。
所有数据源设置均作为**运行参数**传入，不在 Skill 中写死具体路径、Zotero Collection 或项目名称。

## 4.1 通用运行参数

推荐调用结构：

```yaml
run_config:

  # 输出语言
  output_language: zh-CN
  # 可选：
  # zh-CN = 简体中文
  # en = English

  # 数据源
  source_config:
    source_mode: uploaded_files
    # 可选：
    # uploaded_files
    # local_folder
    # zotero
    # mixed

    uploaded_files:
      enabled: true

    local_folder:
      enabled: false
      paths: []
      recursive: true
      include_extensions:
        - pdf
        - docx
        - txt
      exclude_patterns:
        - "~$*"
        - "*.tmp"

    zotero:
      enabled: false
      library: "My Library"
      collections: []
      include_subcollections: true
      attachment_types:
        - pdf
      prefer_fulltext_attachment: true

    deduplication:
      enabled: true
      priority:
        - DOI
        - title
        - title_author_year

    source_priority:
      - zotero
      - local_folder
      - uploaded_files

  # 文献包与补充材料
  document_bundle:
    enabled: true

    # 每篇论文建立一个逻辑文献包：
    # 正文 + 在线附录 + Supplementary Material + Measurement Appendix
    # + Codebook + Replication files + 其他与变量构造相关材料
    supplementary_materials:
      discover: true
      discovery_sources:
        - explicit_links_in_primary_document
        - publisher_page
        - zotero_child_attachments
        - sibling_files_in_local_folder
      download_policy: explicit_or_authorized
      # 可选：
      # explicit_or_authorized = 仅明确链接或用户授权后下载
      # metadata_only = 只登记，不下载
      # local_only = 仅使用已有本地附件

      include_types:
        - online_appendix
        - supplementary_material
        - measurement_appendix
        - codebook
        - variable_dictionary
        - replication_code
        - replication_data_description
        - questionnaire
        - scale_items
        - other_relevant_attachment

      target_subfolder: "_supplementary"
      preserve_original_filename: true

    bundle_layout:
      primary_subfolder: "00_primary"
      supplementary_subfolder: "01_supplementary"
      packet_subfolder: "02_reading_packet"
      output_subfolder: "03_output"

  # 执行角色
  execution_profile:
    mode: hybrid_preferred
    # 可选：
    # chatgpt_full
    # hybrid_preferred
    # codex_prepare
    # codex_conservative

    semantic_analysis_engine: chatgpt
    file_execution_engine: codex

    codex_rules:
      allow_semantic_inference: false
      extract_source_passages: true
      build_reading_packet: true
      write_excel_from_validated_analysis: true

  # 可选目标研究
  target_research:
    research_question:
    dependent_variable:
    focal_variable:
    sample:
    analysis_level:
    period:
    preferred_data:
    theoretical_focus:
    constraints:
```

---

## 4.2 数据源模式

### A. `uploaded_files`

适用于当前会话直接上传的 PDF、DOCX、TXT 等文件。

```yaml
source_config:
  source_mode: uploaded_files
  uploaded_files:
    enabled: true
```

### B. `local_folder`

适用于具有本地文件访问权限的运行环境，例如 Codex、本地代理或具备文件系统访问能力的执行环境。

```yaml
source_config:
  source_mode: local_folder

  local_folder:
    enabled: true
    paths:
      - "D:/Research/Literature"
    recursive: true
    include_extensions:
      - pdf
      - docx
      - txt
```

规则：

1. `paths` 可以为一个或多个目录；
2. `recursive=true` 时读取所有子目录；
3. 只读取 `include_extensions` 中允许的文件；
4. 忽略临时文件、缓存文件和 `exclude_patterns` 指定文件；
5. 不得假设运行环境一定能够访问用户本机路径；
6. 若当前环境无文件系统权限，必须明确说明无法直接读取该路径，而不能声称已经扫描。

### C. `zotero`

适用于能够通过 Zotero API、Zotero MCP、插件、连接器或其他授权接口访问 Zotero 的运行环境。

```yaml
source_config:
  source_mode: zotero

  zotero:
    enabled: true
    library: "My Library"
    collections:
      - "Collection A"
      - "Collection B"
    include_subcollections: true
    attachment_types:
      - pdf
    prefer_fulltext_attachment: true
```

规则：

1. `collections` 指 Zotero **逻辑 Collection**，不是 Zotero `storage` 物理目录；
2. 优先通过 Collection 关系获取文献条目及附件；
3. 不得通过扫描 Zotero `storage` 目录猜测 Collection 归属；
4. 若存在 PDF 全文附件，优先读取全文；
5. 若只有元数据、摘要或网页快照，必须降低证据完整度；
6. 若当前环境没有 Zotero 访问能力，必须说明数据源不可直接访问，不得虚构读取结果。

### D. `mixed`

适用于同时读取多个来源。

```yaml
source_config:
  source_mode: mixed

  local_folder:
    enabled: true
    paths:
      - "D:/Research/Literature"

  zotero:
    enabled: true
    library: "My Library"
    collections:
      - "Core Literature"

  uploaded_files:
    enabled: true
```

混合模式必须先统一建立文献清单，再去重，再进入正式拆解。

---

## 4.3 文献包（Document Bundle）与补充材料

正式拆解的最小对象不再定义为“一个 PDF”，而是：

```text
一篇研究文献
=
正文 Primary Document
+
与研究设计相关的补充材料 Supplementary Materials
```

### 4.3.1 补充材料范围

下列材料均可能包含正文未完整披露的变量构造、分类规则、量表、算法或稳健性设计：

```text
Online Appendix
Supplementary Material
Web Appendix
Internet Appendix
Measurement Appendix
Technical Appendix
Variable Dictionary
Codebook
Questionnaire
Scale Items
Replication Code
Replication Data Description
额外表格 / 附表
额外公式说明
出版社单独提供的附件
Zotero 条目下的附属附件
```

这些材料不得被视为“次要文件”。
如果核心变量的定义、计算公式、词典、样本筛选或识别细节位于补充材料中，补充材料与正文具有同等的研究设计证据价值。

---

### 4.3.2 补充材料发现顺序

按以下顺序发现：

```text
1. 正文中明确引用的 Online Appendix / Supplement / Web Appendix
2. 出版社论文页面列出的 Supplementary Files
3. Zotero 同一父条目下的附件
4. 本地同目录或约定补充目录中的关联文件
5. 用户显式提供的其他文件
```

正文出现以下表述时必须触发补充材料检查：

```text
see Online Appendix
see Web Appendix
see Supplementary Material
see Internet Appendix
see Appendix S1
details are provided in the appendix
variable construction is described in the supplementary material
additional details are available online
```

也包括对应的中文表达：

```text
详见网络附录
详见在线附录
具体计算见补充材料
变量构造见附录
详细分类标准见附件
完整量表见网络附件
```

---

### 4.3.3 补充材料下载与存放

只有当前运行环境具备访问和下载权限时才能执行自动下载。

不得：

- 声称已经下载实际上无法访问的附件；
- 根据文件名猜测附件内容；
- 用第三方摘要替代官方补充材料。

建议每篇文献使用逻辑目录：

```text
{paper_bundle}/
├── 00_primary/
│   └── primary_article.pdf
├── 01_supplementary/
│   ├── online_appendix.pdf
│   ├── codebook.xlsx
│   └── replication_readme.txt
├── 02_reading_packet/
│   └── reading_packet.md
└── 03_output/
    └── variable_decomposition.xlsx
```

如果用户指定自己的目标目录，优先遵守用户目录结构。

---

### 4.3.4 正文与补充材料关联规则

每个补充文件必须绑定到唯一的 `文献ID`，并生成 `补充材料ID`。

优先使用以下关联证据：

```text
Zotero parent item
DOI
出版社论文页面
正文明确附件链接
题名 + 作者 + 年份
文件名与正文显式引用
```

禁止仅凭“文件在同一个文件夹”就自动认定为同一论文附件。

---

### 4.3.5 文献包完整度

每篇文献必须判断：

```text
完整
基本完整
缺少补充材料
补充材料待下载
无法确认是否存在补充材料
```

若正文明确指出变量计算见附件，但附件缺失：

- 不得将该变量标记为“已完整复现”；
- `03_测度复现` 中复现难度至少不能判为“低”；
- `原文未说明项` 应写明“正文将具体计算指向补充材料，但当前未获得附件”；
- `01_文献总览` 标记文献包不完整；
- `10_补充材料清单` 登记待获取材料。

---

## 4.4 语义理解与执行引擎分工

本 Skill 不假定不同运行环境具有相同的长文理解能力。

默认优先：

```text
ChatGPT：语义理解 / 学术拆解
Codex：文件、下载、索引、抽取、Excel 与批处理
```

### A. `chatgpt_full`

适用于 ChatGPT 能直接读取正文及补充材料时。

ChatGPT负责：

```text
构念识别
变量边界
测度理解
机制链条
理论关系
识别设计解释
跨文献标准化
迁移判断
```

可直接生成最终 UVD Excel。

---

### B. `hybrid_preferred`（推荐）

推荐工作流：

```text
Codex
↓
定位正文
↓
发现并下载补充材料
↓
建立 Document Bundle
↓
提取关键章节和证据
↓
生成 Reading Packet
↓
ChatGPT
↓
完成语义拆解
↓
形成结构化 UVD 内容
↓
Codex / ChatGPT
↓
写入固定 Excel
```

这是本 Skill 的首选执行方式。

---

### C. `codex_prepare`

Codex 只做准备工作，不做最终变量解释。

允许：

```text
文件扫描
元数据读取
附件发现
附件下载
正文/附件文本抽取
章节定位
关键词定位
原文证据摘录
参考文献基本信息整理
Reading Packet 生成
Excel 空表 / 原文事实字段预填
```

禁止：

```text
自行扩展理论构念
将代理指标反向定义为构念
自行判断相邻构念边界
自行补全作者未说明的测度步骤
自行生成复杂机制链条
将推测写为原文结论
```

这些字段统一留给 ChatGPT 语义审核。

---

### D. `codex_conservative`

只有无法进行 ChatGPT 语义复核时才使用。

Codex 可以填写：

```text
作者明确写出的变量名
公式
数据来源
样本
模型
明确假设
明确结果
正文原句或忠实摘要
```

对需要解释或整合的字段：

```text
理论构念
规范化定义
构念边界
相邻构念
机制链条
迁移评估
新颖性判断
```

优先填：

```text
需语义复核
```

而不是强行推断。

---

### 4.4.1 Reading Packet

当使用 Codex → ChatGPT 两阶段流程时，每篇论文建议生成：

```text
reading_packet.md
```

其作用不是“替代全文”，而是帮助 ChatGPT快速获取所有需要深度理解的原始证据。

固定内容：

```markdown
# 文献基本信息

# 文件清单
- 正文
- Online Appendix
- Supplementary Material
- Codebook
- 其他附件

# 研究问题原文

# 理论与假设相关原文

# 变量定义原文

# 变量测度原文

# 补充材料中的变量计算细节

# 数据与样本原文

# 模型与识别策略原文

# 稳健性与替代测度原文

# 需要语义判断的问题
- 构念边界
- 相邻变量区别
- 多维构念关系
- 机制链条
- 可迁移性
```

关键原则：

> Reading Packet 必须尽可能保留原文语境，不得只输出 Codex 自己的概括。

对于变量定义和计算方法，应优先保留完整相关段落，而非只提取单句关键词。

---

### 4.4.2 ChatGPT 语义拆解规则

ChatGPT 获取正文、补充材料或 Reading Packet 后，应：

1. 先理解作者的研究问题；
2. 再理解变量在理论模型中的角色；
3. 再读取变量定义；
4. 再读取测度与附录；
5. 最后进行构念标准化和迁移判断。

禁止只依据：

```text
变量表
公式
关键词检索结果
摘要
```

直接完成构念解释。

---

## 4.5 数据源去重规则

默认按以下优先级依次去重：

```text
DOI
→ 标准化题名
→ 题名 + 第一作者 + 年份
```

如果 DOI 相同，视为同一文献。

如果题名高度一致但文件版本不同，优先级：

```text
正式出版版
→ Accepted Manuscript
→ Working Paper / Preprint
→ 二手摘录或网页摘要
```

如不同版本在方法、样本或结果上存在实质差异，不得自动合并，必须分别保留并在备注中说明版本关系。

---

## 4.6 数据源可追溯性

Skill 不改变固定 Excel Sheet 和列名。

### 任务级数据源信息

必须在 `00_说明` 中记录：

```text
数据源模式
本地目录
Zotero Library
Zotero Collection
是否包含子目录
去重规则
读取优先级
输出语言
```

### 文献级数据源信息

每篇文献的实际来源写入 `01_文献总览` 的“备注”，建议格式：

```text
source_type=Zotero;
source_locator=My Library/Collection A/Subcollection B;
source_file=paper.pdf
```

或：

```text
source_type=local_folder;
source_locator=D:/Research/Literature/Topic A;
source_file=paper.pdf
```

若同一文献来自多个来源，全部记录并标明最终采用的全文版本。

---

## 4.7 输出语言参数

`output_language` 决定**拆解内容的输出语言**，但不改变文献基本信息的原始语言。

支持：

```yaml
output_language: zh-CN
```

或：

```yaml
output_language: en
```

### 当 `output_language = zh-CN`

无论原始文献是中文、英文或其他语言：

- 研究问题；
- 理论构念解释；
- 规范化定义；
- 构念核心；
- 构念边界；
- 测度概述；
- 构造步骤；
- 机制链条；
- 识别策略说明；
- 迁移评估；
- 风险判断；
- Excel 中所有分析性、解释性字段

均统一使用**简体中文**。

### 当 `output_language = en`

无论原始文献是中文、英文或其他语言：

- research question;
- construct interpretation;
- normalized definition;
- construct core;
- construct boundary;
- measurement summary;
- construction steps;
- mechanism;
- identification strategy;
- transfer assessment;
- risk assessment;
- all analytical / interpretive Excel fields

均统一使用**English**。

### 不受输出语言参数影响的字段

以下属于**文献基本信息**，必须保持原文，不翻译、不英文化、不中文化：

```text
作者姓名
题名
期刊 / 出版物名称
卷
期
页码
DOI
URL
出版社（如适用）
会议名称（如适用）
机构名称（如属于报告）
```

例如：

原始文献：

```text
李玉花, 林雨昕, 李丹丹. 2024. 人工智能技术应用如何影响企业创新. 中国工业经济, (3): 98–116.
```

即使：

```yaml
output_language: en
```

Excel 中“作者”“题名”“期刊/来源”“参考文献”仍保持中文原文。

反之，英文文献在中文输出模式下仍保持英文题名和期刊名称。

---

## 4.8 文献基本信息与参考文献显示规则

“参考文献”字段必须按常见学术参考文献格式整理，同时保持原始语言。

### 中文文献默认格式

```text
作者. 年份. 题名[J]. 期刊名称, 卷(期): 起止页码.
```

若无卷号：

```text
作者. 年份. 题名[J]. 期刊名称, (期): 起止页码.
```

示例：

```text
李玉花, 林雨昕, 李丹丹. 2024. 人工智能技术应用如何影响企业创新[J]. 中国工业经济, (3): 98–116.
```

### 英文文献默认格式

```text
Author, A. A., Author, B. B., & Author, C. C. (Year). Title. Journal Name, Volume(Issue), pages. DOI
```

示例：

```text
Smith, J. A., & Lee, K. M. (2025). Digital capabilities and supply chain adaptation. Journal of Management, 51(2), 123–148. https://doi.org/xx.xxxx/xxxxx
```

### 其他文献类型

#### 图书

中文：

```text
作者. 年份. 书名[M]. 出版地: 出版社.
```

英文：

```text
Author, A. A. (Year). Book title. Publisher.
```

#### 工作论文 / 报告

```text
作者/机构. 年份. 标题[R/Working Paper]. 机构/系列名称. DOI/URL.
```

#### 会议论文

```text
作者. 年份. 题名[C]. 会议名称, 会议地点/机构.
```

### 基本信息整理原则

1. **保持原文语言**；
2. 不翻译作者姓名；
3. 不翻译题名；
4. 不翻译期刊名；
5. 不自行补译出版社或机构名称；
6. DOI 优先使用标准 DOI；
7. DOI 不存在时可保留稳定 URL；
8. 页码、卷期、年份按原文；
9. 如元数据冲突，以正式出版版本为优先；
10. 无法确认的信息写“需全文核验”，不得猜测。

---

## 4.9 最低输入

至少提供以下之一：

- 文献全文；
- 可读取的论文文件；
- 用户已经整理的文献内容；
- 论文题录 + 摘要；
- 已有变量拆解表；
- 一组待比较候选变量；
- 可访问的数据源定位参数。

若仅有题录或摘要，只能进行有限拆解。

# 5. 证据纪律

## Rule 1：严格区分五类信息

### A. 原文明确
作者直接陈述的定义、假设、测度、公式、数据、样本、模型、结果。

### B. 原文结构化
原文分散在多个位置，但可以在不增加新含义的前提下整合为一个结构化描述。

### C. 分析推断
原文未直接表述，但可以由已知事实合理推导。

### D. 外部核验
仅在用户要求检索、验证或补充外部信息时使用。

### E. 迁移建议
面向用户目标研究提出的改造、替代、组合或应用方案。

禁止将 B、C、D、E 写成论文作者的原始结论。

## Rule 2：材料不足时必须降级结论

若只有题名、摘要、二手综述或初筛记录：
- 可以识别研究主题、部分变量名称和大致研究关系；
- 不得编造公式、量表题项、词典、算法、样本规则或数据库；
- 测度细节统一填“需全文核验”；
- 证据完整度标为“低”或“中”；
- 不得以网页摘要替代原文正式研究设计，除非用户明确要求外部核验。

## Rule 3：变量名称、理论构念、经验操作化必须分开

每个变量至少区分：

```text
原文变量名称
标准化变量名称
理论构念
操作化变量 / 经验代理
```

禁止只看变量名字就推定构念，或只看代理指标就反向定义理论构念。

## Rule 4：不得混淆变量的“概念层次、分析角色和经验指标”

这是通用于任意主题的核心防错规则。

### 4.1 概念性质

每个变量必须判断其主要属于哪一类：

```text
资源/投入
存量/状态
能力
行为/行动
过程/实践
认知/态度/感知
暴露/冲击/处理
关系/网络
制度/环境/情境
结构/属性
产出/绩效/结果
风险/约束
复合构念
其他
```

同一主题下的“投入—能力—行为—过程—结果”不能因为语义相近就合并。

### 4.2 模型角色

另行记录：

```text
核心解释变量
被解释变量
中介变量
调节变量
控制变量
工具变量
处理变量
分组/异质性变量
机制结果变量
其他
```

概念性质 ≠ 模型角色。

### 4.3 经验操作化

必须明确作者最终用什么可观察数据代表理论构念。

若多个指标共同形成一个变量，应判断：

```text
单指标
多指标加总
等权综合
加权综合
潜变量
形成式构念
反映式构念
分类变量
文本指标
网络指标
指数
其他
```

不得把构念、模型角色、经验指标三层混写。

## Rule 5：相邻构念必须做边界检查

每个重要变量至少回答：
1. 它最核心测量什么？
2. 它不测量什么？
3. 与最接近的相邻构念差异是什么？
4. 差异来自理论定义、时间维度、分析层级，还是经验测度？
5. 是否存在“旧构念换名称”的风险？

## Rule 6：复合变量必须检查组成逻辑

当论文或用户希望把多个维度合并为一个上位构念时，必须检查：
1. 是否存在明确共同上位概念；
2. 各维度是互补、阶段、并列还是重复测量；
3. 是否遗漏理论上必要维度；
4. 是形成式还是反映式逻辑；
5. 原文是否真的构建综合指标；
6. 若是用户迁移方案，必须标注“迁移建议”。

不得因为多个变量“都与同一主题有关”就直接相加或合并。

# 6. 标准拆解工作流

## Stage 1：文献身份与研究设计识别
提取作者、年份、题名、期刊/来源、DOI/URL、文献类型、研究范式、研究对象、分析层级、国家/地区、样本期与样本量、数据类型、核心研究问题、理论基础和核心变量关系。

## Stage 2：变量枚举
先枚举核心解释变量、被解释变量、中介、调节、关键控制变量、工具变量/处理变量、异质性变量和机制变量，再逐个拆解。用户只要求核心变量时可缩小范围，但 Excel Schema 不变。

## Stage 3：构念拆解
每个变量记录：
原文变量名、标准化变量名、模型角色、概念性质、分析层级、理论构念、原文定义、规范化定义、构念核心、构念边界、上位构念、下位维度、相邻构念和区分依据。

规范化定义只能压缩、统一术语和明确对象/属性/时间/层级，不得加入原文不存在的新理论含义。

## Stage 4：测度复现
还原原文测度概述、计算公式、构造步骤、原始数据、数据来源、访问方式、匹配键、分类/词典/量表来源、算法/模型、时间窗口、滞后、变换、聚合、异常值、缺失值、阈值、编码、替代测度、稳健性测度和前置依赖。

目标是使数据处理人员不重新阅读全文也能判断是否具备复现条件。原文未交代的步骤必须写“原文未说明”。

## Stage 5：机制关系拆解
一条关系一行。记录起点变量、终点变量、关系类型、方向、假设编号、中介/调节条件、理论机制、机制链条、理论基础、原文论证、实证方法、结果方向、支持状态和边界条件。

## Stage 6：模型与识别策略
提取主模型、变量、控制、固定效应、标准误、内生性来源、识别策略、工具变量/自然实验/匹配/DID/RDD 等、机制/稳健性/异质性检验，以及因果识别限制。不得因为方法复杂就自动判定因果识别强。

## Stage 7：复现性评估
判断数据可获得性、样本可匹配性、时间覆盖、算法透明度、分类/词典/量表公开程度、计算复杂度、关键外部依赖、复现难度和复现瓶颈。

复现难度统一使用：低 / 中 / 高 / 无法判断。

## Stage 8：迁移评估
仅当用户存在目标研究时进行实质判断；否则填“未指定目标研究”。

固定映射：

```text
原文构念
→ 原文操作化
→ 目标研究拟捕捉构念
→ 可直接保留部分
→ 必须修改部分
→ 应舍弃部分
→ 拟采用的新测度
```

并判断构念有效性、测度误差、数据风险、内生性风险、变量重叠、理论兼容性、机制兼容性、新颖性和迁移可行性。

## Stage 9：候选变量去重与比较
去重按“原文变量名 → 标准化构念 → 测度变体”，不得仅按名称去重。

比较至少考虑：构念清晰度、可测性、可复现性、数据可获得性、理论解释力、目标研究匹配度、新颖性、与现有变量区分度和主要风险。

若用户要求推荐，优先级使用：优先 / 备选 / 暂缓 / 不建议。除非用户明确要求，不使用单一总分掩盖权衡。

# 7. 新颖性判断

固定检查：

```text
概念新颖性
测度新颖性
数据新颖性
识别设计新颖性
机制新颖性
研究情境新颖性
跨层级迁移新颖性
```

并检查是否只是旧构念换名、换代理、换数据源，是否真正带来新的可检验机制或新增信息。

# 8. 固定 Excel 输出协议

技能包随附的固定模板位于 `assets/固定Excel模板_UVD-2.2.xlsx`。生成正式工作簿时，以该模板为起点另存输出文件，并保留模板文件不变；不要重建工作簿结构或更改 Sheet 名称、顺序和列名。

## 8.1 强制输出

每次生成 Excel 前必须读取 `run_config.output_language` 与 `run_config.source_config`。

- Excel 的分析性内容语言必须完全遵循 `output_language`；
- 文献基本信息必须保持原文；
- `00_说明` 必须记录数据源配置和输出语言；
- `01_文献总览` 的“备注”必须记录文献级来源定位；
- 不得为了语言统一而翻译题名、作者或期刊名。

除非用户明确说“不要生成文件”，每次使用本 Skill 完成正式拆解后，必须生成 `.xlsx`。

工作簿 Sheet 名称、顺序和列名固定，不得因主题改变。

固定 Schema 版本：`UVD-2.2`

推荐文件名：`文献变量拆解_UVD-2.2_YYYYMMDD.xlsx`

若用户要求追加到已有工作簿，应保持 Schema 不变并追加数据行。

## 8.2 固定工作簿结构

### Sheet 00_说明
固定列：项目｜内容｜说明

### Sheet 01_文献总览
固定列：
任务批次ID｜文献ID｜文献包ID｜检索/分组主题｜参考文献｜作者｜年份｜题名｜期刊/来源｜来源层级｜DOI/URL｜文献类型｜研究范式｜研究对象｜分析层级｜国家/地区｜样本期｜样本量｜数据类型｜核心研究问题｜理论基础｜核心变量关系｜是否实证｜是否进入变量拆解｜准入结论｜准入理由｜全文可得性｜是否存在补充材料｜补充材料状态｜文献包完整度｜证据完整度｜备注

### Sheet 02_变量拆解
固定列：
任务批次ID｜文献ID｜变量ID｜原文变量名｜标准化变量名｜模型角色｜概念性质｜分析层级｜理论构念｜原文定义｜规范化定义｜构念核心｜构念边界｜上位构念｜下位维度｜相邻构念｜区分依据｜操作化变量/经验代理｜测度类型｜方向/编码｜单位｜时间属性｜是否复合指标｜维度数｜维度名称｜理论来源｜与研究问题关系｜原文证据位置｜证据状态｜不确定项｜备注

### Sheet 03_测度复现
固定列：
任务批次ID｜文献ID｜变量ID｜测度方案ID｜原文测度概述｜计算公式｜构造步骤｜原始数据｜原始数据单位｜数据来源｜数据访问方式｜样本匹配键｜分类/词典/量表来源｜算法/模型｜时间窗口｜滞后/提前｜标准化/变换｜聚合规则｜缩尾/异常值处理｜缺失值处理｜阈值/分组｜编码规则｜替代测度｜稳健性测度｜前置依赖｜复现难度｜复现瓶颈｜原文未说明项｜原文证据位置｜证据状态｜备注

### Sheet 04_机制关系
固定列：
任务批次ID｜文献ID｜关系ID｜起点变量ID｜起点变量｜终点变量ID｜终点变量｜关系类型｜方向｜假设编号｜中介/调节条件｜理论机制｜机制链条｜理论基础｜原文论证｜实证检验方法｜结果方向｜显著性/支持状态｜异质性/边界条件｜原文证据位置｜证据状态｜可迁移机制｜迁移限制｜备注

### Sheet 05_识别与模型
固定列：
任务批次ID｜文献ID｜主模型｜因变量｜核心解释变量｜控制变量｜固定效应｜标准误处理｜内生性来源｜识别策略｜工具变量/处理变量｜机制检验｜稳健性检验｜异质性检验｜因果识别强度｜识别限制｜原文证据位置｜证据状态｜备注

### Sheet 06_迁移评估
固定列：
任务批次ID｜文献ID｜变量ID｜目标研究/项目｜目标样本｜目标层级｜拟迁移变量名｜拟捕捉构念｜可直接保留部分｜必须修改部分｜应舍弃部分｜拟采用测度｜数据可获得性｜样本可匹配性｜时间覆盖｜可复现性｜构念有效性风险｜测度误差风险｜数据风险｜内生性风险｜与现有变量重叠｜理论兼容性｜机制兼容性｜新颖性来源｜新颖性风险｜迁移可行性｜推荐角色｜优先级｜迁移结论｜核心理由｜待核验事项｜证据状态｜备注

### Sheet 07_证据摘录
固定列：
任务批次ID｜文献ID｜证据ID｜关联变量ID/关系ID｜证据用途｜原文位置｜页码｜章节｜表/图/附录｜原文摘录/忠实摘要｜证据类型｜证据状态｜支持的字段｜是否需全文核验｜外部核验来源｜备注

### Sheet 08_候选变量池
固定列：
候选ID｜标准化变量名｜上位构念｜概念性质｜常见模型角色｜来源文献数｜代表文献ID｜主要测度｜可用数据｜常用分析层级｜构念清晰度｜可测性｜可复现性｜数据可获得性｜理论解释力｜与目标研究匹配度｜新颖性类型｜主要优势｜主要风险｜推荐角色｜优先级｜当前状态｜去重说明｜备注

### Sheet 09_代码表
固定列：字段｜允许值｜定义

至少维护：证据状态、文献类型、模型角色、概念性质、测度类型、复现难度、优先级、当前状态、缺失值编码。

### Sheet 10_补充材料清单
固定列：
任务批次ID｜文献ID｜文献包ID｜补充材料ID｜材料角色｜材料名称原文｜文件名｜文件类型｜来源位置/URL｜获取方式｜本地目标路径｜下载状态｜与正文关联依据｜主要内容｜涉及变量ID｜涉及测度方案ID｜是否必须读取｜读取优先级｜是否已读取｜关键证据位置｜证据状态｜缺失影响｜备注

# 9. 固定值与缺失值编码

不得留含义不明的空白。

```text
原文未说明 = 文献全文存在，但作者未交代
需全文核验 = 当前材料不足
不适用 = 该字段对该类研究不成立
未指定目标研究 = 用户未提供迁移目标
无法判断 = 已有证据不足以形成判断
```

正式输出尽量不用空白替代上述状态。

# 10. Excel 填表粒度

```text
01_文献总览：一篇文献一行
02_变量拆解：一个变量一行
03_测度复现：一种变量×一种测度方案一行
04_机制关系：一条变量关系一行
05_识别与模型：一篇文献一行；多个主识别设计可拆行
06_迁移评估：一个变量×一个目标研究一行
07_证据摘录：一条证据一行
08_候选变量池：一个标准化候选构念一行
09_代码表：一个字段允许值一行
10_补充材料清单：一个补充文件一行
```

# 11. Excel 内容规则

1. 不得修改固定 Sheet 名和列名。
2. 不得因为某主题特殊而新增主题专属列。
3. 特殊信息放入“备注”或已有通用字段。
4. 多维变量使用“下位维度 / 维度名称”记录，不新增列。
5. 多个数据源、理论或机制使用分号 `；` 分隔。
6. 公式优先保留原文数学表达；无法结构化时忠实转写。
7. 原文证据位置尽可能具体。
8. 若使用外部检索，URL 放入 DOI/URL 或外部核验来源。
9. 不得把分析推断写入“原文定义”“原文论证”等字段。
10. “迁移评估”中的建议不得倒灌进原文事实 Sheet。
11. 正文和补充材料必须共享同一 `文献ID` / `文献包ID`。
12. 补充材料中的公式、词典、量表和算法证据必须登记到 `07_证据摘录`，并在 `10_补充材料清单` 中建立文件级索引。
13. 若变量计算依赖缺失附件，不得标记为“完整复现”。
14. Codex 准备模式产生的“需语义复核”不得自动转换成最终分析结论。

# 12. 标准执行顺序

```text
定位正文
→ 检查正文是否指向补充材料
→ 发现 / 获取补充材料
→ 建立 Document Bundle
→ 判断文献包完整度
→ （Codex 准备模式）生成 Reading Packet
→ 文献身份识别
→ 变量枚举
→ 构念拆解
→ 测度复现（正文 + 补充材料）
→ 机制关系拆解
→ 模型与识别拆解
→ 证据登记
→ 迁移评估（如有目标研究）
→ 候选池去重更新
→ 生成固定 Schema Excel
→ 完成一致性检查
```

# 13. 一致性检查

- [ ] 是否将变量名称、构念、操作化分开？
- [ ] 是否区分概念性质与模型角色？
- [ ] 是否说明构念边界和相邻构念？
- [ ] 是否把复合构念组成逻辑说明清楚？
- [ ] 是否把原文事实与推断、迁移建议分开？
- [ ] 是否记录原文证据位置？
- [ ] 是否对材料不足使用“需全文核验”？
- [ ] 是否把测度还原到可复现粒度？
- [ ] 是否记录数据、时间窗口、算法、编码和缺失处理？
- [ ] 是否把变量关系拆成独立关系记录？
- [ ] 是否记录模型与识别策略？
- [ ] 是否避免将高级方法自动等同于强因果识别？
- [ ] 是否按标准化构念而非变量名称去重？
- [ ] Excel 是否包含固定 11 个 Sheet？
- [ ] Sheet 顺序和列名是否完全符合 UVD-2.2？
- [ ] 是否没有添加主题专属列？
- [ ] 缺失信息是否使用统一编码？

- [ ] 是否读取并遵守 `output_language`？
- [ ] 分析性字段是否全部使用用户选择的输出语言？
- [ ] 作者、题名、期刊、卷期页、DOI 是否保持原文？
- [ ] “参考文献”字段是否按常见学术格式整理？
- [ ] 是否记录任务级数据源配置？
- [ ] 是否记录文献级来源定位？
- [ ] 是否检查正文是否引用 Online Appendix / Supplementary Material？
- [ ] 是否建立 Document Bundle？
- [ ] 变量计算依赖的补充材料是否已经获取并读取？
- [ ] 缺失附件是否明确降低“文献包完整度”和复现判断？
- [ ] `10_补充材料清单` 是否完整登记所有附件？
- [ ] 若由 Codex 执行，是否遵守当前 execution_profile？
- [ ] Codex 是否避免把语义推断伪装成原文事实？
- [ ] hybrid 模式下是否生成足够保留原文语境的 Reading Packet？


# 14. 标准文本回答

Excel 是正式结构化交付物；聊天回答只简要说明：
1. 拆解了多少篇文献；
2. 识别了多少个变量和测度方案；
3. 最重要的构念区分或复现风险；
4. 是否存在待全文核验项；
5. 提供 Excel 下载链接。

除非用户明确要求，不在聊天中重复完整 Excel。

# 15. 极简调用模板

```text
请按 universal-literature-variable-decomposition Skill 拆解这些文献。

run_config:
  output_language: zh-CN

  source_config:
    source_mode: uploaded_files

  document_bundle:
    enabled: true
    supplementary_materials:
      discover: true
      download_policy: explicit_or_authorized

  execution_profile:
    mode: hybrid_preferred
    semantic_analysis_engine: chatgpt
    file_execution_engine: codex

严格区分原文明确、原文结构化、分析推断、外部核验和迁移建议。
按照 UVD-2.2 固定 Schema 输出 Excel，不得改变 Sheet、列名和字段含义。
所有拆解与分析内容使用 output_language 指定语言；
作者、题名、期刊、卷期页、DOI 等文献基本信息保持原文，并按常见参考文献格式整理。
材料不足使用“需全文核验”，原文没有说明使用“原文未说明”。
```
