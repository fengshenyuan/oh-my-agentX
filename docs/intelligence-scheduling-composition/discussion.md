# Intelligence Scheduling / Composition

> 本文件是本主题在写入 oh-my-agentX 之前的原始讨论记录。
>
> 按要求，这里不做二次提炼；保留讨论中的中文用户问题与助手回答，只去掉工具调用、系统消息等非对话内容。

---

## User

在日常的开发任务当中，gpt-5.6 Luna, Terra, Sol, 三个模型 + 4档thinking，应该怎么选择？我发现这是个困扰许多人的问题：每档模型配置到底合适什么样的任务？比如我最近开发时遇到一个springboot 应用 在 k8s pod部署期间最后1s日志丢失的问题，我开始以为这个问题很难，所以直接在codex启用了Sol + xhigh 模式，但是发现模型很容易就进入超长(过渡)思考模式，感觉事无巨细都想检查、验证到100% perfect才会甘心进行下一步，有时甚至觉得我应该打断它，因为它一直就一个无关大雅的问题死命纠结，持续验证，什么扫描.local 里的所有mvn package来寻找一个没有什么太多意义的问题的答案。

下面这张图 https://artificialanalysis.ai/ 的index图，也很让人困惑：sol + medium，sol + low，terra + high，terra + xhigh 相对集中，基本在图中部，这四个组合的选择性就很让困惑，怎么选？它们这个档位有适合任何样难度的问题，我可不会认为thinking强度就等价于问题难度，而且问题难度在任务给到agent时针的主观性非常强。agent使用者需要在模型/harness工具/问题难度任务等多个维度上有相当的理解才能给与判断，或者说才能用好agent。但这个标准是什么？怎么磨炼怎么judgement或者说是品味呢？

---

## Assistant

我非常赞同你对这个问题的感觉，而且我认为你已经碰到了 **2026 年 agent coding 一个很核心、但目前还没有形成成熟“使用方法论”的问题**：

> **Reasoning effort 不是“问题难度旋钮”，而更像是“允许 agent 自主探索和自我验证多少”的旋钮。**

所以真正应该学的不是“什么难度的问题用 high”，而是建立一套 **task × model × reasoning × harness** 的判断框架。

我先给你一个我认为非常实用的结论：

> **日常开发不要把 Sol xhigh 当成“最难问题档”。**
>
> 更好的默认策略是：**低成本、高吞吐的任务用 Luna/Terra；真正高不确定性、高耦合、难验证的问题才逐步把 reasoning 往上推。**
>
> 而且很多时候，**“更强模型 + 较低 reasoning”比“较弱模型 + 很高 reasoning”更符合 agentic coding 的实际工作方式。**

OpenAI 自己现在对 GPT-5.6 的官方建议也非常接近这个方向：Sol 定位为复杂 reasoning/coding，Terra 是 intelligence/cost balance，Luna 是高吞吐；reasoning effort 则应该“有意识地设置”，并建议用当前档位和低一档做实际 workload benchmark，而不是默认认为越高越好。

### 1. 你图里的“4个很接近的点”，其实不是“四个差不多的模型”

你框出来的是：

- **Terra + High**
- **Sol + Low**
- **Terra + xHigh**
- **Sol + Medium**

它们在 Cost / Task 图上非常接近。

这恰恰是这张图最容易让人产生误解的地方。

它们不是“四个 intelligence 差不多的 preset”。

而是四个不同的：

> **Capability × inference compute × latency × cost**

trade-off。

所以我不会把：

> Terra High ≈ Sol Low ≈ Terra xHigh ≈ Sol Medium

理解成：

> “它们能力差不多。”

我会理解成：

> **“它们是不同方向上的局部 Pareto-optimal operating points。”**

### 2. 最重要的改变：不要用“任务难度”选择 reasoning

这是你这个问题里最重要的一点。

例如：

> “给我实现一个 OAuth2 登录模块”

可能需要 High。

而：

> “帮我把这个 controller 的返回值从 DTO 改成 record”

可能只需要 Low。

但：

> “为什么 Kubernetes deployment rollout 时最后 1 秒日志偶尔丢失？”

表面上代码工作量很小，但 agent 的**搜索空间非常大**。

这三个任务真正决定 reasoning 的不是“复杂度”，而是：

### A. Uncertainty —— 你不知道答案在哪

是不是可能出现在：

- Spring Boot shutdown lifecycle
- JVM shutdown hook
- SIGTERM
- Kubernetes termination lifecycle
- preStop
- terminationGracePeriodSeconds
- stdout buffering
- container runtime
- CRI
- Docker/containerd
- node logging agent
- Fluent Bit / Vector / Filebeat
- sidecar
- log aggregation
- async logger
- network buffering

这已经是一个巨大 hypothesis space。

### B. Search Space —— agent 要探索多少东西

一个任务可能只有 3 个文件，也可能是：

```text
代码
↓
配置
↓
Dockerfile
↓
Helm
↓
K8s manifest
↓
logging agent
↓
runtime
```

即使最后修改只有两行，**搜索空间也可能非常大**。

### C. Verification Difficulty —— 找到答案以后容易不容易证明

这是非常关键、但很多人忽略的维度。

比如：

```text
把变量名改一下
```

verification：compile，几秒钟。

但是：

```text
为什么 deployment termination 时最后 1 秒日志丢失？
```

verification：必须构造 termination condition → rollout → observe → repeat → compare。

这就完全不同。

### D. Blast Radius —— 错了以后会不会产生严重后果

例如 rename method 错了是 compile error；修改 transaction / concurrency / deployment lifecycle 错了可能是 production outage。

### E. Reversibility —— 错了容易不容易回来

这是我建议你特别加入的一维。

例如写一个测试很容易回滚，而重构整个 persistence layer 很难回滚。

### 3. 所以真正的“难度”应该改成这个公式

> **Difficulty ≠ Lines of Code**
>
> **Difficulty ≈ Uncertainty × Search Space × Verification Cost × Blast Radius**

而 reasoning effort 大致是在控制：

> **允许 agent 为降低上述不确定性付出多少 inference compute。**

### 4. 你的 Spring Boot / K8s 日志问题，其实很容易把 Sol xHigh “骗进去”

这是一个非常好的例子。

你一开始的直觉是：

> “这个 bug 好奇怪 → 应该很难 → 上 Sol xHigh。”

但 agent 得到的信息实际上可能是：

> “现象很诡异 + 根因未知 + repo 很大 + 我有大量 shell/tool 可以调用 + 我拥有 xHigh reasoning budget。”

于是 agent 非常容易进入：

```text
Hypothesis A
↓
检查 5 个文件
↓
不够确定
↓
Hypothesis B
↓
搜索 Maven
↓
发现另一个 warning
↓
继续验证
↓
扫描 .local
↓
发现另一个 dependency
↓
继续验证
↓
……
```

这不一定是“模型太笨”。

恰恰相反：

> **这是一个很强的 agent 被允许进行无限 exploratory search 后的典型行为。**

它在优化：

> “我要把所有可能性排除。”

而你真正想要的往往是：

> “我要在合理证据下找到最可能的 root cause，然后用一个最小实验确认它。”

这是完全不同的 optimization objective。

### 5. 这也是为什么“更高 reasoning”有时候反而更差

Agent coding 和传统 benchmark 最大的区别之一：

Benchmark 往往是：

```text
Problem
↓
Thinking longer
↓
Better answer
```

Agent coding 更像：

```text
Problem
↓
Hypothesis
↓
Tool call
↓
New evidence
↓
Hypothesis update
↓
Tool call
↓
Modify code
↓
Test
↓
Observe
↓
Stop
```

所以：

> **reasoning token 并不是唯一 intelligence resource。**

还有：

- tool calls
- context
- repository exploration
- shell execution
- tests
- feedback loops
- human intervention

这也是为什么 coding agent 的 harness 会极大影响模型表现。

### 6. 所以我现在反而不建议你记“任务 → 模型”

我建议建立的是：

> **任务 → uncertainty profile → inference strategy**

一个很实用的 mental model 是：

| Task 类型 | 典型特征 | 我的第一选择 |
|---|---|---|
| Mechanical | 明确、局部、可验证 | Luna Low/Medium |
| Routine coding | 有少量 repo exploration | Luna High / Terra Medium |
| Debugging | root cause 不明确 | Terra High |
| Cross-layer debugging | 多系统、多 hypothesis | Sol Medium / Terra xHigh |
| Architecture | 大量 trade-off | Sol Medium/High |
| Very hard research | 极高 uncertainty + 长链验证 | Sol High/xHigh |
| Production incident | 高 blast radius | Sol + 较高 effort + 强制验证 |

注意：

> **“非常难”并不自动意味着 xHigh。**

真正应该让你往 xHigh 走的是：

> **高 uncertainty + 高 verification cost + 高 blast radius**

而不是：

> “我觉得这个 bug 很难。”

### 7. 你圈出来的四个点到底怎么选？

**Sol Low**：强模型 + 不让它过度思考。适合有一定 ambiguity，但希望 agent 快一点。是一个非常容易被低估的配置。

**Terra High**：更便宜、更长时间、更充分探索。适合“我知道这个问题不简单，而且让 agent 搜索一会儿没关系”。

**Terra xHigh**：让 Terra 获得更大的 test-time compute，而不是“我需要一个更聪明的模型”。

**Sol Medium**：我真的希望 agent 有很强的基础能力，但又不想把它释放成一个无限探索器。这可能是一个值得重点关注的黄金档。

### 8. 这产生了一个二维图

不要画：

```text
简单 → 困难
```

而应该想象：

```text
                     uncertainty
                          ↑
                          │
             Sol H/xH     │
                          │
             Sol M        │
                          │
        Terra H           │
                          │
   Luna M / Terra M       │
                          │
 Luna Low                 │
                          └──────────────→ verification / search space
```

也就是说：

> **reasoning effort 应该主要跟 uncertainty / search space / verification cost 走。**

而不是跟“代码多少行”走。

### 9. “打断 agent”的直觉是对的

你说：

> “有时我甚至觉得我应该打断它。”

我认为你应该。

真正优秀的 agent user，不是能提前选择 perfect model，而是知道什么时候模型的 search trajectory 已经失控。

例如：已经修改代码、测试通过，却仍继续搜索、检查 unrelated dependency、扫描 `.local`、继续验证——这时候 marginal information gain 已经接近 0。

如果下一次 tool call 能增加的 useful information 很低，就应该 Stop。

### 10. Agent Operator

传统 programmer：

```text
Human
  ↓
Code
```

Agent programmer：

```text
Human
  ↓
Task formulation
  ↓
Model selection
  ↓
Reasoning configuration
  ↓
Harness
  ↓
Agent
  ↓
Tool execution
  ↓
Observation
  ↓
Human intervention
```

所以未来优秀 developer 的能力组成会逐渐变成：

```text
Coding skill
+
System understanding
+
Agent understanding
+
Task decomposition
+
Model selection
+
Stopping judgment
```

这就是所谓的“品味”。

### 11. 但这种“品味”未必应该永远由用户承担

拿真实工作中的 20～50 个 task，记录：

```text
Task
Model
Reasoning
Harness
Tools
Time
Token
Interventions
Result
```

尤其记录：

```text
Was higher reasoning actually useful?
Where did agent waste time?
When did I intervene?
What would I choose next time?
```

几周以后可以形成自己的 policy。

但随后讨论出现了更重要的产品问题：**这种判断本身为什么必须由终端用户承担？**

---

## User

你说的太多了！几乎淹没了本质问题：1.不从参数角度cost，time来说，Luna，terra，sol各自intelligence index的得分差异代表着什么？比如Luna medium得分比sol low低好多，这是否意味着Luna medium就是存在实打实的智能差异，即这种配置下的Luna就是会有问题发现不了，或者胡说，产生幻觉等问题，儿slo虽然是在low模式下也会好很多！如果是，这意味着什么？或者说Luna和slo模型在本质上就是智能的差异吗？

---

## Assistant

对，你这个问题应该把 **cost/time 全部拿掉**，只讨论一个核心：

> **Luna Medium 和 Sol Low 的 Intelligence Index 差很多，是否意味着 Sol 在“本质智能”上就是比 Luna 高一个档次，即使 Sol 只开 Low？**

**答案是：基本可以这么理解，但要非常准确地理解“智能差异”是什么。**

Artificial Analysis 的 Intelligence Index 是跨一组 benchmark tasks 的综合能力指标。因此，如果图里 **Sol Low ≈ 50+，而 Luna Medium ≈ 39 左右**，这不是“Sol Low 只是多想了一点点”的结果。

它说明：

> **在这些测试任务上，Sol Low 能稳定解决一批 Luna Medium 更容易失败的问题。**

而且这种差异不是简单的 reasoning token 数量能够解释的。

### 为什么会这样？

可以把模型能力粗略拆成：

**模型本身的基础能力 × test-time reasoning**

Luna 和 Sol 是两个不同的模型/模型档位，所以：

```text
Luna
  └── Low → Medium → High → xHigh
```

是在**同一个能力底座上增加推理计算**。

而：

```text
Luna → Terra → Sol
```

首先改变的是**模型本身的能力底座**。

因此：

```text
Sol Low
```

虽然 reasoning budget 很低，但它仍然拥有 Sol 的：

- 更强的问题理解
- 更强的知识/模式识别
- 更强的代码能力
- 更强的复杂推理能力
- 更好的错误识别能力
- 更强的长链任务能力

所以它完全可能出现：

> **Sol Low > Luna Medium**

甚至在某些任务上：

> **Sol Low > Luna High**

这并不矛盾。

### 一个非常重要的区别

假设一个任务是：

> “这个 bug 的真正 root cause 是什么？”

Luna Medium 可能：

```text
理解问题
→ 找到 hypothesis A
→ A 看起来合理
→ 给出答案
```

而 Sol Low 可能：

```text
理解问题
→ 发现 A 有一个 subtle contradiction
→ 排除 A
→ 找到 B
→ 给出答案
```

这里 Sol 并不一定是因为“思考了更多”。

而是：

> **它在较短 reasoning budget 内，就具有更好的 discrimination ability。**

也就是**更会判断哪些东西值得想、哪些 hypothesis 不成立**。

这点对 agent 特别重要。

### 所以 Luna → Terra → Sol 更像“智力底座”的变化

而 Low → Medium → High → xHigh 更像：

> **给这个智力底座多少 test-time compute，让它有多少机会把自己的能力发挥出来。**

现实不是这么干净的二维分解，但作为 mental model 很有用。

因此，如果你的观察是：

> **“为什么 Sol Low 看起来已经明显比 Luna Medium 聪明？”**

**是的，这个观察基本成立。**

而这也意味着你之前那个问题——**“为什么不直接所有事情都 Sol Low？”**——其实才是接下来真正值得研究的问题。这里就会涉及 **Sol Low vs Terra High vs Sol Medium 到底在什么类型的真实 coding task 上发生 capability crossover**，这比单纯看 Intelligence Index 排名有意思得多。

---

## User

对，这就是我真正想问的问题。“为什么不直接所有事情都 Sol Low？”——其实才是接下来真正值得研究的问题。这里就会涉及 Sol Low vs Terra High vs Sol Medium 到底在什么类型的真实 coding task 上发生 capability crossover是什么？作为终端用户，到底应该怎么理解这里的gap呢？

---

## Assistant

对。现在真正的问题可以压缩成一个非常清晰的模型：

> **Sol Low、Terra High、Sol Medium 的差别，不是“谁更聪明”这么简单，而是：到底是“能力底座不够”，还是“能力够但没给足推理预算”。**

### 1. 两种完全不同的 gap

把一个模型解决任务的能力想成：

**最终能达到的能力上限 × 当前实际发挥程度**

那么：

- **Luna → Terra → Sol**：主要改变的是 **能力底座 / capability**
- **Low → Medium → High → xHigh**：主要改变的是 **test-time compute / ability to realize that capability**

所以：

> **Sol Low 的优势，是它的“能力地板”已经很高。**
>
> **Sol Medium/High 的优势，则是允许这个高能力底座进一步发挥。**

### 2. 为什么不永远 Sol Low？

因为 **Sol Low 可能“知道怎么解决”，但没有充分把这个能力展开出来。**

例如：

> “找出这个分布式系统偶发数据丢失的根因。”

Sol Low 可能已经有能力意识到：

> “这里应该重点怀疑 A/B/C。”

但 Low 下它可能：

```text
A 看起来合理
→ 给出答案
```

而 Sol Medium：

```text
A
→ 检查证据
→ 发现 A 与现象 X 矛盾
→ 排除 A
→ 检查 B
→ 找到 B
→ 验证
```

所以这里不是 **Sol Medium 比 Sol Low 更“聪明”**。

而是：

> **Sol Low 已经具备解决问题的能力，但没有把搜索空间充分展开。**

这就是 **reasoning gap**。

### 3. 反过来，Terra High 为什么可能打败 Sol Low？

这正是 **capability crossover**。

假设任务需要：

> 很深的代码理解 + 一个非常 subtle 的 insight。

可能出现：

```text
Terra High
████████████████░░░░
Sol Low
███████████████████░
```

Terra 虽然底座低一点，但给它更多 compute 后，可以：

> **靠更长的搜索，把一部分能力差距补回来。**

但这种补偿不是无限的。

最终可能出现：

```text
简单任务：
Terra High ≈ Sol Low

复杂推理：
Terra High < Sol Low

非常深的任务：
Terra High << Sol Low
```

这就是 **capability ceiling**。

### 4. 三种情况

| 情况 | 典型表现 | 应该怎么升档 |
|---|---|---|
| **模型能力不够** | 根本没发现关键问题、错误理解、错误方向 | **Terra → Sol** |
| **模型能力够，但没想够** | 已经找到正确方向，但验证/规划不充分 | **Low → Medium/High** |
| **已经陷入无效搜索** | 不断检查边缘问题、重复验证、scope 扩张 | **不要升，应该打断/降档** |

第三种恰恰就是之前的 **Sol xHigh + K8s 日志问题**。

### 5. Sol Low 的真正位置

我会把它理解成：

> **“高能力模型的快速模式。”**

它特别适合：

> **我希望模型本身足够聪明，但我不希望它花大量时间证明自己。**

而：

**Sol Medium**：我认为这个问题值得让高能力模型认真展开一次。

**Terra High**：这个问题本身未必需要 Sol 的能力底座，我愿意让 Terra 多花一点时间把能力发挥出来。

### 6. Artificial Analysis 的 Intelligence Index 不应被理解成同一能力

假设：

```text
Sol Low       53
Terra High    53
```

不能推出“两者 intelligence 一样”。

正确理解应该是：

> **在 AA 的测试任务分布上，两者最终成功率/得分的平均结果相近。**

真实任务可能是：

```text
Task A：Terra High = 95%，Sol Low = 95%
Task B：Terra High = 80%，Sol Low = 95%
Task C：Terra High = 95%，Sol Low = 75%
Task D：Terra High = 60%，Sol Low = 90%
```

平均以后 53 ≈ 53，但能力分布完全不同。

### 7. 真正的诊断思维

```text
              Agent失败
                  │
          ┌───────┴───────┐
          │               │
     没找到问题        找到了问题
          │               │
     Capability gap    Reasoning gap
          │               │
       换更强模型       增加 thinking
          │               │
     Terra → Sol      Low → Medium
```

以及第三条：

```text
       Agent已经找到答案
               │
       但还在无效探索
               │
          Search runaway
               │
            打断它
```

核心是判断：

> **“现在缺的是脑子，还是时间？”**

以及更重要的第三种：

> **“其实什么都不缺，它只是已经在浪费时间了。”**

---

## User

但这不就跟汽车刚发明的时候使用手动挡一个道理，其实都是对终端用户的心智负担，全自动必然是未来趋势！我无法想象，面对一个相对复杂的问题就需要使用者在Codex或claude code cli里切换数次model和thinking effort将是多大的折磨！

---

## Assistant

对，而且我认为这个类比**基本击中了未来 agent 产品真正要解决的问题**。

但汽车的类比还可以再往前推一步：

> **现在的 model + reasoning effort 选择，很可能确实类似早期汽车的手动挡。**
>
> 用户现在被迫充当一个很原始的 **inference scheduler**。

你不应该需要知道：

> “这个任务是 capability gap 还是 reasoning gap？”

这应该是 **Codex / Claude Code 自己判断的**。

理想状态应该是：

```text
Human
  ↓
“解决这个问题”
  ↓
Agent
  ├─ 先用便宜 inference
  ├─ 发现 uncertainty ↑
  ├─ 自动增加 reasoning
  ├─ 发现 capability 不够
  ├─ 自动升级 model
  ├─ 调工具验证
  ├─ 发现进入无效搜索
  ├─ 自动停止
  ↓
Answer
```

也就是说：

> **Model selection + reasoning allocation 本身应该成为 agent loop 的一部分。**

### 这是未来 coding agent 一个非常重要的竞争维度

未来真正成熟的 harness，不应该只是“帮模型调用 shell / git / browser”，而应该具备一个 **Inference Controller**：

```text
                 Task
                   ↓
          ┌────────────────┐
          │ Inference      │
          │ Controller     │
          └───────┬────────┘
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
    Model      Reasoning    Tools
    choice       budget
       │          │
       └──────────┼──────────┘
                  ↓
               Agent
                  ↓
             Observation
                  ↓
          Controller again
```

它甚至应该动态决定：

> Luna Low → Terra Medium → Sol Low → Sol Medium

而用户完全不需要看到这些。

### 为什么现在还没有完全做到？

因为这个 controller 本身非常难。

它必须预测：

> **“再花 $0.20 / 30 秒，到底能不能显著提高成功概率？”**

这其实是一个很难的 meta-reasoning 问题。

而且它必须解决：

> **什么时候升级 model，什么时候增加 reasoning，什么时候调用更多 tools，什么时候直接问用户？**

这已经不是传统 LLM inference 了。

实际上已经开始接近 **Agent Runtime / Inference OS**。

### 更准确的类比：CPU scheduling

我觉得这个比汽车更准确。

你不会告诉操作系统：

> “这个线程比较重要，所以给它 13.7ms CPU quantum；那个线程 complexity 较高，所以换一个更大的 core。”

OS 自己做：

```text
task
 ↓
scheduler
 ↓
CPU allocation
 ↓
observe
 ↓
reschedule
```

Agent 未来应该也是：

```text
task
 ↓
inference scheduler
 ↓
model / reasoning / tools allocation
 ↓
observe
 ↓
reschedule
```

最终用户看到的应该只有“Fast / Balanced / Deep”，甚至连这个都不应该需要。

### 为什么这个 controller 难

因为 router 自己也是一个会犯错的模型。

例如：

```text
真正需要 Sol
      ↓
Router 判断“简单”
      ↓
Luna
      ↓
失败
```

或者：

```text
真正只需要 Luna
      ↓
Router 过度保守
      ↓
Sol xHigh
      ↓
任务成功
```

所以 router 面临的是：

> **False Cheap vs False Expensive**

而且两者代价并不对称。

更麻烦的是，任务开始之前通常不知道任务到底难在哪里。

所以最好的 router 根本不能只看最初 prompt，而必须持续观察 trajectory，然后动态调整。

---

## User

这不会是我一个人的疑问和痛点。必然有很多人表达过或注意到过这个事情了，也必然有很多人尝试或者开始去解决这些问题了。比如manus好像是自动挡的，cursor还是openrouter好像是支持某种自动切换的，但我疑惑的是这个问题居然仍然是Codex和Claude code这种上亿用户级别的产品的问题。这说明就像你说的，“这其实是一个非常难的 meta-reasoning 问题”。那业界当前的state是如何的？学术界方向的反馈呢？

---

## Assistant

是的，而且我查完截至 **2026 年 8 月**的业界和学术进展后，我会把现在的状态概括成一句话：

> **这个问题已经被业界和学术界明确识别为一个独立问题，而且正在快速解决；但目前已经有“自动变速箱”的第一代产品，还远没有达到人类可以完全放心地把挡位选择交出去的程度。**

更有意思的是：**你今天在 Codex / Claude Code 里感受到的痛点，恰好就是这个研究方向正在攻的核心问题。**

### 先看业界：已经开始真正自动挡了

**Cursor 是目前我看到最明确的产品化答案。**

2026 年 7 月刚发布的 Cursor Router，会在每个 request 执行前根据 query、context、task complexity、domain 等信息进行分类，然后自动选择模型；它甚至有 Intelligence / Balance / Cost 三个不同的 routing objective。

**Claude Code 其实也已经在往“自动挡”走，但还不是你想象的那个层次。**

Claude Code 目前有一个很有意思的 `opusplan`：

> **Plan 阶段自动 Opus → Execution 阶段自动 Sonnet。**

也就是说 Anthropic 已经在做：

> **根据 agent 当前所处阶段动态选择模型。**

同时 Claude 的 effort 已经从传统固定 thinking budget 演进到 **adaptive thinking + effort**：模型可以自己决定什么时候需要思考、思考多久。

所以 Claude Code 并不是完全没有自动化。

但你指出的问题仍然存在：

> **它主要是“预定义策略 + 用户可调旋钮”，还不是一个真正的全局 cognition scheduler。**

### 为什么这个问题这么难？

以前研究的是：

> **Which model is best for this prompt?**

现在 agent 时代真正的问题变成：

> **Given the current state of a long-running trajectory, what computational action should I take next?**

这个 current state 包括：

- 当前任务
- 当前代码
- 已经读了什么
- 已经尝试了什么
- 当前 hypothesis
- 工具调用历史
- 测试结果
- 模型刚刚失败在哪里
- 是否应该继续 reasoning
- 是否应该换模型
- 是否应该调用工具
- 是否应该停止

所以它实际上已经不是普通的 **model routing**。

而是：

> **Agent-level adaptive computation / inference scheduling。**

### 学术界已经形成这个方向

2025 年已经有系统性的 survey 把它叫做 **Adaptive / Controllable Test-Time Compute**，明确指出一个核心问题：固定 inference compute 会导致简单问题 overthinking、困难问题 underthinking。

到了 2026 年，研究进一步往动态选择 reasoning effort、model routing、agent/trajectory-level routing 发展。

例如 **Ares**：不再给整个任务固定 Low/Medium/High，而是**每一步动态预测需要多少 reasoning effort**。

它的 router 根据 interaction history 判断当前 step 最低需要的 reasoning level。

### Model routing 也开始有专门 benchmark

**LLMRouterBench** 是 2026 ACL Findings 的大规模 routing benchmark：

- 33 个模型
- 21+ 数据集
- 40 万+ instances
- 10 种 routing algorithms

非常关键的结论是：

> **Oracle 和实际 router 之间仍然存在很大的 gap。**

主要问题之一是 router 经常根本没把正确模型召回出来。

所以：

> **“自动选择模型”本身并没有想象中那么容易。**

### Coding agent 的 routing 又比普通 LLM routing 难一个数量级

普通 routing：

```text
User prompt
   ↓
Router
   ↓
Model
   ↓
Answer
```

Coding agent：

```text
Prompt
 ↓
Model
 ↓
Tool call
 ↓
Observation
 ↓
Model
 ↓
Tool call
 ↓
Observation
 ↓
Model
 ↓
...
```

所以 router 必须解决：

> **下一次 call 用什么模型？**

而不是：

> “整个任务用什么模型？”

**TwinRouterBench** 就明确指出：传统 routing benchmark 基本都是 one-shot，而 coding agent 需要 **step-level routing**；它直接在 SWE-bench trajectory 中研究每一次 LLM call 应该选择什么模型。

### 更前沿的方向：Adaptive Computation

真正前沿的方向正在变成：

> **Adaptive Computation**

也就是：

```text
          Agent state
               ↓
        ┌──────────────┐
        │  Controller  │
        └──────┬───────┘
               │
       ┌───────┼────────┐
       ↓       ↓        ↓
     Model   Thinking  Tool
     change   budget   call
       │       │        │
       └───────┼────────┘
               ↓
             Agent
               ↓
          new state
               ↓
          Controller
```

也就是说 Agent 不仅要解决问题，还要决定“接下来应该用多少计算去解决问题”。

### 为什么 Codex / Claude Code 还没完全解决？

一个重要原因是：router 自己也是一个会犯错的模型。

如果真正需要 Sol，router 判断“简单”而选 Luna，trajectory 可能被污染；反过来，如果简单任务被过度保守地送到 Sol xHigh，又会浪费大量 compute。

而且对于 coding agent，错误 routing 会污染后面的 trajectory。

### 一个很有意思的最新发现

开始有实验系统研究“过度 thinking”。coding-agent benchmark 发现，prompt 中要求“think deeply”“比较多个方案”等，会显著增加 reasoning token，但不一定增加正确率。

而且 harness 本身的影响甚至可能比 prompt 更大：同样的 model + task + prompt，在不同 harness 下，每成功一次的成本可以相差几个数量级。

所以：

> **“更多 thinking”绝对不等于“更好的 reasoning”。**

### 2026 年 8 月的判断

可以粗略分成三个阶段：

**第一代：Manual transmission**

```text
用户选择：
Model
+
Reasoning effort
```

Codex / Claude Code 目前仍主要属于这里。

**第二代：Automatic transmission**

```text
Router
↓
自动选择 model / effort
```

Cursor Router、Claude `opusplan`、Ares 这类东西属于这里。

**第三代：真正的 Cognitive Scheduler**

不是：

> “这个 prompt 应该用哪个 model？”

而是：

> **“在 agent trajectory 的当前 state 下，下一步最值得花什么 intelligence？”**

它可以选择：

```text
Luna Low
   ↓
Luna Medium
   ↓
Terra High
   ↓
Sol Low
   ↓
tool call
   ↓
Sol Medium
   ↓
stop
```

甚至：

```text
Sol
↓
“我不知道”
↓
ask human
```

这才是真正意义上的“自动挡”。

---

## User

我觉得这里还可以讲得更详细一点儿。Cognitive Scheduler的本质问题在于编排不同intelligent level的intelligent source/provider形成最终的intelligence产品或者产出。你说的由于模型自己也可能出错，所以不走让模型持续monitor Task trajectory继而实现adaptive computation的论断我觉得只是目前的权宜之计。另外，模型出错其实没有关系，或者说我们应该design with this，只要模型的调用足够快，足够便宜，模型出一次两次错误，后面发现及时纠正过来就可以了。人类协作不也是这样，没有人可以拿任何问题永远做出perfect的判断的，但我们可以有方法论尽可能降低这一点。这对模型或者agent编排通用适应。

---

## Assistant

对，我觉得你这次把问题又往前推进了一层。你说的比我上一条的 **“trajectory-aware adaptive inference”** 更接近本质。

我上一条有一个隐含前提：**把 scheduler 理解成“一个 controller 监控一个 agent，然后决定下一步怎么做”。**

而你说的其实是更一般的东西：

> **Cognitive Scheduler 本质不是监控 agent，而是编排一个由不同 intelligence source/provider 构成的“智能系统”。**

这两个概念差别非常大。

### 1. Cognitive Scheduler 的真正抽象

不应该是：

```text
Task
 ↓
Agent
 ↓
monitor
 ↓
换模型
```

而应该是：

```text
                    Task
                      ↓
              Cognitive Scheduler
                      ↓
       ┌──────────────┼──────────────┐
       ↓              ↓              ↓
   Cheap/fast      Strong model    Specialist
   intelligence    intelligence    intelligence
       ↓              ↓              ↓
       └──────────────┼──────────────┘
                      ↓
                Evidence / State
                      ↓
              Cognitive Scheduler
                      ↓
                   ...
                      ↓
                 Outcome
```

这里的 **intelligence source** 甚至不一定是模型。

可以是：

- Luna
- Terra
- Sol
- Claude
- Gemini
- DeepSeek
- 一个专门的 code model
- 一个 verifier
- 一个 search engine
- compiler
- test suite
- static analyzer
- browser
- 人类
- 另一个 agent

所以最终产品其实不是：

> **一个模型。**

而是：

> **一个 intelligence composition。**

这实际上比“model routing”高了一个抽象层。

### 2. “模型犯错”根本不是 scheduler 的核心问题

你说：

> 模型出错其实没有关系，应该 design with this。

我非常同意。

如果目标设成：

> **每一次 inference 都必须正确**

那这个系统几乎必然变得昂贵、缓慢、过度保守。

人类协作恰恰不是这样：

```text
A 提出假设
 ↓
B 觉得不对
 ↓
C 做实验
 ↓
发现 A 错
 ↓
修正
```

没人要求 A 第一次就 perfect。

真正重要的是：

> **系统有没有快速发现错误并纠正错误的能力。**

所以 cognitive scheduler 的 objective 不应该是：

> maximize probability(each individual inference is correct)

而应该是：

> **maximize probability(final outcome is correct) under bounded time/compute/cost.**

这是一个非常重要的转变。

### 3. Intelligence 不再是一个 scalar

真正的 agent system 里：

> **intelligence 应该是一组可以被组合的资源。**

例如：

```text
Sol Low
→ 强 reasoning

Luna Low
→ 快速 exploration

Compiler
→ 极强的局部 correctness signal

Test
→ empirical verification

Search
→ external knowledge

Human
→ high-level semantic judgment
```

最终 intelligence 可以来自：

```text
Sol reasoning
+
Luna exploration
+
compiler verification
+
test feedback
+
search
```

五个东西组合起来，可能远远超过 Sol xHigh 单模型硬想。

### 4. 最强模型不等于最强 agent

假设：

```text
Sol xHigh
```

本身 benchmark intelligence 极高，但可能：

```text
想错了
↓
继续想
↓
建立在错误假设上继续推理
↓
越来越确信
```

而另一个系统：

```text
Luna Low
↓
提出 hypothesis A
↓
Test
↓
失败
↓
Terra Medium
↓
重新分析
↓
Compiler
↓
失败
↓
Sol Low
↓
解决
```

最终后者可能明显更可靠。

因此：

> **Agent intelligence ≠ model intelligence**

更准确地说：

> **Agent intelligence 是 intelligence sources + orchestration + feedback + verification 的系统属性。**

### 5. “自动换模型”只是最初级的 Cognitive Scheduling

现在很多产品说“我们有 model router”，其实比较初级，因为它解决的是：

```text
Task → Model
```

真正成熟的 scheduler 应该解决：

```text
Task
 ↓
What intelligence do I need?
 ↓
Which source?
 ↓
How much?
 ↓
In what order?
 ↓
What feedback should I obtain?
 ↓
Did it work?
 ↓
What intelligence should I invoke next?
 ↓
Stop?
```

所以真正的问题不是：

> **Model Selection**

而是：

> **Intelligence Allocation / Composition**

### 6. “便宜、快速、允许犯错”会导致完全不同的 architecture

例如：

```text
Sol xHigh
$1 / 30 seconds / 95% success
```

和：

```text
Luna Low      $0.05 / 1 sec
↓
Verifier      $0.01 / 0.2 sec
↓
Terra Medium  $0.15 / 5 sec
↓
Test          $0 / 2 sec
↓
Sol Low       $0.20 / 4 sec
```

即使前面 Luna 犯了一次错，整个 system 仍然可能比 Sol xHigh 更快、更便宜、更可靠。

因为利用的是：

> **cheap mistakes + fast correction**

而不是：

> **expensive perfection attempt**

### 7. 人类团队是很好的 analogy

一个 10 人团队不会让最聪明的 Staff Engineer 独自思考 8 小时直到 100% 确定，而是：

```text
Junior
↓
快速调查

Senior
↓
review

Specialist
↓
解决局部问题

Test / experiment
↓
验证

Staff
↓
最终架构判断
```

没有要求每个人 perfect。

团队能力来自：

> **角色分工 + feedback + escalation。**

Agent system 完全可以复制这个结构。

### 8. Cognitive Scheduler 的定义

> **“在给定任务、状态、时间、成本和可靠性目标下，动态编排可用 intelligence sources，使最终 outcome 的 expected utility 最大化。”**

其中包括：

- Intelligence source：谁来贡献 intelligence？
- Allocation：给谁多少 compute？
- Sequencing：先谁、后谁？
- Feedback：什么结果可以改变下一步决策？
- Escalation：什么时候引入更强 intelligence？
- Verification：什么时候应该调用一个独立 source 验证？
- Termination：什么时候已经足够好了？

### 9. 行业为什么还像 model routing

因为现在行业主要在做：

> **Which model?**

而这里真正问的是：

> **How should intelligence itself be orchestrated?**

可以粗略分成：

```text
Level 1
Model
  ↓
Inference

Level 2
Model + reasoning
  ↓
Adaptive inference

Level 3
Multiple models
  ↓
Model routing

Level 4
Multiple intelligence sources
  ↓
Cognitive scheduling

Level 5
Self-organizing intelligence system
  ↓
Dynamic composition of reasoning,
tools, verification, models, humans
```

我认为现在行业大体处于 Level 2 → Level 3，而这里讨论的东西明显是在讨论 Level 4。

### 10. 对 oh-my-agentX 的意义

这已经与 Agent Runtime / Inference OS 非常接近：未来 runtime 最重要的能力之一，可能根本不是“把模型接进来”，而是决定每一秒钟到底应该给 agent 多少 intelligence。

**未来 runtime 最重要的能力之一，就是 Intelligence Scheduling / Composition。**
