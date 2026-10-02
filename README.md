# Amadeus-SpriteForge-Animator

A planned character runtime SDK and standalone player for SpriteForge character
packs. It aims to turn expressions, animation cues and speech playback signals
into a coherent character performance, with shared rendering and resource
management that applications can embed.

**Status: placeholder and design draft.** This repository currently contains
documentation only. There is no installable SDK, runnable player or stable API.

## Purpose

[SpriteForge](https://github.com/Code-Amadeus/Amadeus-SpriteForge) creates and
exports character assets and behavior graphs. Today, additional performance
rules live in [Amadeus](https://github.com/Code-Amadeus/Amadeus): speaking routes,
transition holds, silence mouth overlays, post-speech exits and runtime loading.

Amadeus-SpriteForge-Animator is intended to give those reusable responsibilities
one home, so the authoring preview, a standalone player, Amadeus and other
applications can use the same runtime and improve it independently.

## Planned scope

- Behavior-graph execution and character performance rules: expressions,
  transitions, loops, holds, speech completion and interruption.
- Mouth presentation: silence closure, frame-specific anchors, overlays and
  closed-mouth settling after speech.
- Animation delivery from parsed TXT or other scripted cues, with an optional
  text-format adapter and synchronization to actual audio playback.
- Sprite rendering, KTX2 loading, prefetching, texture residency budgets and
  explicit resource cleanup.
- A standalone player for replaying and inspecting performances, plus thin
  adapters for embedding the SDK in host applications.

Hosts supply semantic intent, audio playback state and the selected presentation
owner. SpriteForge owns asset production, editing and export. Animator owns how
the selected performance plays out. The initial direction is an embeddable
JS/TS runtime with a PixiJS renderer; process or network integration can be added
when a real caller needs it.

See [PLAN.md](PLAN.md) for the draft boundary, migration stages, acceptance
criteria and unresolved decisions. The plan is a proposal, not a release schedule
or compatibility promise.

## 中文说明

Amadeus-SpriteForge-Animator 计划提供 **SpriteForge 角色运行时 SDK 与独立播放器**，
把表情、动作、解析后的 TXT 演出事件和语音播放信号组织成连贯的角色表演。

**当前为占位仓库与设计草案，只有文档；SDK、播放器和稳定 API 尚未实现。**

目前，角色行为图之外的说话路线、过渡停留、静音闭口、说完后的收尾与资源加载等
逻辑仍在 Amadeus。这个项目希望把可复用的角色演出逻辑独立出来，让 SpriteForge
预览、独立播放器、Amadeus 和其他应用共享同一套实现，单独修改、验证和优化。

计划涵盖行为图与导演规则、嘴部覆盖和逐帧锚点、演出事件与音频同步，以及 KTX2、
纹理缓存、预加载、资源预算和释放。宿主决定表达内容、音频播放状态与呈现来源；
SpriteForge 负责素材创作和导出；Animator 负责角色怎样演出来。

详细边界、迁移阶段和第一版验收条件见 [计划草案](PLAN.md#中文计划草案)。
