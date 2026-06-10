# vortex_trader · Vortex 量化研究院枢纽

> 本库是 Vortex 量化平台的**总装与文档枢纽**，也是"研究院／交易员"的大脑：
> 向上沉淀整体设计、接口协议与镜像部署文档；向下编排 `data → backtest → qmt`
> 的研究与交易闭环。本库目标是**输出技术文档**并承载**持续自主的量化策略研究**。

## 平台组成

Vortex 由四个同级 Git 库组成，各司其职、均已容器化：

| 库 | 角色 | 服务端口（内外一致） | 成熟度 |
|---|---|---|---|
| **vortex_data** | 数据服务：Tushare 采集 → Parquet 落盘 → Qlib 导出 | 8765 | 成熟（1.2G 行情 + 778M qlib 导出，79 测试绿） |
| **vortex_backtest** | 回测引擎：A股分钟级撮合、异步作业、看板 | 8766 | 成熟（真实数据已跑通） |
| **vortex_qmt** | 实盘交易：目标组合→风控→QMT 桥接→对账 | 8767 | 框架成熟（paper/dry-run 通；未接真实 QMT） |
| **vortex_trader**（本库） | 研究院／编排／文档枢纽 | 8768（预留） | 建设中 |

> 端口规范权威来源：vortex_common 的 `config/registry.yml` + [ADR-003](../vortex_common/docs/adr/ADR-003-unified-config-architecture.md)。端口一号到底、容器内外一致，对外暴露由 `VORTEX_<SVC>_BIND_ADDR` 控制。

数据流：**Tushare → vortex_data（parquet + qlib）→ vortex_backtest（回测）→ 策略产出目标组合 → vortex_qmt（实盘）**。

## 文档索引

| 文档 | 内容 |
|---|---|
| [design/00-总览与定位.md](design/00-总览与定位.md) | 平台愿景、研究院定位、本库职责边界 |
| [design/01-端到端架构.md](design/01-端到端架构.md) | 四库关系、端到端数据流、公共契约、集成缺口、架构图 |
| [design/02-接口协议汇总.md](design/02-接口协议汇总.md) | data／backtest／qmt 三服务 HTTP API、CLI、关键数据契约 |
| [design/03-镜像与部署.md](design/03-镜像与部署.md) | 镜像继承链、端口/卷映射、compose 编排、Windows 桥接部署 |
| [design/04-研究院方法论与自主研究闭环.md](design/04-研究院方法论与自主研究闭环.md) | 研究闭环 SOP、agent 研究员分工、数据资产盘点、研究方向建议、自动化路线 |
| [design/05-L3自治交易能力阶梯.md](design/05-L3自治交易能力阶梯.md) | L3 在环交易的分级灰度（L3a→L3e）、进入门禁、横切风控与治理 |

**架构决策（ADR）：** [docs/adr/ADR-001-investment-agent-architecture.md](docs/adr/ADR-001-investment-agent-architecture.md) —— 投资 agent 架构：agent 负责"找与决策"、确定性代码负责"关与限"，目标 L3 在环交易。

架构图源文件：[diagrams/architecture.mermaid](diagrams/architecture.mermaid) · [diagrams/deployment-topology.mermaid](diagrams/deployment-topology.mermaid)

## 快速指引

```bash
# 1. 启动数据服务（采集 + 看板 + qlib 导出）
cp <vortex_data>/.env.example <vortex_data>/.env   # 填 TUSHARE_TOKEN
vortex run up data
# 看板：http://127.0.0.1:8765

# 2. 启动回测引擎（只读挂载 data 的 workspace）
vortex run up backtest                           # http://127.0.0.1:8766

# 3. 启动实盘服务（默认 dry-run；实盘需 Windows 桥接 + 三重门禁）
cp <vortex_qmt>/.env.example <vortex_qmt>/.env
vortex run up qmt                                # http://127.0.0.1:8767

# 一键拉起全平台（共享卷自动注入；宿主机根默认 ~/vortex/{workspace,state}）
vortex run deploy
```

> 详细部署、端口、卷、安全与桥接拓扑见 [design/03-镜像与部署.md](design/03-镜像与部署.md)。

## 当前状态

已完成：四库 + vortex_common/qmt-bridge 调研、端到端契约梳理、数据资产盘点、枢纽文档与架构图。
角色已定为**投资 agent**（自主量化研究员/交易员），目标形态 **L3 在环交易**，核心原则"agent 负责找与决策、确定性代码负责关与限"——见 [ADR-001](docs/adr/ADR-001-investment-agent-architecture.md) 与 [design/05](design/05-L3自治交易能力阶梯.md)。
下一步：vortex_trader 服务化（端口 8768）+ 研究层 spike（qlib/RD-Agent 自动挖因子）+ 先搭 L3a 影子模式。
