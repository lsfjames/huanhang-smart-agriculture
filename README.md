<div align="center">

# 寰航智农 · Huanhang Smart Agriculture

**本地优先的智慧农业 AI 平台 —— 叶片病害分类 · 病斑检测 · 知识图谱问答 · 微服务业务闭环**

[English](README_EN.md) · [系统架构](docs/ARCHITECTURE.md) · [技术栈](docs/STACK.md) · [实验指标](docs/METRICS.md) · [演示脚本](docs/DEMO_SCRIPT.md) · [项目状态](#8-项目状态)

![License](https://img.shields.io/github/license/lsfjames/huanhang-smart-agriculture)
![Status](https://img.shields.io/badge/状态-Windows%20本地全栈已验证-2f855a)
![Showcase](https://img.shields.io/badge/仓库-Showcase%20·%20核心源码私有-6b7280)
![Stack](https://img.shields.io/badge/技术栈-Vue3%20·%20Spring%20Cloud%20·%20FastAPI%20·%20Neo4j-4378ff)

</div>

> 本仓为**项目展示仓（Showcase）**，完整源码保存在私有仓 [`huanhang-smart-agriculture-core`](https://github.com/lsfjames/huanhang-smart-agriculture-core)。
> 面向岗位：AI 应用工程师 / FDE · AI 解决方案 / AI 算法工程师。

---

## 目录

1. [业务问题](#1-业务问题)
2. [技术方案](#2-技术方案)
3. [系统架构](#3-系统架构)
4. [产品截图](#4-产品截图)
5. [已验证成果](#5-已验证成果)
6. [工程问题与解决](#6-工程问题与解决)
7. [演示](#7-演示)
8. [项目状态](#8-项目状态)
9. [验证范围](#9-验证范围)
10. [已知限制](#10-已知限制)
11. [Roadmap](#11-roadmap)
12. [文档导航与许可](#12-文档导航与许可)

---

## 1. 业务问题

- **识别**：田间叶片症状与混淆病（早疫 / 晚疫 / Septoria 等）肉眼难分，**客服 / 专家成本高** → 需要分类与病斑框作为参考（**非正式诊断**）。
- **可信问答**：通用大模型易给出剂量、预测等危险建议 → 农技问答需要**知识卡 + 来源审核 + 引用核验 + 拒答**。
- **协作**：农户上传、专家复核、管理员治理，而非单一 Demo API。
- **工程**：从原型到微服务、异步任务、权限、测试与发布审计的完整链路。

---

## 2. 技术方案

```text
上传叶片 ──► MobileNetV3 三分类 ──► 诊断入库
          └─► YOLOv12n 病斑框 ──► 叠加展示（标签空间隔离）
专家复核 ◄── 待复核队列（跨农户）     管理端：用户/模型版本/知识审核
问答 ──► Neo4j 图谱 + hybrid RAG ──► 引用核验 / 资料不足拒答
巡检 ──► Offline Replay（历史图回放，非无人机）
```

- **做法**：分类 + 检测 + **可溯源 RAG 问答**，配合农户 / 专家 / 管理三角色业务闭环。
- **形态**：Vue 3 前端 · Spring Cloud · FastAPI（AI）· MySQL · Neo4j · Docker Compose。
- **有意保持简单**：不堆砌未使用框架；服务发现 / 配置中心等按需取舍。

### 技术栈

| 层 | 选型 | 职责 |
| --- | --- | --- |
| **前端** | Vue 3 · TypeScript · Vite · Element Plus | 登录/地块/诊断/历史/问答/图谱/专家台/管理台/巡检回放 |
| **后端** | Spring Boot 3.4 · Spring Cloud Gateway · JWT · Flyway | 账号权限、诊断业务、跨服务调用、审计字段 |
| **AI 推理** | FastAPI · PyTorch | 分类/检测/语言独立生命周期与超时 |
| **分类** | MobileNetV3-Small（迁移学习） | 番茄叶三分（早疫/晚疫/健康） |
| **检测** | YOLOv12n（官方实现，AGPL） | PlantDoc 29 类病斑/叶部框 |
| **语言** | Qwen2.5-0.5B-Instruct | 本地生成，审核卡约束 |
| **检索** | keyword / graph / hybrid + Neo4j Cypher | 证据卡与图路径 |
| **数据** | MySQL 5.7 · Neo4j 5.26 | 业务库 + 知识图谱 |
| **工程** | JUnit · unittest · Docker Compose · GitHub Actions | 测试与交付（Docker/CI 见[项目状态](#8-项目状态)） |

明细见 [docs/STACK.md](docs/STACK.md)。

---

## 3. 系统架构

```mermaid
flowchart TB
  subgraph Client
    UI[Vue 3 + TS + Element Plus]
  end
  subgraph Edge
    GW[Spring Cloud Gateway :8888]
  end
  subgraph Java
    US[user-service :8101<br/>JWT · 地块 · 角色]
    DS[diagnosis-service :8102<br/>识别任务 · 专家复核 · 巡检 · 代理]
  end
  subgraph AI
    VIS[FastAPI vision :8001<br/>MobileNetV3 + YOLOv12]
    LLM[FastAPI language :8002<br/>Qwen2.5 + hybrid RAG]
  end
  subgraph Data
    MY[(MySQL 5.7<br/>users/plots + diagnoses)]
    NEO[(Neo4j 5<br/>知识图谱)]
  end
  UI --> GW
  GW --> US
  GW --> DS
  US --> MY
  DS --> MY
  DS -->|HTTP| VIS
  DS -->|HTTP| LLM
  LLM --> NEO
  VIS -. weights .- W[models/ 不入 Git]
```

模块职责详见 [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)。

### 业务功能与权限（农户 / 专家 / 管理员）

| 角色 | 能力 |
| --- | --- |
| **农户** | 注册登录、地块、上传叶片→分类+检测、历史、问答、发起巡检回放任务 |
| **专家** | 跨农户**待复核队列**、诊断详情、CONFIRM / OVERRIDE / UNCERTAIN 与意见、历史复核 |
| **管理员** | 用户角色管理、**模型版本清单**（sha256/run_id）、**知识卡审核台**、全链路状态 |

权限：FARMER 无队列 / 模型清单（403）；跨用户诊断 404；复核写接口 EXPERT / ADMIN。

---

## 4. 产品截图

| | | |
| :---: | :---: | :---: |
| ![登录](docs/screenshots/01_login.png) | ![注册](docs/screenshots/02_register.png) | ![首页](docs/screenshots/03_home.png) |
| **登录** | **注册** | **首页** |
| ![地块](docs/screenshots/04_plots.png) | ![识别结果](docs/screenshots/05_diagnose_result.png) | ![识别异常](docs/screenshots/06_diagnose_error.png) |
| **地块管理** | **识别结果** | **识别异常处理** |
| ![历史](docs/screenshots/07_history.png) | ![问答拒答](docs/screenshots/08_qa_strict_refusal.png) | ![问答作答](docs/screenshots/09_qa_experimental_answer.png) |
| **诊断历史** | **问答：资料不足拒答** | **问答：可溯源作答** |

---

## 5. 已验证成果

### AI 能力与实验指标

> 以下均为**验证集 / 离线实验**结果，性质说明见 [验证范围](#9-验证范围)；完整实验记录见 [docs/METRICS.md](docs/METRICS.md)。

| 模型 | 任务 | 真实指标 | 说明 |
| --- | --- | --- | --- |
| **MobileNetV3-Small** | 番茄叶 3 分类 | **Accuracy ≈ 0.935 · Macro-F1 ≈ 0.934** | 固定 checkpoint 验证；softmax **未校准** |
| **YOLOv12n** | PlantDoc 29 类检测 | **mAP@50 = 0.074 · mAP@50-95 = 0.045 · P/R = 0.40/0.15** | ~11 epoch 后 **4GB GPU OOM 中断**；基线可演示，**非高精度** |
| **Qwen + RAG** | 限定知识问答 | hybrid 检索命中率 **0.9286**（dev 28 题） | 资料不足**拒答**；引用 ⊆ 上下文 |
| **图谱** | 可溯源事实 | **71 节点 / 118 边** | 卡片 + 来源 + 审核记录 |

### 工程与测试

| 套件 | 结果 |
| --- | --- |
| JUnit（真 MySQL + Flyway） | **26 / 26** |
| 知识审核并发 / 原子写 | **6 / 6** |
| 系统审查探针（联调 + 权限 + AI） | **17 / 17** |
| 网关 E2E（阶段 9） | **11 / 11** |
| 故障注入（超时 / 越权 / 坏上传） | **9 / 9** |

要点：缺 multipart → 400；非法 retriever → 422；Neo4j 宕机 hybrid 降级并告警。

---

## 6. 工程问题与解决

自主设计并实现的微服务系统，在以下工程问题上的解决方式：

1. **可信 RAG**：审核状态门禁 + 来源已核实 + 答案引用核验 + 超范围拒答，降低「编造农技建议」风险。
2. **检测 / 分类标签隔离**：PlantDoc 框类 ≠ 番茄三分病名，UI 并列展示、禁止伪造成框。
3. **4GB 显存约束**：YOLOv12n + AMP；OOM 后可续训；指标如实披露。
4. **微服务边界**：Gateway + user/diagnosis；AI 独立 FastAPI；跨库所有权清晰。
5. **离线巡检**：明确 **Offline Replay**，不冒充无人机实时链路。

---

## 7. 演示

- **脚本（3–5 分钟）**：[docs/DEMO_SCRIPT.md](docs/DEMO_SCRIPT.md)
- **截图**：[docs/screenshots/](docs/screenshots/)
- 视频可按脚本录制；未包含真实客户数据。

---

## 8. 项目状态

| 方式 | 状态 |
| --- | --- |
| **Windows 原生进程** | **已验证**（MySQL / Neo4j / FastAPI / Spring / Vue 全栈业务闭环） |
| **Docker Compose** | **未验证**（本机未安装 Docker；配置已备，见私有仓 `DOCKER_INSTALL_PLAN.md`） |
| **GitHub Actions** | **工作流已编写，未在远程跑通前不宣称 CI 通过** |

本地运行与 Compose 设计说明：私有工程仓 `docs/` 与 `docker-compose.yml`。

---

## 9. 验证范围

- **指标性质**：第 5 节全部 AI 指标为**验证集 / 离线实验**，不是田间部署效果。
- **测试性质**：第 5 节测试结果为**程序行为验证**（本机与工程仓测试套件），不代表田间可用性。
- **问答边界**：问答域为**番茄相关公开推广资料**；资料不足时拒答，引用须落在给定上下文内。
- **巡检语义**：巡检为**历史图回放（Offline Replay）**，非无人机实时链路。
- **未验证项**：Docker Compose 与远程 CI 状态见 [项目状态](#8-项目状态)，跑通前不作通过宣称。

---

## 10. 已知限制

**请勿当作成熟植保产品：**

- 非医疗器械 / 非正式植保结论；分类分数**未校准**。
- 检测 mAP 低；训练因 4GB 显存 OOM 中断（见 [已验证成果](#5-已验证成果)）。
- Docker / CI 本机**未完成实测**（见 [项目状态](#8-项目状态)）。
- 知识域以**番茄相关公开推广资料**为主。

---

## 11. Roadmap

| 版本 | 目标 |
| --- | --- |
| v1.1 | Docker Compose 与 CI 在干净环境实测；检测续训提升 mAP |
| v1.2 | 知识审核入库与多副本；模型灰度切换 UI |
| v2.0 | 真实田间数据闭环（替换 Offline Replay）与移动端 |

---

## 12. 文档导航与许可

| 文档 | 内容 |
| --- | --- |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | 系统架构与模块职责 |
| [docs/STACK.md](docs/STACK.md) | 技术栈明细 |
| [docs/METRICS.md](docs/METRICS.md) | 实验指标与记录 |
| [docs/DEMO_SCRIPT.md](docs/DEMO_SCRIPT.md) | 演示脚本 |
| [docs/LICENSES.md](docs/LICENSES.md) | 第三方许可清单 |

**许可**

- 本展示文档与首方说明：MIT（见 [LICENSE](LICENSE)）。
- 完整工程源码在私有仓，其中含 **YOLOv12 / Ultralytics AGPL-3.0**、**PlantDoc CC-BY-4.0**、Qwen 等第三方条款（见 [docs/LICENSES.md](docs/LICENSES.md)）。
- **请勿**将本展示仓误认为完整 AGPL 源码镜像。

© 2026 寰航智农 Project Showcase · Maintainer: [lsfjames](https://github.com/lsfjames)
