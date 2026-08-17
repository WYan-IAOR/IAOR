# 从模糊意图到零冲突并行交付：Manager 如何把歧义变成有序的工作

**作者**: W.Y.
**日期**: 2026-08-12

---

> 上一篇给了 IAOR 一个定义。跑通之后，我又把那次实验的数据翻出来看。翻着翻着，我注意到一件有意思的事：这份简报连"谁写哪个文件"都没说清楚，可最后交出来的，是 25 个零冲突的文件、一次通过的构建。我想把"模糊是怎么变成有序的"单独讲讲。

## 问题

这里有一个在真实开发中不断发生的场景。产品经理把一段描述扔给团队：

> "做一个 Kanban 风格的任务看板。用户可以创建、编辑、删除任务。任务经历状态流转（todo → in_progress → done，再循环回 todo）。要有过滤、搜索、统计和持久化。目标大约 20-25 个源文件。`npm run build` 必须零错误通过。"

这是一份产品描述，不是工程计划。它没有说：
- 要创建哪些文件
- 哪个工程师负责哪个文件
- 谁负责共享的 `constants.ts`
- 是否有人可以并行工作

一个单个 Agent 收到这个，会直接开始写代码。它会隐式地、独自地做出文件归属的决策——没有人跟它冲突，但也没有协调、没有并行、没有验证，也没有办法发现自己看不到的盲点。

一个组织面临更难的考验：它必须把这个模糊的意图，变成干净、并行、零冲突的工作。这篇文章要讲的，就是 IAOR 如何做到这一点——以及一次真实运行的、具体的数据。

## 实验设置：三层刻意的模糊

为了测试一个 Manager 是真正能解决歧义，还是只是执行一份清晰的规格，实验被刻意在三处加了难度：

**1. 一份产品简报，而不是任务清单。** 简报（TaskFlow-Project-Brief.md）描述的是*产品*——功能、数据模型、验收标准——但没有给文件拆分。

**2. 一个归属不明的共享文件。** 简报对 `constants.ts` 说：

> "Shared constants (status labels, priority colors, storage keys) should live in a single `src/constants.ts` file that multiple components import from"

这句话说 `constants.ts` 是"多个组件共享的"——但从来没说谁来创建。从逻辑上它属于基础设施层（它定义全局常量）；从语义上它听起来又跟组件有关。这个矛盾是刻意制造的。Manager 会怎么决定？

**3. 没有"避免重叠"的提示。** Manager 的任务指令删除了任何"assign files so there is minimal overlap"（分配文件使重叠最小化）的指导，并把"who should own each file"（谁负责每个文件）弱化成"who should work on what"（谁该做什么）。Manager 必须自己决定如何处理边界模糊的文件。

## Manager 做了什么

Manager 花了 57 秒理解项目（读简报、读脚手架、确认 `src/` 是空的），然后在一个规划步骤里做出了三个决策：

### 决策 1：它解决了模糊文件的归属

Manager 没有被"多个组件共享"的描述迷惑。它从技术架构角度判断 `constants.ts`——这是一个基础设施层文件（全局常量），跟 `types.ts`、`utils.ts` 同层。于是它把它分配给了基础设施 Operator（004），并明确归属：

> "You own src/types.ts, src/constants.ts, src/utils.ts, src/api.ts, src/store.ts."

而对前端 Operator（003），它只给了 import 权限，从没给写权限：

> "What you import (these files will exist, built by 004): import { STATUS_LABELS, PRIORITY_COLORS, STATUS_ORDER } from '../constants'"

003 从头到尾没写过 `constants.ts`。**一个模糊的文件，由一个归属决策解决——不是把它拆成两半，不是让两个 Operator 协商。**

### 决策 2：用预填契约消除串行依赖

前端 Operator（003）依赖基础设施 Operator（004）的 types 和 store。一个天真的 Manager 会串行执行：004 写完，003 才开始。相反，这个 Manager **把 004 的接口签名直接预填进 003 的任务描述**——完整的类型定义和完整的 Zustand store API。这让 003 可以立即开始编码，并行进行，无需等待。

### 决策 3：选择了合理的拆分粒度

Manager 规划了 25 个文件——5 个给基础设施（004: types/constants/utils/api/store），20 个给前端（003: 入口文件 + 8 个组件 × 每个 2 文件）。这符合简报的"20-25"范围，也遵循标准的 React 组织方式（每个组件一个 `.tsx` + 一个 `.css`）。

## 结果：6 分 26 秒，25 个文件，零冲突

两个 Operator 在同一个 LLM 轮次里被分派。并行执行立即开始。完整时间线：

| T+ | 事件 |
|----|------|
| 0s | Manager 首次 LLM 调用，读简报 |
| 57s | Manager 同时分派 003 和 004 |
| 63s | 004 首次 LLM 响应（开始基础设施） |
| 67s | 003 首次 LLM 响应（开始组件） |
| 87s | 004 完成全部 5 个基础设施文件 |
| 189s | 004 发消息给 003，附上完整接口文档 |
| 200s | Manager 发现 `TaskCard.tsx` 缺失 |
| 210s | Manager 自主分派 003 修复 |
| 275s | 003 的 `npm run build` 成功 |
| 300s | 003 向 Manager 汇报成功 |
| 314s | 003 向 004 发送交叉集成确认 |
| 366s | 004 完成二次验证，向 Manager 汇报 |
| 386s | Manager 确认，进入 idle |

**有效工作时间：6 分 26 秒**（从 T+57s 分派到 T+386s 完成）。

| 指标 | 值 |
|------|-----|
| 产出的源文件 | 25（5 基础设施 + 20 前端） |
| 并行工作的 Operator | 2 |
| 被两个 Operator 同时写的文件 | 0（零冲突） |
| LLM 调用 | 66（Manager 25，003 23，004 18） |
| 工具调用 | 240 |
| `npm run build` | 通过，53 modules，0 错误 |
| TypeScript | 零错误，无 `any` |
| 人工介入 | 0 |

关键数字是**零冲突**。没有任何一个文件被两个 Operator 同时写入。那个可能成为碰撞点的 `constants.ts` 被 004 独占，003 只读取和 import 它。

## 为什么单个 Agent 做不到

单个 Agent 一开始就不会遇到这个冲突——它拥有所有东西，所以没什么需要协调的。但这恰恰是重点：**单个 Agent 无法并行，也无法从独立验证者那里受益。**

对比两者：

| | 单个 Agent | IAOR 组织 |
|---|-----------|-----------|
| 模糊归属 | 独自隐式决策 | Manager 明确解决 |
| 并行 | 串行——一次一件事 | 两个 Operator 并行 |
| 依赖处理 | 对着自己未提交的工作编码 | Manager 预填契约，零等待 |
| 速度 | 一个 Agent，一个线程 | 25 个文件在 6:26 并行完成 |

一个单个 Agent，被要求构建同一个 25 文件的 React 应用，会串行执行——大概需要几十分钟，没有谁帮它发现盲点，也无法分工。

<figure>
  <img src="article-images/fig3-parallel.png" alt="两个 Operator 并行工作">
  <figcaption><b>图 3 —— 并行执行。</b> Manager 的时间线总结把并行点明了：<i>"Both operators worked in parallel: 004 built the type/store foundation while 003 built components against those interfaces."</i> 两个 Operator 各按契约并行推进。</figcaption>
</figure>

## 更深的点：契约是组织并行化的关键

让 6:26 成为可能的不是原始速度。而是**契约**。Manager 把一个模糊的整体，变成两个由精确接口契约连接的解耦工作流：

- 004 基于清晰的归属列表构建基础设施
- 003 基于预填的类型/store 契约构建前端
- 两者从不碰同一个文件，因为契约精确定义了谁拥有什么

这就是真实组织并行化的方式：不是"更快地工作"，而是**定义让并行安全的契约**。契约是使能的人工产物——在 IAOR 里，Manager 就是那个产出并分发契约的角色。

## 一点诚实

这个实验并非完美。有两点值得直说：

1. **这次任务的模糊是刻意制造的。** 简报被写成产品描述而非工程清单、`constants.ts` 归属不明、Manager 指令里刻意去掉了"避免重叠"的提示——这些难度是人为加上的，目的是观察 Manager 如何自己解决歧义。真实场景里，模糊可能更随意，也可能更少。
2. **一个 Operator 难以单独 type-check 自己的文件。** `tsc` 检查整个项目，所以基础设施 Operator 在前端存在之前，无法干净地只验证自己的 5 个文件。这是引擎层的一个缺口（部分类型检查），我正在解决。

## 结论

一个模糊的、歧义的、潜在冲突的高层意图，可以被变成干净的并行工作——25 个文件、零冲突、6 分 26 秒——如果存在一个能解决歧义、定义契约、并行分派的层。那一层就是 IAOR 的 Manager 所提供的。

单个 Agent 做不到这一点，因为单个 Agent 没有东西可组织。

---

*W.Y. · 2026-08-12*
