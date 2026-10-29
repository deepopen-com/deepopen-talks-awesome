# 交互视频前沿 2026

## ——从 Web 播放器到可玩游戏世界：20+ 开源项目深度解读

---

# 前言

2026 年，"视频"正在经历自流媒体诞生以来最大的一次范式转移。

过去十年，视频是"扁平"的：无论内容多精彩，观众始终是旁观者，能做的只有播放、暂停与拖动进度条。而 2026 年的开源社区里，两个方向的汇流正在终结这个时代——

**第一个方向是"交互性烧录进视频"**。从把状态机编译进视频字节的 AVAL，到"编辑器即播放器"的 FrameTrail，视频不再只是媒体文件，而成为可编程的、带状态的、可分支的活物。

**第二个方向是"视频在被实时生成"**。Skywork 的 Matrix-Game 让你在浏览器里用键鼠玩一个神经网络正在逐帧画出来的世界；NVIDIA 为这类"交互式世界模型"专门造了推理服务库；北大袁粒团队用 14B 参数模型把实时长视频生成的成本压到一张 H20 之下。视频的"播放"与"渲染"正在合二为一。

本书源于一次系统的 GitHub 调研：从「Web 交互视频框架」「AI 实时交互视频生成/世界模型」「分支叙事工具」三个方向出发，筛出 2026 年内仍在活跃更新的 20+ 前沿项目，逐一深入解读其定位、原理、架构与用法。书中所有数据（star 数、FPS、显存需求、更新日期）均取自 2026 年 10 月下旬各项目仓库与论文的实时状态。

**本书适合谁读**：前端工程师（第一篇）、AI/图形学研究者与推理工程师（第二篇）、互动叙事创作者与技术负责人（第三篇），以及所有想理解"视频的下一形态"的人。

**如何阅读**：三篇相互独立，可按需跳读。赶时间者可直接读第 21、22 章的技术版图与趋势总结。

---

# 导论：什么是交互视频

在展开具体项目之前，先建立一个统一的分析框架。

**交互视频（Interactive Video）** 泛指一切允许观众输入影响视频呈现的视频形态。按"交互信号"与"视频来源"两个维度，可以划出一张谱系图：

按交互信号分：

| 交互信号     | 代表形式      | 典型项目                         |
| -------- | --------- | ---------------------------- |
| 时间点（cue） | 到点弹题、弹层   | QuizPlayer、IVLabs Core       |
| 点击热区     | 超链接视频、注释  | FrameTrail、IVKit             |
| 状态机      | UI 动效状态切换 | AVAL                         |
| 分支选择     | 互动剧、CYOA  | OpenFMV、Sogni Worlds、NextCut |
| 键鼠/手柄    | 可玩世界      | Matrix-Game、ForgeWM          |
| 相机位姿     | 3D 场景漫游   | HunyuanWorld-Voyager         |
| 语音       | 实时数字人对话   | LiveTalk、Vidu S              |
| 稀疏关键帧    | 视频修复控制    | SparkVSR                     |
| 运动轨迹     | 逐帧引导生成    | MotionStream                 |
| 潜在动作     | 无监督动作迁移   | Olaf-World                   |

按视频来源分，只有两大类：

1. **预生成视频 + 交互运行时**（第一、三篇的主角）：视频内容提前制作完毕，交互层决定"看哪段、何时看、以什么状态看"。技术上可控、成本低，但内容是封闭集。
2. **实时生成视频 + 交互条件**（第二篇的主角）：视频本身在被逐帧合成，用户的键鼠、语音、相机动作直接作为生成条件注入扩散模型。这是 2026 年最前沿、进展最快的方向。

一个重要的判断贯穿全书：**这两个方向不是竞争关系，而是同一枚硬币的两面**。实时生成本质上是把"内容创作"也从预生成搬到了运行时——当生成速度追上播放速度，交互视频与"可玩世界"就不再有边界。

---

# 第一篇 Web 交互式视频框架与引擎

## 篇首语：五个抽象层级

第一篇的五个项目看似同属"Web 交互视频"，实则各占一个抽象层级，合起来恰好构成一张完整的层次图：

- **编解码格式层** —— AVAL：把交互性（状态机）直接编译进视频文件的字节里；
- **播放引擎层** —— Liqvid：视频即 React 应用，时间轴驱动的可编程视频；
- **数据标准层** —— FrameTrail：以 W3C Web Annotation 为基础的开放超视频数据模型；
- **组件内核层** —— Interactive Video Labs Core：可嵌入任何前端框架的最小 cue 内核；
- **创作工具层** —— OpenFMV：给非程序员用的可视化分支叙事编辑器。

理解了这个分层，选型就变得直观：你要改的是"视频字节"还是"播放行为"？要服务的是"程序员"还是"创作者"？答案不同，归宿不同。

---

## 第 1 章 AVAL：把状态机烧录进视频字节

> 仓库：pixel-point/aval ｜ 约 1.6k★ ｜ TypeScript ｜ 2026-07 创建，持续更新

### 1.1 它解决什么问题

设想一个网页里的"支付成功"动画：用户点击后，画面要从"加载中"的循环精准过渡到"成功"的庆祝帧。传统做法是播放一段预渲染视频，到点用 `seek()` 跳到另一个片段——但 `seek` 有延迟、有接缝、有卡顿，状态越多越糟。

AVAL 的答案激进而优雅：**不要在运行时去 seek 视频，而是把状态机在编译期就烧录进视频文件本身**。一个 `.avl` 文件不只是视频字节，它同时携带状态图、时间轴、转场授权和完整性校验数据——交互性成了视频格式的一部分。

### 1.2 核心概念：交互视频格式（wire format 1.1）

一个 `.avl` 文件由六部分构成：有界容器、基础编解码器载荷、状态图、时间轴、完整性校验关系、资源声明。发布一个逻辑动画时，编译器会产出**一组编解码器 bundle**——同一份状态逻辑分别编码为 AV1、VP9、H.265、H.264 四个 `.avl` 文件，浏览器按固定梯队自动选择（通过 `data-codec` 属性声明，与 DOM 顺序无关）。

状态机层面，AVAL 支持一套相当丰富的原语：部分循环、有限体（finite bodies）、held bodies、portals、finish routes、locked bridges、cuts、resident reversible transitions——状态、触发器、可逆转场全部是编译期授权数据，且**每个 codec 文件中的状态图完全一致**。

最独特的设计当属 **packed alpha 见证机制**：1.1 格式要求带透明通道的 rendition 附带 1–8 个采样点"证人"（坐标 + 期望值域），运行时在播放前解码这些帧并校验 alpha 像素——如果某个浏览器"能解码但解错了"，会在就绪检查阶段被直接拒绝。这相当于给解码器输出做资格认证，是笔者在视频格式领域罕见的工程巧思。

### 1.3 技术架构

AVAL 是一个 monorepo，职责划分清晰：

- `@pixel-point/aval-graph`：确定性状态与路由引擎。`setState()` 采用 latest-wins 语义，失败以类型化的 `AbortError`/`RouteError` 抛出；
- `@pixel-point/aval-format`：严格的解析、校验与写入器；
- `@pixel-point/aval-element`：Web Component `<aval-player>`，是唯一的浏览器加载器、解码调度器与渲染器；
- `@pixel-point/aval-compiler`：基于系统 FFmpeg（libaom-av1 / libvpx-vp9 / libx264 / libx265）的编译器，产出 `vp9.avl` + `build.json` 等产物目录；
- 另有 React / Svelte 集成包（SSR 安全，通过 client ref 注册）。

关键点：**转场不通过 seek video element 实现**，循环接缝不会因媒体 seek 而停顿——帧精确是结构保证，不是性能优化。

### 1.4 典型用法

先用 `motion.json` 声明 canvas、帧率、units（如 `{id:"idle.body", range:[0,120], playback:"loop"}`）、states、edges、bindings，然后编译：

```bash
npx @pixel-point/aval-compiler compile motion.json --out dist/motion
```

浏览器侧：

```html
<aval-player id="motion" width="320" height="320">
  <source src="/motion/vp9.avl" data-codec="vp9">
</aval-player>
```

```js
defineAvalElement();
await motion.prepare();      // 失败抛 AvalPlaybackError
await motion.setState("success");
```

一个值得注意的设计哲学：AVAL **刻意不提供 fallback UI**——应用必须自行处理 `error` 事件与 `failure.code`（如 `unsupported-browser`）。这把兼容性责任明确推给了集成方。

### 1.5 评价与局限

AVAL 与 Lottie、Rive 同处"UI 动效"赛道，但策略不同：后两者用运行时引擎画矢量，AVAL 直接发布视频编码，换来的是影视级质感与浏览器原生解码性能。2026 年 7 月创建、三个月 1.6k star，是本篇热度最高的新项目。

局限同样明显：项目非常年轻（兼容性表、官方演示素材在 TODO 中尚未完成）；编码链依赖系统 FFmpeg；只覆盖"短循环动效 + 状态切换"，不适合长视频叙事；格式中的编解码专利责任明确归发布者承担。适合营销落地页、品牌动画、微交互动效等场景先行试点。

---

## 第 2 章 Liqvid：视频即 React 应用

> 仓库：liqvidjs/liqvid ｜ 约 820★ ｜ TypeScript ｜ 2018 年立项，2026-10 仍活跃

### 2.1 定位与血统

如果说 AVAL 在改造"视频文件"，Liqvid 则干脆取消了"视频文件"这个概念——**视频不是媒体，而是一个按时间轴驱动的 React 应用**。它脱胎于数学教学视频的实战（生态内置 KaTeX、MathJax、Desmos、XyJax 集成），2018 年立项至今七年，版本 2.x，2026 年 10 月仍在活跃提交。

### 2.2 核心概念：Playback / Script / Player 三层

- **Playback**：处理播放、倒退、音量、倍速，刻意模仿 `HTMLMediaElement` 接口且**不依赖 React**——这为将来支持 Vue 或 Custom Elements 留了后路；
- **Script**：将 Playback 切分为命名 **markers**（标记），内部存储为 `[name, start, end]` 三元组；创建时先写 `[name, duration]`，以便后续重排片段；
- **Player**：提供图形界面外壳。

标记驱动的显隐是 Liqvid 的标志性写法：`data-during="intro/"` 表示当前 marker 前缀匹配时该元素可见，`data-from-first` / `data-from-last` 控制子元素的出现时机。一个源自实战的细节：内部用 `opacity:0; pointer-events:none` 隐藏而非 `display:none`——避免 reflow 与布局闪现（教训来自 MathJax 的慢渲染），且不受 React 重渲染管理。

### 2.3 独特卖点：录制约时

Liqvid 的开发流程是为"讲解型视频"量身定制的：开发者按 E/W 键在 marker 之间实时翻转，**录制约时自动回填 duration**。也就是说，你可以像讲 PPT 一样"演一遍"视频，时间轴自动生成——这在对口播节奏敏感的数学科普场景里是杀手级体验。

### 2.4 动画与交互

动画有两条路线：① `playback.newAnimation()` 包装 Web Animations API，返回的回调作为 ref 附加到元素，`combineRefs()` 可组合多个动画；② `useTime()` hook，每次时间推进或 seek 时回调当前毫秒值，适合完全自定义的绘制。

交互方面：默认点击画布即暂停，但智能豁免 `<a>`、`<button>`、`<input>` 等天然可点击的标签，其他元素加 `data-affords="click"` 即可豁免；另提供 `dragHelperReact` 拖拽工具与 `canvasClick` 事件总线。

最小示例：

```bash
npm init liqvid my-video
```

```tsx
const script = new Script([["intro/", "0:01.5"], ["plan/1", "0:01"]]);
<Player script={script}>
  <section data-during="intro/">
    <h1>Hello <span data-from-first="intro/world">World!</span></h1>
  </section>
</Player>
```

### 2.5 评价与局限

与 Remotion 对比最能看清定位：Remotion 面向"渲染成 MP4"（React 只是渲染模板），Liqvid 面向"在浏览器里活着的视频"（React 应用本身就是视频）。前者交付文件，后者交付体验。

局限：社区规模有限；captioning、Web Components 化的 `<Player>`（v3 方向）、GSAP/MathJax 深度集成仍在开发中；移动端需要 fake fullscreen 与 polyfill 补丁；无可视化编辑器，纯代码创作门槛较高——它是给"会写 React 的科普作者"的工具。

---

## 第 3 章 FrameTrail：开放超视频的十年长跑

> 仓库：OpenHypervideo/FrameTrail ｜ 132★ ｜ 2016 年立项，2026-10 密集更新

### 3.1 理念：与"渲染"为敌

FrameTrail 是本书调研中历史最悠久的项目（2016 年至今近十年），由超视频（Hypervideo）学者 Joscha Jäger 等人维护。它开宗明义的宣言值得全文引用：

> **"Film shall be programmed with Open Web Technologies."**（电影应当用开放的 Web 技术来编程。）

传统的"渲染"是把影片永久封存——把媒体资产、剪辑、文字叠加、特效、动画不可逆地烧进一个平面视频文件。FrameTrail 反其道而行：创造一个**永久开放的格式**，可以在任何浏览器里被观看、混剪、再编辑，直到永远。

### 3.2 三个核心概念

1. **Player and Editor are one**：编辑器与播放器不分离。"能看就能改"——每个 hypervideo 都自带 FrameTrail 本体，拥有编辑权限即可直接保存修改，或下载导出到其他实例继续编辑。
2. **Timebased Documents**：任何视频、甚至"只有时长的空画布"都可以作为时间轴基底，用来同步交互字幕、叠加层与注释流。
3. **Hyperlinked Videos**：用可点击热区把多个视频连成非线性网络——分支叙事、互动讲解的原生表达。数据模型遵循 **W3C Web Annotation** 标准。

### 3.3 架构：无数据库的可移植性

纯 Web 技术栈（JavaScript + PHP 服务端），所有数据存为 `_data` 目录下的 **JSON 文件 + JSON Schema 校验**——没有数据库。这意味着：把 `_data` 文件夹拷走，实例就完成迁移；内置导出功能可以把整个 hypervideo 导出为**一个 HTML 文件**，离线双击即可播放，再导入任何 FrameTrail 实例又能继续编辑。

三种运行模式对应三档能力：

| 模式                            | 依赖          | 能力                                          |
| ----------------------------- | ----------- | ------------------------------------------- |
| Server mode                   | PHP 7.4+    | 多用户认证、服务端转码/缩略图、协作编辑                        |
| Local folder mode             | Chrome/Edge | 用 File System Access API 直接读写本地 `_data` 文件夹 |
| In-memory / Project File mode | 无           | 整个项目就是一个 HTML 文件，保存即写回自身                    |

### 3.4 典型用法

零代码使用为主：`docker compose up -d` 或 `php -S localhost:8080` → 设置向导 → 编辑模式新建 hypervideo → 把资源拖上时间轴作为 overlay/annotation → Ctrl+S 保存。文档体系极为完整：ARCHITECTURE、DATA-MODEL、TYPES、HTML-FORMAT、EXTENDING（不改核心的插件机制）、EVENTS（供宿主页与分析系统消费的事件 API），甚至还有 PAIRING——用二维码把手机配对为第二屏同步显示。

### 3.5 评价与局限

FrameTrail 的价值在于**数据主权与理念完整性**：它是唯一"编辑器与播放器一体 + 数据完全可移植 + 遵循 W3C 标准"的方案。典型场景是学术教育（LMS 集成、OIDC 认证、API token）、纪录片注释、数字人文与博物馆展示。

局限：PHP 服务端路线在 2026 年略显非主流；132 star 属小众学术 + 独立创作社区；协作编辑仅在 server mode 可用；交互深度受限于 overlay/annotation 类型体系，没有分支状态机类编程接口。2026 年 10 月它经历了密集的现代化重构（Docker 化、测试体系、配套屏配对、Quiz 类型扩展），呈现出"老树发新芽"的态势——如果你要做注释驱动的视频项目，它仍是开放标准一侧的最佳选择。

---

## 第 4 章 Interactive Video Labs Core：最小可用的 Cue 内核

> 仓库：interactive-video-labs/interactive-video-core ｜ 早期项目（个位数 star）｜ TypeScript ｜ 2026-10 活跃

### 4.1 定位

`@interactive-video-labs/core` 是一个框架无关的 TypeScript 交互视频引擎，口号是"轻量、可扩展、cue 驱动"。项目 2025 年 7 月创建，star 数仍是个位数，但它代表了一种值得记录的架构样本：**把交互视频压缩为一个最小内核，再用四套薄 wrapper 适配主流框架**——React、Vue、Svelte、Angular 各一个独立仓库，全部 2026 年 10 月更新。

### 4.2 核心概念：一切皆 Cue

Cue-based 模型把所有交互统一为"挂在视频时间点上的对象"：

```ts
{ id: string, time: number, label: string, payload: object }
```

payload 携带交互定义（选择题、文本输入、视频分段跳转）或纯消息。视频播放时，`timeupdate` 事件驱动 Cue Handler 逐个触发到点的 cue；用户对"choice-video-segment-change"类型 cue 做出选择后，Segment Manager 无缝切换到另一段视频——这就构成了最小可用的分支叙事原语。

### 4.3 架构剖析

内核模块清单本身就是一份"交互视频播放器应有什么"的教科书答案：

- `IVLabsPlayer`：总编排（播放、cue、交互、分析）；
- **播放器状态机**：idle / playing / waitingForInteraction 等状态迁移；
- Cue Handler 与 Interaction Manager：渲染各型交互并处理响应；
- Segment Manager：多段视频的无缝过渡；
- Analytics hook 系统；
- **`DecisionAdapter` 接口**：决策持久化抽象，内置 `InMemoryDecisionAdapter` 与 `LocalStorageDecisionAdapter`，提供 `getDecisionHistory()` / `clearDecisionHistory()`——为个性化学习与学情分析预留了数据出口。

用法一目了然：

```ts
const player = new IVLabsPlayer('player-container', {
  videoUrl: 'https://...mp4',
  cues: [{
    id: 'segmentChange', time: 10, label: 'Segment Change',
    payload: { interaction: { type: 'choice-video-segment-change',
      options: [
        { level: 'Segment A', video: '...1.mp4' },
        { level: 'Segment B', video: '...2.mp4' }] } } },
  ],
  initialState: 'idle',
  decisionAdapter: new LocalStorageDecisionAdapter(),
});
player.init();
```

### 4.4 评价与局限

这是全书中成熟度最低、但架构最干净的项目之一。交互类型有限，分支只有一层"跳段"语义，没有嵌套状态图；但它演示了 cue-based 引擎应有的全部关节（状态机、cue 调度、交互管理、决策持久化），且四框架 wrapper 让它可直接作为产品内嵌组件。与 H5P 互动视频相比，它更轻、无平台依赖；与 Liqvid 相比，它是"配置驱动"而非"代码驱动"。适合在线教育/微课程团队作为起步内核或教学案例。

---

## 第 5 章 OpenFMV：本地优先的可视化分支叙事编辑器

> 仓库：Comedian1926/OpenFMV ｜ 229★ ｜ Next.js + Electron ｜ 2026-05 创建，快速迭代

### 5.1 定位：AI Native + Local-first

OpenFMV 自称"AI Native 交互内容编辑器"，用于制作交互视频、分支叙事和互动短剧，最终可打包为独立桌面应用。它是个人开发者项目，2026 年 5 月创建，五个多月积累 229 star，提供中/日/韩/英四语 README——对中文读者尤其友好。

两条设计底线贯穿项目：**Local-first**——无账号、无托管数据库、无云存储，项目 JSON 与媒体资产全部留在本机；**AI 只做辅助**——帮助写作与构思，不做端到端全自动生成。

### 5.2 双层创作模型

OpenFMV 的创作界面分为两层：

- **宏观层 Blueprint graph**：React Flow 节点图，把交互场景、分支输出、故事流连接成一张图——这是"剧情结构"的视图；
- **微观层 FlowTimeline**：每个节点/场景是一条独立时间轴，包含 **media tracks**（视频/图像/音频 clip）与 **interaction tracks**（按钮、热区 hotspot、暂停门 pause gate、定时分支 timed branch、变量动作 variable action 的交互 clip）——这是"场景内部"的视图。


这种"图 + 轨"双层结构，恰好对应互动叙事的两级抽象：故事怎么走（图），每一步怎么演（轨）。

### 5.3 技术架构

Next.js 16 App Router + React + TypeScript，Electron 桌面壳（打包后后台启动本地 Next.js standalone 服务再打开主界面），React Flow 做图编辑，Zustand 管状态，next-intl 做国际化。最有价值的工程决策是：**preview 与 export 共用同一个 `shared/runtimeCore.mjs` 运行时**——你在编辑器里预览到的行为和导出包的行为严格一致，不存在"预览一个样、成品另一个样"。导出时重写 timeline clip 的 `src`/`poster` 为本地包结构，桌面发布走 Electron Builder + NSIS 自动更新（`latest.yml` + `.blockmap`）。

开发纪律也写得明确：禁止引入托管后端与云同步。

```bash
npm install
npm run dev            # Web 版 http://localhost:3000
npm run desktop:dev    # Electron 开发
npm run package:desktop
```

### 5.4 评价与局限

OpenFMV 是第一篇里"发展最快但也最不成熟"的项目：单人维护，Windows 桌面打包为主要目标，明确列出边界——无协作、无云同步、无 Windows EXE 故事打包、AI 非全自动；变量动作/暂停门等机制的文档尚浅。

与同类对比：FrameTrail 是"注释/混剪优先 + Web 一体"，OpenFMV 是"分支叙事优先 + 桌面编辑器"；与 Twine 相比，它多了真实媒体时间轴与视频播放运行时。它的更大意义在于代表了一股趋势：**local-first + AI Native 的创作工具正在从开源视频编辑器浪潮中生长出来**（致谢名单里的 OpenCut 即是同浪潮项目）。

---

## 第 6 章 第一篇总结：选型对照

| 项目 | 交互模型 | 创作方式 | 数据形态 | 成熟度 | 最佳场景 |
|---|---|---|---|---|---|
| AVAL | 编译进资产的状态机 | `motion.json` 声明式编译 | 二进制 `.avl`（多 codec bundle） | 新但工程质量高 | UI 动效、品牌页 |
| Liqvid | marker + 可编程 React | 纯代码 + 录制约时 | React 应用本身 | 7 年 / 2.x | 数理科普、讲解视频 |
| FrameTrail | 热区链接 + overlay/注释 | 可视化编辑器=播放器 | JSON + W3C Web Annotation | 10 年 / 学术血统 | 教育、纪录片、人文 |
| IVLabs Core | cue 回调 + 跳段 | 代码配置 cues | TS 对象 + 适配器持久化 | 早期 | 在线教育内嵌组件 |
| OpenFMV | Blueprint 图 + 交互轨道 | 可视化双层面板 | 本地项目 JSON | 快速演进期 | 互动短剧原型 |

一句话选型：**改视频字节选 AVAL，写可编程视频选 Liqvid，做注释与 remix 选 FrameTrail，嵌产品组件选 IVLabs，给创作者做分支剧选 OpenFMV。**

---

# 第二篇 AI 实时交互视频生成与交互式世界模型

## 篇首语：2026 年最热的战场

第二篇的项目来自高校、大厂与创业公司，方法各异，但共享同一个目标：**让视频在播放的同时被生成，并让用户的输入实时改变生成结果**。阅读本章前，先建立三条主线：

**主线一：蒸馏初始化之争。** 自回归视频扩散要实时，必须把双向扩散模型蒸馏成少步因果模型。2025 年的 Self Forcing 有视觉伪影与曝光偏差问题；2026 年，清华与生数科技的 **Causal Forcing (+ +)** 给出了理论上更正确的配方，并迅速成为事实标准——Matrix-Game 4.0、ForgeWM、minWM 等多家 SOTA 世界模型都以它为初始化。这是本篇的"地基"，故第 7 章先行。

**主线二：交互信号谱系。** 键盘/鼠标（Matrix-Game、ForgeWM）、相机位姿（Voyager）、语音（Vidu S、LiveTalk）、稀疏关键帧（SparkVSR）、运动轨迹（MotionStream）、潜在动作（Olaf-World）——交互信号的选择决定了产品形态。

**主线三：硬件民主化。** 从 H100 数据中心（Helios 基准），到消费级 RTX 5080/5070 Ti（Matrix-Game 4.0），到 6GB 显存 offload（Helios 蒸馏版），再到免本地 GPU 的云端 SDK（Xmax）——实时交互视频生成正在快速下探。

---

## 第 7 章 Causal-Forcing：实时交互视频的"地基"

> 仓库：thu-ml/Causal-Forcing ｜ 约 1.0k★ ｜ ICML 2026 ｜ 2026-10 活跃

### 7.1 为什么把它放在第一篇之首

严格说，Causal Forcing 不是产品，而是**被整个行业采用的训练配方**。DreamX-World、Astronex-World、Zing-0.5、ZYT-World、Matrix-Game 3.5 与 4.0——2026 年多家 SOTA 交互世界模型的技术报告里都写着同一句话：以 Causal Forcing 为初始化。它的两个版本：Causal Forcing（arXiv:2602.02214，ICML 2026）与 Causal Forcing++（arXiv:2605.15141），出自清华朱军团队（THUML，联合生数科技、UT Austin、人大）。

### 7.2 问题：Self Forcing 错在哪

要把双向扩散模型变成实时的自回归生成器，主流做法是 Self Forcing：让学生在自身生成的轨迹上训练。但直接在早期做 DMD（分布匹配蒸馏）有两个恶果：一是出现网格状/蜡质伪影，二是早期误差经 self-rollout 逐步放大，形成**曝光偏差**——模型越生成越偏。

### 7.3 方法：三段式蒸馏

Causal Forcing 的答案是一套三段式管线：

- **Stage 1**：自回归扩散训练（frame-wise 或 chunk-wise）；
- **Stage 2**：用 **Causal ODE 蒸馏**做初始化。核心洞察：ODE/一致性蒸馏要求师生沿同一轨迹行进，所以必须用自回归教师做轨迹对齐的 ODE 蒸馏。Causal Forcing++ 进一步用 **Causal Consistency Distillation** 取代 ODE——免除昂贵的 ODE 配对数据生成（只用真值数据），并首次做出 frame-wise 2 步模型（质量反而优于 4 步）与 1 步模型；
- **Stage 3**：非对称 DMD。此时可以用双向强教师了——因为只要求匹配最终分布，不再要求轨迹对齐，配合动态时间步偏移。

一句话概括其理论贡献：**把"因果强制"作为不对称 DMD 的理论上正确的初始化**，视觉质量与运动动态全面超越 Self Forcing，而训练预算与推理效率不变。

### 7.4 实践要点

- 权重与代码：基于 Self-Forcing/CausVid/Wan2.1 代码底座，Apache 系开源，HuggingFace 提供 `zhuhz22/Causal-Forcing`；
- 推理示例：

```bash
python inference.py \
  --config_path configs/causal_forcing_dmd_framewise_2step.yaml \
  --checkpoint_path checkpoints/causal-forcing++/framewise-2step.pt \
  --data_path prompts/demos.txt --use_ema
```

- 训练规模：8 节点 × 8 卡（64 GPU）；
- 原生支持 T2V 与 I2V（帧级模型首 latent 直接置条件图）；原生不超 81 帧，配合 Rolling Forcing / Rolling Sink / Infinity-RoPE 可扩展到分钟级；
- 配套 minWM：基于混元 1.5-8B 的动作条件世界模型；
- 注意事项：BF16 训练可能崩，官方建议 FP32；2026-10 社区已有 ComfyUI 节点。

### 7.5 评价

如果你只想读一篇论文来理解 2026 年实时交互视频生成的技术核心，就是它。局限：训练需要大规模集群，长视频依赖外挂方案。对多数读者，它的价值是"读懂下游一切项目的前提"。

---

## 第 8 章 Matrix-Game 4.0：可玩游戏世界的工程标杆

> 仓库：SkyworkAI/Matrix-Game ｜ 约 2.8k★ ｜ Apache-2.0 ｜ 4.0 于 2026-10-15 发布

### 8.1 从 1.0 到 4.0

Matrix-Game 是昆仑万维 Skywork AI 的开源交互世界模型系列：1.0（2025-05）首发动作条件世界模型，2.0 实时长视频，3.0 流式交互加长程记忆，4.0（2026-10，arXiv:2610.27999）带来关键的架构转向。开源内容包含推理代码、浏览器 Demo、**三款原生游戏源码**与 HuggingFace 权重（Medieval Castle、Ancient Egypt、City Racing 三场景）。

### 8.2 核心思想："引擎管逻辑，模型管渲染"

纯世界模型有个根本缺陷：游戏逻辑不可靠、物理会漂移。4.0 的转向是**引擎-模型解耦**：

- 可执行游戏引擎负责玩法逻辑、物理与玩家输入，产出**低多边形（low-poly）RGB 帧**；
- 因果视频模型作为"**流式神经渲染器**"，把低模画面实时增详细为电影级视觉；
- HUD 与音效保留在模型之外的呈现层。

这个思路既保住了玩法确定性，又白拿了神经渲染的画质，是 2026 年交互世界模型的主流工程范式。

### 8.3 两个关键技术细节

**长时稳定性——future sink RoPE**：当 21-latent 的 KV 窗口填满后，把参考锚帧（sink）的 key 重新旋转到当前块前方 11 个位置，使锚帧与生成帧保持恒定相对距离。对比 absolute 策略，它能避免 2000+ 帧后建筑和路面"融化"。

**推理系统优化**：TAEHV 编解码、单步去噪、FP8/INT8 量化线性层、融合内核、CUDA Graph 回放，可选 SageAttention 后端。训练初始化正是基于 Causal Forcing 系列——与第 7 章遥相呼应。

### 8.4 性能：消费级显卡可以玩了

| GPU | 帧率（864×480） | 显存 |
|---|---|---|
| A100 80GB | 38–39 FPS | — |
| RTX 5080 | 42 FPS（SageAttention 后 67 FPS） | ~9.6–12.8GB |
| RTX 5070 Ti | 41 FPS（Sage 后 58 FPS） | ~9.2GB |

### 8.5 快速上手

```bash
cd Matrix-Game-4
matrix-game download --scene ancient-egypt   # 拉取权重与游戏资产（SHA256 校验）
# 在线游玩：启动 MediaMTX 后
bash scripts/serve.sh --scene ancient-egypt --preset realtime --fps 40
# 浏览器访问 http://127.0.0.1:8025/ —— 客户端无需 GPU
```

### 8.6 评价与局限

Matrix-Game 4.0 是"可玩游戏世界"落地的标杆：极少数把延迟压到消费级 GPU 可玩水平、并完整开源游戏引擎 + 模型 + 资产的项目。局限：仅三个封闭场景，不是开放式生成世界；5080/5070 Ti 仅支持固定 prompt；单用户 Demo 而非云端服务；`--fps 40` 只是采集节奏，不等于端到端延迟保证。

---

## 第 9 章 Helios：14B 模型的实时长视频生成

> 仓库：PKU-YuanGroup/Helios ｜ 约 2.2k★ ｜ Apache-2.0 ｜ arXiv:2603.04379

### 9.1 定位：两个"不依赖"

北大袁粒团队的 Helios 自称 "**Real** Real-Time Long Video Generation Model"——两个 Real 不是口误，而是针对同行宣传的反击。它的口号很硬：14B 模型比 1.3B 更便宜、更快、但更强。单张 H100 端到端 19.5 FPS（社区实测最高 20.89 FPS），生成分钟级长视频（1449 帧 ≈ 60s@24FPS），昇腾 NPU 上约 10 FPS。

Helios 的最大卖点是"双不用"：**不依赖任何抗漂移技巧**（无 Self-Forcing、error bank、关键帧采样、inverted sampling），**不依赖任何常规加速技巧**（无 KV-cache、因果掩码、稀疏/线性注意力、TinyVAE、量化）——只靠结构性的架构与训练管线创新。

### 9.2 四项结构性创新

1. **Unified History Injection**：把双向预训练模型改造为自回归生成器的架构适配；
2. **Easy Anti-Drifting + Multi-Term Memory Patchification**：抑制长程漂移；
3. **Pyramid Unified Predictor-Corrector**：激进的多尺度金字塔采样，把 50 步压到 **2+2+2 步**；
4. **Stage-3 对抗分层蒸馏**（基于 DMD）：去掉 CFG，降至 3 步。

自回归按 chunk 生成，每 chunk 33 帧。

### 9.3 生态完备度

Helios 的工程生态在学术项目里罕见地齐全：Day-0 支持 Diffusers（`HeliosPyramidPipeline`）、SGLang-Diffusion、vLLM-Omni、昇腾 NPU；Cache-DiT 加速；**Group Offloading 下蒸馏模型仅需约 6GB 显存**；Ulysses/Ring/Unified Attention 多卡上下文并行；配套 HeliosBench 评测基准。支持 T2V/I2V/V2V/Interactive 四种模式。

快速上手：

```bash
conda create -n helios python=3.11.2
bash install.sh
huggingface-cli download BestWishYSH/Helios-Distilled
bash scripts/inference/helios-distilled_t2v.sh
```

### 9.4 评价与局限

Helios 证明了一件事：**架构与训练管线的创新可以替代推理 hack 实现实时**，且 14B 模型在 6GB 显存的 offload 模式下即可消费级可用——目前开源实时长视频质量-效率比的最强梯队。局限：Interactive 模式官方自注"results may not always meet expectations"，尚不成熟；主战场是文生视频而非动作条件世界模型；384×640 原生分辨率偏低，4K 依赖社区方案。

---

## 第 10 章 HunyuanWorld-Voyager：相机驱动的 3D 世界探索

> 仓库：Tencent-Hunyuan/HunyuanWorld-Voyager ｜ 约 1.6k★ ｜ arXiv:2506.04225

### 10.1 定位与系列脉络

Voyager 是腾讯混元 HunyuanWorld 系列的"可探索 3D 场景生成"模型（2025-09 开源）。需要注意时间线：它是该技术线的早期代表作，系列此后快速迭代——WorldMirror（1.1）、WorldPlay（1.5，实时世界创建与游玩）、HY-World-2.0（2026-04，自称 SOTA 3D 世界模型）。读 Voyager 的正确姿势，是把它当作"相机条件 RGBD 视频扩散"这条线的奠基之作。

### 10.2 核心技术

两个核心组件：

1. **世界一致的视频扩散**：统一架构同时生成空间对齐的 RGB 视频与深度视频，并以既有世界观测（context block）为条件保证全局一致性。用户从单张图片出发，指定相机轨迹（前进/后退/左右平移/左右转），模型沿轨迹生成 3D 一致的场景视频。
2. **长程世界探索**：带**点云剔除（point culling）的高效世界缓存** + 自回归推理 + 平滑视频采样，迭代扩展场景。

配套数据引擎同样重要：对任意视频自动做相机位姿估计与度量深度预测，无需人工 3D 标注即构建 10 万+视频片段（实拍 + UE 合成）。生成的对齐 RGB+Depth 可直接重建点云（导出 ply），桥接了视频生成与 3D 重建。WorldScore 基准 77.62 分（超过 WonderWorld 的 72.69）。

### 10.3 实践参数

Voyager 是**离线高显存模型**，不满足实时交互：540p 单卡峰值显存 60GB（推荐 80GB 的 H20/H100）；512×768、49 帧、50 步在 8×H20 上延迟 288 秒（xDiT USP 并行下 6.69 倍加速），单卡 1925 秒。典型命令：

```bash
python3 data_engine/create_input.py --image_path xxx --type forward
python3 sample_image2video.py --model HYVideo-T/2 --input-path examples/case1 \
  --prompt "..." --infer-steps 50 --i2v-stability
```

### 10.4 评价与局限

Voyager 开源了"视频扩散 → 3D 一致世界"的完整路线（含数据引擎），是相机可控世界探索的开源基准级工作。局限：交互性体现在"预先规划相机路径"而非逐帧输入，非实时。它的 RGB-D 联合生成与点云缓存机制，是理解后续实时世界模型（WorldPlay 等）的前置知识。

---

## 第 11 章 FlashDreams：世界模型的"vLLM 时刻"

> 仓库：NVIDIA/flashdreams ｜ 约 529★ ｜ Apache-2.0 ｜ 2026-10-28 更新

### 11.1 定位：推理基础设施层

LLM 有 vLLM 和 SGLang，世界模型有什么？NVIDIA 给出的答案是 FlashDreams——**交互式自回归视频/世界模型的高性能推理与服务库**，源自 GTC 2026 OmniDreams 闭环自动驾驶 Demo 的优化运行时（配套论文 arXiv:2606.03159）。面向游戏、自动驾驶、机器人、仿真环境。

### 11.2 架构要点

- **分层 API**：Demo API（用世界模型搭交互 Demo，面向应用开发者）与 Inferencing API（接入自研世界模型，面向模型开发者）；
- 内置 OmniDreams 集成与 **Ludus 渲染器**（独立打包，含自研 cudaraster CUDA 光栅化内核）；
- 集成模板：Cosmos3 Nano Sim Auto（Cosmos 世界模型多视角驾驶，支持 HD Map 与可选 LiDAR）；
- 示例含闭环（closed-loop）交互式驾驶与 crazy-robotaxi 游戏化 Demo——模型输出反馈给规控器再产生新输入，是自动驾驶仿真级的实时世界模型推理。

### 11.3 评价与局限

FlashDreams 的信号意义大于当下的代码量：**世界模型正在平台化、服务化**——从各家自带专用推理代码，走向通用运行时，且由 NVIDIA 全家桶（Cosmos、Omniverse）背书。局限：README 信息密度低（细节在 nvidia.github.io/flashdreams 文档站），模型生态以 NVIDIA 自家 Cosmos 为主，通用性有待观察；OmniDreams 本体权重未开源。

---

## 第 12 章 Vidu S：闭源阵营的产品化样本

> 仓库：shengshu-ai/Vidu-S ｜ 约 512★ ｜ arXiv:2607.03118 / 2609.11638

### 12.1 开源的是接口，不是模型

生数科技（Vidu 团队）的 Vidu S 系列代表另一条路线：**闭源模型 + 开发者平台**。仓库不开放权重与训练代码，开放的是 API 文档、Agent Skill（`vidu-s-api` 可装进 Claude Code/Codex 做集成）与生态接口。产品线两条：S1（2026-07）与 S2（2026-09）。

### 12.2 三大能力

- **S2-Avatar**：720p 实时交互数字人，25–42 FPS，支持舞蹈等复杂语音指令，直播流中任意时刻可换参考图；
- **S2-Editing**：实时视频流编辑——风格迁移、虚拟试衣、角色替换、背景替换，且保留源视频运动；
- **空间视频**：为沉浸式显示与 VR 头显生成同步立体视图——这个方向几乎别家无人做。

S1 则是语音实时控制数字角色：用户随时用口语改变生成内容，540p、最高 42 FPS、无限时长。

### 12.3 关键技术

论文层面有两个贡献值得记录：**Self-Replay Forcing**——训练时对自身生成轨迹重新加噪后回放，减少流式分段间的误差累积（自回归蒸馏家族的新变体）；推理栈 **TurboDiffusion + TurboServe**——高效注意力、低比特 GEMM、内核优化、多 GPU 流水线。接入走 platform.vidu.com / vidu.cn 的 REST API，网页端 vidu.com 可直接体验。

### 12.4 评价与局限

国内首个把"实时交互 + 实时编辑 + 空间视频"打包成产品线的团队，Self-Replay Forcing 是实打实的论文贡献。局限：完全闭源，无法复现训练；本书引用依赖其论文与技术博客。它提醒我们：**开源榜之外的产业前沿，同样塑造着交互视频的走向。**

---

## 第 13 章 LiveTalk：实时多模态交互数字人

> 仓库：GAIR-NLP/LiveTalk ｜ Apache-2.0 ｜ arXiv:2512.23576 ｜ 2026-01 更新

### 13.1 定位

GAIR 团队（刘鹏飞组）的 LiveTalk 是实时多模态交互数字人（talking avatar）的系统化实现：基于 Wan2.1-1.3B + OmniAvatar 底座，目标是与语音大模型打通做对话式 AI。核心指标：**24.82 FPS 吞吐、0.33 秒首帧延迟、20 倍推理加速**。

### 13.2 方法：改进的 On-Policy 蒸馏

直接把 Self Forcing 套到多模态条件（参考图 + 音频 + 文本）上会闪烁、黑帧、退化。LiveTalk 的三板斧：

1. **精选多模态条件数据**：参考图超分 + 语义一致、聚焦运动的文本 prompt；
2. **收敛的 ODE 初始化**：延长到 2 万步；
3. **激进优化**：2 倍学习率、CFG=6，在有限 DMD 窗口内最大化学习。


### 13.3 端到端系统架构

- **Thinker/Talker**：流式音频 LM 实时产出语音；
- **Performer**：4 步因果视频扩散，block-wise 自回归（每块 3 个 latent 帧），条件为流式音频 + 参考形象图 + 运动文本；
- KV Cache 流式预填保持时序连贯；
- **AHIS（Anchor-Heavy Identity Sinks）**：KV 窗口部分固定为"身份锚"，小滚动窗口管上下文——保证分钟级交互中人物外观稳定；
- 去噪与 VAE 解码流水线并行，生成跑在播放前面。

多轮交互基准上对标 Veo3、Sora2 有竞争力。

### 13.4 实践与局限

```bash
git clone https://github.com/GAIR-NLP/livetalk.git && cd livetalk
git clone https://github.com/Omni-Avatar/OmniAvatar && bash scripts/add_patch.sh
bash ./scripts/inference.sh
# 输入：人物参考图 + 16kHz WAV 语音 + 文本 prompt；显存约 20GB（RTX 4090 实测）
```

局限：开源的是推理验证脚本，实时流式服务层未开源；输出 16 FPS 低于电影帧率；1.3B 底座画质有限。但它是 on-policy 蒸馏在多模态条件下的首个系统化成功案例，AHIS 身份锚机制与"音频 LM + 视频扩散"的端到端组合，是对话式视频智能体的雏形。

---

## 第 14 章 SparkVSR：用稀疏关键帧"导演"超分辨率

> 仓库：taco-group/SparkVSR ｜ 约 745★ ｜ ECCV 2026 ｜ arXiv:2603.16864

### 14.1 独特的交互定义

SparkVSR（Texas A&M 与 YouTube/Google 合作）把"交互性"引入了一个意想不到的赛道：视频超分辨率。用户不再被动接受黑盒输出，而是**用少量修复好的关键帧控制整段视频的修复结果**——交互信号是稀疏关键帧。

### 14.2 方法

用户任选现成图像超分模型（如 PiSA-SR 或商用 API）修复少量关键帧，SparkVSR 将关键帧先验传播到全序列，同时以原低清视频的运动为锚。训练基于 CogVideoX1.5-5B-I2V，采用**关键帧条件的 latent-pixel 两阶段管线**：Stage-1 在 latent 空间学习融合 LR 视频 latent 与稀疏 HR 关键帧 latent；Stage-2 在 pixel 空间做感知细节精修。推理时提供 **reference-free guidance** 持续平衡"贴关键帧"与"盲修复"——即使关键帧缺失或质量不佳也稳健。

效果：CLIP-IQA +24.6%、DOVER +21.8%，RealVSR 4× SOTA（Papers with Code 认证），并可零样本地外推到老电影修复与视频风格化。

### 14.3 实践

```bash
python sparkvsr_inference_script.py --input_dir test_input \
  --model_path checkpoints/sparkvsr-s2/ckpt-500-sft \
  --ref_mode pisasr --upscale 4 --ref_indices 0
```

关键帧选取三模式：手动指定（`--ref_indices`，间隔须 >4）、编码器 I 帧提取、随机采样；`--ref_guidance_scale` 控制遵从强度。训练需 4×A100；推理单卡，非实时——主打质量与可控性。

### 14.4 评价

SparkVSR 在交互视频图谱中独占"后处理/修复"生态位，证明交互控制信号可以是稀疏关键帧而非动作/相机/语音——这个通用框架视角颇具启发性。YouTube 的合作背景暗示它指向生产级修复管线。

---

## 第 15 章 Xmax SDK：从研究 Demo 到多端基础设施

> 仓库：Xmax-AI/XmaxSDK-Web（另有 iOS/Android/Flutter/HarmonyOS/MacOS/RN）｜ MIT ｜ 2026-10-27/28 密集更新

### 15.1 概览

Xmax AI 开源的全平台 SDK 家族，配套云端 `x2_1` 模型，做的是"实时视频流编辑"范式：媒体采集（摄像头/参考图）→ WebRTC 视频流传输 → 逐帧云端生成 → 端侧渲染。端到端延迟数百毫秒级，支持最高 1080p 实时生成，官方宣称推理只需单张 RTX 5090——比 H100 数据中心方案低数个量级的成本。

### 15.2 典型用例

直播角色替换、虚拟试衣、视频风格重绘、AI 陪伴（摄像头画面中召唤可交互角色）、Live Photo（画轨迹驱动图片动起来）。与 Vidu S2-Editing、Helios 的 V2V 属同一路线。

### 15.3 工程形态

技术栈依赖腾讯云 TRTC / 声网 Agora / 火山 VeRTC 三选一做 RTC 传输、腾讯 COS 做媒体存储；`npm install @xmaxai/web-sdk` 即可接入，API key 走后端短时令牌，含 React 组件子包与完整示例。

### 15.4 意义

Xmax 的价值不在单项技术，而在**样本意义**：它是"实时交互视频生成"从研究 Demo 走向多端 SDK 商用基础设施的代表，且 iOS/Android/鸿蒙全矩阵的布局在国内厂商中少见——这预示着 2027 年交互视频的竞争主战场之一是端侧接入体验。

---

## 第 16 章 前沿群像：四个值得记录的支线项目

### 16.1 ForgeWM：全开源的世界模型复现

社区驱动的完全开源动作条件世界模型全配方（arXiv:2608.14022，Apache-2.0）：以 Matrix-Game 2 为底座、GameFactory 开源数据（约 70 小时 Minecraft 玩法）训练，把 Causal Forcing 蒸馏范式落成四阶段渐进管线（双向 SFT → teacher-forced 因果 AR → 一致性蒸馏 → on-policy DMD），**8×H20 即可复现**。产出 ForgeWM-1/2/4 三档少步学生模型：1 步 72.1 FPS、168ms/chunk（H20，352×640），并含 First-Frame Enhancement（首块用满 4 步保清晰锚帧）与 CrossFPS（键鼠→手柄跨域迁移）。它是对 Matrix-Game 封闭训练的社区回应，证明顶级交互世界模型可以在全开源栈上复现。

### 16.2 MotionStream：轨迹引导的实时生成

ICLR 2026 论文（SNU + Adobe Research + CMU），"Real-Time Video Generation with Interactive Motion Controls"——用户通过稀疏运动轨迹（点轨迹拖拽）实时引导视频内容。作者团队含 Self-Forcing 作者 Huang Xun，技术上属轨迹条件 + 自回归蒸馏的结合。**代码因公司内部审查尚未开源**，仅论文与项目页可用——作为方法论条目记录。

### 16.3 Olaf-World：从互联网视频自学动作

NUS Show Lab 的 "Orienting Latent Actions for Video World Modeling"（arXiv:2602.10104，ICML 2026）：用 **Latent Action Model（LAM）** 从无标注视频中学潜在动作表征驱动世界模型——基于 V-JEPA 2 做对齐式 LAM，配 SkyReels-V2-I2V-1.3B 底座与 Self-Forcing 蒸馏。亮点是**零样本动作迁移**：把参考视频的潜在动作序列直接套到目标首帧上，无需微调。已开源 LAM 训练/推理与权重。它代表"从互联网视频自学动作语义"的无监督交互范式。

### 16.4 ActionSplice：同帧动作切换

Stanford 系的 "In-Flight Action Editing for Interactive World Models"（arXiv:2609.08230，Apache-2.0）解决一个极细但极痛的问题：**当前 chunk 还没采样完，用户就改了输入，怎么办？** 等待增加延迟；直接换条件会留下旧动作塑形的求解器中间态；重启浪费算力。它提出 Counterfactual State Transport（CST）校正器：学习把被打断的求解器状态搬运到同一求解步的反事实状态，再让冻结骨干正常续跑；提供 CST-R（整块重定向）与 CST-T（保前缀只修后缀）两变体，适配 minWM 与腾讯 HY-WorldPlay 后端。这是"action-to-photon 延迟优化"这个新兴细分方向的代表工作。

---

# 第三篇 分支叙事与轻量创作工具

## 篇首语：光谱的四个刻度

第三篇的四个项目规模都不大（star 数从 0 到 6），却恰好覆盖了交互叙事光谱的四个刻度：

- **IVKit**：声明式播放器内核，轻至 9.5 KiB；
- **QuizPlayer**：垂直场景深耕——测验 + 服务端可信验证；
- **Sogni Worlds Kit**：Agent 生产整个世界的内容，人类只做品味把关；
- **NextCut**：企业级工程方法论——状态机、证据链与发布门禁。

三条共同趋势值得预先点出：① 声明式 JSON DSL 正在成为 **AI 可编译的目标格式**；② 几乎所有项目都选择**预生成内容 + 确定性运行时**——没有一个在播放时现场生成视频；③ **AI 生成结果的人工/自动审核管线**正在成为标配而非可选项。

---

## 第 17 章 IVKit：9.5 KiB 的分支叙事内核

> 仓库：Daniel-Glz/ivkit ｜ v0.1.1 ｜ TypeScript ｜ 2026-10 创建

### 17.1 定位

IVKit 把自己定义为"把任意视频变成交互视频"的轻量 TypeScript 库：分支剧情（branching）、提问（questions）、动画热点（animated hotspots）三类交互，以单个 Web Component `<ivkit-player>` 交付，零依赖、gzip 后不足 9.5 KiB。项目 2026 年 10 月 26 日创建、28 日即上架 npm——非常年轻，但工程素养高：有 CI、npm 双包、完整文档站，甚至设置了"体积预算"约束。

### 17.2 核心：声明式 Story JSON

统一的版本化故事格式（`version: "0.1"`）是 IVKit 的灵魂：

- `Story`：`start`（起始场景）+ `scenes`（场景字典）+ 可选 `vars`（状态变量）；
- `Scene`：`src`（视频 URL）、`events`（Question/Hotspot/Cue 事件）、`next`（播完后的下一场景）、`branches`（**条件分支，优先于 next**）、`set`（进入场景时改写变量）；
- `Question`：在 `at` 秒暂停视频展示选项；选项可带 `goto`（跳场景）、`set`（改变量）、`if`（条件显示）、`correct`（测验标记）、`seek`（原地回跳）；支持多选与超时默认答案；
- `Hotspot`：时间窗内可点击区域，坐标 0–1 归一化，支持矩形/椭圆/多边形与关键帧跟踪运动物体；
- `Branch`：基于 `vars` 的条件表达式，首个成立者生效——声明式条件叙事的关键机制。

用法：

```html
<ivkit-player id="player" controls></ivkit-player>
<script type="module">
  import "@ivkit/player";
  document.getElementById("player").story = {
    version: "0.1", start: "intro",
    scenes: { intro: { src: "/videos/intro.mp4" } },
  };
</script>
```

### 17.3 架构

双包设计：`@ivkit/player` 是基于 Custom Elements 的开箱播放器（支持 CSS 自定义属性、`::part()` 与 slot 主题化）；`@ivkit/core` 是 **DOM-free 引擎**——`StoryEngine` 事件驱动（`scene`/`interaction`/`hotspots`），`MediaController` 预加载下一场景实现无黑帧切换，`validateStory` 做格式校验。宿主只需把 `timeupdate` 喂给 `engine.update(currentTime)`——这意味着它可以嵌入 React、Vue，甚至游戏引擎与 Canvas。

### 17.4 评价与局限

IVKit 的气质像"交互视频领域的 hls.js"：不绑框架、不绑后端，声明式 Story JSON 使其成为 **AI 生成剧情图的理想编译目标**。对比 Twine（文本叙事、无视频原生支持）与 Eko（编程式 JS 交互），它更声明式、更轻。局限：版本仅 0.1、无社区；无后端验证方案（答案全在客户端，可作弊）；暂无 HLS/DASH 流媒体官方支持；热点坐标靠手写。

---

## 第 18 章 QuizPlayer：会暂停、响铃、提问的视频播放器

> 仓库：IrvingSamuel/quizplayer ｜ MIT ｜ 2026-10 创建 ｜ 核心约 14 kB gzip、零依赖

### 18.1 定位

QuizPlayer 面向 e-learning/LMS 场景，口号是"any frontend, any backend"。它选择的交互模型是**线性的时间轴测验点**而非图状分支：视频在预设秒数暂停 → Web Audio 合成提示音（无需音频文件）→ 弹出问题 → 即时反馈 → 自动继续。虽然不及分支叙事炫目，但它把这一个场景做到了罕见的完整度——2026 年 10 月 6 日创建、7 日即功能完备：CI、多语言文档、demo 站、协议规范、**覆盖 React/Vue/Blade/PHP/Go/Node 六端 SDK**。

### 18.2 数据与协议

```jsonc
{ "id": "q1", "time": 42.5, "question": "…",
  "options": [{"id":"a","text":"…"}], "correct": "b",
  "explanation": "…", "audioUrl": "…/narration.mp3" }
```

差异化亮点是**防作弊架构**：配置 `endpoint` 后，正确答案不出现在页面——由服务端 `POST /answer` 校验并落库；`preventSkip` 阻止拖进度条跳过未答题；`previousAnswers` 与 `GET /answers` 实现断点续答。后端是一份约 4 个端点的微型 JSON 协议（`/answer`、`/answers`、`/quizzes`、`/results`），官方 SDK 覆盖 Node（Express/Next/Hono/Bun/Deno）、PHP 8.1+（含 Laravel 自动发现 + Blade 组件 + 迁移）与 Go，其他语言"约 50 行"即可实现。

纯 HTML 用法通过 `data-quizplayer` 属性 + JSON `<script>` 声明即可自动初始化，天然适配 Blade/Twig/Django/ERB 等服务端模板。无障碍也做足了：键盘导航、焦点陷阱、ARIA live、`prefers-reduced-motion`、内置 7 语言文案。

### 18.3 评价与局限

与 IVKit 的分工清晰：QuizPlayer 做"深"的单视频测验（服务端验证、成绩、防跳过），IVKit 做"广"的多视频分支。对标付费 SaaS（H5P、Wistia），"开源 + 全栈 + 防作弊"的组合是独特卖点。局限：不做分支叙事；`preventSkip` 作者自认"不是 DRM"；多选题/自由文本、YouTube/Vimeo 适配、SCORM/xAPI 在路线图上尚未实现。

---

## 第 19 章 Sogni Worlds Kit：Agent 当游戏引擎

> 仓库：Sogni-AI/sogni-worlds-kit ｜ MIT ｜ 2026-09 创建，持续活跃

### 19.1 定位："Agent-as-game-engine"

Sogni Worlds Kit 是本篇唯一带商业背景的项目（Sogni AI，去中心化 GPU 网络）。它的设想极具未来感：输入一个概念或你的照片，由你的编程 Agent（Claude Code/Codex/Hermes）配合 Sogni API，**用 9 个开源模型在一夜之间生成一个"电影化点击式冒险（CYOA）世界"**。官方 demo《The Dream Thread》（23 个场景，纯文字概念生成）与《The Long White Cloud》（22 个场景，来自旅行照片，63 条 2K 影片）可在线游玩。

### 19.2 世界观设计："活的静帧 + 穿越式转场"

每个地点是一张 2K 静帧，点击帧中的物体（门、船、吉他），一条以该物体开场的生成影片带你"穿过"它到达下一地点；配 TTS 旁白与配乐。交互点三类型：`crossing`（静帧 A → 静帧 B 的旅程）、`loop`（让静帧呼吸）、`moment`（发生又回归的事件）。致命选择会**倒放影片**回到选择点；可收集物品有 3D 模型可拿起翻转。五条导演准则写入文档（如"每段旅程必须穿过某物，绝不淡入淡出或变形"）——这是把电影语言写进了生成管线。

### 19.3 生产管线与人在环

九步流水线：`paint/photos → ingest → plan(world.yaml) → select(SAM 3 描边) → render(MiniMax H3 首尾帧两段式) → screen(自动质检) → review(人工批准) → build(world.json) → play/export`。模型矩阵覆盖绘景（Krea 2 Turbo）、角色一致性（Identity Edit）、点击描边（SAM 3）、细节补全（Qwen Image Edit）、带声影片（MiniMax H3）、旁白（Qwen3-TTS，可克隆自己的声音）、配乐（MiniMax Music 3）、3D 收藏品（Pixal3D）。

**人类在环是核心设计**：`node world review` 逐片批准/拒绝，还能框选画面区域给 Agent 留返工批注。首版 120 条候选影片只有 57 条成片——生成一致性仍需大量人工把关。

```bash
node world setup && node world new my-world
# 然后对 Agent 说："Build a Sogni World from this concept: …"
node world review && node world play && node world export
```

### 19.4 评价与局限

Sogni Worlds 代表了"AI 负责生产、人类负责品味"的内容生产闭环，与 Twine（人工写内容）、OpenFMV（播放既有 FMV）形成三代工具的对照。局限：强绑定 Sogni 平台与付费（一个世界约 $50–98 按量成本）；所有内容预生成，无实时交互；生成一致性返工率高。

---

## 第 20 章 NextCut：把"发布资格"做成一等公民

> 仓库：yusei56/NextCut ｜ 2026-08 创建 ｜ Next.js + LangGraph.js

### 20.1 定位：两天的工程假设验证

NextCut 是一个中文 AI 原生交互电影 demo，验证链路是把公版小说《猴爪》编译成两条经审核的互动视频分支。2026 年 8 月创建后未再更新、0 star，纯作品集性质——但 README 的诚实度和工程严谨性是全书所有项目中最高的：明确列出已实现/部分实现/未实现清单，25 项单元测试 + 3 条 Playwright E2E 全部通过。

它的核心判断是：**AI 负责提出与生产候选，确定性系统负责状态、证据、版本和发布资格**。创作与游玩严格分离：创作阶段离线生产并审核；玩家阶段不现场生成任何视频，只做确定性剧情跳转、即时播放已审核成片。

### 20.2 多 Agent 规划链

创作侧的多 Agent 流水线：Director（定因果与场景起止状态）→ Writer / Shot Designer / Decision Designer 并行 → Continuity Critic（连续性审核）→ Prompt Compiler（输出类型化 Render Spec）。关键设计是**所有中间结果都有 Zod Schema**，世界状态只能由领域事件（Canonical Action → Domain Events → Child World Snapshot）改变，违规/超时走显式修复或降级路径——不是"六个 Agent 自由聊天"。

### 20.3 三大工程机制

1. **视觉 Bible**：角色/地点/道具作为版本化资产（多角度转面图、比例表），带 `manifest.json` 与 SHA-256 哈希，镜头通过 `referenceAssetIds` 引用；
2. **Render Spec**：显式 `startState`/`endState`、分时动作节拍、摄影/灯光/原生声音、连续性锁与禁止生成内容、与下一镜头逐字一致的边界状态；
3. **发布门禁**：11 项固定标准按 10 个时间证据点逐帧抽查，关键项 fail/uncertain 即不可发布；失败样本留档（猴爪畸变候选被 BLOCKED），审核记录绑定 `takeId` + 素材哈希。

播放端用 **180ms 双缓冲交叠预缓冲**解决换源停顿。

```bash
pnpm install && pnpm dev   # 默认用审核缓存，无需 API Key
# 在线模式：TEXT_MODEL_API_KEY=… STORY_PLANNING_MODE=LIVE
```

### 20.4 评价与局限

NextCut 的价值不在工具本身（固定单一故事、仅一个互动节点有完整分支、审核靠脚本与人工），而在它证明的方法论：**生成成功 ≠ 可发布**——当 AI 生成内容要走向真实发布，"发布资格"必须作为一等公民被工程化。参考系明确（ink、LTX Studio、MovieAgent），路线图指向创作者工作台/离线生产中心/付费玩家平台。它与 Sogni Worlds 构成有趣的对照：一个追求自动生成世界，一个追求生成过程的可审计。

---

# 第四篇 趋势与展望

## 第 21 章 2026 交互视频技术版图

把全书 20+ 项目放回一张图，可以读出完整的版图结构：

### 21.1 按抽象层次

```
创作工具层    OpenFMV ｜ Sogni Worlds ｜ NextCut ｜ MotionStream(方法论)
─────────────────────────────────────────────────
组件内核层    IVLabs Core ｜ IVKit ｜ QuizPlayer
播放引擎层    Liqvid ｜ FrameTrail
数据标准层    FrameTrail(W3C Web Annotation) ｜ aval format
编解码格式层  AVAL(.avl 状态机烧录)
─────────────────────────────────────────────────
推理基础设施  FlashDreams ｜ Xmax SDK ｜ Vidu TurboServe
训练配方层    Causal-Forcing(+ +) ｜ Self-Replay Forcing
模型层        Matrix-Game ｜ Helios ｜ Voyager ｜ LiveTalk ｜ SparkVSR ｜ ForgeWM
```


### 21.2 核心结论

1. **上下两层同时在变，中间三层相对稳定**。模型层（生成什么）与创作工具层（谁来创作）在 2025–2026 年剧变，而 cue/marker/hotspot 这些中间抽象已被证明足够稳定。
2. **蒸馏配方层已收敛**。Causal Forcing (+ +) 成为事实标准，Matrix-Game、ForgeWM、minWM 等皆以它为初始化——这类似 LLM 时代 RLHF 的位置。
3. **交互信号持续多样化**。从时间点、热区、分支，扩展到键鼠、相机、语音、关键帧、轨迹、潜在动作——每一个新信号都开辟一个新产品类别。
4. **推理基础设施化正在发生**。FlashDreams（NVIDIA）与 Xmax SDK 预示着世界模型即将拥有自己的 vLLM 时刻。

## 第 22 章 趋势观察与选型建议

### 22.1 三大趋势

**趋势一：热点从"播放框架"转向"实时生成世界"。** 2026 年交互视频的星火明显在第二篇：用键鼠/文本/音频实时操控生成中的视频，是 Skywork、NVIDIA、腾讯混元、清华、北大集体押注的方向。传统 Web 框架方向仍在演进（FrameTrail 与 aval 是其中最佳），但增量注意力已经转移。

**趋势二：AI 进入了创作与审核环节，而非只进入播放环节。** Sogni Worlds（Agent 生产整个世界）、NextCut（多 Agent 规划 + 发布门禁）、OpenFMV（AI 辅助写作）共同指向同一个未来：**交互视频的创作成本趋近于零，而审核与品味成为新的稀缺资源**。

**趋势三：硬件与接入民主化。** 消费级 5080/5070 Ti 可玩世界模型（Matrix-Game 4.0）、6GB 显存 offload 跑 14B 实时生成（Helios）、免本地 GPU 的多端 SDK（Xmax）——2027 年，实时交互视频将抵达浏览器与手机的原生体验层。

### 22.2 选型速查

| 你的需求 | 推荐 |
|---|---|
| UI 动效要帧精确状态切换 | AVAL |
| 数理/讲解型可编程视频 | Liqvid |
| 视频注释、混剪、教育 | FrameTrail |
| 产品内嵌问答/分支组件 | IVLabs Core / IVKit |
| 给非技术创作者做互动剧 | OpenFMV（原型）/ Eko 类商业平台（生产） |
| e-learning 测验视频 | QuizPlayer |
| AI 批量生成交互世界 | Sogni Worlds（内容）/ NextCut（方法论） |
| 自研实时交互生成 | 底座 Causal-Forcing → 参考 Matrix-Game 4.0 / ForgeWM 工程 |
| 实时数字人对话 | LiveTalk（自部署）/ Vidu S（API） |
| 视频修复/超分可控化 | SparkVSR |
| 世界模型推理服务 | FlashDreams |

### 22.3 给不同读者的行动建议

- **前端工程师**：先精读 AVAL 的格式设计（它重新定义了"视频文件"），再用 Liqvid 或 IVKit 做一个小项目体感"视频即应用"。
- **研究者**：Causal Forcing 两篇论文 + Matrix-Game 4.0 技术报告 + ActionSplice，是从蒸馏理论到系统优化的完整阅读路径。
- **创作者/创业者**：用 OpenFMV 或 Sogni Worlds 快速原型，但认真研究 NextCut 的"发布门禁"章节——那才是走向产品时真正缺的东西。
- **所有人**：关注 2027 年的三个信号——世界模型通用推理运行时的成熟度、浏览器原生支持实时生成视频流的进度、以及"AI 审核管线"是否会标准化。

---

# 附录

## 附录 A：项目速查表

| 项目 | 方向 | Stars | 最近更新 | 一句话定位 |
|---|---|---|---|---|
| pixel-point/aval | Web 框架 | ~1.6k | 2026-09 | 状态机烧录进视频字节的交互视频格式 |
| liqvidjs/liqvid | Web 框架 | ~820 | 2026-10 | React 可编程视频，数学科科普利器 |
| OpenHypervideo/FrameTrail | Web 框架 | 132 | 2026-10-24 | 十年长跑的开放超视频平台 |
| interactive-video-labs/core | Web 框架 | 个位数 | 2026-10-28 | 四框架适配的最小 cue 内核 |
| Comedian1926/OpenFMV | Web 框架 | 229 | 2026-06 | 本地优先可视化分支叙事编辑器 |
| SkyworkAI/Matrix-Game | AI 生成 | ~2.8k | 2026-10-28 | 消费级 GPU 可玩的流式神经渲染世界 |
| PKU-YuanGroup/Helios | AI 生成 | ~2.2k | 2026-08 | 14B 实时长视频，双"不依赖" |
| Tencent-Hunyuan/HunyuanWorld-Voyager | AI 生成 | ~1.6k | 2026-04 | 相机条件 RGBD 世界探索奠基作 |
| thu-ml/Causal-Forcing | AI 生成 | ~1.0k | 2026-10-24 | 行业事实标准的自回归蒸馏配方 |
| NVIDIA/flashdreams | AI 生成 | ~529 | 2026-10-28 | 世界模型的 vLLM 时刻 |
| shengshu-ai/Vidu-S | AI 生成 | ~512 | 2026-09 | 实时交互+编辑+空间视频产品线 |
| GAIR-NLP/LiveTalk | AI 生成 | — | 2026-01 | 24.8 FPS 实时多模态数字人 |
| taco-group/SparkVSR | AI 生成 | ~745 | 2026-08 | 稀疏关键帧导演的视频超分 |
| Xmax-AI/XmaxSDK-* | AI 生成 | — | 2026-10-28 | 实时交互视频生成的全端 SDK |
| asdfo123/ForgeWM | AI 生成 | — | 2026-08 | 全开源世界模型复现配方 |
| alex4727/MotionStream | AI 生成 | ~587 | 2026-03 | 轨迹引导实时生成（代码未开源） |
| showlab/Olaf-World | AI 生成 | ~128 | 2026-10 | 潜在动作驱动的世界模型 |
| PardisTaghavi/ActionSplice | AI 生成 | 6 | 2026-09 | 世界模型同帧动作切换 |
| Daniel-Glz/ivkit | 叙事工具 | 1 | 2026-10 | 9.5 KiB 分支叙事内核 |
| IrvingSamuel/quizplayer | 叙事工具 | 1 | 2026-10 | 防作弊的视频测验播放器 |
| Sogni-AI/sogni-worlds-kit | 叙事工具 | 6 | 2026-10 | Agent 生产整世界 CYOA |
| yusei56/NextCut | 叙事工具 | 0 | 2026-08 | 发布资格工程化方法论 |

## 附录 B：术语表

- **Cue（提示点）**：挂在视频时间轴上的交互触发对象，到点即触发预设行为。
- **Hypervideo（超视频）**：以超链接结构组织的非线性视频网络，视频间通过可点击热区跳转。
- **自回归扩散蒸馏**：把多步双向扩散模型压缩为少步因果（逐块生成）模型的训练技术，是实时视频生成的核心手段。
- **DMD（分布匹配蒸馏）**：让学生模型的输出分布匹配教师分布的蒸馏方法，不要求逐轨迹对齐。
- **Causal Forcing**：以因果 ODE/一致性蒸馏为初始化、再接非对称 DMD 的三段式蒸馏配方，2026 年行业事实标准。
- **On-Policy 蒸馏**：学生在自身生成轨迹上向教师学习的蒸馏方式。
- **世界模型（World Model）**：能按动作条件逐帧预测环境未来画面的生成模型，可"游玩"。
- **Action-to-Photon 延迟**：从用户输入到对应画面呈现的端到端延迟，交互世界模型的关键指标。
- **Local-first**：数据默认存于用户本地、无云端强依赖的软件架构理念。
- **CYOA（Choose Your Own Adventure）**：读者/观众通过选择决定剧情走向的叙事形式。
- **发布门禁（Release Gate）**：内容发布前必须通过的一组固定质量标准与证据审查流程。

---

*本书数据截至 2026 年 10 月 29 日，基于各项目 GitHub 仓库、技术报告与论文的公开信息整理。开源世界日新月异，请以各仓库最新状态为准。*


