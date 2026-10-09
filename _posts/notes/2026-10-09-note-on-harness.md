---
layout: post
title: "Notes on Agent Harness"
date: 2026-10-09
categories: [Notes]
tags: [agent,harness]
math: true
---

# Agent Harness 学习指南

---

## 目录

- [0. 一分钟定位：harness 是什么](#0-一分钟定位harness-是什么)
- [1. 接口层：模型怎么表达「我要调工具」](#1-接口层模型怎么表达我要调工具)
- [2. 工具从哪来：三种来源](#2-工具从哪来三种来源)
- [3. 执行通道：工具在哪跑](#3-执行通道工具在哪跑)
- [4. 校验与执行都在 harness 侧](#4-校验与执行都在-harness-侧)
- [5. 环境层：从「工具」升级到「环境」](#5-环境层从工具升级到环境)
- [6. 生态地图：现成项目怎么选](#6-生态地图现成项目怎么选)
- [7. 最小实现：一份 registry 喂三个 harness](#7-最小实现一份-registry-喂三个-harness)
- [8. 进阶：还没覆盖但同样重要的六件事](#8-进阶还没覆盖但同样重要的六件事)
- [9. 常见误解 FAQ](#9-常见误解-faq)

---

## 0. 一分钟定位：harness 是什么

**一句话：模型只会「说」，harness 负责「听懂 → 校验 → 执行 → 回写 → 打分」。**

模型一次推理的全部产出就是一段文本。它**永远不会自己**打开文件、调 API、跑代码。让一段文本变成一次真实副作用（读文件、检索、执行脚本），中间的整套机制就是 harness。

### 三层坐标

| 层             | 干什么                     | 典型项目          |
| ------------- | ----------------------- | ------------- |
| **模型服务层**     | 给一段 token 序列，吐下一段 token | vLLM、SGLang   |
| **Harness 层** | 工具定义、校验、执行、上下文组装、奖励计算   | **本指南主题**     |
| **训练器层**      | 消费轨迹，算 loss，更新参数        | verl、prime-rl |

关键认知：**harness 和训练器是可分离的**。你可以只用 harness 收集轨迹而不训练；也可以换训练器而复用同一套环境。生态里绝大多数混乱，来自把这两层（甚至三层）混着谈。

### 一个最小 harness 循环只有 5 步

```python
messages = [{"role": "system", ...}, {"role": "user", "content": task}]
tools    = REGISTRY.openai_tools()          # ① 声明
for turn in range(max_turns):
    msg = llm(messages, tools=tools)        # ② 模型只输出请求文本
    messages.append(msg)
    if not msg.tool_calls: break
    for tc in msg.tool_calls:               # ③ 校验 + ④ 执行（都在 harness）
        r = REGISTRY.call(tc.function.name, json.loads(tc.function.arguments))
        messages.append(r.tool_message(tc.id))   # ⑤ 回写，进下一轮
```

就这么多。后面几千字全是在解释这 5 步里每一步有哪些坑、哪些现成实现可以抄。

---

## 1. 接口层：模型怎么表达「我要调工具」

**一句话：OpenAI 的 `tools` / `tool_calls` / `role="tool"` 是事实标准，各家模型底层格式不同但被 tool parser 抹平。**

### 1.1 标准形态

- 请求侧：`tools` 数组，每项 `{"type": "function", "function": {name, description, parameters}}`，`parameters` 是 JSON Schema。
- 响应侧：`message.tool_calls`，每项有 `id` + `function.name` + `function.arguments`。
- 回写侧：一条 `{"role": "tool", "tool_call_id": ..., "content": ...}` 消息。

vLLM / SGLang / TGI 全实现了这套 wire 格式，所以**你写的 schema 换个推理后端也能直接用**——这是整个栈里唯一真正稳定的契约。

### 1.2 Qwen3 实际吐的什么（源码实测）

Qwen3 能用 OpenAI 的 `tools` schema，但模型底层吐的是 **Hermes 风格 XML，包在 ChatML 里**（源自 NousResearch Hermes 2 Pro，Qwen2.5 / Qwen3 沿用）。实测 `Qwen/Qwen3-8B` 的 `tokenizer_config.json`：

| token                                  | id              | special   |
| -------------------------------------- | --------------- | --------- |
| `<\|im_start\|>` / `<\|im_end\|>`      | 151644 / 151645 | True      |
| `<tool_call>` / `</tool_call>`         | 151657 / 151658 | **False** |
| `<tool_response>` / `</tool_response>` | 151665 / 151666 | **False** |
| `<think>` / `</think>`                 | 151667 / 151668 | **False** |

三个反直觉点，直接决定你的循环怎么写：

1. **工具结果是 `user` 角色，不是 `tool` 角色。** Qwen3 模板把 `role=="tool"` 渲染成 `<|im_start|>user\n<tool_response>…\n</tool_response><|im_end|>`。手写拼接时别自己造 tool turn。
2. **`arguments` 是嵌套 JSON 对象，不是 OpenAI 的字符串。** 两边互转最容易出双编码 bug。
3. **这些标记 `special=False`**——`skip_special_tokens=True` 剥不掉它们，所以 vLLM 的 hermes parser 才能从解码文本里正则恢复。对做 token-in/token-out 是好事。

部署参数：`vllm serve ... --enable-auto-tool-choice --tool-call-parser hermes`（Qwen 官方 `function_call.md` 指定，要求 vllm ≥ 0.8.5）。

### 1.3 并行调用没有 id，靠顺序配对

**Qwen3 的模型输出层面根本没有 tool-call id。**

- 模板里 `'tool_call_id'` 一次都没出现；渲染 `role=="tool"` 时**只读 content**，不看 id 也不看 name。
- 并行就是同一 assistant turn 内连续写 N 个 `<tool_call>…</tool_call>`，**第 i 个 `<tool_response>` 对应第 i 个 `<tool_call>`**，唯一对齐依据是位置。
- vLLM 返回给你的 id 是服务端后补的：parser 构造 `ToolCall(...)` 时没有 id，由 `serving.py` 兜底 `ToolCall(id=tc.id or make_tool_call_id(), ...)`，默认生成 `chatcmpl-tool-{random_uuid()}`。**这些 id 从不进 prompt**。

**硬约束**：N 个 call 必须紧跟 N 个 response，顺序严格一致；工具超时/失败也要补占位 `<tool_response>error: timeout</tool_response>`，少一个就整体错位——这是**不出 traceback 的静默错误**。

> 建议：**直接禁用并行**。每轮只发一个 `<tool_call>`。Search-R1、ReAct 都是这个形态，对 RL 更友好（action space 更小、credit assignment 更干净、rollout 更好 cache）。

### 1.4 token 保真：为什么不能事后重新 tokenize

vLLM 官方 blog（2025-10-22，与 Agent-Lightning 合作）列了漂移三来源，其中一条就是 **tool-call 序列化重渲染**：parser 把模型吐的 JSON 解析成对象再渲染回文本，空白/格式会被改写甚至自动纠错。**事后拿消息列表重新 tokenize 补不回来**。

所以要做 token-in/token-out：

- vLLM ≥ 0.10.2 起，`/v1/chat/completions` 支持 `return_token_ids: true`，响应多出 `prompt_token_ids` 与 `token_ids`。
- 高阶封装（如 `openai-agents` 的 `ModelResponse`）**不携带**这些字段——要做零漂移，只有自己写循环这一条路。
- verl 文档明确要求 token-in/token-out，实测 decode→encode 不是单射，会导致单轮 PPO 都不收敛。

---

## 2. 工具从哪来：三种来源

**一句话：只有「内置原生工具」和「MCP」会产生 tool call；Skill 不产生任何工具条目，它是按需加载的说明书。**

Claude Code 官方文档原话：

> To add custom tools, connect an MCP server. To extend Claude with reusable prompt-based workflows, write a skill, which **runs through the existing `Skill` tool rather than adding a new tool entry**.

| 来源             | 形态                          | 进 `tools` 数组？ | 例子                                                                                    |
| -------------- | --------------------------- | ------------- | ------------------------------------------------------------------------------------- |
| **内置原生工具**     | 客户端代码里的函数                   | ✅             | Claude Code 的 `Read`/`Grep`/`Glob`/`Bash`/`Edit`/`Write`/`WebFetch`；你自己写的 `read_file` |
| **MCP server** | 常驻进程（stdio / HTTP），JSON-RPC | ✅             | `mcp__<server>__<tool>`；官方 filesystem / fetch server                                  |
| **Skill**      | 一个目录：`SKILL.md` + 可选脚本      | ❌             | 经内置 `Skill` 工具触发，只把指令注入上下文                                                            |

内置原生工具和 MCP 都 ✅，那它们进数组后长得一样吗、schema 由谁提供——见 2.3。

### 2.1 Skill 的三个特征

1. **渐进式披露**：常驻只有 name + description（几十 token），命中后才加载正文，且正文作为单条消息**在后续轮次常驻**（不会每轮重读）。
2. **不执行任何东西**。官方 20 个 skill（pdf / xlsx / pptx / docx …）**无一使用 `allowed-tools`**，真实做法是正文里写命令让模型用 Bash 跑，例如 docx skill 的 `python scripts/office/validate.py out.docx --original doc.docx`。
3. **`allowed-tools` 只是当轮免弹窗授权，不限制**；真要移除工具得用 `disallowed-tools`。

### 2.2 那 filesystem 为什么是 MCP？

因为它需要 **typed schema + 服务端强制的沙盒边界**（allowed directories 是进程内路径校验，Skill 里写一句「别出这个目录」改不了模型行为），且要跨客户端复用。

**但反例同样硬**：如果 agent 有 shell，`cat` / `grep -r` 就够了（mini-swe-agent、OpenHands 完全不用 MCP）。Claude Code 自己也在 macOS/Linux/WSL 上**默认不提供 `Glob`/`Grep`**，直接让模型用 Bash 的 grep。

> 真正的判据不是「filesystem 天然属于 MCP」，而是**你要不要那个结构化边界 + 跨客户端复用**。自己写 harness 且工具只有几个时，直接写函数进 `tools` 数组是最优解。

### 2.3 MCP 工具的 schema 也要进 tools 数组吗？

**要「进」数组，但通常不是以 schema 的形式进。** 一共三种声明形态（2026-10-09 官方文档实测）：

| 形态                      | `tools` 数组里写什么                                                                                                                                                                                                                              | schema 谁提供            | 谁执行调用   |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------- | ------- |
| **A 自写 function**       | `{"type":"function","function":{"name":...,"description":...,"parameters":{...}}}`                                                                                                                                                          | **你（client 侧手写）**     | 你       |
| **B MCP → function 转换** | 同 A，但内容是运行时从 `tools/list` 拉下来转成的                                                                                                                                                                                                            | **MCP server（运行时发现）** | 你       |
| **C 托管 MCP（hosted）**    | OpenAI：`{"type":"mcp","server_label":...,"server_url":...,"allowed_tools":[...],"require_approval":...}`<br />Anthropic：`mcp_servers` 数组 + `tools` 里的 `{"type":"mcp_toolset","mcp_server_name":...,"default_config":{...},"configs":{...}}` | **API 服务端自己拉**        | API 服务端 |

C 形态下 `tools` 数组里装的&#x662F;**「引用 + 配置」，不是 schema**——schema 由服务端在运行时去找 server 要，再注入上下文。Anthropic 甚至专门出了一个 beta（`mcp-client-2026-09-15` + `mcp_tool_listing` 块）来**把拉到的工具列表 pin 住**，因为不 pin 的话 server 中途改工具会改变模型看到的东西——这跟 8.2 的确定性是同一个问题。

#### 格式差异：MCP Tool vs OpenAI function

MCP 的 `Tool` 是**扁平结构**（`schema/2026-07-28/schema.ts` 实测）：

```jsonc
{ "name": "read_file",
  "title": "...",            // 可选
  "description": "...",
  "inputSchema": { "type": "object", ... },   // JSON Schema 2020-12，根必须 object
  "outputSchema": { ... },   // 可选
  "annotations": { "readOnlyHint": true, "destructiveHint": false, ... } }  // 可选，全为 hints
```

OpenAI 的 function 是**包一层**的，而且字段改名：

```jsonc
{ "type": "function",
  "function": { "name": "read_file",
                "description": "...",
                "parameters": { "type": "object", ... },   // ← inputSchema
                "strict": false } }                        // ← 无 outputSchema / annotations
```

#### 三条实操结论

1. **自托管 vLLM / Qwen3 只能走 B**。vLLM 官方 `docs/features/tool_calling.md` 全文无 MCP 字样（实测），`/v1/chat/completions` 没有托管 MCP 发现——你必须自己发 `tools/list` 再转成 function。（vLLM 另有 `--tool-server` 支持 MCP 的说法来自第三方 wiki，本指南未一手核实，别依赖。）
2. **名称不变、服务端用独立字段标记来源**。Anthropic 的 `mcp_tool_use` 块里 `name` 是原始工具名，`server_name` 是单独字段——不在 name 上做前缀。（Claude Code 客户端侧常见的 `mcp__<server>__<tool>` 前缀是客户端约定，本指南未从官方 tools.md 核实到。）
3. **对模型来说三种形态完全等价**。最后都会被渲染成同一段 ChatML 的 `<tool_calls>`（Qwen3）或 `tool_use` 块（Claude），模型分不清工具来自哪。差别全在工程侧：谁维护 schema、谁控制 token 占用、谁执行。

---

## 3. 执行通道：工具在哪跑

**一句话：接口维度（typed tool vs Bash）和实现维度（进程内 / 子进程 / 常驻 HTTP / 容器）是两个正交维度，别混。**

### 3.1 四档

| 档           | 生命周期     | 适用                                           | 例子                                      |
| ----------- | -------- | -------------------------------------------- | --------------------------------------- |
| **进程内**     | 函数调用，同进程 | 只读 / 纯函数 / 毫秒级 / 可信                          | `read_file`、`search_files`、`list_files` |
| **本地子进程**   | 一次调用一个进程 | 慢（秒级）、**会崩 / 死循环**、依赖外部二进制（rg / fd / pandoc） | 大语料 grep、PDF 抽取                         |
| **常驻 HTTP** | 长命服务     | **有状态**（索引常驻内存）、独立 GPU、多 rollout 共享          | FAISS / BM25 检索服务                       |
| **容器 / 远端** | 隔离沙盒     | 不可信代码、需要网络/文件/凭据隔离                           | Docker、Modal、K8s、Gondolin 微 VM          |


### 3.2 一条必须纠正的直觉

很多人以为「进程内也能用线程池或 `signal.alarm` 做硬超时」——**不能**：

- Python 线程无法被安全 kill，`future.result(timeout=)` 只是**不再等待**，线程仍在跑（占 CPU、泄漏）。
- `signal.alarm` 只在主线程生效，对阻塞在 C 扩展里的调用无效。

**所以能真正「杀掉」的硬超时只有 subprocess 一条路。** 这是把工具划到子进程档的**首要理由**——不是因为慢，而是「不能让一个失控调用拖死整个 rollout 进程」。进程内只能用 `try/except` 兜异常，兜不住死循环。

### 3.3 MCP 不是 subprocess 的替代品

MCP 是**通信协议**，subprocess 是**运行时档位**。而且 MCP 的默认形态恰恰就是子进程——官方 transports 页原话：stdio 是 *"newline-delimited messages over the standard streams of a **client-launched subprocess**"*；同时 FastMCP 支持 **in-memory transport**，同进程、无子进程、无网络。

**真正的分界线是 one-shot vs 常驻**：`subprocess.run` 跑一次脚本是典型 one-shot，恰恰是 MCP 最不擅长的用法；MCP 的主场是「一个有状态的长命服务」。

**MCP 的价值是纯粹的 N×M 问题**：N 个 harness × M 个工具，没有协议要写 N×M 份胶水；有了协议各实现一次，N+M。你只有 1 个 harness × 4 个工具时，协议收益接近于零。

### 3.4 MCP 的规范要点（schema 2026-07-28 实测）

- **`Tool`**：`name` + `description` + `inputSchema`（JSON Schema 2020-12，根必须 `type:"object"`）+ 可选 `outputSchema` + `annotations`（`readOnlyHint` / `destructiveHint` / `idempotentHint` / `openWorldHint`）。注意官方注释：这些**全是 hints**，"Clients should never make tool use decisions based on ToolAnnotations received from untrusted servers"。
- **`tools/list`** → `ListToolsResult extends PaginatedResult, CacheableResult`（支持 cursor 分页 + 缓存 TTL）。
- **`tools/call`** → `CallToolResult { content: ContentBlock[]; structuredContent?; isError? }`。
- **`isError` 那条原话值得直接抄进自己的 harness**：
  > Any errors that originate from the tool SHOULD be reported inside the result object, with `isError` set to true, **not** as an MCP protocol-level error response. Otherwise, the LLM would not be able to see that an error occurred and self-correct.

**三角色**：host（应用，如 Claude Code / 你的 harness）持有完整对话；client 由 host 创建，与 server **1:1**；server 由能力/服务方提供。**server 读不到完整对话，也看不进其他 server**。官方 reference server 只有 7 个（Everything / Fetch / Filesystem / Git / Memory / Sequential Thinking / Time），且 README 明确警告这些是 *"educational examples … **not as production-ready solutions**"*。

---

## 4. 校验与执行都在 harness 侧

**一句话：模型只产生请求文本，校验与执行的控制权 100% 在 harness。**

### 4.1 四步闭环

| 步       | 谁           | 内容                                          |
| ------- | ----------- | ------------------------------------------- |
| 声明      | harness     | `tools` 数组随请求发出，schema 是唯一事实来源              |
| 输出      | 模型          | name + arguments 文本，**无任何副作用**              |
| **校验**  | **harness** | 三关（见下）                                      |
| 执行 + 回写 | harness     | 查 registry 执行 → 结果写成 `role="tool"` 消息 → 下一轮 |

### 4.2 三关校验

1. **name 是否在工具表里** —— 冷启动时模型幻觉出不存在的工具名是常态，这关最容易出错。
2. **arguments 是否符合 JSON Schema** —— 类型 / required / enum / additionalProperties。
3. **业务规则** —— 路径白名单、长度上限、超时、权限。

> **设计决定：校验失败不要抛异常终止，把错误当一条正常 tool 结果返回**：
>
> ```python
> {"error": "unknown tool: extract_tabels", "allowed": ["extract_tables", "grep_corpus"]}
> ```
>
> 模型看到通常能自我纠正；同时这是很有价值的负样本（RL 阶段能学「别瞎编工具名」）。

可以在解码端收紧（guided decoding、`tool_choice`），但**不能只依赖解码端**——parser 会漏，且训练/推理可能用不同后端，校验逻辑必须放在 harness 这一层才一致。

---

## 5. 环境层：从「工具」升级到「环境」

**一句话：训练器不吃 `tools` 数组，它吃的是 `reset()` / `step()`。环境层就是把同一批工具再包一层，顺带把「终止条件」和「奖励」这两个契约定下来。**

### 5.1 为什么需要这一层

前面四节解决的是「一次 rollout 怎么跑」。但一旦要接 RL，训练器（verl / verifiers / OpenEnv）消费的接口是 Gymnasium 风格的 Env，核心只有：

```python
obs, info = env.reset()
obs, reward, terminated, truncated, info = env.step(action)
```

差别不在能力，而在**契约的表达方式**：

| OpenAI rollout 视角     | 环境层视角                                    |
| --------------------- | ---------------------------------------- |
| `tools` 数组            | `action_space`（RL 的可选动作集）                 |
| `message.tool_calls`  | `action = {"name": ..., "arguments": ...}` |
| 工具结果 `role="tool"` 消息 | `observation`                            |
| 模型不再输出 tool call      | `terminated`（自然结束）                        |
| 达到 `max_turns`        | `truncated`（被截断）                          |
| —（接口层根本没有）            | `reward`                                  |

注意最后三行：**终止条件和奖励在接口层是不存在的概念**，只有到了环境层才被定义。这也是为什么「收集轨迹」和「训练」看似同一件事、却必须写两套壳。

### 5.2 核心工程建议：registry 三份分离

别为每个 harness 各写一份工具定义。正确做法是一个 `ToolSpec` 拆成三份数据，registry 上挂几个**视图方法**，adapter 只是「读视图 + 调 `registry.call()`」：

| 数据    | 给谁看            | 内容                                              |
| ----- | -------------- | ----------------------------------------------- |
| **schema** | 模型 / 各 harness | OpenAI function schema（纯数据，可被翻译成 action_space）   |
| **handler** | 执行器            | 纯函数，三种实现形态（inproc / subprocess / http）共用同一签名     |
| **meta** | 策略层            | `mode` / `read_only` / `timeout` / `max_chars` / `mask_observation` / `tags` |

registry 上挂四个方法就够了：

```python
openai_tools(tags)      # 视图 A：给 OpenAI / vLLM rollout，出 tools 数组
action_space_desc()     # 视图 B：给 Gymnasium / OpenEnv，出 action_space 描述
read_only_subset()      # 视图 C：给权限裁剪
call(name, args)        # 执行：三关校验 + 按 mode 分发 + 输出校验
```

三个 adapter 各取所需：

| adapter                 | 用 schema           | 用 handler          | 用 meta                                    |
| ----------------------- | ------------------ | ------------------ | ----------------------------------------- |
| OpenAI / vLLM rollout   | `openai_tools()`   | `call()`           | `max_chars` 截断、`mask_observation` 决定哪些 span 不进 loss |
| Gymnasium / OpenEnv Env | `action_space_desc()` | `call()` in `step()` | `read_only` 过滤可用工具、`mask_observation` 写进 info |
| verifiers v1 Env        | 同上                 | 转发给 Gym env        | 同上                                        |

**硬性要求**：registry 必须是 rollout 与 training **import 的同一份代码**。两侧 schema 或校验逻辑一旦分叉，收集到的轨迹就和训练时渲染的不一致——这正是 token 保真的前提。

---

## 6. 生态地图：现成项目怎么选

**一句话：训练器用 verl；环境/奖励用 verifiers；协议层看 OpenEnv；轨迹收集用 `openai-agents` 或自己写循环。**

### 6.1 能 pip 装的（2026-10-08 实测）

| 包                    | 版本     | 月下载         | stars  | 定位                                      | 装法                                                |
| -------------------- | ------ | ----------- | ------ | --------------------------------------- | ------------------------------------------------- |
| **`verifiers`**      | 0.3.1  | **762,022** | 4,683  | 环境 / 工具 / 奖励 harness + Environments Hub | `pip install verifiers` + `uv tool install prime` |
| **`openenv`**        | 0.8.0  | 49,125      | 2,680  | 环境**协议层**，Gymnasium 风格 + 原生 MCP         | `pip install openenv`                             |
| **`agentlightning`** | 1.0.2  | —           | 18,591 | 零改造接入层：Tracer + LLM Proxy 抓 token ids   | `pip install agentlightning`                      |
| **`harbor`**         | 0.24.0 | —           | 5,916  | 评测 + 容器化 rollout harness                | `pip install harbor`                              |
| `verl`               | —      | 26,607      | 23,791 | **训练器，不是 harness**（量级参照）                | `pip install verl`                                |

⚠️ **API 变化**：`verifiers` 0.3.1 的包根只暴露版本号，注释原话 *"the stack lives in `verifiers.v1`"*。要写 `import verifiers.v1 as vf`，顶层导出 `Env` / `SingleAgentEnv` / `Harness` / `Taskset`，工具走 `verifiers.v1.mcp.Toolset`，runtime 有 `SubprocessConfig` / `DockerConfig` / `PodmanConfig` / `ApptainerConfig` / `PrimeConfig`。

### 6.2 只能源码装的

| 项目               | stars | 状态          | 说明                                                                                                  |
| ---------------- | ----- | ----------- | --------------------------------------------------------------------------------------------------- |
| **ROLL**（阿里）     | 3,415 | 天天更新        | 工具生态最全（SearchTool / MCPTool / PythonCodeTool，YAML 有 `sandbox_type`）；**没上 PyPI**                     |
| **rLLM**         | 5,862 | 活跃          | 自带 `SWEEnv`（backend 可选 kubernetes）；**主仓库已迁到 `rllm-org/rllm`**，旧的 `agentica-project/rllm` 停在 2025-09 |
| **AReaL**（蚂蚁+清华） | —     | 活跃          | 全异步，generation 与 training 解耦；ASearcher 做到 128 轮、单轨迹 100+ 次工具调用                                      |
| **Atropos**      | 1,354 | **停更 3 个月** | 只抄设计                                                                                                |
| **GEM**          | 510   | **停更 9 个月** | 接口干净、被 ROLL 内置依赖，值得抄，别当依赖                                                                           |

⚠️ **同名包陷阱**：PyPI 上的 `atropos` 是高通量测序去接头工具，`gem` 是游戏数学库——**都不是**那两个项目。

### 6.3 轨迹收集（不训练时）

| 包                   | stars | 判断                                                                                                                    |
| ------------------- | ----- | --------------------------------------------------------------------------------------------------------------------- |
| **`openai-agents`** | 30k   | **首选**。`set_default_openai_api("chat_completions")` 可指向 vLLM；`agents.mcp` 支持 MCP；`RunResult.to_input_list()` 导出标准消息列表 |
| **`smolagents`**    | 30k   | 备选。有 MCP，但 CodeAgent 自带 prompt，与 Qwen3 ChatML 不一致                                                                     |
| `pydantic-ai`       | 20k   | 类型化输出强                                                                                                                |
| `litellm`           | 60k   | 网关层，不是 agent 框架                                                                                                       |
| `openhands`         | 90k   | 完整平台（含 EventStream 轨迹），偏代码场景、太重                                                                                       |

⚠️ **`openai-agents` 的一个必须改的默认值**：tracing 默认用 `BatchTraceProcessor` + `BackendSpanExporter` **把 trace 发到 OpenAI 后端**，且 `trace_include_sensitive_data` 默认 True。自托管必须 `set_tracing_disabled(True)` 或换自己的落盘处理器。

### 6.4 标准化程度：三层要分清

| 层                       | 状态                                                                                                   |
| ----------------------- | ---------------------------------------------------------------------------------------------------- |
| **接口层**（模型 ↔ harness）   | ✅ **已事实标准化**。OpenAI wire format，vLLM/SGLang/TGI 全实现                                                  |
| **工具协议层**（harness ↔ 工具） | ⚠️ **部分收敛到 MCP**。verifiers / OpenEnv / Claude Code 是 MCP 原生；ROLL 的 `SearchTool`、GEM 全套是自有 `BaseTool` |
| **运行时层**（工具在哪跑）         | ❌ **完全没统一**，各家绑定自己的 trainer / 云平台                                                                    |

新动向：接口层正出现第二个竞争者 **ACP（Agent Client Protocol，Zed 提出）**，verifiers v1 里已有 `verifiers/v1/acp` 模块。现在还不用管，值得盯着。

### 6.5 其他协议（常被误当成 MCP 替代品）

- **OpenAPI / Swagger**：MCP 的同层竞争者，**比 MCP 更轻**——已有 HTTP 服务把 OpenAPI 描述喂进去就变成工具。代价是只能表达 REST，没有 resources / prompts / sampling 语义。
- **A2A**（Google 发起，2025-06 捐 Linux Foundation）：26k stars，`a2a-sdk` 1.2.2。核心是 **Agent Card**（`/.well-known/agent-card.json`）。**只有当「把另一个 agent 当工具调」时才扮演工具 specs 的角色**，本体是横向编排。
- **ACP 有两个同名协议（最大的坑）**：Zed 的 Agent Client Protocol（编辑器 ↔ agent，「agent 时代的 LSP」，4.4k stars）；IBM 的 Agent Communication Protocol 已**于 2025-08 并入 A2A 并退出**。



---

## 7. 最小实现：一份 registry 喂三个 harness

完整可跑代码在 `/tmp/toolkit/`（`registry.py` + `adapters.py`），这里摘核心骨架。

### 7.1 三份分离的数据结构

```python
@dataclass(frozen=True)
class ToolMeta:                       # 策略层读，模型看不到
    mode: str = "inproc"              # inproc | subprocess | http
    read_only: bool = True
    idempotent: bool = True
    timeout: float = 5.0
    max_chars: int = 32_000
    mask_observation: bool = True     # 结果 token 是否不进 loss
    tags: tuple[str, ...] = ("core",)

@dataclass(frozen=True)
class ToolSpec:
    name: str
    input_schema: dict                # OpenAI function schema，纯数据
    output_schema: dict               # 声明返回结构，换形态时防漂移
    meta: ToolMeta
    handler: Callable | None = None   # inproc
    script: str | None = None         # subprocess
    endpoint: str | None = None       # http
```

handler 一律返回同一契约 `{result, opened_files, n_tokens, truncated}`——`opened_files` 就是 provenance，后面直接变 process reward。

### 7.2 registry 三视图 + 三关校验

```python
class ToolRegistry:
    def openai_tools(self, tags=("core",)):          # 视图 A：tools 数组
        return [s.input_schema for s in self.specs.values()
                if any(t in s.meta.tags for t in tags)]

    def action_space_desc(self):                     # 视图 B：Gymnasium
        return {n: {"arguments": s.input_schema["function"]["parameters"],
                    "read_only": s.meta.read_only,
                    "mask_observation": s.meta.mask_observation}
                for n, s in self.specs.items()}

    def call(self, name, args) -> ToolResult:
        spec = self.specs.get(name)
        if spec is None:                             # ① name 是否在表里
            return ToolResult(name, False, {"allowed": self.names()},
                              f"unknown tool: {name}", 0, 0, False)
        jsonschema.validate(args, spec.input_schema["function"]["parameters"])  # ②
        try:
            payload = {"inproc": lambda: spec.handler(**args),
                       "subprocess": lambda: _run_subprocess(spec, args),
                       "http": lambda: _run_http(spec, args)}[spec.meta.mode]()
        except Exception as e:                       # 错误进 result，不抛异常
            return ToolResult(name, False, {}, f"{type(e).__name__}: {e}", ...)
        jsonschema.validate(payload, spec.output_schema)   # 防输出漂移
        return ToolResult(name, True, payload, None, elapsed_ms,
                          payload.get("n_tokens", 0), payload.get("truncated", False))
```

### 7.3 Gymnasium adapter（60 行以内）

```python
class ToolGymEnv:
    def reset(self, *, seed=None, options=None):
        self.steps, self.opened, self.done = 0, set(), False
        return self.task, {"available_tools": self.available}

    def step(self, action: dict):
        self.steps += 1
        info = {"step": self.steps}
        if "finish" in action:                       # terminated：自然结束
            self.done = True
            info["answer"] = action["finish"]
            return "", self.reward_fn(info), True, False, info

        name, args = action.get("name"), action.get("arguments", {})
        if name not in self.available:
            return (json.dumps({"error": f"unknown tool: {name}",
                                "allowed": self.available}),
                    0.0, False, self.steps >= self.max_steps, info)

        r = self.reg.call(name, args)
        for f in r.payload.get("opened_files", []):
            self.opened.add(f)                       # provenance → process reward
        info.update(tool=name, ok=r.ok, n_tokens=r.n_tokens,
                    gold_hit=len(self.opened & self.gold_files),
                    mask_observation=self.reg.specs[name].meta.mask_observation)
        return (r.content(self.reg.specs[name].meta.max_chars),
                self.reward_fn(info), False,
                self.steps >= self.max_steps,        # truncated：被截断
                info)
```

**换实现形态只改 `ToolSpec.mode`，返回结构由 `output_schema` 兜住，轨迹格式不漂移。**

---

## 8. 进阶：还没覆盖但同样重要的六件事

前七节讲的是「harness 是什么、怎么搭起来」。下面六件事论文里几乎不写，但决定你搭出来的 harness **能不能真的跑完一轮训练**。

| #  | 主题             | 为什么它是 harness 的核心问题                                   | 一句话做法                                                             |
| -- | -------------- | ------------------------------------------------------ | ----------------------------------------------------------------- |
| ①  | **并发与吞吐**      | harness 是 RL 的 **critical path**——卡的往往是环境而不是 GPU       | rollout 与工具调用全 async；工具服务配并发池 / 连接池 / 限流                          |
| ②  | **确定性**        | GRPO 的组内方差是真信号，环境抖动是假信号；两者混在一起就学不出东西                  | 训练期不用会漂移的真 API；时间戳 / PID / 绝对路径先清洗；版本 hash 入轨迹                   |
| ③  | **终止条件与奖励接口**  | 「何时结束」和「谁算 reward」都属于**环境契约**，不属于模型                   | `terminated` 与 `truncated` 必须分开；reward 各分量分开记                    |
| ④  | **工具输出是不可信输入** | 外部内容可以间接给模型下指令（prompt injection）                      | 返回一律当数据、不当指令；只读优先；凭据留服务端；长度与条数硬上限                                |
| ⑤  | **上下文渲染与截断策略** | chat template 是唯一真理，手工拼接必漂                            | 永远用同一份模板渲染；超窗策略显式选定并作为变量上报                                       |
| ⑥  | **可观测性**       | agent 的失败不是「整体变差」，而是「某一条轨迹悄悄变差」，没日志只能靠猜               | 单条轨迹存 messages + token ids + 每次调用元信息，可独立重放；失败按类型计数；留固定 probe 任务 |

### 8.1 ① 并发与吞吐：环境才是瓶颈

同步 rollout 里 GPU 经常在等工具返回。文件 IO、subprocess、HTTP 检索都是阻塞的——**一个 rollout 卡住，整批都卡住**。

- rollout 主循环与工具调用都写成 async（`AsyncOpenAI` + `asyncio.gather`），工具服务侧配并发池 / 连接池。
- **inproc 工具也要防阻塞**：CPU 密集的（大语料 grep）挪到线程池或 subprocess，别堵住事件循环。
- 检索服务单独起、独立限流与缓存，跟 rollout 进程解耦。

### 8.2 ② 确定性：最容易被低估的一条

RL 的梯度来自**组内相对差异**。如果同一条 prompt、同一个 seed 跑两次得到不同结果（工具返回漂移、时间戳变化、并发顺序不同），这部分噪声会被当成「探索」，抬高组内假方差，dynamic sampling 也会误判。

- 训练期**不要**用会漂移的真搜索 API；用本地语料快照或缓存代理。
- 工具输出进上下文前先清洗：绝对路径、时间戳、PID、exit code、ANSI 色码、harness 注入的提示文本。
- 工具实现 / 脚本的 hash、语料版本、索引版本写进每条轨迹元信息。

### 8.3 ③ 终止条件与奖励接口

`terminated`（任务自然结束，如模型提交答案）和 `truncated`（被 `max_steps` / 超时截断）**必须分开返回**。两者在 RL 里语义完全不同：truncated 的轨迹末状态不是真的终局，需要 bootstrap 估值；混在一起会引入系统性偏差。

奖励同理——outcome reward、格式奖励、process reward（如是否打开过 gold 文档）**分开记录**，不要只落一个总和，否则后面改奖励权重就得重跑。

### 8.4 ④ 工具输出是不可信输入

模型无法区分「用户指令」「系统提示」和「工具返回的内容」。而工具返回可能来自网页、文件、数据库——**第三方可以往里面塞指令**。

- 工具返回一律当**数据**处理，不当指令执行。
- 只读优先（不注册 `write_file` / `edit_file`），凭据留在服务端不下发给模型。
- 长度与命中条数硬上限，既是防 prompt injection 也是防 OOM。

### 8.5 ⑤ 上下文渲染与截断策略

**永远用同一份 chat template 渲染，不要手工拼字符串。** Qwen3 的模板靠 `content.startswith('<tool_response>') and endswith('</tool_response>')` 判断这条 user 消息是不是工具结果，据此决定历史 assistant turn 要不要补空的 `<think>` 块——**多一个空格都会误判**，导致 rollout 与训练渲染出的 token 序列不一致。

另外要显式决定超窗怎么办（截断最旧 / 截断中间 / 摘要 / 直接终止），并把这个策略**作为变量上报**，否则你的对比实验里混了不止一个自变量。

### 8.6 ⑥ 可观测性

单条轨迹要能独立重放：完整 messages + 每轮 `prompt_token_ids` / `token_ids` + 每次工具调用的元信息（名字、耗时、`n_tokens`、`truncated`）。有了这三样，任何一条异常轨迹都能离线复现，不用重跑整个 rollout。

再加两个工程习惯：**失败按类型计数**（解析失败 / 未知工具 / 超时 / 越界 / 截断），以及**留一组固定的 probe 任务**，每次改 harness 都跑一遍做回归——否则你永远不知道某次改动是不是悄悄把成功率打掉了。

> **如果只挑两条先学：② 和 ⑥。** ② 错会让你白跑几天却查不出 RL 为什么不收敛；⑥ 缺失会让你每次改 harness 都不知道自己改坏了什么。

---

## 9. 常见误解 FAQ

| #  | 误解                                      | 正解                                                                                                                                        |
| -- | --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| 1  | 「Skill 是一种工具」                           | ❌ Skill 不产生 tool call，只注入指令。要产生 tool call 必须进 `tools` 数组（内置或 MCP）。                                                                        |
| 2  | 「脚本放进 skill 就能跑」                        | ❌ Skill 不提供执行通道。官方 20 个 skill 全部靠 Bash 跑脚本。自建 harness 可用 `run_script` 白名单 typed tool 替代。                                                  |
| 3  | 「`run_script` 本质就是 bash tool」           | ❌ bash 的定义性特征是「模型写自由 shell 字符串 + shell 解释」。`run_script` 是 enum name + 结构化 args + argv 列表 + `shell=False`；更进一步可以直接进程内函数调用，一个进程都不起。         |
| 4  | 「Bash 的轨迹不好 mask」                       | ❌ 这句不准确。Bash 是标准 tool call，mask 和 `Read` 一样容易。真正变差的是：provenance 抽不出（process reward 废掉）、长度旋钮失灵、输出脏污染组内方差、单条可复合多动作导致 credit 变粗。             |
| 5  | 「Claude Code 的 Read/Grep 是 MCP 或 Skill」 | ❌ 都是**内置原生工具**（第三条路）。且 `Glob`/`Grep` 在 macOS/Linux/WSL **默认缺席**，这些平台直接走 Bash。                                                             |
| 6  | 「上了 MCP 就得起进程」                          | ❌ MCP 支持 in-memory transport，同进程、无子进程、无网络。                                                                                                |
| 7  | 「MCP 是 subprocess 的替代品」                 | ❌ 不同维度：MCP 是协议，subprocess 是运行时档位。MCP 的默认形态恰恰就是 subprocess。                                                                                |
| 8  | 「进程内也能做硬超时」                             | ❌ 线程无法安全 kill，`signal.alarm` 只对主线程有效。能真正 kill 的只有 subprocess。                                                                             |
| 9  | 「工具多了就该上延迟加载」                           | ⚠️ 延迟加载至少多一整轮往返，且中途改 `tools` 数组会让 radix prefix cache 从该点起全部失效（Qwen3 把 tools 渲染在 system 里，即上下文最前面）。工具少时正确做法是**按任务类型在 rollout 开始前一次性静态裁剪**。 |
| 10 | 「校验失败应该抛异常」                             | ❌ 错误进 result 返回（MCP 规范 `isError` 原文：否则 LLM 看不见就无法自纠）。                                                                                     |

---


