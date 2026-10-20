# 技术栈与实验指标 / Stack & Metrics

## 技术栈分类

| 类别 | 技术 |
| --- | --- |
| 前端 | Vue 3、TypeScript、Vite、Element Plus、Pinia |
| 后端 | Spring Boot 3.4、Spring Cloud Gateway、Spring Security JWT、Flyway |
| AI 服务 | FastAPI、Uvicorn、PyTorch 2.10+cu128 |
| 模型 | MobileNetV3-Small、YOLOv12n（sunsmarterjie turbo）、Qwen2.5-0.5B-Instruct |
| 数据 | MySQL 5.7、Neo4j Community 5.26 |
| 工程 | JUnit 5、unittest、Docker Compose（配置）、GitHub Actions（配置）、保护清单脚本 |

## 真实实验指标（离线验证）

### MobileNetV3-Small · 番茄叶三分类

| 指标 | 数值 |
| --- | --- |
| Validation Accuracy | **0.9346** |
| Macro-F1 | **0.9340** |
| 权重 run_id | `baseline_20261006_175350_f39a01` |
| 说明 | softmax 分数**未校准**；非田间效果 |

### YOLOv12n · PlantDoc 病斑/叶部检测

| 指标 | 数值 |
| --- | --- |
| mAP@50 | **0.0742** |
| mAP@50-95 | **0.0445** |
| Precision / Recall | **0.402 / 0.147** |
| 训练 | 25 计划 / **约 11 epoch 后 CUDA OOM**（RTX 3050 4GB） |
| 数据 | PlantDoc VOC 真框 → YOLO；val 225 图（剔 11 泄漏） |
| 说明 | **基线可演示**；不得宣传为高精度检测 |

### Qwen + RAG + Neo4j

| 项 | 结果 |
| --- | --- |
| hybrid vs keyword 命中率 | **0.9286 vs 0.8214**（dev 28 题） |
| 疾病对齐率 | hybrid **0.8929** |
| 策略 | approved_only 拒答 / 引用核验 citation_violation 检测 |
| 图规模 | 71 节点 / 118 边 / 14 知识卡 |

### 测试计数

| 套件 | 通过 |
| --- | --- |
| JUnit | 26/26 |
| knowledge store | 6/6 |
| 审查探针 | 17/17 |
| 网关 E2E | 11/11 |
| 故障注入 | 9/9 |
