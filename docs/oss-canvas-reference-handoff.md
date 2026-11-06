# 开源画布项目借鉴与实施任务书

> 基准：Skill 平台「AI 剧集 + 画布」技术实现评估文档（`skill-development` + `skill-frontend`，基线 2026-11-06）
> 调研日期：2026-11-06
> 所有仓库结构与许可证均已在本地 clone 后逐一核实，文中给出的代码路径为实际存在的路径。

---

## 0. 本文档怎么用

| 读者 | 读哪里 |
|---|---|
| 决策者 | 第 1 节（许可证边界）+ 第 4 节开头的优先级表 |
| 执行 Agent | 第 2 节拉代码 → 第 3 节找参考位置 → 第 4 节领取任务包 |

第 4 节的每个任务包都是**自包含**的，可以整段复制给一个独立 Agent 执行，不需要额外上下文。

---

## 1. 许可证边界（动代码前必读）

### 1.1 结论速查

| 项目 | 许可证（已核实） | 能否复制代码进 Skill |
|---|---|---|
| open-ai-canvas（影策） | MIT，`LICENSE` 完整，`NOTICE` 保留上游署名 | ✅ 可以 |
| infinite-canvas | MIT，`LICENSE` 完整 | ⚠️ 可以，但见 1.2 |
| Doodle-Canvas-Service | MIT，`LICENSE` 完整 | ✅ 可以 |
| Curiso | MIT，`LICENSE` 完整 | ✅ 可以 |
| Mosaic | MIT，`LICENSE` 完整 | ✅ 可以 |
| canvas-chat | ⚠️ **仓库内无 `LICENSE` 文件**，仅 README 写了 MIT | ⚠️ 见 1.2 |
| AI Canvas Tauri | Source-Available，**非开源** | ❌ 禁止 |
| tldraw | tldraw 专有许可 | ❌ 生产需付费授权 |
| Dify | 改版 Apache 2.0 | ⚠️ 多租户需书面授权 |
| AFFiNE | 编辑器 MIT / 后端 MPL-2.0 | ⚠️ 分许可证判断 |

### 1.2 三条硬规则

**规则一：AI Canvas Tauri 的代码一行都不能进 Skill。**
它采用自定义的 Source-Available License，README 明确声明「并非 OSI 定义下的开源项目」，并禁止「将本项目作为同类产品进行商业化」。Skill 与它是同类产品，这条直接命中。
允许的做法：阅读其 README、在线文档与产品演示，理解设计意图后**独立实现**。
禁止的做法：复制任何代码片段、结构体定义、配置 schema。

**规则二：canvas-chat 只做设计参考，不直接复制。**
仓库内没有 `LICENSE` 文件，只有 README 里一行「MIT」。在没有正式许可证文件的情况下，默认版权归属作者、未授予使用权。如果确实要复用其代码，先向作者索取正式授权或等其补齐 LICENSE 文件。读懂算法逻辑后自己写一遍是安全的。

**规则三：infinite-canvas 复用前过一遍法务。**
许可证是标准 MIT，但 README 额外要求「二次开发与 PR 请保留原作者信息和前端页面标识」。这超出 MIT 仅要求保留版权声明的范围。在 Skill 这种商业闭源产品里复用其代码，建议先确认这条请求的约束力。

**另：tldraw SDK 不是开源软件。** 生产环境（HTTPS 非回环域名 + `NODE_ENV=production`）必须持有有效 license key，否则 SDK 在控制台报错并在五秒后停止渲染。其 starter kit 示例代码是 MIT，但运行依赖 SDK 本体。本任务书中 tldraw 仅作为**设计阅读材料**出现。

---

## 2. 一键获取参考代码

```bash
mkdir -p ~/refs && cd ~/refs

# 可直接复用代码的（MIT）
git clone --depth 1 https://github.com/ddcat-ai/open-ai-canvas.git        # 影策 · 短剧五步流水线 · Go + React
git clone --depth 1 https://github.com/basketikun/infinite-canvas.git     # 无限画布 · 插件与渠道适配 · React
git clone --depth 1 https://github.com/kggzs/Doodle-Canvas-Service.git    # 多渠道调度与计费 · Node + Vue
git clone --depth 1 https://github.com/metaspartan/curiso.git             # 本地模型聚合与 RAG · React + Tauri
git clone --depth 1 https://github.com/versus184-py/Mosaic.git            # 多模型并联对比 · React Flow 12

# 仅供阅读，禁止复制代码
git clone --depth 1 https://github.com/ericmjl/canvas-chat.git            # 矩阵评估与多模型合议 · Python + 原生 JS
git clone --depth 1 https://github.com/Tenney95/AI-Canvas-tauri.git       # 产品设计参考 · Source-Available
```

仓库链接汇总：

- 影策 open-ai-canvas — https://github.com/ddcat-ai/open-ai-canvas
- infinite-canvas — https://github.com/basketikun/infinite-canvas
- Doodle-Canvas-Service — https://github.com/kggzs/Doodle-Canvas-Service
- Curiso — https://github.com/metaspartan/curiso
- Mosaic — https://github.com/versus184-py/Mosaic
- canvas-chat — https://github.com/ericmjl/canvas-chat
- AI Canvas Tauri — https://github.com/Tenney95/AI-Canvas-tauri
- tldraw（设计阅读） — https://github.com/tldraw/tldraw/tree/main/templates/agent
- AFFiNE（CRDT 参考） — https://github.com/toeverything/AFFiNE
- Dify（provider 抽象参考） — https://github.com/langgenius/dify

---

## 3. 逐项目参考清单

### 3.1 影策 open-ai-canvas —— 最高价值参考

| 项 | 内容 |
|---|---|
| 仓库 | https://github.com/ddcat-ai/open-ai-canvas |
| 许可证 | MIT（`LICENSE` + `NOTICE`，上游为 `basketikun/infinite-canvas`） |
| 技术栈 | 后端 Go 1.25，前端 Bun + Vite + React |
| 与 Skill 的关系 | 五步流水线同构：剧本 → 角色 → 分镜 → 成片 → 剪辑 |

#### 3.1.1 声明式渠道插件契约（本次调研最有价值的发现）

`plugin-packages/` 下有 **111 个渠道插件包**，覆盖 OpenAI、Anthropic、Gemini、Vertex、Bedrock、火山方舟、阿里百炼、智谱、Kimi、DeepSeek、MiniMax、Kling、Runway、Luma、Pika、Vidu、Recraft、Ideogram、Flux、Replicate、fal、Ollama、vLLM、LocalAI、LiteLLM、RunningHub、ComfyUI 等。

**每个包只有三个文件，没有一行适配代码：**

```
plugin-packages/<provider-id>/
├── manifest.json        # 完整的声明式契约
├── docs/interface.md    # 自动生成的接口文档
└── README.md
```

`manifest.json` 的契约结构（取自 `plugin-packages/vllm-chat/manifest.json`，已核实）：

```jsonc
{
  "apiVersion": "yingce.plugin/v2",
  "id": "vllm-chat",
  "version": "2.0.0",
  "permissions": ["generation.run", "media.read"],
  "configuration": {
    "fields": [{ "name": "apiKey", "type": "secret", "required": true }]
  },
  "contributes": {
    "providers": [{
      "id": "vllm-chat",
      "capabilities": ["text"],                    // text | image | video | audio
      "scopes": ["admin.system-channel", "user.custom-channel", "canvas", "creation", "agent"],
      "baseUrl": "http://127.0.0.1:8000",
      "auth": { "type": "bearer", "field": "apiKey" },
      "parameters": [                              // 统一字段 → 上游字段的映射声明
        { "name": "model", "type": "string", "required": true, "mapping": "model" },
        { "name": "temperature", "type": "number", "required": false, "mapping": "temperature" }
      ],
      "create": {                                  // 上游请求模板
        "method": "POST",
        "path": "/v1/chat/completions",
        "body": {
          "model": { "$ref": "request.model" },
          "temperature": { "$omitEmpty": { "$ref": "request.providerOptions.vllm-chat.temperature" } },
          "max_tokens": { "$omitEmpty": { "$coalesce": [
            { "$ref": "request.extra.max_tokens" },
            { "$ref": "request.providerOptions.vllm-chat.max_tokens" }
          ]}}
        }
      },
      "agent": { "body": { "$merge": [ /* ... */ ] } },
      "response": {                                // 响应映射
        "textPaths": ["choices.0.message.content", "choices.0.text"],
        "reasoningPaths": ["choices.0.message.reasoning_content"],
        "usage": { "$ref": "response.usage" },
        "errorPaths": ["error.code"],
        "messagePaths": ["error.message"]
      },
      "agentResponse": {
        "toolCallsPath": "choices.0.message.tool_calls",
        "toolCallNamePaths": ["function.name"],
        "toolCallArgumentsPaths": ["function.arguments"]
      }
    }]
  }
}
```

关键在于这套模板表达式 DSL：`$ref` 取值、`$omitEmpty` 空值省略、`$coalesce` 多来源回退、`$merge` 对象合并。有了它，**接入一个新渠道等于写一个 JSON 文件，不需要发版、不需要改后端代码**。异步任务类渠道还有 `query`（轮询）、`cancel`（取消）、结果下载等段。

对照 Skill 现状：你们的 `app/provider/{ark,openrouter,byteplus,heygen}` 是逐厂商手写 client，`MODEL_IDS` / `PRICES` 是代码常量，新增渠道必须发版。

建议阅读顺序：

1. `plugin-packages/openai-chat-completions/manifest.json` —— 最标准的同步文本协议
2. `plugin-packages/volcengine-ark-seedance/manifest.json` —— 异步视频任务，与你们现有 Seedance 链路可直接对照
3. `plugin-packages/volcengine-ark-seedream/manifest.json` —— 图片生成
4. `plugin-packages/vllm-chat/manifest.json`、`ollama-chat` —— 本地推理接入
5. `plugin-packages/runninghub-workflow/manifest.json` —— 工作流型渠道怎么声明
6. `plugin-packages/generate-catalog.mjs`、`build-packages.sh`、`package-directories.mjs` —— 打包与目录生成工具链
7. `backend/internal/provider/{interface.go,registry.go}` —— 宿主侧的 provider 接口与注册
8. `backend/internal/providercatalog/{catalog.go,pricing.go,source.go}` —— 模型目录与定价，含 `source_retry_test.go`

#### 3.1.2 其他可借鉴模块（路径已核实）

| 能力 | 路径 | 说明 |
|---|---|---|
| 技能库 | `backend/internal/skills/`（`service.go`、`skill_packages*.go`、`skill_curation*.go`、`skill_library_categories.go`、`seed/`） | 技能包的发现、编辑、归档、GitHub 回源、分类策展，带完整测试 |
| 内置技能内容 | `backend/builtin/`（39 个目录） | 「一镜到底短剧流水线」「叙事短片导演分镜」「剧本撰写」「剧本工业化机器」「短剧实战手册」等，是现成的提示词资产 |
| 剪映草稿导出 | `backend/internal/jianying/draft.go` | 把成片工程导出为剪映草稿，用户可在剪映里继续精剪。**这是绕开自建多轨编辑器的捷径** |
| 影视制作链路 | `backend/internal/production/`（`storyboard.go`、`segment_prompt.go`、`segment_writer.go`、`looks.go`、`timing.go`、`dialogue_check.go`、`assets.go`） | 与你们 `app/storyboard` + `app/unit` 直接对位，可做实现对比 |
| 画布内核 | `backend/internal/canvas/`（`realtime.go`、`edit_leases.go`、`canvas_history.go`、`canvas_share.go`、`canvas_asset_guard.go`、`operations.go`、`layout/`、`capability/`、`contract/`） | 它做了实时协同、编辑租约、历史、分享、资产守卫。`canvas_share.go` 对应你们缺失的画布分享 |
| 生成引擎 | `backend/internal/generation/`（`engine.go`、`registry.go`、`types.go`） | 四类任务统一队列的抽象边界 |
| 前端画布 | `web/src/pages/canvas/`、`web/src/stores/canvas/`、`web/src/components/canvas/` | React 实现，可与 `skill-frontend/web/src/studio/canvas/` 对照 |

---

### 3.2 infinite-canvas

| 项 | 内容 |
|---|---|
| 仓库 | https://github.com/basketikun/infinite-canvas |
| 许可证 | MIT（README 有额外署名要求，见 1.2 规则三） |
| 技术栈 | Bun + Vite + React，前端直连 OpenAI 兼容接口（BYOK） |

| 能力 | 路径（已核实） | 说明 |
|---|---|---|
| 插件运行时 | `web/src/lib/canvas/`（`plugin-loader.ts`、`plugin-registry.ts`、`plugin-runtime.ts`、`plugin-node-context.ts`、`node-registry.ts`） | 节点插件的加载、注册、运行时沙箱与上下文注入 |
| 插件示例 | `plugins/canvas/`、`plugins/infinite-canvas/` | 官方插件包样例 |
| 画布导入导出 | `web/src/lib/canvas/canvas-export.ts` | 对应你们缺失的画布 JSON 导出 |
| 画布 Agent / MCP | `canvas-agent/`（`src/`、`agent-instructions.md`、`codex-server.md`） | 本地 Agent 通过 MCP 操作画布，可与你们 `app/canvas/mcp_server.py` 对照 |
| 提示词库 | `web/src/pages/prompts/`、`web/src/components/prompts/`（`prompt-card.tsx`、`prompt-select-dialog.tsx`、`use-prompt-list.ts`） | 多来源提示词聚合与选择器 UI |
| 节点工厂与几何 | `web/src/lib/canvas/`（`canvas-node-factory.ts`、`canvas-node-geometry.ts`、`canvas-node-size.ts`、`canvas-resource-references.ts`） | 节点创建、尺寸与资源引用管理 |
| 生成辅助 | `web/src/lib/canvas/canvas-generation-helpers.ts`、`web/src/lib/canvas/canvas-video-frame.ts` | 首尾帧处理等 |

---

### 3.3 Doodle-Canvas-Service

| 项 | 内容 |
|---|---|
| 仓库 | https://github.com/kggzs/Doodle-Canvas-Service |
| 许可证 | MIT |
| 技术栈 | Node.js + Vue 3 + MySQL + Redis |
| 规模 | 很小（2 星），**建议只取设计不取代码** |

| 能力 | 路径（已核实） | 说明 |
|---|---|---|
| 渠道适配器 | `server/src/adapters/`（`openai.js`、`aliyun.js`、`doubao.js`、`hunyuan.js`、`stepfun.js`、`agnes.js`、`custom.js`、`shared.js`、`index.js`） | 7 种 Provider 适配器的组织方式 |
| 多渠道优先级调度 | `server/src/scheduler/index.js` | 同一逻辑模型绑定多个物理渠道，按优先级选线与故障转移 |
| 生成队列 | `server/src/services/generation-queue.js`、`generation.js` | 租约领取、预扣、幂等退款 |
| 计费 | `server/src/services/`（`billing.js`、`coins.js`、`user-groups.js`） | 金币预扣结算与用户组倍率分层定价 |
| 运营后台服务 | `server/src/services/`（`model-management.js`、`admin-users.js`、`records.js`、`error-logs.js`、`announcements.js`） | 完整运营后台的服务层划分 |
| 存储与安全 | `server/src/services/storage.js`、`server/src/middleware/` | 签名访问、软删除、远程媒体下载的 SSRF 防护 |

---

### 3.4 Curiso

| 项 | 内容 |
|---|---|
| 仓库 | https://github.com/metaspartan/curiso |
| 许可证 | MIT |
| 技术栈 | React + Tauri（桌面端） |

| 能力 | 路径（已核实） | 说明 |
|---|---|---|
| 本地模型接入 | `client/src/lib/localmodels.ts` | Ollama、LM Studio、vLLM、Jan、Exo 的统一接入 |
| 云端模型接入 | `client/src/lib/remotemodels.ts` | 十余家云厂商的统一抽象 |
| 推理速度指标 | `client/src/lib/toksec.ts`、`client/src/lib/metricstore.ts`、`client/src/components/MetricsDialog.tsx` | tok/s 与 token 总量的采集与展示 |
| 本地 RAG | `client/src/lib/rag.ts`、`embeddings.ts`、`db.ts`、`scraper.ts`；UI 在 `components/RAGManager.tsx`、`RAGSelector.tsx` | 文档与网站导入、本地嵌入、IndexedDB 向量库 |
| 密钥加密存储 | `client/src/lib/encryption.ts`、`secureStorage.ts` | 本地加密（你们是服务端托管，架构不同，仅作参考） |

---

### 3.5 Mosaic

| 项 | 内容 |
|---|---|
| 仓库 | https://github.com/versus184-py/Mosaic |
| 许可证 | MIT |
| 技术栈 | React + React Flow v12 + Zustand + Tauri（**与 skill-frontend 画布同栈**） |

| 能力 | 路径（已核实） | 说明 |
|---|---|---|
| 多 provider 统一流式层 | `src/api/`（`providers.ts`、`openai.ts`、`anthropic.ts`、`gemini.ts`、`mistral.ts`、`ollama.ts`、`config.ts`，含 `__tests__`） | AsyncGenerator 统一五家 provider 的流式接口 |
| 画布状态管理 | `src/store/`（`canvasStore.ts`、`canvasManagerStore.ts`、`ragStore.ts`、`analyticsStore.ts`、`pruneStore.ts`、`uiStore.ts`） | 多画布管理与分析统计的 store 切分，可与你们每画布一个 zustand 实例的设计对照 |
| 画布与节点 | `src/components/canvas/`（`MosaicCanvas.tsx`、`MessageNode.tsx`、`NodeInput.tsx`、`DistillEdge.tsx`、`TendrilEdge.tsx`、`LiquidEdge.tsx`） | React Flow 12 的自定义节点与边实现 |
| 沙箱执行 | `src/workers/` | Web Worker 沙箱（与剧集场景无关，略） |

---

### 3.6 canvas-chat（仅阅读，勿复制）

| 项 | 内容 |
|---|---|
| 仓库 | https://github.com/ericmjl/canvas-chat |
| 许可证 | ⚠️ 无 LICENSE 文件，README 声明 MIT |
| 技术栈 | Python FastAPI + litellm + llamabot，前端原生 JS |

| 能力 | 路径（已核实） | 说明 |
|---|---|---|
| 矩阵交叉评估 | `src/canvas_chat/plugins/matrix_handler.py` | 从上下文节点解析行列、填充矩阵单元格，SSE 流式返回。基于 `AsyncStructuredBot` + Pydantic 结构化输出 |
| 多模型合议 | `src/canvas_chat/static/js/app.js`、`slash-command-menu.js`、`streaming-manager.js` | `/committee` 斜杠命令的调度与结果合成 |
| CRDT 图模型 | `src/canvas_chat/static/js/crdt-graph.js` | 画布图的 CRDT 实现，是轻量级实时协同参考 |
| 自动布局 | `src/canvas_chat/static/js/layout.js` | 左右层级 DAG 自动排列 |
| 画布内搜索 | `src/canvas_chat/static/js/search.js` | 跨节点全文检索 |
| 撤销管理 | `src/canvas_chat/static/js/undo-manager.js` | 对照你们的 30 步 undo |
| 插件机制 | `src/canvas_chat/static/js/`（`feature-registry.js`、`feature-plugin.js`、`node-registry.js`、`plugin-events.js`、`example-plugins/`） | 前端特性插件注册体系 |
| 联网检索 | `src/canvas_chat/static/js/web-grounding.js`、`plugins/ddg_endpoints.py` | 搜索增强 |
| 设计文档 | `docs/designs/`、`docs/specs/`、`docs/llds/` | 作者写了完整的设计文档与低层设计，**比读代码更高效** |

---

### 3.7 AI Canvas Tauri（禁止复制代码）

仓库：https://github.com/Tenney95/AI-Canvas-tauri
许可证：Source-Available，禁止商业再分发。

**只读 README、`docs/` 与在线演示，理解后独立实现。** 值得研究的产品设计：

1. **分集多画布** —— 每集一张画布，整部剧共享角色库与项目记忆。用产品结构绕开单画布节点膨胀，对应你们 `MAX_NODES=500` 的硬上限。
2. **资产库** —— 项目级 + 全局级的人物/场景/道具，多参考图、单图精准引用、使用位置反查、失效素材重连、资产包跨项目导入导出。
3. **快捷指令编排** —— 用户可保存复用的多步宏，步骤可引用原始输入或上一步结果，支持顺序或依赖并发执行，可导出 JSON 分享。
4. **只读子智能体 + AI 应用节点** —— 能力绑定到具体画布素材而非全局 scope，探查类操作降权执行。
5. **MCP 默认关闭，远程连接需显式确认。**
6. **存储健康面板** —— 区分文件占用与应用数据配额，清理前复核引用。

---

## 4. 可直接派发的任务包

### 优先级总览

| # | 任务 | 解决的问题 | 主要参考 | 改动范围 |
|---|---|---|---|---|
| A | 渠道声明式接入层 | 单一供应商风险、新增渠道要发版 | open-ai-canvas | 后端 `app/provider`、`app/catalog` |
| B | 图像后期编辑能力 | 只能生成不能修 | open-ai-canvas 插件包、Lobe Chat Pro | 后端 provider + 前端画布节点 |
| C | 多模型并联对比与评估矩阵 | 批量只能同模型多张 | Mosaic、canvas-chat | 前端画布 + 后端批量提交 |
| D | 画布 JSON 导入导出与分享 | 画布无法备份迁移 | infinite-canvas、open-ai-canvas | 前端 + 后端 canvas 路由 |
| E | 共享技能库 / 提示词库 | 提示词无法团队复用 | open-ai-canvas | 后端新模块 + 前端新页面 |
| F | 分集多画布 + 共享资产库 | 500 节点上限、资产不可跨项目 | AI Canvas Tauri（仅设计） | 领域模型调整，改动较深 |
| G | 成片交付升级（剪映草稿导出） | 线性 concat 无法精剪 | open-ai-canvas `jianying/draft.go` | 后端合成模块 |

建议顺序：**A → B → D → C → E → G → F**。A 是上线前风险且多个项目都给出了成熟答案；F 触及领域模型，放最后。

---

### 任务 A · 渠道声明式接入层

```
【任务】为 Skill 后端引入声明式渠道接入层，替换当前逐厂商手写 client 的模式。

【背景】
Skill 当前在 app/provider/{ark,openrouter,byteplus,heygen} 下为每个厂商手写 client，
模型 ID 与价格是 app/provider/ark/client.py 里的 MODEL_IDS / PRICES 代码常量。
后果：新增一个渠道或上游出新模型都必须发版；默认走 OpenRouter（海外），
国内上线的连通性与切换成本是明确风险。

【参考实现】
仓库：https://github.com/ddcat-ai/open-ai-canvas （MIT，可复用代码）
核心：plugin-packages/ 下 111 个渠道插件包，每包仅 manifest.json + docs/interface.md + README.md，
零适配代码。宿主侧接口在 backend/internal/provider/{interface.go,registry.go}，
模型目录与定价在 backend/internal/providercatalog/{catalog.go,pricing.go,source.go}。

必读的 manifest 样例（按此顺序）：
1. plugin-packages/openai-chat-completions/manifest.json  （同步文本）
2. plugin-packages/volcengine-ark-seedance/manifest.json  （异步视频，与 Skill 现有 Seedance 链路对位）
3. plugin-packages/volcengine-ark-seedream/manifest.json  （图片）
4. plugin-packages/vllm-chat/manifest.json 与 ollama-chat/  （本地推理）

契约要点：apiVersion / permissions / configuration.fields（secret 类型）/
contributes.providers[] 下的 capabilities、scopes、baseUrl、auth、parameters（统一字段→上游字段映射）、
create（请求模板）、agent、query（轮询）、cancel、response（textPaths/usage/errorPaths）、
agentResponse（toolCalls 映射）。
模板表达式 DSL：$ref 取值、$omitEmpty 空值省略、$coalesce 多来源回退、$merge 对象合并。

【交付物】
1. 一份 Skill 版渠道契约 schema（JSON Schema 或 Pydantic 模型），覆盖同步与异步两类生命周期。
2. 一个契约解释器：读契约 → 组装 HTTP 请求 → 解析响应 → 归一为现有的统一结果结构。
   必须复用现有的 app/billing/quote.py 报价与 app/generation 的任务状态机，不要另起一套。
3. 用契约重新实现现有三个渠道（ark seedance、ark seedream、openrouter chat）作为验证，
   行为必须与现状逐字段一致。
4. 模型目录从代码常量迁移为可配置数据，支持新增模型不发版。

【约束】
- 不要引入多租户、不要改动 BillingOperation 的 HELD/结算语义。
- planning fingerprint（app/episode/service.py:411 _planning_fingerprint）依赖 model_id 与路由名，
  迁移后必须保证指纹计算输入不变，否则会误判排队中的 run 失败。
- 保持 app/provider/route.py 现有的路由切换能力。

【验收】
- 现有剧集五步链路与画布逐节点生成全部回归通过。
- 新增一个此前未支持的渠道，只需新增一个契约文件，不改 Python 代码。
- tests/ 下补充契约解释器的单元测试，覆盖 $ref/$omitEmpty/$coalesce/$merge 四种表达式。
```

---

### 任务 B · 图像后期编辑能力

```
【任务】为 Skill 画布增加图像后期编辑能力：局部重绘、图片扩展、高清放大、移除背景。

【背景】
Skill 当前图像能力只有「文生图」与「参考图编辑」（app/cast/prompt.py 的定妆照模板、
unit 的 MultimodalReference）。定妆照脸部有瑕疵或分镜首帧构图要加宽时，
用户只能整张重新生成，迭代慢且重复扣点数。

【参考实现】
1. https://github.com/ddcat-ai/open-ai-canvas （MIT）
   plugin-packages/wavespeed-image-edit/、image-tools/、volcengine-jimeng-image/ 的 manifest.json
   —— 看图像编辑类渠道的参数如何声明（蒙版、扩展方向、放大倍数）。
2. https://github.com/basketikun/infinite-canvas （MIT，复用前见许可证规则）
   web/src/lib/canvas/canvas-image-data.ts、canvas-node-factory.ts —— 画布侧图像数据与节点处理。

【交付物】
1. 后端：在生成链路中新增 image-edit 类任务，支持蒙版（inpaint）、扩展（outpaint）、
   放大（upscale）、抠图（remove-background）四种操作。
   复用现有 Generation 表与 quote → hold → settle 计费三段式，不要绕过。
2. 前端：画布图片节点新增编辑入口。蒙版需要一个轻量涂抹工具，
   在 skill-frontend/web/src/studio/canvas/nodes/ 下新增，不要引入重型图像库。
3. 编辑结果作为新 take 落回画布，保留原图（参考现有 UnitTake 的历史语义）。

【约束】
- 严格遵守现有的 canvas rules.json 槽位校验与 lock_version CAS。
- 编辑任务必须走 app/generation 的统一队列，不要用 BackgroundTasks。
- 任务 A 若已完成，新渠道一律用契约文件接入，不要再手写 client。

【验收】
- 定妆照节点可局部重绘脸部并保留其余区域不变。
- 分镜首帧可向左右扩展画幅。
- 四种操作的点数报价与实际结算一致，失败自动退点。
```

---

### 任务 C · 多模型并联对比与评估矩阵

```
【任务】让 Skill 画布支持同一输入并联多个模型横向对比，以及 prompt × 模型 × 参数的网格评估。

【背景】
Skill 画布逐节点生成支持批量 1–4 张（app/generation/router_studio.py:347），
但那是同一模型同一参数的多次采样。定妆照定形象、镜头方案选型本质是在比不同模型，
现在要人工建多个节点分别配模型再肉眼比。

【参考实现】
1. https://github.com/versus184-py/Mosaic （MIT，与 skill-frontend 同栈 React Flow 12 + Zustand）
   src/api/providers.ts —— AsyncGenerator 统一多 provider 流式接口
   src/store/canvasStore.ts —— 多路并发结果如何进 store
   src/components/canvas/MessageNode.tsx —— 并联结果的节点呈现
2. https://github.com/ericmjl/canvas-chat （⚠️ 无 LICENSE 文件，只读逻辑不复制代码）
   src/canvas_chat/plugins/matrix_handler.py —— 矩阵交叉评估的后端实现，
   用 AsyncStructuredBot + Pydantic 从上下文解析行列并填充单元格，SSE 流式返回。
   src/canvas_chat/static/js/app.js 中的 /committee —— 多模型合议后综合。

【交付物】
1. 画布新增「对比组」节点：一个 prompt 扇出到 N 个模型配置，结果并排呈现，支持一键采纳其中之一。
2. 评估矩阵：选定一个维度组合（如 模型 × 分辨率 × 时长），生成网格并批量提交，
   结果填回对应单元格。
3. 报价必须覆盖整组：提交前一次性报出全组点数，用户确认后整体 hold，
   逐项结算，失败项单独退。

【约束】
- 复用现有 idempotency_key 与 expected_credits 协议，整组共用一个幂等键。
- 对比组的节点数要计入 MAX_NODES=500，必要时在 schemas.py 里按组折算。
- 不要引入新的轮询通道，复用 react-query 2.5s 回显。

【验收】
- 同一分镜 prompt 可一次性发给 seedance2.5 / 2.0 / 2.0-fast 并排比较。
- 矩阵提交的点数预估与实际结算误差为零（多退少不补按现有规则）。
```

---

### 任务 D · 画布 JSON 导入导出与分享

```
【任务】为 Skill 画布增加文档级 JSON 导出、导入与分享链接。

【背景】
Skill 当前只有成片的 ZIP 流式打包（app/episode/router_studio.py:449 _zip_stream），
画布本身无法备份、迁移、分享或版本留档。对比项目中这是标配。

【参考实现】
1. https://github.com/basketikun/infinite-canvas （MIT）
   web/src/lib/canvas/canvas-export.ts —— 导出实现
   web/src/lib/canvas/canvas-resource-references.ts —— 导出时如何处理媒体引用
2. https://github.com/ddcat-ai/open-ai-canvas （MIT）
   backend/internal/canvas/canvas_share.go —— 分享链接的服务端实现
   backend/internal/canvas/canvas_history.go、canvas_revision_test.go —— 版本与历史

【交付物】
1. 导出：画布 document + 引用的媒体清单导出为单个 JSON（媒体用引用而非内联，附 sha256）。
2. 导入：每次导入创建一张全新画布，不覆盖已有画布（参考 infinite-canvas 的语义）。
   导入时必须跑现有的 fromServer / migrateNode 版本迁移。
3. 分享：只读分享链接，响应必须经过现有的 metadata() 剥离逻辑，不外发 URL 与 object_key。

【约束】
- 导出的文档要能被 app/canvas/schemas.py 的形状与大小校验接受。
- 导入时的节点 id 必须全量重映射，并用 remap_prompt_parts 的同款逻辑重写附件引用
  （参考 app/branch/service.py:418 create_branch 的做法）。
- 分享链接的鉴权走现有 session 体系，不要新增 token 类型。

【验收】
- 导出再导入后，画布拓扑、节点数据、附件引用完全一致。
- 跨环境（dev → staging）导入可正常打开。
- 分享链接在未登录浏览器中只读可见且不泄露任何媒体直链。
```

---

### 任务 E · 共享技能库 / 提示词库

```
【任务】为 Skill 增加团队级可复用的技能库（结构化提示词资产）。

【背景】
Skill 的提示词目前沉淀在各用户自己的 prompt 节点里（prompt_parts），
没有团队级的复用、分享与收藏机制。LLM 任务提示词集中在 app/llm_task/catalog.py，
是开发者资产而非用户资产。

【参考实现】
https://github.com/ddcat-ai/open-ai-canvas （MIT）
- backend/internal/skills/service.go —— 技能服务主体
- backend/internal/skills/skill_packages*.go —— 技能包的发现、编辑、归档、文件管理
- backend/internal/skills/skill_curation*.go、skill_library_categories.go —— 策展与分类
- backend/internal/skills/seed/ —— 初始化数据
- backend/builtin/ —— 39 个内置技能的实际内容，其中「一镜到底短剧流水线」
  「叙事短片导演分镜」「剧本撰写」「剧本工业化机器」「短剧实战手册」「角色设计」
  与 Skill 的业务高度相关，可直接作为内容蓝本（注意 MIT 署名要求）。

【交付物】
1. 技能数据模型：支持创建、公开、收藏、被画布节点引用。
2. 技能在画布中的引用方式：选中技能后物化为 prompt_parts，而不是运行时动态拉取
   （与现有 apply_fill → _assemble_prompt 的物化语义保持一致）。
3. 初始化一批内置技能，覆盖剧集五步各阶段。

【约束】
- 技能内容变更不得影响已物化到画布的 prompt_parts（避免历史结果不可复现）。
- 若技能参与计费路径，必须纳入 planning fingerprint 的考量，或明确声明不参与。

【验收】
- 用户可把自己调好的分镜提示词存为技能并公开给团队。
- 从技能库插入画布后，生成结果与手写同样内容的 prompt 一致。
```

---

### 任务 F · 分集多画布 + 共享资产库

```
【任务】把单张大画布拆分为「每集一张画布 + 整部剧共享资产库」的结构。

【背景】
Skill 画布未开启虚拟化（onlyRenderVisibleElements 明确不开），
靠 app/canvas/schemas.py 的 MAX_NODES=500 硬上限兜底，文档整档读写无懒加载。
在渲染层做虚拟化改造风险高；用产品结构拆分更顺着已有的 Episode 领域模型。

【参考设计】
https://github.com/Tenney95/AI-Canvas-tauri
⚠️ Source-Available 许可证，禁止商业再分发。
只读 README 与 docs/ 理解设计意图，然后独立实现。严禁复制任何代码片段或配置 schema。

要理解的设计点：
- 剧集与分集：一部剧拆成多张画布，每集一张
- 角色库与素材整部剧共享，不随画布复制
- 资产使用位置反查、失效素材重连
- 资产包跨项目导入导出

【交付物】
1. 画布与 Episode 的关联模型调整，支持一个项目下多张画布。
2. 资产（CastItem / CastItemAsset / 媒体）从画布中解耦，升级为项目级共享资源。
3. 资产使用位置反查：给定一个资产，列出引用它的所有画布节点与 Unit。
4. 断链检测与重连。

【约束】
- 这是本任务书中改动最深的一项，必须先出设计文档并评审，不要直接动手。
- 现有 create_branch（app/branch/service.py:418）的全量 id 重映射语义会受影响，
  要同步设计分支在多画布结构下的复制策略。
- episode_in_use 的删除保护逻辑需要扩展为通用的引用计数。

【验收】
- 单张画布的节点数在真实短剧项目中稳定低于 200。
- 删除被引用的资产时能准确列出所有引用位置。
```

---

### 任务 G · 成片交付升级（剪映草稿导出）

```
【任务】让 Skill 的成片可以导出为剪映草稿，供用户在剪映中精剪。

【背景】
Skill 的 app/episode/synthesis.py render_episode() 是逐 clip 归一化后线性 concat，
加 ASS 字幕与 BGM 混音，产出 MP4 + receipt JSON。成片之后用户无法调整单个片段的
时长、音量或转场。自建多轨时间线编辑器成本很高。

【参考实现】
https://github.com/ddcat-ai/open-ai-canvas （MIT）
backend/internal/jianying/draft.go —— 剪映草稿工程的生成。
这是绕开自建编辑器的捷径：把编排数据导出为剪映工程，让用户用专业工具精剪。

【交付物】
1. 把一集的 clip 序列、字幕、BGM 导出为剪映草稿工程包。
2. 媒体文件随包导出或给出可下载清单。
3. 与现有 ZIP 流式打包（_zip_stream）统一入口。

【约束】
- 纯本地操作，不调用任何上游 provider，必须维持 0 点数（与画布本地合成一致）。
- 复用现有的 _episode_synthesis_budget 超时预算与存储水位检查。
- 导出不得阻塞合成队列。

【验收】
- 导出的草稿在剪映中可正常打开，片段顺序、字幕时间轴、BGM 与 MP4 成片一致。
```

---

## 5. 不要做的事

1. **不要为了对齐开源项目而推翻 Skill 已有的优势实现。** 以下五项 Skill 明显强于全部对比项目：
   - 计费三段式（quote → HELD → settle，带 idempotency_key 与 price_revision_label，409 price_changed）
   - 画布并发正确性（lock_version CAS + 逐字段三方合并 + 人工冲突评审）
   - MCP 权限（逐路由 scope 矩阵 + 限速 + 响应剥离媒体地址）
   - planning fingerprint 的协议变更纪律
   - 测试覆盖（后端 7 个画布测试文件 + 前端 30 个 canvas-* 文件）

2. **不要盲目上实时协同。** Skill 的三方合并 + 冲突评审在异步多端编辑场景已经够用。
   只有当产品明确要求多人同时编辑同一张画布时才引入 Yjs，且会推翻现有冲突评审 UI 的投入。

3. **不要指望开源项目解决分布式渲染。** 调研的全部项目要么同样单机 FFmpeg，
   要么把渲染推给桌面客户端，对 SaaS 不适用。规模化瓶颈只能自建渲染集群或接云端转码服务。

4. **不要把「四种媒体类型不加更多」当缺陷。** 这是 owner 明确裁决
   （skill-frontend CanvasFlow.tsx L107），本任务书不包含增加节点类型的任务。

5. **不要在未核实的前提下断定 Skill「没有」某功能。** 以下项在实现文档中未提及，
   但不能据此断定确实缺失，执行前先在代码库中核实：画布小地图、移动端适配、
   多画布标签页、运营后台的实际覆盖范围。

---

## 6. 附：调研基础

本文档中所有代码路径均来自 2026-11-06 对以下仓库 `--depth 1` clone 后的实际目录核实：

| 仓库 | 核实内容 |
|---|---|
| ddcat-ai/open-ai-canvas | 111 个 plugin-packages 目录、manifest.json 契约字段、backend/internal 下 38 个模块、backend/builtin 下 39 个技能目录 |
| basketikun/infinite-canvas | web/src/lib/canvas 下 15 个文件、canvas-agent/、plugins/、提示词库组件 |
| kggzs/Doodle-Canvas-Service | server/src 下 adapters(9)、scheduler、services(15)、middleware |
| metaspartan/curiso | client/src/lib 下 19 个文件、components 下 18 个组件 |
| versus184-py/Mosaic | src/api(7)、src/store(7)、src/components/canvas(7) |
| ericmjl/canvas-chat | src/canvas_chat/plugins(10)、static/js(34)、docs/designs 等设计文档目录 |

许可证均通过各仓库根目录 `LICENSE` 文件首行核实；canvas-chat 确认无 `LICENSE` 文件。
AI Canvas Tauri 与 tldraw 未 clone，许可证依据其官方 README 与 tldraw.dev 许可证页面。
