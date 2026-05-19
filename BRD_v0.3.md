# XDC BioFoundry 业务需求文档（BRD）v0.3

> **文档版本**：v0.3（用于 HTML 原型生成 + 开发输入）
> **编写日期**：2026-05-18
> **项目名称**：XDC BioFoundry / XDC 偶联工艺平台实验室数字化系统
> **文档用途**：作为 HTML 原型生成（v0 / Claude Code / Cursor）以及 React + TypeScript 开发的输入，覆盖框架、字段、状态机、组件级颗粒度，并包含 UI/UX 详细设计规范
>
> **v0.3 相比 v0.2 的变化**
> - §7.3 编码规则简化
> - §10.5 CSV 文档责任人修订为"对应业务 owner"
> - §11.1 / §11.2 I 期 MVP 范围扩展，纳入 Project + Study Plan 基础对象
> - **新增 §15 UI/UX 设计规范**（含色彩/字体/间距/组件库/核心页面 wireframe/交互模式）
>
> **v0.2 相比 v0.1 的变化**
> - 明确双层流程模型（Project/Study Plan 层 + Batch Run 层）
> - 修正 AS / XAS / BCOE / FL 的组织与角色关系
> - 明确"PD 替代 ELN、AS 仅做报表监控、BCOE 主数据目标替代 PE ELN"的分层定位
> - 补充 DOE/RSM、Holding Stability、Tech Transfer Package、客户提供物料等模块
> - 补充 Part 11 / GAMP 5 / ALCOA+ 合规映射
> - 增加数据对象字段级定义、状态机、组件设计

---

## 目录

1. [文档信息与读者](#1-文档信息与读者)
2. [业务背景与现状](#2-业务背景与现状)
3. [系统定位与目标](#3-系统定位与目标)
4. [用户与角色](#4-用户与角色)
5. [双层业务流程模型](#5-双层业务流程模型)
6. [功能模块设计](#6-功能模块设计)
7. [数据对象与关系](#7-数据对象与关系)
8. [接口与集成](#8-接口与集成)
9. [非功能需求](#9-非功能需求)
10. [合规与验证（Part 11 / GAMP 5 / ALCOA+）](#10-合规与验证)
11. [MoSCoW 优先级与分期计划](#11-moscow-优先级与分期计划)
12. [风险与待确认事项（TBD 清单）](#12-风险与待确认事项)
13. [验收标准](#13-验收标准)
14. [附录](#14-附录)
15. [UI/UX 设计规范（原型生成核心）](#15-uiux-设计规范)

---

## 1. 文档信息与读者

### 1.1 适用读者

| 角色 | 关注章节 |
|---|---|
| 产品/PM | 全文，重点 §3、§5、§11 |
| 原型设计师 | §4、§5、§6、§14.E（菜单） |
| TL / 开发 | §6、§7、§8、§9、§14.B（字段字典）、§14.C（状态机） |
| QA / CSV | §10、§13 |
| 业务负责人（PD/AS/BCOE） | §3、§4、§5、§6 |

### 1.2 文档约定

- **TBD** = 待业务确认（业务侧未给输入或系统侧需要更多上下文）
- **P0/P1/P2** = MoSCoW 中的 Must / Should / Could 优先级映射
- **§ 编号** = 章节交叉引用
- 字段定义使用 TypeScript interface 表达，便于直接转代码

---

## 2. 业务背景与现状

### 2.1 业务背景

XDC 是国内 ADC（抗体偶联药物）领域的 CRDMO/CDMO 服务商，核心业务围绕**偶联工艺（Bioconjugation Process）**展开，涉及以下三个主要部门：

| 部门 | 全称 | 职能 | 本系统覆盖范围 |
|---|---|---|---|
| **PD** | Process Development | 偶联工艺开发，包含 BCPD（Bioconjugation Process Development） | **核心覆盖**：ELN 替代 + 工艺协作 |
| **AS** | Analytical Science | 分析科学大部门，PD 的送样接收方 | **报表与流程监控**：检测仍在 LIMS/ELN 内执行 |
| ├─ XAS | XDC Analytical Science | 理化/结构表征（DAR、SEC、free drug、residual solvent、聚集体、降解体等） | 报表监控 |
| └─ BCOE | Bio Center of Excellence | 活性/生物学检测（bioassay、potency）+ **细胞库管理 + 物料管理** | **主数据替代**：目标替代 PE ELN |
| **BCDS** | Bio Conjugation Drug Substance | 类似内部 CRO 的部门 | **Out of Scope**（本期不考虑） |

### 2.2 现状痛点

1. PD 工艺记录目前在 ELN 中执行，但缺少**工艺级协作视图**（Study Plan、Batch Run、DOE 矩阵都散落在 Excel 和 ELN 字段里）。
2. AS 送样流程目前是**人工 + Excel 计划表**（参考无锡送样流程 PPT），PD 看不到送样和检测的实时状态，依赖邮件和微信追问。
3. BCOE 的细胞库和物料管理目前**都在 PE 的 ELN 中**，存在搜索不便、状态不透明、跨 site（上海/无锡）信息不互通等问题。
4. 客户提供的 Drug（payload）作为高价值受控物料，缺少专门的使用、剩余量、销毁追溯。
5. 偶联工艺涉及大量 DOE/RSM 设计，目前用 JMP/Design-Expert + Excel，结果回流到 ELN 是手工的。
6. Tech transfer to GMP 当前是 paper-based，没有标准化的工艺包导出机制。

### 2.3 信息来源

| 来源 | 说明 |
|---|---|
| ADC preparation route（XDC 业务图） | Layer 2 批次流程（8 个 stage）的依据 |
| BCPD&XAS project timeline | Layer 1 项目流程（5 个月 timeline）的依据 |
| Conjugation Process Development activities | Stage / Activity / Testing Items 映射 |
| 0414 无锡送样流程 PPT | 送样数据契约的参考 |
| BRD v0.1 | 范围、模块清单的起点 |
| 业务访谈补充 | 组织关系、CSV 范围、I 期边界 |

---

## 3. 系统定位与目标

### 3.1 一句话定位

> **XDC BioFoundry 是面向 XDC 偶联工艺平台的实验室数字化系统**，核心服务于 PD 工艺开发与协作，并为 AS 送样监控、BCOE 受控主数据管理提供统一入口。

### 3.2 分层定位（关键）

| 业务域 | 系统定位 | 与现有系统的关系 |
|---|---|---|
| **PD 工艺记录** | **替代 ELN**（在 PD 范围内） | 目标全量替代现有 ELN 的 PD 模块 |
| **PD 工艺协作**（Study Plan、Batch Run、DOE） | **新建** | 现有 ELN 无此能力 |
| **AS 检测执行**（XAS + BCOE 检测部分） | **不替代**，检测仍在外部 LIMS/ELN | 仅做报表和流程监控，从外部送样系统**单向拉数** |
| **BCOE 主数据**（细胞库、物料管理） | **目标替代 PE ELN** | 边界待 BCOE + QA 确认（见 §12 TBD-04） |
| **BCDS** | Out of Scope | 本期不考虑 |
| **ERP** | 不替代 | 单向接收物料到货数据 |
| **设备数据**（AKTA / UF-DF / Easy Max） | I 期不接入 | 后续扩展，预留接口 |

### 3.3 项目目标

| 目标 | 度量指标（建议） |
|---|---|
| 建立 PD 实验员统一工作台 | I 期上线 3 个月内 80% 的新 BCPD 项目在系统内创建 Study Plan |
| 替代 ELN 在 PD 范围内的使用 | II 期上线后，PD 新增实验记录 100% 在新系统 |
| 提供 AS 送样状态可视化 | 送样状态 dashboard 与外部送样系统的数据延迟 < 4 小时 |
| 替代 PE ELN 的 BCOE 主数据管理 | III 期上线后，细胞库 + 物料管理迁移完成，PE ELN 相关模块下线 |
| 满足 21 CFR Part 11 / ALCOA+ | GxP 模块 CSV 验证 0 critical findings |

### 3.4 In Scope（按 §11 分期）

详见 §11 MoSCoW 矩阵。

### 3.5 Out of Scope

| 范围 | 原因 |
|---|---|
| 外部送样流程发起 | 由独立的送样系统承担 |
| AS 检测执行（XAS 仪器操作、BCOE 活性检测执行） | 仍在外部 LIMS/ELN 内 |
| BCDS 部门业务 | 业务边界外 |
| 设备直接控制（AKTA 等） | I 期不做，预留扩展接口 |
| ERP 业务流程（采购、入库审批） | ERP 自有流程，本系统仅展示到货后的物料生命周期 |
| 接口管理前台（API Key、连接器配置） | 后台运维能力，不暴露给业务用户 |

---

## 4. 用户与角色

### 4.1 角色定义

#### 4.1.1 PD 侧

| 角色 | 主要目标 | 核心操作 | 系统权限 |
|---|---|---|---|
| **PD Manager** | 跨项目管理、资源分配、风险跟踪 | 项目组合视图、跨项目报表、模板审批 | 全 PD 读，配置流程模板 |
| **PD PM / Study Lead** | 单项目 / 单 Study Plan 推进 | 创建 Study Plan、维护 Batch Run 计划、跟踪 stage 完成度 | 项目内全权限 |
| **PD FL（Function Lead / 实验员组长）** | **送样监控、异常闭环、组内任务分配** | **送样 dashboard、SLA 告警处理、异常单跟进** | 监控视图 + 异常处理 |
| **PD Scientist / 实验员** | 执行 Batch Run、记录工艺参数、采样 | Batch Record 填写、样品创建、物料领用、电子签名 | 本人记录写、本组读 |
| **BCPD 工艺开发** | DOE 设计、参数优化、process lock | Study Plan 设计、DOE 矩阵、scale up 记录 | 工艺开发全流程 |

> **关于 FL 角色的说明**：FL = Function Lead，是 PD 内部角色（不是独立部门）。BRD v0.1 中"AS/FL 监控"的提法不准确——FL 是 **PD 侧** 看 AS（XAS+BCOE）检测进展的角色。

#### 4.1.2 AS 侧

| 角色 | 主要目标 | 核心操作 | 系统权限 |
|---|---|---|---|
| **XAS 检测员** | 接样、执行理化检测 | 在外部 LIMS 操作，本系统**只读**送样信息 | 极少使用本系统（只读视图） |
| **BCOE 检测员** | 接样、执行活性检测 | 在外部 LIMS 操作，本系统只读 | 同上 |
| **BCOE 主数据管理员** | **细胞库 + 物料库的 process owner** | 细胞库台账维护、物料效期管理、QA 审核 | 主数据全权限 |
| **AS Lead** | 跨 XAS/BCOE 调度 | 看整体检测产能和瓶颈 | 报表读 |

#### 4.1.3 横向

| 角色 | 主要目标 | 核心操作 | 系统权限 |
|---|---|---|---|
| **QA** | 合规边界、签核、审计 | 模板审核、电子签名复核、审计轨迹查询 | 全局读、受控变更审批 |
| **CSV 工程师** | 系统验证、变更控制 | 验证文档管理、change control、periodic review | 验证文档管理 |
| **系统管理员** | 用户、权限、字典、运维 | 用户角色、字段字典、流程规则、监控阈值 | 系统级管理 |
| **客户**（II 期+） | 查看自家项目进度 | 客户门户（client portal）只读视图 | 项目内只读，敏感字段脱敏 |

### 4.2 RACI 矩阵

| 流程活动 | PD Mgr | PM/Lead | FL | Scientist | BCPD | BCOE Mgr | AS Lead | QA | Admin |
|---|---|---|---|---|---|---|---|---|---|
| I 期范围冻结 | A | R | C | C | C | C | C | C | I |
| Study Plan 模板设计 | A | R | I | C | R | I | I | C | C |
| Batch Record 模板设计 | A | R | I | C | R | I | I | C | C |
| Batch Run 执行 | I | I | C | R/A | C | I | I | I | I |
| 送样监控与异常闭环 | I | C | **R/A** | C | C | I | C | I | I |
| 检测结果回写（自动） | I | I | I | I | I | I | I | I | A |
| 细胞库出入库 | I | I | I | C | I | **R/A** | I | C | I |
| 物料开瓶/dispose | I | I | I | R | C | A | I | C | I |
| 受控字段变更 | I | I | I | R | C | C | I | A | I |
| CSV 验证 | C | C | I | I | C | C | C | A | R |

> R = Responsible，A = Accountable，C = Consulted，I = Informed。

---

## 5. 双层业务流程模型

### 5.1 流程模型概述

XDC 偶联工艺涉及**两个时间尺度**的流程，必须在系统里分层建模，否则用户视图会混乱。

```
┌─────────────────────────────────────────────────────────────────┐
│ Layer 1：项目级流程（Project Lifecycle）                          │
│ 时间尺度：5 个月（参考 BCPD&XAS project timeline）                │
│ 主体：Project → Study Plan                                       │
│ 用户：PD Manager、PM、BCPD Lead                                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼ 一个 Study Plan 包含多个 Batch Run
┌─────────────────────────────────────────────────────────────────┐
│ Layer 2：批次级流程（Batch Run Lifecycle）                        │
│ 时间尺度：1–3 天（参考 ADC preparation route）                    │
│ 主体：Batch Run → 8 个 Process Stage                              │
│ 用户：PD Scientist、FL                                           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼ 每个 stage 可能触发送样
┌─────────────────────────────────────────────────────────────────┐
│ 送样监控（Sample Submission Monitor）                             │
│ 主体：Sample → Test Request → Test Result                         │
│ 数据源：外部送样系统（拉数）                                       │
│ 用户：PD FL（异常处理）、PM（结果查看）                            │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Layer 1：项目级流程

#### 5.2.1 流程阶段（参考 BCPD&XAS timeline）

| Phase | 名称 | 典型时长 | 主要活动 |
|---|---|---|---|
| P1 | Feasibility | 第 1 月 | Feasibility study using transferred process |
| P2 | Material Generation | 第 1–2 月 | Material generation for AS and XBPD study |
| P3 | Conjugation Process Development | 第 2–3 月 | DSD/RSM/DOE/OFAT study + confirmation study |
| P4 | Scale Up | 第 3 月 | Gram scale up + UF/DF optimization + ACF optimization |
| P5 | Holding Stability | 第 3 月 | Holding stability study for process intermediates |
| P6 | Process Lock | 第 4 月 | Process lock run with optimized conditions |
| P7 | Use Test & Non-GMP | 第 4–5 月 | Use test for lock run + Use test for Non-GMP run + Non-GMP run |
| P8 | Tech Transfer | 第 5 月 | Tech transfer to GMP team |

并行的 XAS / BCOE 工作流：
- ADC assay development（M1–M5 全程）
- Biosafety testing for mAb（M3–M5）

#### 5.2.2 Study Plan 设计（**系统侧规划，业务待确认**）

由于业务侧目前没有 Study Plan 的既定规范，本系统**建议采用以下结构**：

> **建议**：一个 Project 下按 Phase 拆分多个 Study Plan，典型一个项目 4–6 个 Study Plan。

| Study Plan 类型 | 对应 Phase | 典型 Batch Run 数 |
|---|---|---|
| Feasibility Study Plan | P1 | 1–2 |
| Material Generation Plan | P2 | 1–3 |
| Conjugation DOE Plan | P3 | 8–20（取决于 DOE 设计） |
| Scale-up Plan | P4 | 2–4 |
| Holding Stability Plan | P5 | 1–2（多时间点） |
| Process Lock Plan | P6 | 1–3 |
| Non-GMP Plan | P7 | 1–2 |

> ⚠️ **TBD-01**：Study Plan 的标准分类与命名规则需 BCPD 业务确认。原型中按上表展示，业务侧反馈后调整。

### 5.3 Layer 2：批次级流程（Batch Run）

#### 5.3.1 8 个 Process Stage（参考 ADC preparation route）

| Stage | 名称 | Inputs | Activities | Outputs | In-Process Testing |
|---|---|---|---|---|---|
| S1 | mAb intermediate thaw | mAb（from WXB） | 解冻条件控制 | Thawed mAb | 外观、浓度 |
| S2 | Reduction | mAb + TCEP（或其他还原剂） | 还原反应 | Free Thiol Produced | DAR、SEC、free thiol |
| S3 | UF/DF Purification | Reduced mAb | TFF or Unicorn，去除 TCEP/free payload/盐 | Purified mAb | SEC、free drug、residual solvent |
| S4 | Conjugation | mAb + Drug（from clients） | 偶联反应（pH/temp/time/Drug 比例） | ADC | DAR、SEC |
| S5 | Quench | ADC | 终止偶联 | Quenched ADC | 反应终止确认 |
| S6 | ACF Purification | Quenched ADC | Active carbon filter，去除小分子杂质 | Filtered ADC | 小分子杂质 |
| S7 | HIC Purification | Filtered ADC | 分离不同 DAR、去除聚集体/降解体 | Purified ADC | DAR distribution、聚集体、降解体 |
| S8 | DS Formulation | Purified ADC | 制剂、存储、下游处理 | DS（Drug Substance） | 放行前关键项目、稳定性 |

#### 5.3.2 Stage Activity 详细映射（参考 Conjugation Process Development activities）

| Study Stage | Activities | Testing Items |
|---|---|---|
| Small scale feasibility & material generation | For formulation team and analytical method development | DAR, SEC |
| Conjugation process development with OFAT / DoE | Optimization of：mAb intermediate reaction conc.、Reducing agent to mAb ratio、Drug/mAb、Organic solvent %、Reaction pH、Reaction temp.、Reaction time | DAR, SEC |
| Scale up for purification development | Time course, optimization of purification parameters | DAR, SEC, free drug, residual solvent |
| Process lock ADC DS | mAb intermediate input to lock conjugation and purification process; intermediate stability study | Based on analytical panel for lock DS |
| Use-test & ADC Non-GMP DS | Large scale mAb intermediate input from mAb Non-GMP batch | Based on analytical panel for tox DS |
| Tech transfer to GMP | Paper-based tech transfer protocol to GMP site（**本系统目标数字化此环节**） | — |

#### 5.3.3 IPC（In-Process Control） vs Release Testing

两类检测的处理逻辑不同，**系统中必须区分**：

| 维度 | IPC（In-Process Control） | Release / Characterization |
|---|---|---|
| 触发时机 | Stage 间快速检测 | Stage 完成后或最终放行 |
| 执行地点 | PD 内部快测 | 送至 XAS / BCOE |
| 数据回流 | 小时级 | 天级 |
| 用途 | 决定下一步是否继续 | 工艺评估、放行 |
| 在系统中的位置 | Batch Run 内嵌字段 | 送样监控视图 |
| SLA | 紧（小时） | 松（按 method 设定） |

### 5.4 Layer 1 → Layer 2 关系

```
Project (1)
  └── Study Plan (n)
        ├── DOE Design Matrix（可选，针对 Conjugation DOE Plan）
        └── Batch Run (n)
              ├── Stage S1 (1) ─── IPC Sample (0..n)
              ├── Stage S2 (1) ─── IPC Sample (0..n) ─── Test Request → XAS
              ├── Stage S3 (1) ─── IPC Sample (0..n)
              ├── Stage S4 (1) ─── IPC Sample (0..n) ─── Test Request → XAS
              ├── Stage S5 (1)
              ├── Stage S6 (1)
              ├── Stage S7 (1) ─── Release Sample → XAS / BCOE
              └── Stage S8 (1) ─── Release Sample → XAS / BCOE
```

### 5.5 状态机定义

#### 5.5.1 Project 状态机

```
[Draft] → [Active] → [On Hold] ↔ [Active] → [Completed]
                                          ↘ [Cancelled]
```

#### 5.5.2 Study Plan 状态机

```
[Draft] → [Pending Approval] → [Approved] → [In Progress] → [Completed]
                              ↘ [Rejected]                ↘ [Cancelled]
```

#### 5.5.3 Batch Run 状态机

```
[Planned] → [In Progress] → [Stage S1 Done] → ... → [All Stages Done]
                                                          ↓
                                                  [Pending Review]
                                                          ↓
                                              [Reviewed/Signed Off]
                                                          ↓
                                                     [Archived]

异常分支：任何状态 → [Deviation Raised] → [CAPA] → 回到原状态 或 [Cancelled]
```

#### 5.5.4 Sample 状态机

```
[Created] → [Labeled] → [Stored] → [Submitted for Testing] → [Under Testing]
                                                                  ↓
                                                          [Result Received]
                                                                  ↓
                                                  [Result Reviewed]
                                                                  ↓
                                                  [Archived] 或 [Disposed]
```

#### 5.5.5 Test Request 状态机（从外部送样系统同步）

```
[Submitted] → [Received by Lab] → [Testing In Progress] → [Result Available]
                                                              ↓
                                                  [Result Synced to BioFoundry]
                                                              ↓
                                                  [Reviewed]

异常分支：任何状态 → [Delayed] (SLA 超时) → [Issue Raised] → 闭环
```

#### 5.5.6 Material 状态机

```
[ERP Arrived] → [Received in Lab] → [In Storage] → [Opened/In Use] → [Used Up]
                                                  ↘ [Quarantine]   ↘ [Disposed]
                                                                   ↘ [Expired]
```

#### 5.5.7 Cell Bank Vial 状态机

```
[Created] → [QA Released] → [In Storage] → [Withdrawn for Use] → [Consumed]
                                          ↘ [Transferred to Another Site]
                                          ↘ [Disposed]
```

---

## 6. 功能模块设计

### 6.0 模块清单

| 模块编号 | 模块名 | 业务 Owner | 优先级 | CSV 等级 |
|---|---|---|---|---|
| M1 | Dashboard / 工作台 | PD/AS | P1（I 期需要 FL 视图） | Non-GxP（数据可信即可） |
| M2 | Project & Study Plan | PD | P0 | GxP |
| M3 | Batch Record（电子批记录） | PD | P0 | GxP（关键） |
| M4 | DOE / RSM 支持 | BCPD | P2 | GxP（Read-only 导入 OK） |
| M5 | 样品管理 | PD | P0 | GxP |
| M6 | 送样与检测监控 | PD-FL | **P0（I 期 MVP）** | Non-GxP（监控） |
| M7 | 细胞库管理 | BCOE | P1 | GxP |
| M8 | 物料管理 | BCOE | P1 | GxP |
| M9 | Holding Stability | BCPD | P2 | GxP |
| M10 | Tech Transfer Package | PD | P2 | GxP（导出受控） |
| M11 | 报表与看板 | 全角色 | P1 | Non-GxP |
| M12 | 系统管理后台 | Admin | P0（基础） | 支持 GxP |

### 6.1 M1：Dashboard / 工作台

#### 6.1.1 页面：实验员工作台 `/dashboard/scientist`

**布局**：左侧待办、中间活跃 Batch Run、右侧告警/通知

| 卡片 | 数据来源 | 字段 |
|---|---|---|
| 我的待办 | 待 Batch Record 填写、待复核、待签核 | task_id, type, batch_run_id, due_date, priority |
| 活跃 Batch Run | 当前实验员负责的 Run | run_id, stage_current, progress_pct, status |
| 最近样品 | 本周创建的样品 | sample_id, project, stage, status |
| 物料即将到期 | 已开瓶且 7 天内到期 | material_lot, opened_at, expire_at |

#### 6.1.2 页面：FL 监控工作台 `/dashboard/fl` 【**I 期 MVP 核心**】

详见 §6.6 送样与检测监控。

#### 6.1.3 页面：PM/Manager 工作台 `/dashboard/manager`

| 卡片 | 数据来源 | 字段 |
|---|---|---|
| 项目组合 | 所有 Active Project | project_id, phase, risk_level, completion_pct |
| 跨项目 Study Plan 进度 | 所有 In Progress Study Plan | plan_id, project, type, batch_count, completed_count |
| 本周风险项目 | Risk Level >= Medium 的项目 | project_id, risk_reason, owner |
| 关键里程碑预警 | 未来 14 天内到期的里程碑 | milestone, due_date, owner |

#### 6.1.4 组件设计

```typescript
// 通用卡片组件
interface DashboardCard {
  id: string;
  title: string;
  type: 'list' | 'metric' | 'chart' | 'alert';
  refreshInterval: number; // 秒
  permissions: string[]; // role IDs
  dataSource: string; // API endpoint
}

// 待办卡片
interface TodoItem {
  id: string;
  type: 'batch_record_fill' | 'review' | 'signoff' | 'sample_action' | 'deviation';
  title: string;
  link: string;
  dueDate: string; // ISO date
  priority: 'high' | 'medium' | 'low';
  createdAt: string;
}
```

### 6.2 M2：Project & Study Plan

#### 6.2.1 页面：项目列表 `/projects`

**筛选**：项目状态、Phase、负责人、客户、业务线
**列**：project_code, project_name, client, phase, status, lead, start_date, target_completion_date

#### 6.2.2 页面：项目详情 `/projects/:id`

**Tab 结构**：
- Overview（项目基本信息 + phase 时间轴）
- Study Plans（项目下所有 Study Plan）
- Batch Runs（跨 Study Plan 的所有 Run）
- Samples（项目下所有样品）
- Materials Used（领用物料记录）
- Test Results（关联检测结果汇总）
- Documents（附件、SOP、客户协议）
- Audit Trail（项目级审计轨迹）

#### 6.2.3 页面：Study Plan 创建/编辑 `/study-plans/new` / `/study-plans/:id`

**字段**：
```typescript
interface StudyPlan {
  id: string;
  code: string; // 自动生成：项目代码-Phase-序号
  projectId: string;
  type: 'Feasibility' | 'Material_Generation' | 'Conjugation_DOE'
      | 'Scale_Up' | 'Holding_Stability' | 'Process_Lock'
      | 'Non_GMP' | 'Other'; // TBD-01: 待业务确认
  phase: 'P1' | 'P2' | 'P3' | 'P4' | 'P5' | 'P6' | 'P7' | 'P8';
  title: string;
  objective: string;
  scope: string;
  designApproach?: 'OFAT' | 'RSM' | 'DSD' | 'DoE' | 'N/A';
  designMatrixUrl?: string; // 链接到 JMP / Design-Expert 文件
  plannedBatchCount: number;
  startDate: string;
  targetCompletionDate: string;
  leadId: string; // 用户 ID
  status: StudyPlanStatus;
  templateId?: string;
  attachments: Attachment[];
  // 审批
  approvers: ApproverChain[];
  // 关联
  relatedBatchRunIds: string[];
  relatedSampleIds: string[];
  // 审计
  createdBy: string;
  createdAt: string;
  signedOffBy?: string;
  signedOffAt?: string;
  signatureMeaning?: 'Author' | 'Reviewer' | 'Approver';
}
```

#### 6.2.4 关键交互

- **创建 Study Plan**：选择模板 → 填写基本信息 → 自动生成 code → 进入 Draft → 提交审批
- **审批链**：可配置（默认 Lead 提交 → Manager 审批 → QA 知会，TBD-02 待业务确认）
- **批量创建 Batch Run**：DOE 类型的 Study Plan 可基于 DOE 矩阵自动批量创建 Run（含参数）

### 6.3 M3：Batch Record（电子批记录）

#### 6.3.1 页面：Batch Run 列表 `/batch-runs`

**筛选**：所属 Project、Study Plan、Stage 进度、状态、执行人、日期
**列**：run_code, study_plan, stage_current, progress, status, scientist, start_date

#### 6.3.2 页面：Batch Run 详情（Stage Walkthrough） `/batch-runs/:id`

**核心 UI**：左侧 8 stage 时间轴（可点击切换），右侧当前 stage 的 record 表单。

```
[S1 ✓] → [S2 ✓] → [S3 ●] → [S4 ○] → [S5 ○] → [S6 ○] → [S7 ○] → [S8 ○]
                     ↑ 当前 stage（高亮）
```

每个 stage 内：
- **Header**：stage 名称、操作人、开始/完成时间、状态
- **Parameters**：工艺参数表单（按 stage 模板，含必填校验）
- **Inputs**：使用的 mAb / reagent / drug 批次（关联物料管理）
- **Outputs**：产出物（中间体浓度、体积、外观）
- **IPC Samples**：本 stage 采集的 IPC 样品（按钮：创建样品）
- **Submissions to AS**：本 stage 触发的送样请求（关联送样监控）
- **Deviations**：偏差记录
- **Attachments**：仪器导出文件、照片
- **Signatures**：执行人签名 + 复核人签名

#### 6.3.3 字段定义

```typescript
interface BatchRun {
  id: string;
  code: string;
  studyPlanId: string;
  projectId: string; // 冗余便于查询
  doeRunNumber?: number; // 若来自 DOE 矩阵
  doeParameters?: Record<string, number>; // DOE 设计的参数值
  scientistId: string;
  reviewerId?: string;
  startDate: string;
  endDate?: string;
  status: BatchRunStatus;
  currentStageId: string;
  stages: BatchStage[];
  signatures: ElectronicSignature[];
  attachments: Attachment[];
  auditTrail: AuditEntry[];
}

interface BatchStage {
  id: string;
  stageCode: 'S1' | 'S2' | 'S3' | 'S4' | 'S5' | 'S6' | 'S7' | 'S8';
  stageName: string;
  status: 'Not_Started' | 'In_Progress' | 'Done' | 'Skipped' | 'Deviated';
  startedAt?: string;
  completedAt?: string;
  operatorId?: string;
  parameters: StageParameter[];
  inputMaterials: MaterialUsage[];
  outputSamples: SampleRef[];
  ipcResults: IPCResult[];
  testRequestIds: string[]; // 关联送样请求
  deviations: Deviation[];
  notes?: string;
  signatures: ElectronicSignature[];
}

interface StageParameter {
  fieldKey: string; // e.g. 'reduction.tcep_ratio'
  fieldLabel: string;
  value: string | number;
  unit?: string;
  expectedRange?: { min?: number; max?: number };
  isOutOfRange?: boolean;
  recordedAt: string;
  recordedBy: string;
}
```

#### 6.3.4 Stage 模板示例：S2 Reduction

```typescript
const S2_REDUCTION_TEMPLATE: StageTemplate = {
  stageCode: 'S2',
  stageName: 'Reduction',
  parameters: [
    { key: 'reduction.mab_input_volume', label: 'mAb input volume', unit: 'mL', required: true },
    { key: 'reduction.mab_input_concentration', label: 'mAb concentration', unit: 'mg/mL', required: true },
    { key: 'reduction.reducing_agent', label: 'Reducing agent', type: 'select', options: ['TCEP', 'Other'], required: true },
    { key: 'reduction.agent_ratio', label: 'Reducing agent to mAb ratio', type: 'number', required: true },
    { key: 'reduction.reaction_ph', label: 'Reaction pH', type: 'number', expectedRange: { min: 7.0, max: 8.0 } },
    { key: 'reduction.reaction_temp', label: 'Reaction temperature', unit: '°C', expectedRange: { min: 20, max: 25 } },
    { key: 'reduction.reaction_time', label: 'Reaction time', unit: 'min', required: true },
  ],
  ipcTesting: [
    { key: 'free_thiol', label: 'Free Thiol', method: 'Ellman assay' },
  ],
  releaseTesting: [], // S2 通常不送样，根据业务调整
};
```

### 6.4 M4：DOE / RSM 支持（P2，先占位）

#### 6.4.1 范围

I 期不开发，II 期至少支持：
- DOE 矩阵导入（CSV / JMP export）
- 矩阵展开为多个 Batch Run（每个 run 自动填入参数）
- 结果回收按 DOE 维度可视化（main effect plot、interaction plot）

III 期可考虑系统内 DOE 设计向导。

> **决策建议**：DOE 设计继续由 BCPD 用 JMP/Design-Expert 完成，本系统负责"矩阵导入 → 批量生成 Run → 结果回收"。

### 6.5 M5：样品管理

#### 6.5.1 页面：样品列表 `/samples`

**筛选**：项目、Study Plan、Batch Run、Stage、样品类型、状态、位置、日期
**列**：sample_id, type, project, stage, status, location, created_at

#### 6.5.2 页面：样品详情 `/samples/:id`

**Tab**：
- Overview（基本信息）
- Lineage（样品谱系树：来源、衍生、拆分、合并）
- Status Log（状态变更历史）
- Test Requests（关联送样请求）
- Test Results（检测结果汇总）

#### 6.5.3 字段定义

```typescript
interface Sample {
  id: string;
  code: string; // 自动生成
  type: 'mAb_intermediate' | 'reduced_mAb' | 'conjugation_intermediate'
      | 'ADC_DS' | 'IPC' | 'stability_timepoint' | 'retain' | 'other';
  projectId: string;
  studyPlanId?: string;
  batchRunId?: string;
  stageId?: string; // 来源 stage
  parentSampleId?: string; // 谱系
  childSampleIds?: string[];

  // 物理属性
  volume?: number;
  volumeUnit?: 'mL' | 'µL' | 'L';
  concentration?: number;
  concentrationUnit?: 'mg/mL' | 'µg/mL';
  storageCondition: 'Room_Temp' | '4C' | '-20C' | '-80C' | 'LN2';
  location: {
    site: 'Shanghai' | 'Wuxi' | string;
    building?: string;
    freezer?: string;
    shelf?: string;
    box?: string;
    position?: string;
  };

  status: SampleStatus;
  labelGenerated: boolean;
  barcode?: string;
  qrcode?: string;

  createdBy: string;
  createdAt: string;
  archivedAt?: string;
  disposedAt?: string;
  disposeReason?: string;
}
```

### 6.6 M6：送样与检测监控【**I 期 MVP 核心**】

#### 6.6.1 页面：FL 监控 Dashboard `/monitoring/fl`

**这是 I 期唯一硬性目标功能。**

**布局**：
- **顶部 KPI 卡片**：本周送样数、在测数、超时数、异常未关闭数
- **中部主表格**：按 stage 维度展示所有送样请求
- **右侧告警面板**：实时 SLA 超时、结果缺失、异常待处理

**主表格列**：

| 列 | 说明 |
|---|---|
| 送样单号 | 来自外部送样系统 |
| Project / Study Plan / Batch Run | 关联上下文 |
| Stage | S1–S8 |
| 检测通道 | XAS / BCOE |
| 检测项 | DAR, SEC, free drug, etc. |
| 优先级 | High / Medium / Low |
| 送样时间 | 数据契约字段 |
| 接样时间 | 数据契约字段（可能为空） |
| 当前状态 | Submitted / Received / In Progress / Completed |
| 预期回报时间 | 基于 SLA |
| 实际回报时间 | |
| SLA 状态 | On Track / At Risk / Overdue |
| 异常 | 是否有未闭环异常 |
| 责任人 | 当前 FL |

**关键交互**：
- 点击送样单 → 进入送样详情
- SLA Overdue → 触发告警，FL 可点击"标记跟进中"或"创建异常单"
- 多维筛选：Project、Stage、检测通道、SLA 状态、优先级

#### 6.6.2 页面：送样详情 `/test-requests/:id`

只读视图，包含：
- 送样基本信息（来自外部送样系统）
- 关联样品、Batch Run、Stage 上下文
- 检测项清单与各自状态
- 检测结果（如已回写）
- 异常处理记录
- 时间线（提交 → 接样 → 在测 → 完成 → 回写各时点）

#### 6.6.3 数据契约（与外部送样系统）

```typescript
interface ExternalTestRequest {
  // 外部送样系统主键
  externalRequestId: string;
  externalSystemName: 'AS_Submission_System'; // TBD-03: 实际系统名待确认
  // 关联 BioFoundry 上下文（关键！）
  bioFoundryProjectCode?: string;
  bioFoundryStudyPlanCode?: string;
  bioFoundryBatchRunCode?: string;
  bioFoundryStageCode?: string; // S1-S8
  bioFoundrySampleCode?: string;
  // 送样信息
  submittedAt: string;
  submittedBy: string;
  priority: 'High' | 'Medium' | 'Low';
  expectedTurnaroundDays: number;
  // 检测项
  testItems: {
    itemCode: string; // e.g. 'DAR', 'SEC', 'free_drug'
    itemName: string;
    channel: 'XAS' | 'BCOE';
    method?: string;
  }[];
  // 状态
  currentStatus: 'Submitted' | 'Received' | 'In_Progress' | 'Completed' | 'Cancelled';
  receivedAt?: string;
  completedAt?: string;
  // 结果（如已完成）
  results?: TestResult[];
  // 同步信息
  syncedAt: string;
  syncSource: string;
}
```

> ⚠️ **TBD-03**：外部送样系统的具体名称、接口形式（API / 数据库直读 / 定时文件交换）、刷新频率待业务和 IT 共同确认。原型中按"4 小时刷新一次的 REST API"模拟。

#### 6.6.4 SLA 配置（系统管理后台可配）

```typescript
interface SLAConfig {
  testItemCode: string;
  channel: 'XAS' | 'BCOE';
  priority: 'High' | 'Medium' | 'Low';
  expectedTurnaroundHours: number;
  warningThresholdPct: number; // e.g. 80%（用了 80% 时间未完成 → At Risk）
}
```

### 6.7 M7：细胞库管理（BCOE 主数据，P1）

#### 6.7.1 页面：细胞库台账 `/cell-banks`

**列**：cell_line_code, batch_no, passage, project, location_site, qa_status, vial_count, withdrawal_count

#### 6.7.2 页面：细胞库详情 `/cell-banks/:id`

**Tab**：
- Overview（基本信息：细胞系、来源、代次）
- Vials（库存 vial 列表，按位置组织）
- Withdrawal History（出库历史）
- Transfer History（site 间转移）
- QA Status & Audit Trail

#### 6.7.3 字段定义

```typescript
interface CellBank {
  id: string;
  cellLineCode: string;
  cellLineName: string;
  source: string; // 来源
  parentBankId?: string; // 主库引用
  bankType: 'MCB' | 'WCB' | 'EOPCB' | 'Research'; // Master/Working/End-of-Production
  batchNumber: string;
  passage: string;
  projectIds: string[];
  totalVialCount: number;
  remainingVialCount: number;
  storageCondition: 'LN2' | '-80C';
  primarySite: 'Shanghai' | 'Wuxi';
  qaStatus: 'Pending' | 'Released' | 'Quarantine' | 'Rejected';
  qaReleasedAt?: string;
  qaReleasedBy?: string;
  documents: Attachment[];
}

interface CellBankVial {
  id: string;
  cellBankId: string;
  vialBarcode: string;
  location: {
    site: string;
    freezer: string;
    rack: string;
    box: string;
    position: string;
  };
  status: 'In_Storage' | 'Withdrawn' | 'Consumed' | 'Transferred' | 'Disposed';
  createdAt: string;
  withdrawnAt?: string;
  withdrawnBy?: string;
  withdrawalPurpose?: string;
}
```

#### 6.7.4 关键流程

- **入库**：BCOE 主数据管理员创建 vial 记录，扫码贴标，分配位置
- **出库**：实验员申请 → BCOE 审批 → 扫码出库 → 状态变更
- **盘点**：定期盘点流程，记录差异
- **跨 site 转移**：源 site 出库 → 转运 → 目标 site 入库

### 6.8 M8：物料管理（BCOE 主数据，P1）

#### 6.8.1 页面：物料台账 `/materials`

**筛选**：物料类型（化学试剂 / 客户提供 Drug / 耗材）、状态、效期、供应商、批号
**列**：material_code, name, lot, supplier, quantity_remaining, opened_at, effective_expire, status

#### 6.8.2 页面：物料详情 `/materials/:id`

**Tab**：
- Overview（物料基本信息、批次、效期）
- Lifecycle Events（到货 → 入库 → 开瓶 → 使用 → dispose）
- Usage Records（使用记录，关联 Batch Run）
- QA / CoA（合规文档）

#### 6.8.3 字段定义

```typescript
interface Material {
  id: string;
  materialCode: string; // 物料主数据 code
  materialName: string;
  materialType: 'chemical' | 'reagent' | 'client_drug' | 'consumable' | 'mAb_intermediate';
  // 客户提供物料的特殊字段
  isClientProvided: boolean;
  clientId?: string;
  clientReference?: string;
  cmcRestrictions?: string; // 客户 CMC 协议要求

  lotNumber: string;
  supplierName?: string;
  manufacturerLot?: string;

  // 数量与单位
  initialQuantity: number;
  remainingQuantity: number;
  quantityUnit: string;

  // 效期管理
  originalExpireDate: string;
  isOpened: boolean;
  openedAt?: string;
  openedBy?: string;
  postOpenExpireRule?: string; // e.g. "30 days after opening"
  postOpenExpireDate?: string; // 自动计算

  // 状态
  status: MaterialStatus;

  // 位置
  storageLocation: {
    site: string;
    location: string;
    condition: string;
  };

  // QA
  qaStatus: 'Pending' | 'Released' | 'Quarantine' | 'Rejected';
  coaAttachments: Attachment[];

  // ERP 来源
  erpReference?: string;
  erpArrivalDate?: string;
}

interface MaterialEvent {
  id: string;
  materialId: string;
  eventType: 'arrival' | 'received' | 'opened' | 'used' | 'transferred' | 'disposed' | 'expired';
  eventTime: string;
  operatorId: string;
  quantity?: number; // 涉及数量变化的事件
  reason?: string; // dispose / quarantine 原因
  relatedBatchRunId?: string; // 使用事件关联 Batch
  approvalRequired: boolean;
  approvedBy?: string;
  approvedAt?: string;
}
```

#### 6.8.4 开瓶后效期规则引擎

```typescript
interface PostOpenExpireRule {
  materialCode?: string; // 特定物料
  materialType?: string; // 物料类型
  ruleType: 'fixed_days' | 'min_of_two' | 'custom';
  fixedDays?: number; // e.g. 30
  // min_of_two: min(original_expire, opened_at + days)
  description: string;
  version: string; // 规则版本
  effectiveFrom: string;
}
```

> ⚠️ **TBD-05**：物料开瓶后效期规则的具体配置（哪些物料 30 天、哪些 60 天、哪些按原效期）需 BCOE + QA 共同确认并维护规则版本。

### 6.9 M9：Holding Stability（P2，先占位）

#### 6.9.1 需求

支持同一中间体（如 reduced mAb）在多个时间点取样测试：
- 创建 Holding Stability Plan（关联 Study Plan）
- 定义时间点：0h, 4h, 8h, 24h, 48h, 72h, ...
- 系统自动按时间点提醒取样
- 多时间点结果可视化对比

### 6.10 M10：Tech Transfer Package（P2，先占位）

#### 6.10.1 需求

Process lock 完成后，一键导出 Tech Transfer Package：
- 工艺参数终值（Stage S1–S8 锁定参数）
- IPC / Release 检测方法清单
- 关键样品稳定性数据
- 偏差与 CAPA 汇总
- 物料 BOM
- 导出为 PDF + Excel 双格式，含电子签名

### 6.11 M11：报表与看板

| 报表 | 受众 | 字段 |
|---|---|---|
| AS 监控周报 / 月报 | FL、PM、PD Manager | 送样数、在测数、超时率、关闭率、异常 TOP 列表 |
| BCPD Phase 进度报表 | PM、PD Manager | 按项目和 phase 展示完成度 |
| 实验记录合规率 | QA | 记录完整性、签核完整性、审计异常数 |
| 样品状态看板 | PD、PM | 在库、待送检、检测中、已归档 |
| 物料效期看板 | BCOE、实验员 | 即将到期、已开瓶、超期、dispose 状态 |
| 模块采用率 | PD Manager、IT | 用户活跃度、记录量、模块覆盖率 |

### 6.12 M12：系统管理后台

| 功能 | 说明 |
|---|---|
| 用户管理 | 创建/禁用、密码策略、SSO 集成 |
| 角色权限 | RBAC 配置、字段级权限、操作级权限 |
| 字段字典 | 全局枚举、单位、stage 模板、检测项主数据 |
| 流程规则 | 状态机配置、审批链配置 |
| SLA 配置 | 检测项 SLA、告警阈值 |
| 电子签名配置 | 签名 meaning、双因子、session 策略 |
| 审计查询 | 全局审计 trail 检索 |
| 数据迁移 | 历史数据导入工具 |
| 系统监控 | 接口同步状态、错误日志、性能 |

---

## 7. 数据对象与关系

### 7.1 实体关系图（文字表达）

```
Client (1) ─── (n) Project
Project (1) ─── (n) StudyPlan
StudyPlan (1) ─── (n) BatchRun
StudyPlan (1) ─── (0..1) DOEMatrix
BatchRun (1) ─── (8) BatchStage  [fixed: S1-S8]
BatchStage (1) ─── (n) StageParameter
BatchStage (1) ─── (n) MaterialUsage
BatchStage (1) ─── (n) Sample
BatchStage (1) ─── (n) IPCResult
BatchStage (1) ─── (n) TestRequest [外部送样]
BatchStage (1) ─── (n) Deviation

Sample (1) ─── (n) Sample [parent-child 谱系]
Sample (1) ─── (n) TestRequest

TestRequest (1) ─── (n) TestItem
TestRequest (1) ─── (n) TestResult

Material (1) ─── (n) MaterialEvent
Material (1) ─── (n) MaterialUsage [关联 BatchStage]

CellBank (1) ─── (n) CellBankVial
CellBankVial (1) ─── (n) VialEvent

User (1) ─── (n) Role
User (1) ─── (n) ElectronicSignature
任意 GxP 实体 (1) ─── (n) AuditEntry
```

### 7.2 核心实体清单

| 实体 | 关键字段 | GxP |
|---|---|---|
| Client | id, name, code, contact, contracts | Non-GxP |
| Project | id, code, name, clientId, phase, status, leadId | GxP |
| StudyPlan | id, code, projectId, type, phase, approach, status | GxP |
| BatchRun | id, code, studyPlanId, status, stages | GxP（关键） |
| BatchStage | id, runId, stageCode, parameters, status | GxP（关键） |
| Sample | id, code, type, status, location, lineage | GxP |
| TestRequest | id, externalId, samples, items, status | Non-GxP（监控） |
| TestResult | id, requestId, itemCode, value, unit, reviewedBy | GxP（如用于决策） |
| Material | id, code, lot, expiry, status, qaStatus | GxP |
| MaterialEvent | id, materialId, eventType, quantity, operator | GxP |
| CellBank | id, code, batch, passage, qaStatus | GxP |
| CellBankVial | id, bankId, location, status | GxP |
| User | id, email, roles, ssoId | 支持 GxP |
| Role | id, name, permissions | 支持 GxP |
| ElectronicSignature | id, userId, action, target, meaning, signedAt | GxP（强制） |
| AuditEntry | id, userId, action, entity, field, oldValue, newValue, reason, timestamp | GxP（强制，不可关闭） |
| Deviation | id, runId, stageId, description, severity, capaId | GxP |

### 7.3 ID 与编码规则建议

**设计原则**：编码要短、人类可读、能反推上下文；不要把所有层级都堆进编码（层级关系由数据库的外键保证，不靠编码字符串）。

| 实体 | 编码格式 | 示例 | 说明 |
|---|---|---|---|
| Project | `P-{Year}-{Seq}` | P-26-001 | 年份用 2 位，序号 3 位 |
| Study Plan | `SP-{Seq}` | SP-0042 | 全局自增，4 位序号 |
| Batch Run | `BR-{Seq}` | BR-1023 | 全局自增 |
| Sample | `S-{Seq}` | S-58291 | 全局自增 |
| Material | `M-{Seq}` | M-3012 | 全局自增 |
| Cell Bank | `CB-{Seq}` | CB-0125 | 全局自增 |
| Test Request | `TR-{Seq}` | TR-9821 | 来自外部送样系统 |
| Deviation | `DEV-{Seq}` | DEV-0034 | 全局自增 |

**关联关系**：在 UI 上展示时通过面包屑和上下文链接体现，不靠编码层级。例如 Sample S-58291 详情页面包屑展示：`P-26-001 / SP-0042 / BR-1023 / Stage S2 / S-58291`。

> ⚠️ **TBD-06**：编码规则的最终格式需 PD/BCOE/QA 共同确认。原型中按上表执行。

---

## 8. 接口与集成

### 8.1 集成清单

| 系统 | 方向 | 数据 | 频率 | I 期 |
|---|---|---|---|---|
| **外部送样系统** | 拉数 | TestRequest 状态 + Result | 4 小时 | ✅ I 期 MVP 必须 |
| 现有 ELN | 双向 | 用户、历史记录 | 批量 + 增量 | I.5 期 |
| ERP | 拉数 | 物料到货 | 实时 / 日 | I.5 期 |
| LIMS | 拉数 | 检测结果回写 | 实时 | I 期监控用 |
| PE ELN（BCOE 现用） | 一次性迁移 | 细胞库、物料历史 | 一次性 | II 期 |
| SSO（公司统一认证） | 单向 | 用户认证 | 实时 | I 期 |
| 设备（AKTA / UF-DF） | 拉数 | 仪器数据 | 实时 | III 期 |

### 8.2 接口设计原则

- **统一中间层**：所有外部数据通过 BioFoundry Integration Layer（独立服务）接入，业务模块只与中间层交互
- **数据契约文档化**：每个集成点维护 schema 文档 + version
- **失败处理**：重试 3 次 + 死信队列 + 监控告警
- **审计**：所有外部数据 ingestion 记录 sync_at + source + raw_payload（用于回溯）

### 8.3 接口示例：外部送样系统

```typescript
// GET /api/external/test-requests?since={timestamp}
interface TestRequestSyncResponse {
  syncedAt: string;
  records: ExternalTestRequest[]; // see §6.6.3
  nextSinceToken?: string; // 分页
}
```

---

## 9. 非功能需求

| 类别 | 要求 |
|---|---|
| **性能** | 列表查询 P95 < 1.5s（10万级数据）；Dashboard 加载 < 3s |
| **可用性** | I 期 99.0%（工作时间）；II 期 99.5% |
| **并发** | 至少 100 并发用户 |
| **安全** | HTTPS、SSO、RBAC、字段级权限、敏感字段加密存储、审计不可篡改 |
| **数据保留** | GxP 数据按 product lifetime + 10 年保留 |
| **备份** | 每日全量 + 实时增量，异地备份 |
| **浏览器** | Chrome / Edge 最新两个版本（主要支持），Safari（次要） |
| **响应式** | Desktop 优先，平板可用，手机不强求 |
| **国际化** | 中文为主，英文 II 期支持 |
| **可访问性** | I 期不强制，II 期参考 WCAG 2.1 AA |

### 9.1 技术栈（开发约定）

| 层 | 选型 | 备注 |
|---|---|---|
| 前端 | React + TypeScript + Vite | 与原型保持一致 |
| UI 库 | Ant Design 或 shadcn/ui | TBD-07 待 TL 确认 |
| 状态管理 | TBD（Zustand / React Query 组合） | |
| 后端 | TBD（Node.js / Java） | |
| 数据库 | PostgreSQL（业务）+ 对象存储（附件） | |
| 集成 | REST + 消息队列 | |
| 部署 | 内网 + 公司容器平台 | |

---

## 10. 合规与验证

### 10.1 GxP 模块分级

| 模块 | GxP 等级 | 验证策略 | Business Owner | QA Owner |
|---|---|---|---|---|
| Project & Study Plan | GxP | 完整 CSV | PD Manager | QA |
| Batch Record | **GxP 关键** | 完整 CSV + 加强 UAT | PD Manager | QA |
| 样品管理 | GxP | 完整 CSV | PD Manager | QA |
| 送样监控（read-only） | Non-GxP | 简化验证（数据可信即可） | PD-FL | — |
| 细胞库管理 | GxP | 完整 CSV | **BCOE** | QA |
| 物料管理 | GxP | 完整 CSV | **BCOE** | QA |
| Holding Stability | GxP | 完整 CSV | BCPD | QA |
| Tech Transfer Package | GxP（导出受控） | 完整 CSV | PD Manager | QA |
| Dashboard / 报表 | Non-GxP | 简化验证 | — | — |
| 系统管理后台 | 支持 GxP | 完整 CSV | IT/Admin | QA |

### 10.2 GAMP 5 分类声明

> 本系统按 **GAMP 5 Category 4（配置型软件） + Category 5（部分定制开发）** 处理。
> - Category 4：使用商用框架（React、PostgreSQL 等）
> - Category 5：业务流程、Batch Record 模板、状态机为定制开发

验证文档清单：
- URS（User Requirements Specification）
- FRS（Functional Requirements Specification）= 本 BRD 是输入
- DS（Design Specification）
- IQ（Installation Qualification）
- OQ（Operational Qualification）
- PQ（Performance Qualification）
- Validation Summary Report
- Traceability Matrix（URS ↔ Test Cases）
- Periodic Review（每年一次）

### 10.3 21 CFR Part 11 条款映射

| 条款 | 要求 | 系统实现 |
|---|---|---|
| §11.10(a) | 系统验证 | 完整 CSV 文档套 |
| §11.10(b) | 准确人类可读副本 | Batch Record 导出 PDF，与系统内一致 |
| §11.10(c) | 记录保护 | 数据库 + 备份，retention 期内可读 |
| §11.10(d) | 系统访问限制 | SSO + RBAC + 字段级权限 |
| §11.10(e) | **审计追踪（不可关闭）** | 所有 GxP 数据变更自动写 AuditEntry，**系统级强制开启**，管理员也不可关闭 |
| §11.10(f) | 操作顺序检查 | 状态机强制（如 Stage 必须按 S1→S8 顺序） |
| §11.10(g) | 权限检查 | RBAC + 字段级 + 操作级 |
| §11.10(h) | 设备检查 | 设备数据接入时验证（I 期 N/A） |
| §11.10(i) | 用户培训 | 培训记录模块，用户首次使用前校验培训完成 |
| §11.10(j) | 责任声明 | 用户签署 Electronic Signature Responsibility Statement（onboarding） |
| §11.10(k) | 文档控制 | 系统文档版本化管理 |
| §11.50 | 签名 manifestation | 每个 signature 显示：**Name + UTC Timestamp + Meaning** |
| §11.70 | 签名与记录链接 | Signature 与目标 record 强绑定，删除一方需级联标记 |
| §11.200 | 电子签名组件 | 双组件：UserID（SSO）+ Password / 短信验证码；首次签名输全部，session 内可只输密码 |
| §11.300 | ID/密码控制 | SSO 强密码策略；密码 90 天更换；账号唯一不复用 |

### 10.4 ALCOA+ 数据完整性映射

| 原则 | 系统实现 |
|---|---|
| **A**ttributable | 每条记录绑定 user + timestamp |
| **L**egible | 数据存储为结构化字段，导出格式人类可读 |
| **C**ontemporaneous | 时间戳由服务器生成，不由用户输入 |
| **O**riginal | 原始数据不可修改，修改通过 versioning 保留历史 |
| **A**ccurate | 字段校验、单位强制、范围检查 |
| **C**omplete | 必填校验、完整性 check |
| **C**onsistent | 全局字典、单位统一、状态机一致 |
| **E**nduring | 数据库 + 备份，retention 期 |
| **A**vailable | 授权用户随时可访问 |

### 10.5 CSV 文档与责任人

**原则**：每份文档的 Business Owner = 对应模块的业务负责人；TL 负责技术审核；QA 负责合规放行。

| 文档 | Owner（按模块业务 owner 对应） | Reviewer | Approver |
|---|---|---|---|
| URS - PD 模块（Project、Study Plan、Batch Record、样品、送样监控） | PD Manager | TL + QA | QA |
| URS - BCOE 模块（细胞库、物料） | BCOE Manager | TL + QA | QA |
| URS - 系统管理 | IT/Admin | TL + QA | QA |
| FRS（本 BRD） | TL | PM + 业务 owner | PD Manager + BCOE Manager + QA |
| DS（Design Specification） | TL | Architects | TL |
| IQ/OQ/PQ Protocols | CSV Engineer / TL | QA | QA |
| Validation Report | CSV Engineer / TL | TL + 业务 owner | QA |
| Change Control | 变更发起人 | TL + QA | QA |
| Periodic Review（年度） | CSV Engineer / TL | TL + 业务 owner | QA |

> **说明**：I 期资源有限，CSV Engineer 角色可由 TL 兼任，QA 由公司统一 QA 团队负责。后续 II 期上量后可考虑专职 CSV 工程师。

---

## 11. MoSCoW 优先级与分期计划

### 11.1 分期总览

| 期 | 目标时间 | 范围 | 关键决策 |
|---|---|---|---|
| **I 期 MVP** | 2026 Q4（软目标） | **送样 FL 监控报表** + **Project & Study Plan 基础对象** + 用户权限 + 基础审计 | 不替代 ELN，工艺记录暂留在外部 |
| **I.5 期** | 2027 H1 | Batch Record（8 stage 完整流转）+ 样品管理（完整）+ 基础报表 | 开始替代 PD ELN |
| **II 期** | 2027 H2 | 细胞库 + 物料管理 + 客户门户 + 报表深化 | 替代 PE ELN（需 QA 确认） |
| **III 期** | 2028 H1 | DOE / Holding Stability / Tech Transfer / 设备数据 | 完整工艺数字化闭环 |

### 11.2 MoSCoW 矩阵

#### I 期 MVP（Must Have）

I 期目标是让送样监控真正可用，并搭好 Project / Study Plan 的"骨架"对象，让 II 期 Batch Record 接入时不需要返工。

| 模块 | I 期范围 | 工作量估计（人月） |
|---|---|---|
| M6 送样监控 FL Dashboard | 数据契约 + 拉数 + Dashboard + SLA + 异常单 | 2.0 |
| **M2 Project & Study Plan（基础）** | **Project CRUD + Study Plan CRUD + 列表/详情视图 + 简单审批（单步）** | **1.5** |
| M5 样品管理（最小化） | 样品 CRUD + 列表 + 详情 + 与送样关联（不含谱系） | 1.0 |
| M12 系统管理（基础） | 用户、角色、SSO、字段字典 | 1.0 |
| 审计 + 电子签名（基础） | 强制审计、签名组件、签名 manifestation | 0.5 |
| 集成层（外部送样系统接入） | API / 中间层 / 重试 / 死信队列 | 1.5 |
| 系统框架与基础 UI | 布局、导航、通用组件库 | 1.0 |
| **小计** | | **8.5** |

> **资源约束验证**：1 TL + 2 Dev × 5 个月 ≈ 12 人月可用产能。I 期 8.5 人月留 30% buffer 给设计/测试/CSV 文档，**仍然可行但偏紧**。建议把 M2 的"简单审批"放在 I 期最后做，必要时可降级为"提交即生效 + 后审"。

#### I.5 期（Should Have）

| 模块 | 范围 |
|---|---|
| M3 Batch Record | 完整（8 stage 模板 + 状态机 + 双签名 + 偏差） |
| M5 样品管理（增强） | 样品谱系树 + 标签打印 + 位置选择器 |
| M2 Project & Study Plan（增强） | 多步审批链 + 模板管理 + DOE 矩阵占位 |
| M11 报表（基础几个） | AS 监控周报、Phase 进度、样品状态 |

#### II 期（Should Have）

| 模块 | 范围 |
|---|---|
| M7 细胞库管理 | 完整（依赖 PE ELN 替代决议） |
| M8 物料管理 | 完整（含开瓶后效期规则引擎、客户提供 Drug 特殊管理） |
| 客户门户（轻量） | 客户只读视图 |
| M11 报表深化 | 物料效期看板、合规报表、模块采用率 |

#### III 期（Could Have）

| 模块 | 范围 |
|---|---|
| M4 DOE / RSM | 矩阵导入 + 结果可视化 |
| M9 Holding Stability | 多时间点采样与对比 |
| M10 Tech Transfer Package | 一键导出 |
| 设备数据接入 | AKTA / UF-DF / Easy Max |

#### Won't Have（本期）

- BCDS 部门业务
- 替代 ERP / LIMS
- 接口管理前台
- 设备自动控制
- 完整客户协作平台

---

## 12. 风险与待确认事项（TBD 清单）

| ID | 项目 | 影响 | 阻塞优先级 | 建议动作 |
|---|---|---|---|---|
| **TBD-01** | Study Plan 标准分类与命名规则 | 影响 M2 模板设计 | High | BCPD 业务先行确认，原型按建议表展示 |
| **TBD-02** | Study Plan / Batch Record 审批链 | 影响 M2/M3 工作流 | High | QA + 业务共同确认 |
| **TBD-03** | 外部送样系统的具体名称、接口形式、刷新频率 | **I 期 MVP 阻塞**！ | **Critical** | I 期启动前必须明确 |
| **TBD-04** | BCOE 主数据替代 PE ELN 的边界 | 影响 II 期 | Medium | BCOE + QA 共同确认 |
| **TBD-05** | 物料开瓶后效期规则 | 影响 M8 | Medium | BCOE + QA 共同规则库 |
| **TBD-06** | 全局编码规则（Project / Sample / Material 等） | 影响主数据 | Medium | PD + BCOE + QA 三方会议确认 |
| **TBD-07** | UI 组件库选型（Ant Design vs shadcn/ui） | 影响开发效率 | Low | TL 决策 |
| **TBD-08** | DOE 设计工具集成方案（JMP vs 内置） | 影响 III 期 | Low | BCPD + IT 共同评估 |
| **TBD-09** | Tech Transfer Package 的导出规范 | 影响 III 期 | Low | PD Manager + GMP 团队共同定义 |
| **TBD-10** | 客户提供 Drug 的 CMC 约束规则 | 影响 M8 | Medium | 法务 + PD + 客户合同审查 |
| **TBD-11** | I 期 MVP 上线后的 CSV 验证范围 | 影响合规 | High | QA 在 I 期启动前确认 GxP 边界 |
| **TBD-12** | 设备数据接入的优先级与设备清单 | 影响 III 期 | Low | 业务调研 |

### 12.1 项目风险

| 风险 | 概率 | 影响 | 缓解措施 |
|---|---|---|---|
| 外部送样系统接口未就绪 | 中 | **I 期阻塞** | I 期启动前与对方系统 owner 签订数据契约；准备 mock 数据保证开发不停 |
| BCOE 替代 PE ELN 受阻 | 中 | II 期延后 | II 期与 BCOE/QA 联合启动验证规划 |
| CSV 文档跟不上开发节奏 | 高 | 上线延期 | 文档与开发并行，CSV 工程师在 I 期介入 |
| BCPD 团队不接受 DOE 由 JMP 完成 | 低 | III 期范围扩大 | 早期沟通，明确边界 |
| I 期 1 TL + 2 Dev 资源不足 | 中 | 范围收窄 | 范围已按资源量身定做（送样 FL 单一目标），可行 |

---

## 13. 验收标准

### 13.1 I 期 MVP 验收标准

#### 业务验收

1. PD FL 可在系统中查看穿插于 BCPD 工艺阶段的所有 AS 送样请求及实时状态。
2. SLA 超时告警可在 dashboard 中按 Project / Stage / 检测通道筛选。
3. 异常送样单可在系统中创建、跟进、关闭，并记录责任人与时长。
4. 外部送样系统数据延迟不超过 4 小时（或按 TBD-03 确认的频率）。
5. 系统支持至少 50 名 PD/FL/PM 用户的并发使用。

#### 系统验收

1. 所有 GxP 数据变更触发审计 trail，审计系统级强制开启不可关闭。
2. 电子签名包含 Name + Timestamp + Meaning 三要素。
3. SSO + RBAC + 字段级权限正常工作。
4. 外部送样系统接口失败有重试 + 死信队列 + 告警。
5. 关键页面 P95 响应时间 < 1.5s。

#### 合规验收

1. 完成 GxP 模块的 IQ/OQ/PQ（送样监控为 Non-GxP，简化验证）。
2. Part 11 §11.10/50/70/200/300 全条款 traceability matrix 通过 QA review。
3. ALCOA+ 9 条原则有功能映射且测试覆盖。
4. 用户培训记录模块上线，所有 GxP 用户培训完成方可使用。
5. Validation Summary Report 由 QA Head 签署放行。

### 13.2 I.5 / II / III 期验收

各期单独定义 URS 与 UAT，遵循同样的"业务 + 系统 + 合规"三层结构。

---

## 14. 附录

### 14.A 术语表

| 术语 | 全称 / 说明 |
|---|---|
| ADC | Antibody-Drug Conjugate，抗体偶联药物 |
| AS | Analytical Science，分析科学（XDC 内部部门） |
| BCOE | Bio Center of Excellence，生物中心（AS 下属，含细胞库/物料/活性检测） |
| BCPD | Bioconjugation Process Development，偶联工艺开发 |
| BCDS | Bio Conjugation Drug Substance（本期 out of scope） |
| BRD | Business Requirements Document，业务需求文档 |
| CAPA | Corrective and Preventive Action |
| CMC | Chemistry, Manufacturing, and Controls |
| CSV | Computerized System Validation，计算机化系统验证 |
| DAR | Drug-to-Antibody Ratio |
| DOE / DoE | Design of Experiments |
| DS | Drug Substance |
| DSD | Definitive Screening Design |
| ELN | Electronic Lab Notebook |
| ERP | Enterprise Resource Planning |
| FL | Function Lead（PD 内部角色，负责送样监控等） |
| GAMP 5 | Good Automated Manufacturing Practice v5 |
| GMP | Good Manufacturing Practice |
| HIC | Hydrophobic Interaction Chromatography |
| IPC | In-Process Control |
| LIMS | Laboratory Information Management System |
| mAb | Monoclonal Antibody |
| MCB / WCB | Master Cell Bank / Working Cell Bank |
| OFAT | One Factor At a Time |
| Part 11 | 21 CFR Part 11，FDA 电子记录电子签名法规 |
| PD | Process Development，工艺开发 |
| PE | Process Engineering（现有 ELN 提供方，BCOE 主数据载体） |
| QA | Quality Assurance |
| RSM | Response Surface Methodology |
| SEC | Size Exclusion Chromatography |
| SSO | Single Sign-On |
| TCEP | Tris(2-carboxyethyl)phosphine，还原剂 |
| UF/DF | Ultrafiltration / Diafiltration |
| URS | User Requirements Specification |
| XAS | XDC Analytical Science（AS 下属，理化/结构表征） |

### 14.B 字段字典初版（核心枚举）

```typescript
// 项目阶段
type ProjectPhase = 'P1' | 'P2' | 'P3' | 'P4' | 'P5' | 'P6' | 'P7' | 'P8';

// Stage
type StageCode = 'S1' | 'S2' | 'S3' | 'S4' | 'S5' | 'S6' | 'S7' | 'S8';

// 状态枚举
type ProjectStatus = 'Draft' | 'Active' | 'On_Hold' | 'Completed' | 'Cancelled';
type StudyPlanStatus = 'Draft' | 'Pending_Approval' | 'Approved' | 'Rejected'
                    | 'In_Progress' | 'Completed' | 'Cancelled';
type BatchRunStatus = 'Planned' | 'In_Progress' | 'Pending_Review'
                   | 'Reviewed' | 'Archived' | 'Cancelled';
type SampleStatus = 'Created' | 'Labeled' | 'Stored' | 'Submitted_For_Testing'
                 | 'Under_Testing' | 'Result_Received' | 'Result_Reviewed'
                 | 'Archived' | 'Disposed';
type MaterialStatus = 'ERP_Arrived' | 'Received' | 'In_Storage' | 'Opened'
                   | 'In_Use' | 'Used_Up' | 'Quarantine' | 'Expired' | 'Disposed';
type TestRequestStatus = 'Submitted' | 'Received' | 'In_Progress'
                      | 'Completed' | 'Cancelled' | 'Delayed';

// SLA
type SLAStatus = 'On_Track' | 'At_Risk' | 'Overdue';

// 签名 meaning
type SignatureMeaning = 'Author' | 'Reviewer' | 'Approver' | 'Witness';

// 通用
interface Attachment {
  id: string;
  filename: string;
  url: string;
  size: number;
  mimeType: string;
  uploadedBy: string;
  uploadedAt: string;
}

interface ElectronicSignature {
  id: string;
  userId: string;
  userName: string;
  meaning: SignatureMeaning;
  targetEntityType: string;
  targetEntityId: string;
  signedAt: string; // server timestamp
  ipAddress?: string;
}

interface AuditEntry {
  id: string;
  entityType: string;
  entityId: string;
  fieldKey?: string;
  action: 'create' | 'update' | 'delete' | 'sign' | 'export';
  oldValue?: any;
  newValue?: any;
  userId: string;
  userName: string;
  timestamp: string;
  reason?: string;
  ipAddress?: string;
}
```

### 14.C 状态机汇总

见 §5.5。原型中状态机使用 XState 或类似方案实现，确保转移逻辑可视化。

### 14.D Part 11 详细映射

见 §10.3。开发时每个 GxP 模块的功能必须能映射到至少一条 Part 11 要求。

### 14.E 原型菜单结构

```
顶部导航
├── 工作台（Workspace）
│   ├── 我的工作台 /dashboard/scientist
│   ├── FL 监控台 /dashboard/fl                    【I 期 MVP 核心入口】
│   └── 项目管理台 /dashboard/manager
│
├── 项目与流程（Projects & Process）
│   ├── 项目列表 /projects
│   ├── Study Plan /study-plans
│   ├── Batch Run /batch-runs
│   └── 工艺流程视图 /process-view
│
├── 实验数据（Lab Data）
│   ├── 样品管理 /samples
│   ├── 送样监控 /monitoring                       【I 期 MVP】
│   ├── 检测结果 /test-results
│   └── 偏差与 CAPA /deviations
│
├── 主数据（Master Data）【BCOE】
│   ├── 细胞库 /cell-banks
│   ├── 物料 /materials
│   └── 物料效期看板 /materials/expiry
│
├── 报表（Reports）
│   ├── AS 监控周报 /reports/as-weekly
│   ├── Phase 进度 /reports/phase
│   ├── 样品状态 /reports/samples
│   └── 合规报表 /reports/compliance
│
└── 管理（Admin）
    ├── 用户与角色 /admin/users
    ├── 字段字典 /admin/dictionary
    ├── 流程规则 /admin/workflow
    ├── SLA 配置 /admin/sla
    ├── 模板管理 /admin/templates
    ├── 审计查询 /admin/audit
    └── 系统监控 /admin/system
```

---

## 15. UI/UX 设计规范

> **本章的用途**：作为 HTML 原型生成的核心输入（v0 / Claude Code / Cursor / Bolt 等 AI 工具），以及后续 React + TypeScript 开发的视觉与交互参照。
> 本章按"全局规范 → 通用组件 → 核心页面"三层组织，每个核心页面包含 ASCII wireframe、组件清单、交互说明。

### 15.1 设计原则

#### 15.1.1 五大设计原则

1. **专业克制（Professional Restraint）**：医药行业用户，UI 应传达可信赖感。避免亮色、渐变、装饰性元素。整体偏 Linear / Vercel / Notion 风格。
2. **信息密度优先（Density First）**：实验员每天处理大量样品、批次、检测项。表格密度应接近 Linear / Airtable，**不要 Material Design 的大留白**。
3. **状态可视化（Status-Driven）**：状态是系统的核心信息。用颜色 + 形状 + 文字三重编码（不能只靠颜色，避免色盲不友好）。
4. **路径清晰（Clear Wayfinding）**：用户经常需要在 Project → Study Plan → Batch Run → Stage → Sample 五层之间跳转。面包屑、上下文链接、返回按钮必须完整。
5. **GxP 严肃性（GxP Seriousness）**：受控操作（签名、删除、关键字段修改）必须有二次确认 + 输入原因，不能用普通"确认"按钮。

#### 15.1.2 行业参考产品（借鉴交互、不抄视觉）

| 场景 | 主要参考 | 借鉴点 |
|---|---|---|
| 整体后台框架 | **Linear** | 简洁的左侧导航、紧凑的内容区、命令面板（Cmd+K） |
| 企业级布局成熟度 | **Ant Design Pro** | ProTable、ProForm、ProLayout 模式 |
| Dashboard / 监控台 | **Datadog Monitors** + **Grafana** | KPI 卡片、告警面板、状态分布饼图、时序对比 |
| 数据表格 | **Linear Issues** + **Airtable** | 紧凑表格、列宽自调、行内编辑、批量操作 |
| 8-stage 流转视图 | **GitLab Pipeline** + **Vercel Deployments** | 横向时间线、stage 节点、点击展开详情 |
| 样品 / 物料管理 | **Benchling Inventory** | 树形位置选择器、Vial 网格视图、谱系树 |
| 表单录入 | **Notion** + **Linear Issue Form** | inline 编辑、自动保存、字段分组 |
| 数据可视化 | **Tableau Public** + **Observable Plot** | 简洁、可读、不炫技 |
| 命令面板 | **Linear** + **Raycast** | Cmd+K 跳转、快速搜索 |

#### 15.1.3 反模式（避免）

- ❌ 大圆角卡片（>12px）、阴影过重
- ❌ 渐变背景、彩色按钮（除强调操作外）
- ❌ 大量动画与过渡
- ❌ emoji 装饰
- ❌ Material Design 风格的 FAB（浮动按钮）
- ❌ 过度使用 modal 嵌套（最多 2 层）
- ❌ 表格中混用颜色块作为背景（除告警行外）

### 15.2 视觉规范

#### 15.2.1 色彩系统

**主色调（Primary）**：深蓝灰，传达专业与冷静

```css
/* Primary - Deep Slate Blue */
--primary-50:  #F0F4F8;
--primary-100: #D9E2EC;
--primary-200: #BCCCDC;
--primary-300: #9FB3C8;
--primary-400: #829AB1;
--primary-500: #627D98;
--primary-600: #486581;  /* 主按钮、链接、激活态 */
--primary-700: #334E68;  /* hover */
--primary-800: #243B53;  /* 顶部导航、深色 header */
--primary-900: #102A43;  /* 文字最深 */
```

**中性灰（Neutral）**：用于背景、边框、文字

```css
--neutral-50:  #FAFBFC;  /* 页面背景 */
--neutral-100: #F4F6F8;  /* 卡片悬停、表格斑马 */
--neutral-200: #E4E7EB;  /* 边框 default */
--neutral-300: #CBD2D9;  /* 边框 hover、分割线 */
--neutral-400: #9AA5B1;  /* 占位符、disabled */
--neutral-500: #7B8794;  /* 次要文字 */
--neutral-600: #616E7C;  /* 正文次级 */
--neutral-700: #52606D;  /* 正文 */
--neutral-800: #3E4C59;  /* 标题 */
--neutral-900: #1F2933;  /* 重要标题 */
```

**语义色（Semantic）**：状态、告警

```css
/* Success - 通过、完成、正常 */
--success-50:  #E3F9E5;
--success-500: #31B237;
--success-700: #207227;

/* Warning - At Risk、效期临近 */
--warning-50:  #FFFBEA;
--warning-500: #F0B429;
--warning-700: #B44D12;

/* Danger - Overdue、Critical 异常、Dispose */
--danger-50:   #FFEEEE;
--danger-500:  #E12D39;
--danger-700:  #AB091E;

/* Info - 一般信息、链接、Reviewing */
--info-50:     #E0F1FF;
--info-500:    #3B89E0;
--info-700:    #1A4D89;

/* Pending - 草稿、待审批等中性状态 */
--pending-50:  #F4F6F8;
--pending-500: #7B8794;
--pending-700: #3E4C59;
```

**色彩使用规则**：
- 大面积色块 → 仅用 neutral 系
- 强调点（按钮、链接） → primary-600
- 状态标识 → semantic 色，且必须配文字或图标（双重编码）
- 表格行背景 → 默认白，hover 用 neutral-100，告警行用 danger-50 / warning-50（淡）

#### 15.2.2 字体

**字体栈**：

```css
/* 英文 / 数字 */
--font-sans: 'Inter', 'SF Pro Text', -apple-system, BlinkMacSystemFont, sans-serif;

/* 中文 */
--font-zh: 'PingFang SC', 'Noto Sans SC', 'Microsoft YaHei', sans-serif;

/* 等宽（编码、ID、数值） */
--font-mono: 'JetBrains Mono', 'SF Mono', 'Menlo', 'Consolas', monospace;
```

**字号阶梯**（rem，基础 16px）：

| Token | Size | Line Height | 用途 |
|---|---|---|---|
| `text-xs` | 12px / 0.75rem | 16px | 元信息、辅助说明、表格头 |
| `text-sm` | 13px / 0.8125rem | 20px | 表格内容、表单 label、次要文字 |
| `text-base` | 14px / 0.875rem | 22px | **正文默认** |
| `text-md` | 16px / 1rem | 24px | 卡片标题、副标题 |
| `text-lg` | 18px / 1.125rem | 28px | 页面副标题 |
| `text-xl` | 20px / 1.25rem | 30px | 页面主标题 |
| `text-2xl` | 24px / 1.5rem | 32px | 仪表盘大数字 |
| `text-3xl` | 30px / 1.875rem | 38px | KPI 卡片关键数字 |

> **注意**：基础正文 14px 是 Linear / Notion 的做法，比 Material 的 16px 紧凑。如果觉得太小可以微调到 14.5px。

**字重**：
- 400 Regular：正文
- 500 Medium：链接、表格头、强调
- 600 Semibold：标题、按钮
- 不使用 700 Bold（过于厚重）

#### 15.2.3 间距与圆角

**间距阶梯**（基于 4px 网格）：

| Token | Value | 典型用途 |
|---|---|---|
| `space-1` | 4px | 内联元素间距 |
| `space-2` | 8px | 紧凑卡片内距 |
| `space-3` | 12px | 表单字段间距 |
| `space-4` | 16px | 卡片内距、按钮内距 |
| `space-5` | 20px | 段落间距 |
| `space-6` | 24px | 卡片之间、section 间距 |
| `space-8` | 32px | 页面 section 大间距 |
| `space-10` | 40px | 顶层 padding |
| `space-12` | 48px | 极大间距 |

**圆角**：

| Token | Value | 用途 |
|---|---|---|
| `radius-none` | 0 | 表格 |
| `radius-sm` | 4px | 按钮、输入框、小卡片 |
| `radius-md` | 6px | **默认**：卡片、面板 |
| `radius-lg` | 8px | 模态框、抽屉 |
| `radius-full` | 9999px | 头像、Pill 状态标签 |

#### 15.2.4 阴影

**轻量阴影**（克制使用）：

```css
--shadow-xs: 0 1px 2px rgba(15, 23, 42, 0.04);
--shadow-sm: 0 1px 3px rgba(15, 23, 42, 0.06), 0 1px 2px rgba(15, 23, 42, 0.04);
--shadow-md: 0 4px 6px rgba(15, 23, 42, 0.05), 0 2px 4px rgba(15, 23, 42, 0.04);
--shadow-lg: 0 10px 15px rgba(15, 23, 42, 0.06), 0 4px 6px rgba(15, 23, 42, 0.05);
--shadow-xl: 0 20px 25px rgba(15, 23, 42, 0.08); /* 仅模态框 */
```

使用规则：
- 卡片默认无阴影，靠边框区分
- hover 时使用 `shadow-sm`
- 下拉菜单、tooltip 使用 `shadow-md`
- 模态框使用 `shadow-xl`

#### 15.2.5 边框

```css
--border-default: 1px solid var(--neutral-200);
--border-strong:  1px solid var(--neutral-300);
--border-focus:   2px solid var(--primary-500);
--border-danger:  1px solid var(--danger-500);
```

#### 15.2.6 图标

**图标库**：使用 **Lucide Icons**（开源、风格统一、覆盖广）

**常用图标映射**：

| 概念 | 图标 |
|---|---|
| Project | `folder` 或 `briefcase` |
| Study Plan | `clipboard-list` |
| Batch Run | `flask-conical` |
| Sample | `test-tube` |
| Material | `package` |
| Cell Bank | `dna` 或 `microscope` |
| Test Request | `send` |
| Deviation | `alert-triangle` |
| Dashboard | `layout-dashboard` |
| 报表 | `bar-chart-3` |
| 用户 | `user` / `users` |
| 设置 | `settings` |
| 审计 | `shield-check` |
| 签名 | `pen-line` |
| 搜索 | `search` |
| 筛选 | `filter` |
| 排序 | `arrow-up-down` |
| 更多 | `more-horizontal` |
| 状态正常 | `check-circle-2` |
| 状态警告 | `alert-circle` |
| 状态错误 | `x-circle` |
| 状态进行中 | `loader-2`（旋转） |

**图标尺寸**：14px（内联）、16px（默认）、20px（导航）、24px（page header）

### 15.3 全局布局与导航

#### 15.3.1 整体布局

```
┌───────────────────────────────────────────────────────────────────────┐
│ Top Bar [56px]                                                        │
│ [Logo + 系统名] [搜索/Cmd+K] [新建▼] [通知🔔] [头像 用户名 ▼]          │
├──────────┬────────────────────────────────────────────────────────────┤
│          │                                                            │
│ Sidebar  │  Page Content Area                                         │
│ [240px]  │  ┌─────────────────────────────────────────────────────┐   │
│          │  │ 面包屑：项目 / SP-0042 / BR-1023 / Stage S2          │   │
│ □ 工作台 │  ├─────────────────────────────────────────────────────┤   │
│ □ 项目   │  │ Page Header                                          │   │
│ □ 实验   │  │ 标题  [次级信息]                  [次操作] [主操作]   │   │
│ □ 主数据 │  ├─────────────────────────────────────────────────────┤   │
│ □ 报表   │  │                                                      │   │
│ □ 管理   │  │ Content                                              │   │
│          │  │                                                      │   │
│          │  │                                                      │   │
│          │  └─────────────────────────────────────────────────────┘   │
└──────────┴────────────────────────────────────────────────────────────┘
```

**尺寸**：
- Top bar 高度：56px，背景 `--primary-800`，文字白色
- Sidebar 宽度：240px（展开） / 60px（收起，仅图标），背景白色 + 右边框
- 主内容区：max-width 1440px（宽屏） / 流式（普通屏）
- 内容区左右 padding：32px

#### 15.3.2 顶部导航条（Top Bar）

```
┌────────────────────────────────────────────────────────────────────────┐
│ [🧪 XDC BioFoundry]  [🔍 搜索任何东西 ⌘K]  [+ 新建▼]  [🔔3]  [👤 张明 ▼]│
└────────────────────────────────────────────────────────────────────────┘
   左 16px            居中（最大 480px）       右侧依次排列，间距 12px
```

**组件**：
- **Logo + 系统名**：可点击回首页
- **全局搜索框**：固定宽度 480px，placeholder "搜索项目、批次、样品... (⌘K)"，点击或按 Cmd+K 打开命令面板
- **新建按钮**：下拉菜单（新建 Project / Study Plan / Batch Run / Sample / 送样异常单）
- **通知图标**：未读数小红点（仅显示 0–9，超出显示 9+）
- **用户菜单**：头像 + 姓名，下拉（个人资料 / 切换角色 / 培训记录 / 登出）

#### 15.3.3 侧边导航（Sidebar）

```
┌──────────────────┐
│ 工作台            │
│  ├ 我的工作台      │ ← active 时左侧 3px primary-600 竖线 + 浅色背景
│  ├ FL 监控台      │
│  └ 管理工作台      │
│                  │
│ 项目与流程        │
│  ├ 项目列表       │
│  ├ Study Plan    │
│  └ Batch Run     │
│                  │
│ 实验数据          │
│  ├ 样品管理       │
│  ├ 送样监控       │
│  ├ 检测结果       │
│  └ 偏差与 CAPA   │
│                  │
│ 主数据 (BCOE)    │
│  ├ 细胞库         │
│  └ 物料管理       │
│                  │
│ 报表             │
│ 管理             │
│                  │
│ ───────────────  │
│ ⌘ 帮助文档        │ ← 底部固定
│ ⌘ 反馈与支持      │
└──────────────────┘
```

**样式**：
- 一级分组：`text-xs` + `font-medium` + `text-neutral-500` + uppercase
- 二级菜单项：`text-sm` + `text-neutral-700`，左 padding 16px + 图标 16px
- Active 项：背景 `primary-50` + 文字 `primary-700` + 左 3px `primary-600` 竖线
- Hover：背景 `neutral-100`
- 折叠：点击顶部图标，展开 → 收起为 60px 仅图标
- 角色定制：根据当前用户角色隐藏不可见的菜单项

#### 15.3.4 面包屑（Breadcrumb）

```
首页 / 项目 / P-26-001 / SP-0042 / BR-1023 / Stage S2: Reduction
```

**规则**：
- 每一级可点击跳转
- 当前项不可点击，颜色 `neutral-900`
- 分隔符使用 `/`，颜色 `neutral-300`
- 超过 5 级时中间折叠为 `...`
- 字号 `text-sm`

#### 15.3.5 Page Header

```
┌───────────────────────────────────────────────────────────────────────┐
│  BR-1023 · Reduction Run            [● In Progress]                   │
│  P-26-001 · ADC for Client ABC · SP-0042                              │
│                                  [复制] [导出] [偏差] [完成本 Stage]   │
└───────────────────────────────────────────────────────────────────────┘
```

**组成**：
- **主标题**：实体编码 · 实体名称（`text-xl` + `font-semibold`）
- **状态标签**：右上角紧贴标题
- **次级信息**：上下文路径（`text-sm` + `text-neutral-500`），如所属项目、所属 Study Plan
- **操作区**：右侧，次操作（ghost 按钮）+ 主操作（primary 按钮）
- 整体高度约 80px，底部有 1px 分割线

### 15.4 通用组件库

#### 15.4.1 按钮（Button）

**变体**：

| 变体 | 用途 | 样式 |
|---|---|---|
| `primary` | 主操作 | bg `primary-600`，hover `primary-700`，text 白 |
| `default` | 次操作 | bg 白，border `neutral-300`，text `neutral-800`，hover bg `neutral-50` |
| `ghost` | 三级操作 | 无背景无边框，text `neutral-700`，hover bg `neutral-100` |
| `danger` | 危险操作 | bg `danger-500`，hover `danger-700`，text 白 |
| `link` | 跳转 | 无背景，text `primary-600`，hover 下划线 |

**尺寸**：

| 尺寸 | Height | Padding | 字号 |
|---|---|---|---|
| `xs` | 24px | 0 8px | text-xs |
| `sm` | 28px | 0 12px | text-sm |
| `md`（默认） | 32px | 0 16px | text-sm |
| `lg` | 40px | 0 20px | text-base |

**状态**：
- Disabled：opacity 0.5，cursor not-allowed
- Loading：内嵌 spinner，文字保留

#### 15.4.2 输入框（Input / Textarea / Select）

**通用样式**：
- Height：32px（默认）
- Padding：0 12px
- Border：`1px solid neutral-300`
- Border radius：4px
- Focus：border `primary-500` + 2px ring `primary-100`
- Disabled：bg `neutral-50` + text `neutral-400`
- Error：border `danger-500` + 底部红色 hint 文字

**字段组（Form Field）**：

```
┌─────────────────────────────┐
│ Label *           [Help ⓘ]  │  ← label 13px medium，必填 *红色
│ ┌─────────────────────────┐  │
│ │ Input                   │  │
│ └─────────────────────────┘  │
│ Hint or error text          │  ← hint 12px neutral-500，error 12px danger-700
└─────────────────────────────┘
```

#### 15.4.3 状态标签（Status Badge / Pill）

**形状**：Pill 形（`radius-full`），高度 20px / 22px，padding 0 8px

**双重编码**：颜色 + 圆点 + 文字

```
● Draft          (灰圆点 + neutral-700 文字 + neutral-100 背景)
● In Progress    (蓝圆点 + info-700 文字 + info-50 背景)
● Completed      (绿圆点 + success-700 文字 + success-50 背景)
● Pending        (黄圆点 + warning-700 文字 + warning-50 背景)
● Overdue        (红圆点 + danger-700 文字 + danger-50 背景)
● Cancelled      (灰删除线 + neutral-500 文字)
```

**SLA 专用变体**：

```
✓ On Track       (绿色)
⚠ At Risk        (黄色)
⚡ Overdue        (红色，闪烁动效可选)
```

#### 15.4.4 表格（Data Table）

**整体规格**：
- 行高：40px（紧凑） / 48px（默认）
- 表头：bg `neutral-50`，text `text-xs` + `font-medium` + uppercase + `text-neutral-600`
- 单元格 padding：0 12px
- 行分隔：1px `neutral-200`
- Hover：bg `neutral-50`
- 选中行：bg `primary-50`

**功能**：
- 列宽可拖拽调整
- 列可隐藏/显示（右上角"列设置"）
- 排序：点击列头，显示 ↑↓ 图标
- 多选：左侧 checkbox 列
- 行内操作：右侧固定列，hover 时显示（`...` 菜单）
- 批量操作：选中后顶部出现"已选 N 项"操作栏
- 空状态：插画 + 提示文字 + 主操作按钮
- 加载状态：骨架屏（不用 spinner 覆盖）
- 分页：右下角，默认 50 条/页

**示例（FL 监控台核心表格）**：

```
┌────┬──────────┬─────────┬───────┬─────┬───────┬──────┬─────────┬─────────┬────────┐
│ ☐  │ 送样单号  │ Project │ Stage │通道 │ 检测项 │ 优先 │ SLA状态  │ 责任人  │  操作  │
├────┼──────────┼─────────┼───────┼─────┼───────┼──────┼─────────┼─────────┼────────┤
│ ☐  │ TR-9821  │P-26-001 │  S2   │ XAS │ DAR   │ High │⚡Overdue│ 张明    │  ⋯    │
│ ☐  │ TR-9820  │P-26-001 │  S4   │ XAS │ SEC   │ Med  │⚠ AtRisk│ 李华    │  ⋯    │
│ ☐  │ TR-9819  │P-26-002 │  S7   │ BCOE│Potency│ Med  │✓OnTrack│ 王芳    │  ⋯    │
└────┴──────────┴─────────┴───────┴─────┴───────┴──────┴─────────┴─────────┴────────┘
   40px   140px      120px    60px   60px   100px   60px    100px      80px    60px
```

#### 15.4.5 卡片（Card）

**基础卡片**：
- 背景：白
- 边框：`1px solid neutral-200`
- 圆角：6px
- 内距：16px
- 标题区：底部 1px 分割线
- 标题：`text-md` + `font-semibold`

**KPI 卡片**：

```
┌─────────────────────────┐
│ 本周送样数         📈   │  ← title text-sm neutral-600
│                         │
│  328                    │  ← 数字 text-3xl + font-semibold
│  ↑ 12% vs 上周          │  ← 趋势 text-xs success-700
└─────────────────────────┘
```

#### 15.4.6 表单布局（Form）

**布局模式**：
- **垂直表单**（默认）：label 在上，input 在下，适合移动端和窄屏
- **水平表单**：label 左对齐宽度 120px，input 右侧。适合密集表单
- **网格表单**：2/3 列布局，适合"宽度短的字段"（如温度、pH、时间）

**字段分组**：

```
┌─ 反应条件 ───────────────────────────────────┐
│ pH        [7.5]      温度    [22 °C]         │
│ 反应时间   [30 min]   搅拌速度 [200 rpm]      │
└──────────────────────────────────────────────┘
```

#### 15.4.7 模态框（Modal）

**尺寸**：

| 尺寸 | 宽度 | 用途 |
|---|---|---|
| `sm` | 400px | 确认对话框 |
| `md` | 600px | 简单表单 |
| `lg` | 800px | 复杂表单 |
| `xl` | 1080px | 详情展示 |
| `full` | 100vw | 全屏（少用） |

**结构**：

```
┌──────────────────────────────────┐
│ 标题                          [×] │  ← header 56px
├──────────────────────────────────┤
│                                  │
│ Content                          │
│                                  │
├──────────────────────────────────┤
│              [取消]  [确认]       │  ← footer 64px，右对齐
└──────────────────────────────────┘
```

**GxP 受控操作的二次确认 Modal**：

```
┌──────────────────────────────────────┐
│ ⚠ 受控操作 - 请确认                  │
├──────────────────────────────────────┤
│ 你即将完成 Stage S2 的电子签名        │
│                                      │
│ 操作含义 *                            │
│ ○ 作者 (Author)                       │
│ ● 复核人 (Reviewer)                   │
│ ○ 批准人 (Approver)                   │
│                                      │
│ 签名原因 *                            │
│ ┌──────────────────────────────┐    │
│ │ 已完成本 Stage 所有参数复核   │    │
│ └──────────────────────────────┘    │
│                                      │
│ 密码 *                                │
│ ┌──────────────────────────────┐    │
│ │ ●●●●●●●●                     │    │
│ └──────────────────────────────┘    │
│                                      │
│ ☑ 我确认此操作符合 Part 11 要求      │
├──────────────────────────────────────┤
│                  [取消]  [签名提交]   │
└──────────────────────────────────────┘
```

#### 15.4.8 抽屉（Drawer）

**用途**：从右侧滑入，用于"不离开当前页"的详情查看或快速编辑

**尺寸**：宽度 480px（默认） / 640px（宽）

**典型场景**：
- 表格行点击"查看详情" → 抽屉
- 创建简单实体 → 抽屉
- 查看审计 trail → 抽屉

#### 15.4.9 命令面板（Command Palette，Cmd+K）

**触发**：Cmd+K (Mac) / Ctrl+K (Windows)

```
┌─────────────────────────────────────────┐
│ 🔍 输入命令、ID 或关键词...              │
├─────────────────────────────────────────┤
│ 最近访问                                  │
│   📁 P-26-001 - ADC for Client ABC      │
│   🧪 BR-1023 - Reduction Run            │
│   🧫 S-58291 - mAb intermediate         │
│                                          │
│ 快速操作                                  │
│   + 新建 Project                         │
│   + 新建 Study Plan                      │
│   + 新建 Sample                          │
│                                          │
│ 导航                                      │
│   → FL 监控台                            │
│   → 物料管理                              │
│                                          │
│ 帮助                                      │
│   ⌘? 快捷键列表                          │
└─────────────────────────────────────────┘
```

#### 15.4.10 通知（Toast / Notification）

**位置**：右上角，距离顶部 76px（top bar 56px + 20px）

**类型**：success / info / warning / error

**结构**：

```
┌────────────────────────────────┐
│ ✓ 签名提交成功              [×]│  ← 4 秒自动消失
│   Stage S2 已完成复核签名      │
└────────────────────────────────┘
```

#### 15.4.11 空状态（Empty State）

```
┌─────────────────────────────────┐
│                                  │
│         📋 (示意图标)             │
│                                  │
│    暂无 Study Plan               │
│    在当前项目下创建第一个 Plan    │
│                                  │
│       [+ 新建 Study Plan]        │
│                                  │
└─────────────────────────────────┘
```

#### 15.4.12 审计 Trail Drawer

**用途**：任何 GxP 实体的右上角"查看审计"按钮，打开抽屉

```
┌────────────────────────────────────┐
│ 审计轨迹 - BR-1023            [×] │
├────────────────────────────────────┤
│ 全部 (28)  字段修改 (15)  签名 (3) │
│                                    │
│ ┌──────────────────────────────┐  │
│ │ 2026-05-18 14:23:01 +08:00   │  │
│ │ 👤 李华 (Reviewer)            │  │
│ │ 修改字段: reaction_ph        │  │
│ │ 7.0 → 7.5                    │  │
│ │ 原因: 复核时发现录入错误      │  │
│ └──────────────────────────────┘  │
│                                    │
│ ┌──────────────────────────────┐  │
│ │ 2026-05-18 13:55:22 +08:00   │  │
│ │ 👤 张明                      │  │
│ │ 创建 Stage S2                 │  │
│ └──────────────────────────────┘  │
└────────────────────────────────────┘
```

### 15.5 核心页面详细设计

> 本节针对 I 期 MVP 的核心页面给出详细 wireframe。其他页面遵循同样的设计语言。

#### 15.5.1 P1 - FL 监控工作台（I 期 MVP 核心）

**路由**：`/dashboard/fl`
**用户**：PD FL（Function Lead）
**目标**：30 秒内识别所有送样状态、SLA 风险、未关闭异常

**完整 Wireframe**：

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│ Top Bar                                                                         │
├──────────┬──────────────────────────────────────────────────────────────────────┤
│ Sidebar  │ 首页 / FL 监控台                                                      │
│          ├──────────────────────────────────────────────────────────────────────┤
│          │ FL 监控工作台              ⟳ 4 分钟前更新 [自动刷新 ✓]               │
│          │ 实时监控 AS 送样状态、SLA 与异常闭环         [导出周报] [新建异常]    │
│          ├──────────────────────────────────────────────────────────────────────┤
│          │                                                                       │
│          │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐    │
│          │  │ 本周送样数   │ │ 在测数       │ │ SLA 超时    │ │ 异常未关闭  │    │
│          │  │     328     │ │     142     │ │      12      │ │      5      │    │
│          │  │ ↑12% 上周   │ │             │ │ ↑3 较昨日    │ │ ⚠ 需关注    │    │
│          │  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘    │
│          │                                                                       │
│          │  ┌──────────────────────────┐ ┌─────────────────────────────────┐  │
│          │  │ 按 Stage 分布             │ │ 按检测通道分布                   │  │
│          │  │ S1 ████░░░░ 12            │ │ XAS ████████████░ 78%            │  │
│          │  │ S2 ████████░ 28           │ │ BCOE ████░░░░░░░ 22%             │  │
│          │  │ S3 ███████░░ 24           │ │                                  │  │
│          │  │ S4 ████████████ 41        │ │ 平均 TAT                          │  │
│          │  │ S5 ██░░░░░░░░ 6           │ │  XAS: 3.2 天                     │  │
│          │  │ S6 ████░░░░░░ 12          │ │  BCOE: 5.8 天                    │  │
│          │  │ S7 ██████░░░░ 18          │ │                                  │  │
│          │  │ S8 █░░░░░░░░░ 3           │ │                                  │  │
│          │  └──────────────────────────┘ └─────────────────────────────────┘  │
│          │                                                                       │
│          │  送样监控明细                                                          │
│          │  ┌─────────────────────────────────────────────────────────────────┐ │
│          │  │ [筛选: 全部 ▼] [Stage: 全部 ▼] [通道: 全部 ▼] [SLA: ⚡ 超时 ▼] │ │
│          │  │ [🔍 搜索送样单/项目]                          [⋮ 列设置] [导出] │ │
│          │  ├─────────────────────────────────────────────────────────────────┤ │
│          │  │ ☐│送样单号 │Project │Stage│通道 │检测项│优先│SLA      │责任人│⋯│ │
│          │  │ ☐│TR-9821  │P-26-001│ S2  │XAS  │DAR  │High│⚡Overdue│张明 │⋯│ │
│          │  │ ☐│TR-9820  │P-26-001│ S4  │XAS  │SEC  │Med │⚠AtRisk │李华 │⋯│ │
│          │  │ ☐│TR-9819  │P-26-002│ S7  │BCOE │Pot  │Med │✓OnTrack│王芳 │⋯│ │
│          │  │ ☐│TR-9818  │P-26-003│ S4  │XAS  │DAR  │High│⚡Overdue│张明 │⋯│ │
│          │  │ ... (更多行)                                                     │ │
│          │  └─────────────────────────────────────────────────────────────────┘ │
│          │  显示 1-50 / 共 142 条                              [< 1 2 3 ... >]  │
│          │                                                                       │
└──────────┴──────────────────────────────────────────────────────────────────────┘
```

**组件清单**：

1. **页面标题区**：
   - 标题："FL 监控工作台"，`text-xl` + `font-semibold`
   - 副标题：业务说明 + 最后更新时间
   - 操作按钮：导出周报（default）+ 新建异常（primary）
   - 右侧状态：自动刷新开关 + 最后刷新时间

2. **KPI 卡片区**（4 个，等宽，gap 16px）：
   - 每张卡片：标题 + 主数字 + 趋势文字
   - 标题：`text-sm` + `neutral-600`
   - 数字：`text-3xl` + `font-semibold` + `neutral-900`
   - 趋势：`text-xs`，向上箭头 + 百分比，颜色根据正负（异常类向上为坏，向下为好）
   - hover 显示工具提示，说明数据口径

3. **分布图表区**（2 列）：
   - 左：水平条形图，按 Stage（S1–S8）分布，使用 D3 或 Recharts
   - 右：饼图 + TAT（Turn Around Time）平均值
   - 图表配色：使用 primary 色阶（不同深浅），避免彩虹色

4. **筛选条**：
   - 多个下拉筛选：业务线、Stage、检测通道、SLA 状态、优先级、责任人、日期范围
   - 搜索框：跨字段全文搜索（送样单号、项目、样品 ID）
   - 操作区：列设置（齿轮图标）、导出（下拉支持 Excel/CSV/PDF）
   - 已选筛选条件以 Pill 形式显示，可单独移除

5. **主表格**：
   - 字段见 §6.6.1
   - 整行可点击进入送样详情
   - SLA 状态列使用专用样式（Pill + 图标）
   - "操作"列：hover 时显示 `⋯` 菜单（查看详情 / 标记跟进 / 创建异常单 / 联系负责人）
   - 表格高度自适应，超出部分滚动
   - 行密度：紧凑（40px）

6. **分页**：
   - 右下角，默认 50/页，可选 20/50/100
   - 显示总数 + 当前范围

**交互细节**：

- **自动刷新**：每 4 小时拉一次外部数据，刷新时表格右上角显示一个轻量 loading 条
- **筛选状态保留**：刷新页面后筛选条件通过 URL query 保留
- **行点击**：点击非操作列 → 打开右侧抽屉显示送样详情（不离开当前页）
- **批量操作**：选中多行后顶部出现操作条："已选 N 项 [批量标记跟进] [批量分配] [取消]"
- **SLA Overdue 强提示**：表格中超时行使用淡红色背景 `danger-50`
- **快捷键**：`/` 聚焦搜索，`F` 打开筛选，`R` 刷新

**响应式**：
- 桌面优先，最小支持 1280px 宽度
- KPI 卡片在 < 1280px 时变为 2x2 排列

#### 15.5.2 P2 - 项目列表与详情

**路由**：`/projects` 和 `/projects/:id`

##### P2-A 项目列表

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ 首页 / 项目                                                                   │
├──────────────────────────────────────────────────────────────────────────────┤
│ 项目                                          [全部 12] [我的 5] [+ 新建项目]│
├──────────────────────────────────────────────────────────────────────────────┤
│ [筛选: Phase ▼ 状态 ▼ 客户 ▼ 负责人 ▼] [🔍 搜索]      [⊞ 卡片] [≡ 列表]    │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│ 列表视图：                                                                    │
│ ┌──────────────────────────────────────────────────────────────────────────┐│
│ │项目编码 │项目名称              │客户   │Phase│状态     │Lead │起始日期    ││
│ ├─────────┼──────────────────────┼───────┼─────┼─────────┼─────┼────────────┤│
│ │P-26-001 │ADC for Client ABC    │ABC    │P3   │●Active  │张明 │2026-01-15  ││
│ │P-26-002 │XYZ Bioconjugate      │XYZ    │P4   │●Active  │李华 │2025-12-01  ││
│ │P-26-003 │NextGen ADC Platform  │DEF    │P2   │●On Hold │王芳 │2026-02-10  ││
│ └──────────────────────────────────────────────────────────────────────────┘│
│                                                                              │
│ 卡片视图（备选）：                                                            │
│ ┌──────────────────────┐ ┌──────────────────────┐ ┌──────────────────────┐ │
│ │ P-26-001 ●Active     │ │ P-26-002 ●Active     │ │ P-26-003 ●On Hold    │ │
│ │ ADC for Client ABC   │ │ XYZ Bioconjugate     │ │ NextGen ADC          │ │
│ │ Phase: P3            │ │ Phase: P4            │ │ Phase: P2            │ │
│ │ Lead: 张明           │ │ Lead: 李华           │ │ Lead: 王芳           │ │
│ │ ████████░░ 60%       │ │ ██████████ 92%       │ │ ████░░░░░░ 35%       │ │
│ │ 起始: 2026-01-15     │ │ 起始: 2025-12-01     │ │ 起始: 2026-02-10     │ │
│ └──────────────────────┘ └──────────────────────┘ └──────────────────────┘ │
└──────────────────────────────────────────────────────────────────────────────┘
```

##### P2-B 项目详情

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ 首页 / 项目 / P-26-001                                                        │
├──────────────────────────────────────────────────────────────────────────────┤
│ P-26-001 · ADC for Client ABC              [●Active]                         │
│ 客户: ABC · Lead: 张明 · 起始: 2026-01-15                                    │
│                                  [审计] [导出] [编辑] [创建 Study Plan]      │
├──────────────────────────────────────────────────────────────────────────────┤
│ [概览] [Study Plans] [Batch Runs] [样品] [送样监控] [物料] [文档] [审计]    │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│ ┌─ Phase 时间轴 ────────────────────────────────────────────────────────┐  │
│ │ P1 ●─── P2 ●─── P3 ●═══════════ P4 ○─── P5 ○─── P6 ○─── P7 ○─── P8 ○ │  │
│ │ ✓Done   ✓Done   ●In Progress      Planned                              │  │
│ └──────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│ ┌─ 基本信息 ──────────────┐ ┌─ 关键指标 ────────────────────────────┐    │
│ │ 项目编码  P-26-001       │ │ Study Plans         5 (2 完成)         │    │
│ │ 客户      ABC Inc.       │ │ Batch Runs          18 (12 完成)       │    │
│ │ 业务线    Conjugation    │ │ 样品               86                  │    │
│ │ Phase     P3 (M2)        │ │ 送样总数            42                 │    │
│ │ Lead      张明           │ │ 当前 SLA 超时       3 ⚡               │    │
│ │ Manager   王芳           │ │ 未关闭异常          1                  │    │
│ │ 起始日期  2026-01-15     │ │ 完成度              60%                │    │
│ │ 预期完成  2026-06-15     │ └────────────────────────────────────────┘    │
│ └──────────────────────────┘                                                │
│                                                                              │
│ ┌─ 近期 Study Plans ────────────────────────────────────────────────────┐  │
│ │ SP-0042 · Conjugation DOE Plan       ●In Progress  8/15 Run  张明    │  │
│ │ SP-0041 · Material Generation Plan   ✓Completed    3/3 Run   李华    │  │
│ │ SP-0040 · Feasibility Study Plan     ✓Completed    2/2 Run   王芳    │  │
│ └──────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│ ┌─ 风险与提醒 ──────────────────────────────────────────────────────────┐  │
│ │ ⚡ TR-9821 (DAR @ S2) 超时 6 小时               2026-05-18 09:00      │  │
│ │ ⚠ M-3012 (TCEP Lot LX-2026-03) 将于 5 天后到期                       │  │
│ │ ⚠ BR-1018 偏差 DEV-0028 等待复核                                     │  │
│ └──────────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────────────┘
```

**关键组件**：
- **Phase 时间轴**：水平节点图，每个节点点击可查看该 phase 的 Study Plans。当前 phase 高亮，已完成绿色，未来灰色。
- **Tab 导航**：8 个 tab，使用 underline 样式（不是 box 样式），当前 tab 颜色 `primary-700`
- **关键指标卡片**：紧凑展示，数字使用 mono 字体
- **风险与提醒**：垂直列表，每项 left border 颜色对应严重度

##### P2-C Study Plan 创建/编辑

**路由**：`/study-plans/new`

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ 首页 / 项目 / P-26-001 / 新建 Study Plan                                     │
├──────────────────────────────────────────────────────────────────────────────┤
│ 新建 Study Plan                                                              │
│ 创建一份工艺研究计划，关联到 Project P-26-001                                 │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│ ┌─ 基本信息 ──────────────────────────────────────────────────────────┐    │
│ │                                                                       │    │
│ │ Plan 类型 *                                                           │    │
│ │ ┌─────────────────────────────────────┐                              │    │
│ │ │ Conjugation DOE Plan          ▼     │                              │    │
│ │ └─────────────────────────────────────┘                              │    │
│ │                                                                       │    │
│ │ 关联 Phase                  关联项目                                   │    │
│ │ ┌──────────┐                ┌───────────────────────────┐            │    │
│ │ │ P3    ▼  │                │ P-26-001 (已锁定)         │            │    │
│ │ └──────────┘                └───────────────────────────┘            │    │
│ │                                                                       │    │
│ │ 标题 *                                                                │    │
│ │ ┌─────────────────────────────────────────────────────────────────┐  │    │
│ │ │ DAR optimization with DoE for Project ABC                       │  │    │
│ │ └─────────────────────────────────────────────────────────────────┘  │    │
│ │                                                                       │    │
│ │ 研究目标 *                                                            │    │
│ │ ┌─────────────────────────────────────────────────────────────────┐  │    │
│ │ │ Optimize conjugation parameters to achieve target DAR of 4.0±0.3│  │    │
│ │ │ ...                                                              │  │    │
│ │ └─────────────────────────────────────────────────────────────────┘  │    │
│ │                                                                       │    │
│ └───────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│ ┌─ 设计方法 ──────────────────────────────────────────────────────────┐    │
│ │                                                                       │    │
│ │ 设计方法 *                                                            │    │
│ │ ○ OFAT      ● DoE      ○ RSM      ○ DSD      ○ N/A                  │    │
│ │                                                                       │    │
│ │ 计划 Batch Run 数 *         设计矩阵附件（JMP 等）                    │    │
│ │ ┌──────────┐                ┌────────────────────────────┐          │    │
│ │ │ 15       │                │ 📎 上传文件                 │          │    │
│ │ └──────────┘                └────────────────────────────┘          │    │
│ │                                                                       │    │
│ └───────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│ ┌─ 时间与负责人 ──────────────────────────────────────────────────────┐    │
│ │ 起始日期 *           目标完成日期 *         Lead *                    │    │
│ │ ┌──────────┐         ┌──────────┐         ┌────────────────────┐    │    │
│ │ │2026-05-20│         │2026-06-30│         │ 张明              ▼│    │    │
│ │ └──────────┘         └──────────┘         └────────────────────┘    │    │
│ └───────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│                                              [取消] [保存为草稿] [提交审批]  │
└──────────────────────────────────────────────────────────────────────────────┘
```

**字段组规则**：
- 每个字段组 (`fieldset` 风格) 标题 + 一个浅边框区域
- 字段按字段组内的逻辑分组
- 必填字段 label 后红色 `*`
- 自动保存：每 30 秒静默保存草稿，右下角显示"已保存"状态

#### 15.5.3 P3 - Batch Run 详情（8-Stage Walkthrough）

**路由**：`/batch-runs/:id`
**用户**：PD Scientist
**这是 I.5 期的核心，但 I 期 MVP 可以做"只读视图"作为骨架**

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ 首页 / 项目 / P-26-001 / SP-0042 / BR-1023                                   │
├──────────────────────────────────────────────────────────────────────────────┤
│ BR-1023 · Reduction Run #3              [●In Progress]                       │
│ Study Plan: SP-0042 · Scientist: 张明 · 开始于 2026-05-18 09:00              │
│                                [审计] [偏差] [打印] [完成本 Stage] [完成 Run]│
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│ Stage Pipeline:                                                              │
│ ┌────────────────────────────────────────────────────────────────────────┐  │
│ │ ✓ S1 ──── ✓ S2 ──── ● S3 ──── ○ S4 ──── ○ S5 ──── ○ S6 ── ○ S7 ── ○ S8│  │
│ │  thaw     Reduce    UF/DF    Conjug    Quench    ACF     HIC    DS     │  │
│ │  ✓Done    ✓Done    In Progress  Pending  Pending  ...                  │  │
│ └────────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│ ┌─ Stage S3: UF/DF Purification ─────────────────────────────────────┐    │
│ │ 操作人: 张明  开始: 2026-05-18 11:20  状态: ●In Progress              │    │
│ ├─────────────────────────────────────────────────────────────────────┤    │
│ │ [参数] [输入物料] [输出物] [IPC] [送样] [偏差] [附件] [签名]          │    │
│ ├─────────────────────────────────────────────────────────────────────┤    │
│ │                                                                       │    │
│ │ 工艺参数                                                              │    │
│ │ ┌─────────────────────────────────────────────────────────────────┐│    │
│ │ │ 字段                  值           单位      范围         状态  ││    │
│ │ │ ─────────────────────────────────────────────────────────────── ││    │
│ │ │ 起始 mAb 浓度    [10.5    ]       mg/mL    8-12         ✓     ││    │
│ │ │ 起始体积         [500     ]       mL       -            ✓     ││    │
│ │ │ DF buffer        [PBS pH 7.4 ▼ ]   -        -           ✓     ││    │
│ │ │ Diavolume        [7       ]       -        ≥5           ✓     ││    │
│ │ │ 操作温度         [22      ]       °C       18-25        ✓     ││    │
│ │ │ 终点 mAb 浓度    [____    ]       mg/mL    -            ⚠待填  ││    │
│ │ │ 终点体积         [____    ]       mL       -            ⚠待填  ││    │
│ │ │ TFF 系统         [AKTA-01 ▼ ]      -        -           ✓     ││    │
│ │ └─────────────────────────────────────────────────────────────────┘│    │
│ │                                                                       │    │
│ │ 输入物料                                          [+ 添加物料]       │    │
│ │ ┌─────────────────────────────────────────────────────────────────┐│    │
│ │ │ 物料编码    名称      批号        使用量    剩余    操作      ││    │
│ │ │ M-3012      TCEP      LX-2026-03  50 mg     950 mg  [×]      ││    │
│ │ │ M-3045      PBS       PB-2026-12  500 mL    2.5 L   [×]      ││    │
│ │ └─────────────────────────────────────────────────────────────────┘│    │
│ │                                                                       │    │
│ │ IPC 检测                                          [+ 创建样品]       │    │
│ │ ┌─────────────────────────────────────────────────────────────────┐│    │
│ │ │ 检测项     方法       值        状态                            ││    │
│ │ │ Free Drug  HPLC       [____ ]   ⚠待填                          ││    │
│ │ │ SEC        HPLC       [____ ]   ⚠待填                          ││    │
│ │ └─────────────────────────────────────────────────────────────────┘│    │
│ │                                                                       │    │
│ │ 送样请求                                          [+ 触发送样]       │    │
│ │ ┌─────────────────────────────────────────────────────────────────┐│    │
│ │ │ 送样单号   样品       检测项    通道   状态                     ││    │
│ │ │ TR-9821    S-58291    DAR       XAS    ⚡Overdue (跳转 →)      ││    │
│ │ └─────────────────────────────────────────────────────────────────┘│    │
│ │                                                                       │    │
│ │                              [保存草稿]  [完成 Stage S3 并签名]      │    │
│ └─────────────────────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────────────────────┘
```

**Stage Pipeline 组件细节**：
- 横向时间轴，每个 stage 是一个圆点 + 连接线 + 标签
- 状态映射：
  - ✓ Done（绿色实心圆）
  - ● In Progress（蓝色实心圆 + 脉冲动画）
  - ○ Pending（灰色空心圆）
  - ⚠ Deviated（黄色三角）
  - ✗ Skipped（灰色 X）
- 点击任意 stage：切换到该 stage 的详情面板
- 当前 stage：底部高亮条 + 加粗文字

**Stage 详情面板（标签页结构）**：
- 标签页：参数 / 输入物料 / 输出物 / IPC / 送样 / 偏差 / 附件 / 签名
- 每个标签的内容卡片化呈现
- inline 编辑：点击字段直接编辑，离开焦点自动保存
- 范围校验：填入超出预期范围的值时字段右侧显示 `⚠` 图标，悬停显示警告（不是阻塞，只提示）

**关键交互**：
- **完成 Stage**：必须填完所有必填字段才可点击"完成 Stage"，触发签名 modal
- **签名**：使用 §15.4.7 的二次确认 modal，输入密码 + 原因
- **触发送样**：点击"触发送样"打开 modal，选择检测项 + 优先级 + 关联样品，生成送样单（实际推送到外部送样系统）
- **添加物料**：搜索物料编码或扫码，自动带出剩余量，输入使用量后扣减
- **创建样品**：从 stage 输出物自动创建，预填 stage、batch、type、location

#### 15.5.4 P4 - 样品列表与详情

**路由**：`/samples` 和 `/samples/:id`

##### P4-A 样品列表

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ 首页 / 样品                                                                   │
├──────────────────────────────────────────────────────────────────────────────┤
│ 样品                              [全部 528] [我创建 42] [待送检 18] [+ 新建]│
├──────────────────────────────────────────────────────────────────────────────┤
│ [筛选: 类型 ▼ 状态 ▼ Stage ▼ 位置 ▼] [🔍]      [⊞ 网格] [≡ 表格] [树 谱系]│
├──────────────────────────────────────────────────────────────────────────────┤
│ ☐│样品编码  │类型               │项目     │Stage│位置             │状态     ││
│ ☐│S-58291   │mAb intermediate   │P-26-001 │S2   │SH/F1/B2/A05    │●Stored ││
│ ☐│S-58290   │reduced mAb        │P-26-001 │S2   │SH/F1/B2/A06    │●UnderT ││
│ ☐│S-58289   │ADC DS             │P-26-002 │S8   │WX/F3/B1/C12    │●Archvd ││
│ ...                                                                           │
└──────────────────────────────────────────────────────────────────────────────┘
```

##### P4-B 样品详情

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ 首页 / 样品 / S-58291                                                         │
├──────────────────────────────────────────────────────────────────────────────┤
│ S-58291 · mAb intermediate            [●In Storage]                          │
│ Project: P-26-001 · BR-1023 · Stage S2                                       │
│                                    [打印标签] [送样] [转移] [Dispose] [审计] │
├──────────────────────────────────────────────────────────────────────────────┤
│ [概览] [谱系] [状态历史] [送样] [检测结果]                                    │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│ ┌─ 基本信息 ──────────────┐ ┌─ 物理属性 ──────────────┐                   │
│ │ 编码     S-58291         │ │ 体积      10.5 mL        │                   │
│ │ 类型     mAb intermediate│ │ 浓度      5.2 mg/mL      │                   │
│ │ 项目     P-26-001        │ │ 总量      54.6 mg        │                   │
│ │ Batch    BR-1023         │ │                          │                   │
│ │ Stage    S2 Reduction    │ │ 保存条件  -80°C          │                   │
│ │ 创建于   2026-05-18 11:30│ │ 位置                     │                   │
│ │ 创建人   张明            │ │ Site:    上海            │                   │
│ │                          │ │ Freezer: F1              │                   │
│ │                          │ │ Box:     B2              │                   │
│ │                          │ │ 位置:    A05             │                   │
│ │                          │ │                          │                   │
│ │                          │ │ 标签 [打印]              │                   │
│ │                          │ │  ┌────────────────┐     │                   │
│ │                          │ │  │ ▓▓▓▓▓▓▓▓▓▓▓▓ │     │                   │
│ │                          │ │  │ S-58291        │     │                   │
│ │                          │ │  │ 张明 / 2026-05 │     │                   │
│ │                          │ │  └────────────────┘     │                   │
│ └──────────────────────────┘ └──────────────────────────┘                   │
│                                                                              │
│ ┌─ 谱系（简化） ──────────────────────────────────────────────────────┐    │
│ │                                                                       │    │
│ │   S-58280 (mAb stock)                                                 │    │
│ │      ↓ thaw (S1)                                                      │    │
│ │   S-58288 (thawed mAb)                                                │    │
│ │      ↓ reduction (S2)                                                 │    │
│ │   ● S-58291 (mAb intermediate, 当前)                                  │    │
│ │      ↓ UF/DF (S3)                                                     │    │
│ │   S-58294 (purified intermediate)                                     │    │
│ │                                                                       │    │
│ │                                              [查看完整谱系树 →]        │    │
│ └───────────────────────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────────────────────┘
```

**位置选择器（核心组件）**：

参考 Benchling 设计，多级树形：

```
Site → Building → Freezer → Shelf → Box → Position(grid)

┌─────────────────────────────────────┐
│ 选择存储位置                      [×]│
├─────────────────────────────────────┤
│ Site: ● 上海 ○ 无锡                  │
│                                      │
│ Freezer: [F1 (-80°C) ▼]              │
│                                      │
│ Box: [B2 (9x9 grid)  ▼]              │
│                                      │
│ Position: (点击网格选择)              │
│   1  2  3  4  5  6  7  8  9         │
│ A ░  ░  ░  ░  ░  ░  ░  ░  ░         │
│ B ░  ▓  ░  ░  ▓  ░  ░  ░  ░         │
│ C ░  ░  ●  ░  ░  ░  ░  ░  ░         │  ← ● 当前选中
│ ...                                  │
│ ▓ 已占用  ░ 空闲  ● 当前选中         │
│                                      │
│ 选中位置：A05                         │
│                                      │
│              [取消] [确认]            │
└─────────────────────────────────────┘
```

#### 15.5.5 P5 - 送样详情（抽屉视图）

从 FL 监控台点击行打开右侧抽屉：

```
┌────────────────────────────────────────┐
│ TR-9821                            [×] │
├────────────────────────────────────────┤
│ DAR 检测 · S-58291                     │
│ [⚡ Overdue 6 小时]                    │
│                                        │
│ ┌─ 关联上下文 ─────────────────────┐ │
│ │ Project    P-26-001 →            │ │
│ │ Study Plan SP-0042 →             │ │
│ │ Batch Run  BR-1023 →             │ │
│ │ Stage      S2 Reduction          │ │
│ │ 样品       S-58291 →             │ │
│ └──────────────────────────────────┘ │
│                                        │
│ ┌─ 送样信息 ───────────────────────┐ │
│ │ 送样人      张明                  │ │
│ │ 送样时间    2026-05-15 14:30      │ │
│ │ 优先级      🔴 High               │ │
│ │ 期望 TAT    3 天                  │ │
│ │ 检测通道    XAS                   │ │
│ │ 检测项      DAR                   │ │
│ └──────────────────────────────────┘ │
│                                        │
│ ┌─ 时间线 ──────────────────────────┐│
│ │ ✓ 2026-05-15 14:30 已提交         ││
│ │ ✓ 2026-05-15 15:45 实验室接样     ││
│ │ ● 2026-05-15 16:00 检测中         ││
│ │ ⚡ 2026-05-18 14:30 预期完成（超时）││
│ │ ○ ?              结果回传          ││
│ └──────────────────────────────────┘ │
│                                        │
│ ┌─ 异常 ───────────────────────────┐│
│ │ DEV-0034 · 实验室仪器故障         ││
│ │ 2026-05-16 创建 · 王芳             ││
│ │ 状态: 处理中                       ││
│ │ [查看详情 →]                      ││
│ └──────────────────────────────────┘ │
│                                        │
│ [创建异常单] [联系实验室] [跟进中]    │
└────────────────────────────────────────┘
```

#### 15.5.6 P6 - 物料管理（II 期，但需在原型中展示骨架）

**路由**：`/materials`

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ 首页 / 物料                                                                   │
├──────────────────────────────────────────────────────────────────────────────┤
│ 物料管理 (BCOE)                  [全部 642] [即将到期 8] [已开瓶 25] [+ 新建]│
├──────────────────────────────────────────────────────────────────────────────┤
│ [类型 ▼ 状态 ▼ 效期 ▼ 客户提供 ▼ 供应商 ▼] [🔍]              [扫码入库]   │
├──────────────────────────────────────────────────────────────────────────────┤
│ ┌─ 效期预警条 ────────────────────────────────────────────────────────────┐│
│ │ ⚠ 8 个物料 7 天内到期 · 3 个已超期 · 2 个开瓶后即将到期    [查看全部 →]││
│ └──────────────────────────────────────────────────────────────────────────┘│
│                                                                              │
│ ☐│物料编码 │名称       │批号        │剩余量    │原始效期  │开瓶后效期│状态  ││
│ ☐│M-3012   │TCEP       │LX-2026-03  │950 mg    │2027-03-15│2026-06-15│Opened││
│ ☐│M-3045   │PBS Buffer │PB-2026-12  │2.5 L     │2026-12-31│N/A       │Stored││
│ ☐│M-3088   │Client Drug│CD-26-A1 🔒│15 mg     │2026-08-30│2026-06-30│Opened││
│ ...                                                                           │
└──────────────────────────────────────────────────────────────────────────────┘
```

**客户提供物料标记**：编码后加 🔒 图标，hover 提示"客户提供，CMC 受控"，列表行加左侧 2px 紫色边框

#### 15.5.7 P7 - 细胞库管理（II 期）

**路由**：`/cell-banks`

参考 Benchling Inventory 的 Vial 网格视图：

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ 首页 / 细胞库 / CB-0125                                                       │
├──────────────────────────────────────────────────────────────────────────────┤
│ CB-0125 · CHO-K1 MCB                  [●Released]                            │
│ MCB · 代次 P15 · 总 vial 数 200 · 剩余 187                                   │
│                                    [出库] [盘点] [转移] [审计]                │
├──────────────────────────────────────────────────────────────────────────────┤
│ [概览] [Vial 库存] [出库历史] [转移历史] [QA 文档]                            │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│ Vial 库存（按位置）                                                          │
│                                                                              │
│ Freezer LN-01 (LN2) / Tower 3 / Cane 7                                       │
│ ┌─────────────────────────────────────────────┐                              │
│ │  1     2     3     4     5     6           │                              │
│ │ ✓    ✓    ✓    ✗    ✓    ✓    Row A     │ ← ✓ 在库 ✗ 已使用              │
│ │ ✓    ✓    ✓    ✓    ✓    ✓    Row B     │                              │
│ │ ✓    ✓    ✓    ✓    ○    ○    Row C     │ ← ○ 空位                       │
│ └─────────────────────────────────────────────┘                              │
│                                                                              │
│ 点击 vial 查看详情或操作                                                      │
└──────────────────────────────────────────────────────────────────────────────┘
```

### 15.6 交互模式与动效

#### 15.6.1 通用交互原则

1. **可逆性**：所有非破坏性操作支持 Undo（5 秒 toast 内）
2. **预览**：批量操作前显示影响范围（"将影响 23 个样品"）
3. **延迟加载**：列表 > 100 条时使用虚拟滚动
4. **即时反馈**：表单字段实时校验，错误立即显示
5. **保留状态**：筛选、排序、分页通过 URL 保存，刷新不丢失

#### 15.6.2 动效规范

**原则**：克制、迅速、有意义

| 场景 | 时长 | 缓动 |
|---|---|---|
| 按钮 hover | 100ms | ease-out |
| 抽屉打开 | 200ms | ease-out |
| 模态框打开 | 150ms | ease-out |
| Tab 切换 | 100ms | linear |
| Toast 进入 | 200ms | ease-out |
| Toast 离开 | 150ms | ease-in |
| 状态变更（如 ✓ Done） | 300ms | ease-in-out |
| Loading 骨架屏 | 1500ms 循环 | ease-in-out |
| 状态圆点脉冲（In Progress） | 1500ms 循环 | ease-in-out |

**禁用**：
- 页面切换的滑动/淡入（仅 instant 切换）
- 表格行变化时的"打字机"效果
- 不必要的弹跳、旋转

#### 15.6.3 键盘快捷键

| 快捷键 | 功能 |
|---|---|
| `⌘ K` / `Ctrl K` | 打开命令面板 |
| `/` | 聚焦搜索框 |
| `?` | 查看快捷键帮助 |
| `g d` | 跳转到 Dashboard |
| `g p` | 跳转到项目列表 |
| `g s` | 跳转到样品列表 |
| `g m` | 跳转到监控台 |
| `c` | 创建新对象（上下文相关） |
| `e` | 编辑当前对象 |
| `Esc` | 关闭模态框/抽屉 |
| `j` / `k` | 上下移动（列表中） |
| `Enter` | 进入/确认 |
| `⌘ Enter` | 提交表单 |
| `⌘ S` | 保存草稿 |

### 15.7 响应式与状态

#### 15.7.1 断点

| 断点 | 宽度 | 适配 |
|---|---|---|
| `xs` | <640px | 不支持（移动端 II 期再说） |
| `sm` | 640–1024px | 不优化，可用即可（平板） |
| `md` | 1024–1280px | 兼容（小笔记本） |
| `lg` | 1280–1536px | **主要设计目标** |
| `xl` | >1536px | 宽屏，内容居中 max-width 1440px |

#### 15.7.2 加载状态

- **页面加载**：使用骨架屏（matching 真实内容的灰色块）
- **数据加载**：表格使用骨架行，卡片使用骨架卡片
- **操作中**：按钮文字保留 + 内嵌 spinner（按钮变 disabled）
- **不要使用全屏遮罩 loading**

#### 15.7.3 错误状态

- **网络错误**：右上角 toast + 自动重试 3 次
- **权限错误**：替换内容区为 403 占位（"你没有权限查看此页面"）
- **404**：友好的"页面不存在" + 返回按钮
- **API 500**：内联错误卡片 + 重试按钮 + 反馈链接

#### 15.7.4 空状态

每种列表/视图都需要设计空状态：
- 插画或大图标（neutral 系列灰色）
- 引导文字
- 主操作按钮（如"+ 新建第一个样品"）

#### 15.7.5 受控操作的视觉提醒

**任何 GxP 受控操作（签名、删除、关键字段修改）**：
- 按钮配色使用 `danger-500`（如果是破坏性）或 `primary-600` + 锁定图标
- 点击触发二次确认 modal
- Modal 顶部带 `⚠ 受控操作` 警示条（背景 `warning-50`）

---

### 15.8 原型生成提示词模板（喂给 AI 工具）

当你需要让 AI（v0 / Claude Code / Cursor / Bolt）基于本 BRD 生成 HTML 原型时，建议在 prompt 中包含以下要素：

```
请基于以下 BRD 设计规范生成一个 HTML/React 原型：

【系统定位】
XDC BioFoundry - 偶联工艺平台实验室数字化系统，PD 工艺协作 + AS 送样监控 + BCOE 主数据管理

【设计原则】
- 风格参考：Linear + Ant Design Pro + Benchling（借鉴交互不抄视觉）
- 偏专业克制，避免亮色和装饰
- 信息密度高，类似 Linear / Notion
- 状态颜色 + 图标 + 文字三重编码

【视觉规范】
- 主色：深蓝灰（primary-600: #486581）
- 字体：Inter（英文）+ 苹方（中文），基础 14px
- 圆角：默认 6px（卡片），4px（按钮/输入）
- 间距：4px 网格（4/8/12/16/24...）

【布局】
- Top bar（56px，深色 primary-800）
- Sidebar（240px，白色）
- 内容区 max-width 1440px

【要生成的页面】
[这里指定要生成的页面，参考 §15.5 的 wireframe]

【关键交互】
[这里指定要实现的交互，如表格排序、筛选、抽屉等]

【数据】
请使用合理的 mock 数据填充，包括：
- 12 个 Project (P-26-001 到 P-26-012)
- 50 个 Sample
- 142 个在测的 Test Request
- 真实的 stage 名称（S1: mAb thaw, S2: Reduction, S3: UF/DF...）
- 真实的检测项（DAR, SEC, free drug, potency...）
- 真实的中文名称（张明、李华、王芳等）

【技术栈】
- React + TypeScript
- TailwindCSS（CSS 变量按 §15.2.1 定义）
- Lucide Icons
- shadcn/ui 或自己实现的组件库

【输出要求】
- 单文件 HTML 或模块化 React，由你选择
- 完整的视觉细节，不要 placeholder
- 包含 hover/active 状态
- 表格至少展示 10 条数据
- 包含至少 1 个完整可交互流程（如打开抽屉、筛选、签名 modal）
```

---

**文档结束。**

> **下一步建议**：
> 1. 业务侧（PD Manager + BCOE Manager + QA）审阅本 BRD，重点 review §3 定位、§5 流程模型、§11 分期、§15 UI 设计。
> 2. **TBD-03（外部送样系统接口）必须在 I 期启动前 sign-off**，否则 I 期 MVP 无法启动。
> 3. TBD-01（Study Plan 分类）、TBD-02（审批链）可在原型评审会上与 BCPD 共同确认。
> 4. 使用 §15.8 的提示词模板将本 BRD 喂给 v0 / Claude Code，先生成 FL 监控台（§15.5.1）作为第一个可演示页面。
> 5. 基于原型评审反馈，迭代 BRD v0.4，进入开发排期。
