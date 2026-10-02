# Amadeus-SpriteForge-Animator — Draft Plan

**Status:** discussion draft; documentation only.

**Date:** 2026-10-02.

**Implemented in this repository:** the placeholder README and this plan.

[中文计划草案](#中文计划草案)

## 1. Problem and intended outcome

A SpriteForge character pack contains runtime assets and graph topology, but it
does not currently carry the complete performance behavior used by Amadeus.
Amadeus supplies speaking label groups, intent mappings, transition holds, mouth
presentation and post-speech release behavior. SpriteForge's exact clip preview
therefore does not reproduce an entire speaking turn.

The intended outcome is one reusable Animator runtime, an independently runnable
player, and host adapters. A creator should be able to adjust a character's
performance and verify it without starting the complete Amadeus application.
Amadeus and the SpriteForge performance preview should eventually consume that
same implementation.

## 2. Proposed ownership

| Owner | Responsibility |
| --- | --- |
| SpriteForge authoring | Asset production, takes, QA, graph editing and export. |
| Character pack | Indexed clips, graph topology, mouth geometry and anchors, and the character-specific performance configuration needed by Animator. |
| Animator director | Graph traversal, expression-to-route mapping, transitions, loops, holds, speech lifecycle and visual interruption. |
| Animator rendering and resources | Sprite and mouth composition, texture loading, prefetching, cache budgets and resource disposal. |
| Host adapter | Translate existing host events into Animator input; connect audio playback, asset access and the drawing surface. |
| Application host | Interpret content, choose intent, synthesize/play speech and decide which source owns presentation. Identity, permissions and durable business state stay here. |

These are responsibilities within a proposed SDK, not a commitment to six
packages. Start with the existing JS runtime and PixiJS renderer. Keep the
performance rules independent of the DOM, transport and renderer-specific
texture objects where extraction makes that separation useful.

## 3. Performance contracts

- Preserve authored graph semantics: positive weights drive automatic traversal;
  zero-weight edges are manual routes; clip timing and loop modes remain explicit.
- Move character-specific label groups, mappings and geometric adjustments out
  of Amadeus into a supplied character configuration. Derive the minimum shape
  from existing callers before extending the pack contract.
- Separate silence during speech from completion. Silence keeps the base loop
  moving and closes the mouth through its overlay and per-frame anchor. Completion
  can hold a closed frame, play the appropriate exit and return to idle.
- Apply mouth composition to declared speaking assets. Transition clips preserve
  their authored appearance; existing behavior must be recorded before changing
  asset eligibility.
- Accept expressions, parsed script cues and speech start/progress/completion/
  cancellation from the host. Keep TXT syntax support in an optional adapter;
  arbitrary text interpretation remains with the application or its model.
- Use the host's actual playback state and progress for synchronization. Audio
  submission time alone does not measure acoustic presentation time.
- Keep source arbitration in the host. For events mirrored across synchronized
  surfaces, resolve randomized choices once before fan-out. Make clock and random
  inputs controllable for replay and semantic testing.
- Rendering limits and texture sampling preserve source timeline duration and
  required frames, including frames needed for mouth closure and holds.

This describes behavior, not a finalized public API or a new scripting language.

## 4. Migration stages and completion gates

### M0 — Placeholder (current)

Publish the purpose and draft plan. Make implementation status explicit.

### M1 — Inventory and behavior-preserving extraction

Record real callers of the Python Animator, browser runtime, rendering bridges
and pack loader. Establish replay fixtures for idle, expression changes, speech
pauses, completion and interruption. Extract the director and renderer without
changing observed behavior; retain only compatibility paths with proven callers.

**Gate:** fixtures agree on graph transitions, holds, speech exits and mouth
visibility. The extracted runtime can be exercised without an Amadeus server.

### M2 — Standalone performance player

Build a small player that loads a character pack and consumes parsed script cues
and playback signals. Support replay and inspection of expression, silence,
completion and cancellation. Use redistributable synthetic assets for the
included example; separately supplied character packs can provide additional QA.

**Gate:** a scripted performance completes independently, including a silence
pause, a normal speech exit and an interruption. Invalid packs report an
observable failure without leaving the previous character partially replaced.

### M3 — Shared preview and Amadeus integration

Connect SpriteForge's full-performance preview and Amadeus's character surfaces
to the SDK. Keep exact clip inspection available in the authoring tool. Move
character configuration together with its consumer, exercising producer and
runtime validators for any format change. Reduce Amadeus to host adaptation and
presentation ownership; remove replaced implementations after proving caller
coverage.

**Gate:** the same pack and performance input follow the same rules in the
standalone player, authoring performance preview and Amadeus. Speech completion,
interruption and multi-source handoff remain correct in Amadeus.

### M4 — Measured resource optimization

Consolidate decode concurrency, prefetch priorities, active texture residency and
cache eviction. Define load/unload/dispose behavior and ownership of shared
textures. A current frame or mouth overlay must remain valid while in use.

Measure cold start, first-expression latency, frame pacing and steady-state
resource usage. Separate CPU observations, decoded texture estimates and actual
GPU measurements; do not label one as another. Exercise repeated character loads,
switches and disposal before claiming a memory improvement.

**Gate:** publish a reproducible baseline and comparison, with bounded residency
under the tested budget and no regressions in performance timing or mouth closure.

## 5. First usable release acceptance

- Load a validated pack and render a complete performance without Amadeus's
  server, model stack or installed character assets.
- Consume parsed text cues and external speech playback signals; synchronize
  expressions and mouth presentation with the playback lifecycle.
- Keep motion continuous during speech silence, settle correctly after speech
  and cancel obsolete performance work on interruption.
- Change character-specific performance configuration independently of Amadeus.
- Embed the same SDK through a thin Amadeus adapter and use it in a SpriteForge
  performance preview.
- Expose resource cleanup and enough state to reproduce failures; verify repeated
  load/unload behavior with a redistributable example.

## 6. Decisions to resolve with implementation evidence

- The minimum character configuration and whether a versioned pack extension is
  required; initially consume the existing v1 format and runtime configuration.
- Ownership and packaging of shared pack validation across authoring and runtime.
- The first supported TXT syntax, derived from an actual scripted-player caller.
- Embedding lifecycle and API shape, derived from the player and Amadeus adapter.
- SDK distribution and licensing, with upstream provenance preserved before code
  extraction. This documentation placeholder copies no upstream implementation.
- Whether any caller needs a separate player process or IPC/WebSocket transport.
  The initial target is an embeddable SDK and standalone player.

Release dates, package names and platform guarantees are not assigned yet.

## 7. Source baseline

The following public revisions were checked on 2026-10-02. These references
describe existing source behavior, not completed work in this repository.

- Amadeus: [`97f6d17b22a5c243d1e9036e92326ccb8057ef11`](https://github.com/Code-Amadeus/Amadeus/tree/97f6d17b22a5c243d1e9036e92326ccb8057ef11).
  Its [pack contract](https://github.com/Code-Amadeus/Amadeus/blob/97f6d17b22a5c243d1e9036e92326ccb8057ef11/docs/character_pack_authoring.md#L99-L102)
  leaves character-specific aliases and speaking policy in Amadeus. The
  [Python Animator](https://github.com/Code-Amadeus/Amadeus/blob/97f6d17b22a5c243d1e9036e92326ccb8057ef11/render/spriteforge_animator.py#L546-L680)
  supplies mouth/runtime configuration, and the
  [browser renderer/runtime](https://github.com/Code-Amadeus/Amadeus/blob/97f6d17b22a5c243d1e9036e92326ccb8057ef11/render/web/renderer.js#L1067-L1504)
  executes speaking and post-speech behavior.
- SpriteForge: [`4e5221413278911a78894e9fa463bbe529463be1`](https://github.com/Code-Amadeus/Amadeus-SpriteForge/tree/4e5221413278911a78894e9fa463bbe529463be1).
  Its [architecture notes](https://github.com/Code-Amadeus/Amadeus-SpriteForge/blob/4e5221413278911a78894e9fa463bbe529463be1/docs/architecture.md#L25-L38)
  distinguish host semantics from authoring and state that standalone node preview
  is not an entire speaking-turn implementation.

## 中文计划草案

### 目标与当前状态

本仓库目前只包含占位说明与计划，尚未实现 SDK、播放器或稳定 API。以下内容是
2026-10-02 的讨论草案，供确定边界和实施顺序，不代表已经完成或承诺交付日期。

SpriteForge 导出的素材与行为图还不足以重现 Amadeus 中完整的一段角色表演：
说话节点分组、表情映射、过渡停留、嘴部表现和说完后的退出策略仍由 Amadeus
提供。目标是把这些可复用责任集中到 Animator，让创作者无需启动整个 Amadeus
也能修改、播放和验证角色表现，并让编辑器演出预览、独立播放器与 Amadeus
使用同一套运行时。

### 职责边界

| 部分 | 负责内容 |
| --- | --- |
| SpriteForge 创作工具 | 素材生成、候选与 QA、行为图编辑和导出。 |
| 角色包 | 帧索引、图拓扑、嘴部几何与锚点，以及角色专属的演出配置。 |
| Animator 导演逻辑 | 图执行、表情到路线的映射、过渡、循环、停留、语音生命周期和演出中断。 |
| Animator 渲染与资源 | 帧与嘴部合成、KTX2 加载、预加载、缓存预算和资源释放。 |
| 宿主适配器 | 把现有宿主事件接入 Animator，连接音频播放、素材访问和画布。 |
| 应用宿主 | 理解内容、选择表达意图、生成与播放语音、决定哪个来源拥有呈现权；身份、权限和持久业务状态仍由宿主负责。 |

这些是一个 SDK 内的责任划分，不预设要拆成多个包。第一步沿现有 JS 运行时和
PixiJS 渲染器提取；根据实际需要，让演出规则脱离 DOM、传输和具体纹理对象。

### 导演规则与同步原则

- 保留行为图的现有语义：正权重参与自动跳转，零权重用于手动路线；帧间隔与
  循环模式保持明确。
- 角色专属的节点分组、映射和几何修正进入随角色提供的配置。先从现有调用方
  提取最小契约，再判断角色包是否需要版本扩展。
- 区分两种闭口：说话中的静音保持基础动画运动，通过覆盖层和逐帧锚点闭口；
  一句话结束后则可以停在闭口帧，播放对应的退出，再回到 idle。
- 嘴部合成作用于明确声明的说话素材，过渡片段保持创作时的表现。改变适用范围
  前先记录并验证现有行为。
- 主入口接收表情、解析后的脚本事件，以及语音开始、进度、完成和取消。TXT
  语法放在可选适配器，自然语言理解仍由应用或模型负责。
- 同步依赖实际音频播放状态和进度；音频提交时间本身不等于扬声器出声时间。
- 来源优先级由宿主决定。需要同步的多个显示面复用扇出前已解析的随机选择；
  时钟和随机输入可控制，以便重放和验证。
- FPS 限制与纹理采样保留原时间线总时长和必要帧，包括闭口与停留所需的帧。

这些描述行为契约，具体 API 和脚本语法要由真实调用方验证后确定。

### 分阶段实施与验收

**M0：占位仓库（当前阶段）。** 公开用途与草案，明确实现状态。

**M1：盘点调用方，保持行为地提取。** 检查 Python Animator、浏览器 runtime、
渲染桥和角色包加载器的真实调用方；建立 idle、表情切换、语音暂停、完成与中断
的重放样例，再提取导演与渲染逻辑。兼容路径必须有活跃调用方证据。
验收：图跳转、停留、退出和嘴部可见性与原实现一致，并能脱离 Amadeus 服务验证。

**M2：独立演出播放器。** 加载角色包，消费解析后的脚本事件与音频播放信号，
支持演出重放和检查。仓库示例使用可再分发的合成素材，真实角色包用于额外验收。
验收：独立完成包含静音、正常收尾和中断的演出；非法包产生明确错误，并保留
此前角色的完整状态。

**M3：统一预览与 Amadeus 接入。** SpriteForge 的完整演出预览与 Amadeus
角色界面接入同一个 SDK，创作工具保留精确片段检查。角色配置与消费者一起迁移；
格式变化同时验证生产端和运行时。证明调用方已覆盖后，移除替代掉的实现。
验收：同一角色包与同一演出输入在三个入口遵循相同规则，Amadeus 的语音完成、
中断和多来源交接保持正确。

**M4：依据测量优化资源。** 统一解码并发、预加载优先级、常驻纹理与缓存淘汰，
明确加载、卸载、销毁及共享纹理所有权；使用中的角色帧和嘴部覆盖必须保持有效。
记录冷启动、首个表情延迟、帧间隔和稳定运行资源占用，区分 CPU 观测、纹理容量
估算与实际 GPU 测量，覆盖反复加载、切换与销毁。
验收：提供可复现的基线和对比，在已测预算内保持有界资源驻留，并保留时序与
闭口效果。

### 第一版可用条件

- 无需 Amadeus 服务、模型栈或已安装的角色素材，加载有效角色包并播放完整演出。
- 接收解析后的文本事件与外部语音播放信号，按实际播放生命周期同步表情和嘴部。
- 静音时动作连续，结束后正确收尾，中断后取消过时的演出工作。
- 角色专属演出配置可独立于 Amadeus 修改和验证。
- Amadeus 通过薄适配器接入同一 SDK，SpriteForge 使用它提供完整演出预览。
- 有明确资源释放接口和可重放状态，并以合成示例验证反复加载与卸载。

### 待实现证据确定的事项

角色配置的最小结构与是否扩展 v1；创作端与运行时共享校验的归属；首个 TXT
调用方需要的语法；播放器与宿主共同验证的 API／生命周期；保留上游来源记录的
SDK 分发与许可证安排；是否有实际调用方需要独立进程或 IPC／WebSocket。
目前目标是可嵌入 SDK 和独立播放器，尚未指定发布日期、包名或平台支持承诺。

源代码依据见上面的 [Source baseline](#7-source-baseline)，均链接到已核对的公开
提交；本占位仓库没有复制上游实现。
