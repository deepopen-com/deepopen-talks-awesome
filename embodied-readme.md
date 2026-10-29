# 具身智能前沿 2026

## ——从开源学习地图到人形真机：35+ 项目深度解读

---

# 前言

如果 2024 年的关键词是"大模型"，2025 年是"Agent"，那么 2026 年几乎所有顶级实验室与大厂的路线图上，都写着同一件事：**具身智能（Embodied AI）**。

这一年发生了太多标志性事件：Genesis 以生成式物理引擎斩获 30k star；Hugging Face 的 LeRobot 把"模型 + 数据集 + 工具"的开源范式完整复制到机器人领域；斯坦福 BEHAVIOR-1K 用 1000 项真实人类需求定义了下一代难度标尺；NVIDIA 把世界模型压缩到 4B 参数跑进 Jetson 边缘设备；OSDI 2026 收录了具身强化学习基础设施论文——系统顶会开始为机器人"修路"；而中文社区贡献了从 16.7k star 技术指南到"淘宝 + 嘉立创就能复现的开源人形"的完整光谱。

本书是一次系统的开源生态调研：覆盖**学习指南、VLA 模型、仿真平台与世界模型、人形机器人与真机硬件、训练/推理基础设施**五个方向，筛出 2026 年内仍在活跃更新的 35+ 个项目，逐一深入解读其定位、原理、架构与用法。所有数据（star 数、评测成绩、硬件价格、更新日期）均取自 2026 年 10 月下旬各仓库与论文的实时状态。

**本书适合谁读**：想入门具身智能的学生与转型开发者（第一篇）、机器人/多模态研究者（第二、三篇）、硬件工程师与创客（第四篇）、平台与基础设施工程师（第五篇），以及所有想理解"AI 获得身体"这一进程的人。

**如何阅读**：五篇相互独立，可按需跳读。赶时间者直接读第 35、36 章的版图与趋势总结。

---

# 导论：具身智能的 2026 版图

**具身智能**指依赖身体与环境交互获得智能的 AI 系统——它不只"看懂"世界，还要在世界中"动手"。2026 年的具身智能技术栈可以概括为五层：

```
┌─────────────────────────────────────────────┐
│  学习地图层：指南、教程、论文列表（第一篇）            │
├─────────────────────────────────────────────┤
│  模型层：VLA / 世界模型 / World Action Model（第二、三篇）│
├─────────────────────────────────────────────┤
│  仿真层：物理引擎、照片级渲染、基准（第三篇）           │
├─────────────────────────────────────────────┤
│  本体层：人形机器人、机械臂、遥操作（第四篇）           │
├─────────────────────────────────────────────┤
│  基础设施层：RL 训练、Agent harness、数据与评测（第五篇） │
└─────────────────────────────────────────────┘
```

三个关键术语贯穿全书：

- **VLA（Vision-Language-Action）模型**：视觉-语言骨干负责理解世界，专用动作头负责输出控制信号——它是机器人的"大脑"，也是 2026 年最热的方向；
- **世界模型（World Model）/ WAM（World Action Model）**：能预测世界未来状态的生成模型，直接产出动作即 WAM——它让机器人"在脑内排练"；
- **数据金字塔**：从互联网视频（亿级）→ 人类第一视角视频（十万小时级）→ 遥操作真机数据（千小时级）→ 精修微调数据（几十条），2026 年的主战场是数据配方而非模型架构。

带着这张图，我们逐层展开。

---

# 第一篇 学习地图：入门必看资源

## 篇首语

具身智能的知识密度极高——控制、感知、学习、硬件缺一不可。第一篇的五个项目不写代码、不发模型，却是全书所有项目得以被"读懂"的地基：两本中文深度指南、两套动手教程、一份论文雷达。

## 第 1 章 Embodied-AI-Guide 与星期八社区：两张中文地图

> Embodied-AI-Guide：16.7k★ ｜ Octoday-Hub/Embodied-AI：3.0k★ ｜ 均 2026-10 活跃

### 1.1 Embodied-AI-Guide：深度认知的百科全书

Lumina 具身智能社区维护的中文知识库，2025 年 3 月开源、当年 12 月破万星，是中文世界最热门的具身智能指南。它的组织方式是"一条主线 + 一张全景图"七章结构：

1. **从这里开始**：20 条术语速查表、按背景分类的学习路线表，以及新人六大 FAQ——显存要求、操作 vs 运动选方向、"现在入场晚不晚"等都有直白回答；
2. **动手学习**：以 RoboTwin 2.0 + XPolicyLab 为载体，一周内走通"读论文 → 装环境 → 下 10 万条开源轨迹 → 训 ACT → 评测对照 Leaderboard"的完整生命周期；
3. **认知资料**：社区、实验室、会议、论文列表、年度趋势；
4. **算法篇**：视觉基础模型、Robot Learning、LLM for Robotics、VLA（含 2025 代表工作与 2026 进展专节）；
5. **基础设施篇**：仿真器、基准、数据集、部署工具链；
6. **控制篇**：经典/现代控制、机器人学、WBC 人形全身控制；
7. **硬件篇**：嵌入式、机械、传感器、触觉、数据采集、公司生态。

它的可贵之处在于**观点鲜明**——例如"2026 年公认瓶颈已从模型架构转向数据配方与评测协议"这类判断，直接点出领域主要矛盾。2026 年 1 月完成内容重组，更新勤奋。注意：非商业许可。

### 1.2 Octoday Hub：广度导航与产业地图

星期八具身智能生态社区（公众号"星期八Robotics"）维护的知识索引，每周二、周五固定更新资源核验——更新纪律性在同类仓库中罕见。GitHub 侧重代码类资源：三大导航（开源项目 / 开源工具 / 数据集）+ 五个次级索引（论文合集、课程书籍、学术竞赛、公司目录、求职信息）。

数据集索引按四个场景组织，本身就是一份极好的检索表：真实操作数据（DROID、Open X-Embodiment、AgiBot World）、遥操作示教（RH20T、UMI）、仿真合成（MimicGen、RoboTwin）、评测基准（LIBERO、BEHAVIOR-1K）。

两个社区定位互补：**Embodied-AI-Guide 做"深度认知"，Octoday 做"广度导航"**——先读前者建立框架，用后者快速找资源。

## 第 2 章 Datawhale 双教程：从 Python 到真机

> every-embodied：4.3k★ ｜ dive-into-embodied-ai：648★ ｜ 均 2026-10 活跃

### 2.1 every-embodied：从 0 构建 VLA

Datawhale 出品的 Zero-to-Hero 学习库，获 Hugging Face、上海 AI Lab、达摩院、智元、AMD ROCm 等支持，口号是"仅需 Python 基础"。三阶段路径：机器人学基础/硬件/ROS → CV/运动规划/RL → 模仿学习/VLA/VLN/Sim2Real，对应 20+ 编号章节。

两个核心亮点：**第 6 章 VLA 复现**——在 MuJoCo 上复现 ACT/Pi0/SmolVLA，部署 OpenVLA，并精读 TurboVLA/WALL-OSS/DM0.5 等十余篇 2026 前沿；**第 17 章具身世界模型**——LeWM、τ0-WM、WALL-WM、GE-Sim 2.0 等导读。此外还有竞赛专区（AgiBot World Model、ATEC2026 复盘）与组队学习路径 v6，支持 AMD ROCm 仿真案例和地瓜 RDK-X5 + SO101 真机遥操作。

它的 News 记录几乎每周更新，本身就可以当作"2026 具身智能动态索引"使用。每个复现都带排障记录——这在学习类仓库中非常难得。

### 2.2 dive-into-embodied-ai：完整机器人的系统课程

Datawhale 另一条教程线，定位"从零到一搭建一台具身智能机器人"，Docusaurus 建站（中/英/意三语），目前处于 Alpha 内测。三栏目结构：

- **具身导论**（7 章）：发展脉络、任务与技能、载体、核心挑战、技术栈，以及同类教程少见的**就业岗位篇**——覆盖"大脑/小脑/感知/工程/硬件"五类岗位的面试准备；
- **理论基础**：按"大脑（RL 决策、VLA、World-Model）— 小脑（RL 控制、PID/LQR/MPC、运动规划）— 感官（VLM 感知、标定与 sim2real）— 工程底座（Isaac Sim/MuJoCo、ROS2、数据工程）"四列组织——这个"脑-小脑-感官-底座"的类比是理解具身智能分工的好框架；
- **项目实战**：AMD 专区（ROCm 上双足 PPO）、仿真实战（四足、MicroDuck GPU 并行 PPO、ACT+ALOHA 双臂交接）、真机实战（SO-101 + LeRobot 硬件连通）。

选择建议：**every-embodied 适合"跑起来再理解"的实践派，dive-into-embodied-ai 适合偏好系统课程的学习者**，两者都以 LeRobot/SO-101 生态为真机底座，可以接续学习。

## 第 3 章 awesome-embodied-vla-va-vln：论文雷达

> 3.6k★ ｜ 700+ 篇论文 ｜ 2026-09 更新

收录 700+ 篇论文的英文 Awesome 列表，开篇理念是"robotics will experience its own LLM moment"（机器人学将迎来自己的 LLM 时刻）。按技术线分九节：Survey（30+ 篇综述）、VLA & WAM、VLN、VA（含 Diffusion Policy）、MLLM-based Embodied Learning、Physics-aware Policy、Sim2Real、Benchmark、Simulator，每年内按时间倒序排列，代表作置顶。

它的价值在于**紧跟前沿的时效性**：VLA 节已收录 Qwen-VLA、NVIDIA DreamZero/GR00T 1.7/Cosmos Policy、Physical Intelligence π0.6、腾讯 Hy-Embodied-0.5、蚂蚁 LingBot-VLA 等最新工作，每条尽量带"论文 + 项目主页 + 代码"三链接。Simulator 与 Benchmark 节可直接当工具索引用。

局限：无中文导读、以罗列为主缺乏评述——这正是第一篇前几个项目存在的意义：列表给你全貌，指南给你判断。

---

# 第二篇 VLA 模型：机器人大脑的主战场

## 篇首语：2026 年 VLA 的四个共识

第二篇的九个项目——三个框架、五个模型、一个域外变体——勾勒出 2026 年 VLA 版图。先记住四条共识：

1. **三系统/双系统架构成为主流**：Ψ₀（VLM + 扩散动作专家 + RL 全身控制器）、GigaBrain-0.7（三系统）、starVLA-GR00T、Alpamayo 殊途同归——**VLM 出语义（System-2），专用小动作头出高频控制（System-1）**；
2. **动作解码三流派并存**：FAST 离散自回归、MLP 并行回归（OFT）、**flow matching（已成默认选择）**；
3. **框架竞争白热化**：starVLA（研究广度）、dexbotic（复现保真）、FluxVLA（部署纵深）三足鼎立，均与 RLinf/RL 后训练生态对接；
4. **数据配方重于模型规模**：Ψ₀ 用 80 条轨迹微调出新技能、LingBot 清洗 6 万小时数据、GigaBrain 把世界模型生成数据纳入预训练——数据工程是 2026 年的主战场。

## 第 4 章 starVLA：乐高式 VLA 研究框架

> 3.9k★ ｜ MIT ｜ arXiv:2604.05014

starVLA（谐音 "start VLA"）是社区驱动的 VLA 研发平台，定位"研究基础设施"而非单一模型：把 VLA 研究拆为**可插拔的底座（VLM/世界模型）、动作头、数据、训练器、评测**五根轴，新框架变体通常只需换动作头。

官方提供四种同构框架变体（共享数据接口，仅动作头不同）：

- **StarVLA-FAST**：π₀-fast 式 FAST 离散动作 token 自回归解码；
- **StarVLA-OFT**：OpenVLA-OFT 式 MLP 并行连续动作解码；
- **StarVLA-PI**：π₀ 式 flow-matching 动作专家；
- **StarVLA-GR00T**：双系统架构，VLM 做 System-2、flow-matching 做 System-1。

底座支持 Qwen2.5-VL/Qwen3-VL（0.8B–9B）、InternVL、Florence-2 等，并支持 **WM4A 范式**——直接用视频生成 DiT（Cosmos-Predict2、Wan2.2）作动作预测主干，这是"世界模型做动作头"的先行实验。训练配方范式无关：SFT、多模态多目标共训、跨本体共训、RL 后训练（经 RLinf）。

"乐高式"的工程体现：每个子模块可独立跑冒烟测试、dataloader 只返回模型无关的原始 dict、正则冻结与分模块学习率分组。换小模型用 Florence-2-large，**单张 A100 即可训练**；已支持昇腾 NPU。

评测覆盖 LIBERO、SimplerEnv、RoboCasa-365、RoboTwin 2.0、BEHAVIOR-1K 等十余个基准，RoboCasa365 PandaOmron 上 StarVLA-PI 达 40.64%。它也是最早面向 AI 编码代理优化（提供 agent skills）的 VLA 代码库。局限：是方法集合而非单点 SOTA，结果多为研究参考而非官方榜单提交。

## 第 5 章 dexbotic：把各家 VLA 复现到位的工具箱

> 3.2k★ ｜ MIT ｜ arXiv:2510.23511 ｜ Dexmal 星尘智能出品

Dexbotic 主张 "Pretraining · Fine-tuning · Inference · Evaluation" 一站式闭环，内置 π0、π0.5、CogACT、OFT、GR00T N1、MemVLA、NaVILA 及自研 DM0/DM0.5 等主流策略的复现配方。

它的差异化价值在"**复现保真**"：README 系统给出"DB-*（Dexbotic 复现）vs 原版"对比表——LIBERO 上 DB-CogACT 94.9 vs 原版 93.6；CALVIN 上 DB-CogACT 平均长度 4.063 vs 3.246；最惊人的是 SimplerEnv 上 DB-OFT 76.39 vs 原版 30.23——**复现版本大幅超越原论文数字，说明训练配方本身就是重要贡献**。

架构采用"分层配置 + 工厂注册 + 入口分发"，实验即脚本（`playground/benchmarks/libero/*.py`）；统一数据格式 Dexdata（附 LeRobot/RLDS 转换脚本）；DM 系列提供 Triton 实时推理后端，核心推理提速约 5 倍；支持 LoRA SFT、混合共训、GRPO RL 后训练（RLinf 后端）、FSDP2 分布式。硬件覆盖云端与消费级 GPU，另有 Blackwell 专用镜像，适配 UR5、Franka、ALOHA、SO-101、Unitree G1 等多种本体。

与 starVLA 的分工：**starVLA 求"广"（方法多），dexbotic 求"正"（复现准）**。局限：模型变体广度略小，深度绑定 Dexmal 生态（Dexdata、DM 权重）。

## 第 6 章 Psi0：人形全身操作的开源标杆

> 2.9k★ ｜ RSS'26 + CVPR 2026 Workshop 最佳论文 ｜ USC 等机构

Ψ₀ 是开源人形全身 loco-manipulation（移动操作）VLA 中学术背书最强的工作。它的核心命题是数据配方：**先从大规模人类第一视角视频学任务语义与视觉表征，再用少量真机遥操作数据后训练**——微调仅需约 **80 条轨迹**即可习得新的长时程灵巧技能。

三层系统架构：

- **System-2**：Qwen3-VL-2B 视觉-语言骨干，端到端训练；
- **System-1**：约 500M 参数的多模态扩散 Transformer 动作专家（受 Stable Diffusion 3 启发，flow-based 扩散），预测全身动作块；
- **System-0**：RL 全身跟踪控制器（AMO → NVIDIA SONIC）执行下半身动作，保证物理稳定性。

预训练用 Apple EgoDex（48-DoF 动作重标注）+ Humanoid Everyday，后训练用 Unitree G1 遥操作数据；评测在自研 SIMPLE 仿真器（MuJoCo 物理 + Isaac Sim 渲染），并开源了 GR00T N1.6、OpenPi π0.5、InternVLA-M1、H-RDT、EgoVLA 全套基线——这种"连基线一起开源"的做法在领域内堪称表率。

```bash
bash scripts/train/psi0/finetune-real-psi0.sh $task
bash ./scripts/deploy/serve_psi0-rtc-sonic.sh   # RTC 模式部署
```

局限：动作头体量小（500M）、依赖外置 RL 全身控制器、语言能力受 2B 骨干限制、真机生态绑定 Unitree G1。

## 第 7 章 Mano-P：VLA 思想的域外胜利

> 2.8k★ ｜ 明略科技 ｜ OSWorld 专用模型第一（58.2%）

Mano-P 把 VLA 思想迁移到了机器人之外的领域：**GUI 操作智能体（Computer Use Agent）**。"P" = Private——所有操作在本地 Mac mini/MacBook（M4、32GB+）上推理，截图与任务数据不出设备，主打"私有 AI"。

旗舰模型 Mano-CUA-2.0-4B 基于 Qwen3-VL-4B 级底座做 GUI 专用微调，方法三板斧：Mano-Action 双向自增强学习、三阶段渐进训练（SFT → 离线 RL → 在线 RL）、"think-act-verify"循环推理。边缘化三板斧同样精彩：自研 Cider SDK 补齐 W8A8/W4A8 在线激活量化（prefill 较 MLX 提速 1.4–2.2 倍）、**GSPruning 视觉 token 剪枝**（锚点保全局结构 + 语义离群点识别关键 UI 元素，2–3 倍吞吐提升）。

成绩单：OSWorld 58.2%，领先第二名 13.2 个百分点；4B 本地模型（56%）显著超过云端 Qwen3-VL-Plus（39%），单步 7.9 秒。配套 Mano-AFK 实现"自然语言 → 编码 → 部署 → E2E GUI 测试 → 自修复"的全自动应用构建。

它对本书的意义：**VLA 范式（感知 → 推理 → 动作序列）不专属于机械臂**——GUI 与驾驶（见第 11 章）证明它正在成为通用的"感知-决策-动作"骨架。局限：动作空间是鼠标键盘而非机器人自由度，评测成绩以自报为主，生态强绑定 Apple Silicon。

## 第 8 章 giga-brain-0：世界模型数据驱动的三系统 VLA

> 2.7k★ ｜ arXiv:2608.15875 ｜ 前作曾登顶 RoboChallenge 真机榜

GigaBrain 系列是"世界模型驱动 VLA"的代表。GigaBrain-0.7 的两个关键词：

**异构数据规模**：预训练扩展到 **3.73 万小时**异构具身数据——真机、UMI、第一视角、仿真，以及**世界模型生成数据**（这正是其"世界模型驱动"标签的来源：用世界模型造数据喂 VLA）；

**三系统架构 + 一段式对齐**：统一"理解—预测—行动"，联合优化视觉-语言理解与多本体动作生成。底座 PaliGemma2-3B，动作侧用 Physical Intelligence 的 FAST 分词器 + flow matching；跨本体通过 embodiment ID（0–7，覆盖 AgileX、智元 G1、灵巧手、UMI 等）+ per-embodiment 状态/动作投影与 delta-action 掩码实现。

团队生态也值得一记：GigaBrain-0.1 曾登顶 RoboChallenge 真机排行榜，0.5M* 是从世界模型强化学习学习的 VLA，并在 CVPR 2026 主办 GigaBrain Challenge（三赛道竞赛）。

局限：37k 小时数据只公开组成图而非全量；开源部分以 3.5B 模型与工作流为主；配置系统（embodiment ID / delta mask / norm stats）对新手较陡峭。

## 第 9 章 opendm（DM0.5）：成绩单最漂亮的开放世界 VLA

> 2.2k★ ｜ Apache-2.0 ｜ Dexmal 出品 ｜ 已被 LeRobot 官方集成

DM0.5 延续"原生具身建模"路线，系统性升级开放指令、长时程任务、动态干扰与多本体控制。架构是 VLM 语义骨干 + flow matching 动作生成（三路图像输入 Head/Left wrist/Right wrist，chunk-size 50），四个技术亮点：

1. **DM05-MEM 记忆变体**：RoboDojo Memory 赛道登顶的关键；
2. **K=4 multiflow 训练**：同一观测前缀只编码一次，复用给 4 组独立噪声/时间步采样取平均 flow-matching 损失；
3. **fast 推理后端**：TensorRT 视觉引擎 + Triton 前后缀 kernel + FlexAttention + 前缀 MLP FP8 校准 + CUDA Graph；
4. **数据注册机制**：把任意机器人的状态维度语义化（joint/gripper 描述），训练自动计算归一化统计。

它的横向成绩单可能是当前开源 VLA 中最好的：**LIBERO 99.0%**（π0 94.4、π0.5 96.9）；RoboTwin2.0 Clean 93.6%（π0.5 82.7）；VLA-Arena L2 44.1%（π0.5 24.5）；RoboColiseum 四个榜单**唯一全满贯**；真机 RoboChallenge Table30V2 得分 54.42（π0.5 仅 31.48）。KDDI Research 基于 DM0.5 微调的模型已登顶 RoboChallenge Table30 V2 真机榜。

局限：README 基本不披露底座 VLM 与参数规模细节，深度信息需读技术博客；生态绑定 Dexmal 数据注册体系。

## 第 10 章 lingbot-vla / v2：务实路线的完整公开配方

> V1 1.9k★ + V2 1.1k★ ｜ 蚂蚁 LingBot 团队 ｜ arXiv:2601.18692 / 2607.06403

LingBot-VLA 是业内"务实派"代表——不追架构 novelty，靠数据质量与训练工程把指标做扎实。

**V1（4B）**：2 万小时真机预训练数据、9 种主流双臂构型，训练效率较现有 VLA 代码库提速 1.5–2.8 倍；Qwen2.5-VL-3B 底座 + π0 式 flow-matching 动作专家，提供无深度/深度蒸馏两个版本。


**V2（6B）**五项升级值得逐条记录，它几乎是 2026 年 VLA 数据工程的教科书：

1. 预训练语料扩至 **6 万小时**（5 万小时机器人轨迹、20 种构型 + 1 万小时第一视角人类视频），配严苛清洗管线（视频-状态对齐、速度/加速度/加加速度异常过滤）；
2. **统一动作空间**：异构本体映射到 55 维规范状态/动作向量（双臂 14+14、末端位姿 14、夹爪 2、灵巧手 12、腰 4、头 2、底盘 3）；
3. 动作专家内嵌**稀疏 MoE**（细粒度专家 + 共享专家隔离）；
4. **双查询蒸馏**：附加当前与未来感知 query，分别从 LingBot-Depth（几何）与 DINO-Video（时序语义）蒸馏，培养"预测性动力学建模"；
5. **Muon 优化器** + 基于 TorchTitan FlexShard 的 Distributed Muon（单节点 H20 后训练提速 37.6%）。

真机成绩：V2 在 GM-100 自建基准 AgileX 上 34.4 SR（超 π0.5 的 32.2），RoboTwin 93.52/92.80，RTX 4090D 单次推理约 130ms。局限：GM-100 为自建基准，第三方可比性有限；FP32 推理要求较重（BF16 明显掉成功率）。

## 第 11 章 Alpamayo：VLA 开上高速公路

> 2.1k★ ｜ NVIDIA ｜ CES 2026 发布 ｜ arXiv:2511.00088

Alpamayo 1 是 NVIDIA 的自动驾驶 "Reasoning-VLA" 模型：**Cosmos-Reason 视觉-语言骨干 + 扩散动作专家**，定位是自动驾驶长尾场景的推理-动作桥梁。

核心创新是 **Chain-of-Causation（CoC）因果链推理**：通过"人在回路的混合自动标注"产出因果推理轨迹，模型输出"轨迹 + 推理文本"双模结果——先解释"为什么"（前面行人在过马路），再输出"怎么做"（10Hz、64 个路点的 6.4 秒轨迹）。这与机器人侧 Ψ₀/GigaBrain 的 System-2/1 分层互为镜像，是 VLA 分层架构的跨域印证。

工程规格：推理需 ≥24GB 显存（3090/4090/A100 均可）；动作专家支持 CUDA Graph 加速；训练数据为 NVIDIA PhysicalAI-AV 数据集（gated）。注意：本仓库已停止积极开发，新版本在 alpamayo-recipes（Alpamayo 1.5 已发布），RL 权重未释出——引用时需注意版本状态。官方明确免责：未经车规验证，非完整驾驶栈。

## 第 12 章 FluxVLA：把部署侧做成一等公民

> 767★ ｜ arXiv:2609.17210

FluxVLA Engine 是全栈端到端具身智能工程平台，本质是一个"**多模型统一运行时**"：

- **模型侧**同时支持 VLA（OpenVLA、SmolVLA、X-VLA、GR00T N1.5/N1.7、Pi0、Pi0.5）与 WAM 世界-动作模型（DreamZero、FastWAM 及其 IDM/Joint 变体、Cosmos3 全档、DiT4DiT），甚至支持免 checkpoint 的 API 模型（GPT-6 Astra、MiniMax M3.1 Flash）经 OpenAI 兼容接口做策略推理；
- **工程特性**齐全：单配置文件管全流程、FSDP+DDP/LoRA、断点续训、RTC 实时分块、Triton 融合算子 + CUDA Graph（GR00T-RTC 在 RTX 5090 达 **45Hz**）、Jetson Orin 边缘部署（GR00T-N1.5 达 7.4Hz）、ZMQ 远程推理、SARM/ARM 奖励建模、FluxDAgger 数据聚合。

它贡献了罕见的大规模同协议横评：LIBERO 平均 DiT4DiT 98.65、FastWAM-Joint 98.35、PI0.5 97.95，而 SmolVLA 仅 84.7——清晰给出 VLA/WAM 谱系的能力梯度；RoboDojo 上 PI0.5 13.61 显著领先，说明**长时程/记忆类任务仍是全行业短板**。

与 starVLA/dexbotic 同属框架派，但 FluxVLA 工程纵深最足——从 Jetson 边缘、RTC 低延迟到奖励建模与 DAgger 数据闭环，是唯一把"部署侧"做成一等公民的开源平台。局限：无自研预训练基座；安装体系复杂（FFmpeg/TorchCodec/FlashAttention 轮子匹配）。

## 第 13 章 第二篇总结：VLA 选型对照

| 项目 | 类型 | 一句话定位 | 适合谁 |
|---|---|---|---|
| starVLA | 框架 | 乐高式方法集合，四种动作头同构对比 | 研究者做消融实验 |
| dexbotic | 框架 | 复现保真的一站式工具箱 | 工程团队找可靠基线 |
| FluxVLA | 框架 | 部署侧最深的统一运行时 | 上真机/边缘设备的团队 |
| Psi0 | 模型 | 人形全身操作学术标杆 | 人形方向研究者 |
| DM0.5 (opendm) | 模型 | 综合成绩最强的开源 VLA | 要开箱好成绩的团队 |
| lingbot-vla v2 | 模型 | 务实路线完整配方 | 学数据工程的人 |
| giga-brain-0 | 模型 | 世界模型数据驱动规模化 | 关注 WAM×VLA 的人 |
| Mano-P | 域外 VLA | GUI 智能体边缘化 | Agent 方向开发者 |
| Alpamayo | 域外 VLA | 自动驾驶推理 VLA | 自动驾驶研究者 |

---

# 第三篇 仿真平台与世界模型：机器人的"排练场"

## 篇首语

机器人不能在真实世界里试错，于是仿真平台就是它的"排练场"；而世界模型更进一步——把排练场搬进神经网络内部。本篇七个项目构成一条完整光谱：从传统物理仿真（SAPIEN）、多物理统一（Genesis）、照片级渲染（SPEAR）、人类需求基准（BEHAVIOR-1K），到数据与策略的开源生态（LeRobot），再到"模型即仿真器"的世界模型平台（Cosmos、LongLive）。

## 第 14 章 Genesis：把五种物理引擎装进一个场景

> ~30k★ ｜ Apache-2.0 ｜ 前身 2024-12 学术项目，2026-05 升级 Genesis World 1.0

Genesis 是本书调研中 star 数最高的具身智能项目。它的核心理念：把**物理仿真、照片级渲染、跨平台编译**三层能力整合到一套 Pythonic 接口之后，从笔记本单卡一路扩展到数据中心级 GPU。技术架构是清晰的四层栈：

1. **Simulation Interface**：资产解析（URDF/MJCF/OBJ/GLB/USD）、实体访问、控制器、传感器（深度相机、IMU、LiDAR、触觉、接触力、温度场）、异构并行环境；
2. **Physics——统一多物理引擎**：这是 Genesis 最独特的差异点。刚体（参考 MuJoCo）、FEM、MPM 物质点法、PBD/SPH 粒子、IPC 沙盒接触整合在同一场景、同一状态中，通过显式 Coupler 实现**刚-柔-流体跨求解器耦合**——布料贴合刚体、水车、切割这类"混合物理"任务是其他引擎难以企及的；
3. **Render**：三条渲染通路——自研 Nyx 渲染器（3D Gaussian Splat + PBR）、Luisa 光线追踪 DSL、Pyrender 光栅化；
4. **Compiler**：Quadrants（从 Taichi fork 而来），把 Python kernel 编译到 CUDA、ROCm、Metal、Vulkan、x86、ARM64。

安装极简：`pip install genesis-world`（Python 3.10–3.14）。一段值得记录的历史：2024 年末初版 README 曾宣传"比 CPU 仿真快数十万倍"引发学界争议，1.0 版已不再出现性能数据，转而强调可读性与多后端兼容——**社区评价焦点已从"速度神话"转向"多物理统一"与"跨硬件编译"这两个真实差异化能力**。这本身是开源社区纠偏机制的一次示范。

## 第 15 章 LeRobot：机器人界的 Hugging Face

> ~28.6k★ ｜ Apache-2.0 ｜ ICLR 2026 论文 ｜ 2026-10-29（昨日）仍在更新

如果说 Genesis 重造了仿真引擎，LeRobot 重造的则是**生态**：把 Hugging Face 在 NLP/CV 领域的"开源模型 + 共享数据集 + 低门槛工具链"范式完整复制到机器人领域。三大核心组件：

1. **硬件无关的 Robot 接口**：统一 `connect / get_observation / send_action` 三步式控制循环；原生支持 SO-101 低成本主从臂、OpenARM、LeKiwi、Unitree G1、Reachy 2 等整机与手柄/键盘/手机遥操作；
2. **LeRobotDataset 数据格式**：视觉用同步 MP4、状态/动作用 Parquet，托管在 HF Hub 支持流式加载——直指机器人领域"数据碎片化"痛点；
3. **SoTA 策略库**（纯 PyTorch）：模仿学习（ACT、Diffusion Policy、VQ-BeT）、强化学习（HIL-SERL、TDMPC）、**VLA 多达 14 个**（Pi0/Pi0.5、GR00T N1.7、SmolVLA、XVLA、EO-1、WALL-OSS 等）、世界模型（VLA-JEPA、FastWAM 等）与奖励模型。

训练一条命令：

```bash
lerobot-train --policy.type=act --dataset.repo_id=lerobot/aloha_mobile_cabinet
lerobot-eval --env.type=libero --env.task=libero_object
```

生态是其最大护城河：第三方硬件插件（UFACTORY、UR5e、Franka、AgileX Piper、GELLO、Meta Quest、ROS 2 桥接）通过 `lerobot_robot_*` 前缀自动发现注册；配套 LeLab 浏览器工具、免费课程、活跃 Discord。可以说，**2026 年具身智能的开源数据与模型流通，事实标准就是 LeRobot**。局限：VLA 基座微调显存要求高（40GB+ 或需 LoRA）；插件质量参差；真机调试仍需工程能力。

## 第 16 章 BEHAVIOR-1K：用真实人类需求定义的难度标尺

> 1.8k★ ｜ 斯坦福（李飞飞、Jiajun Wu 等）｜ arXiv:2403.09227

BEHAVIOR-1K 最大的特色是**任务定义方式**：团队通过大规模时间使用调查与偏好研究，直接问人们"希望机器人帮你做什么"，从回答中筛选出 **1000 项日常家务活动**（清洁、烹饪、整理）——而非研究者拍脑袋定义的任务集。

基准构成：1000 个活动 × 50 个完全可交互场景 × 10,000+ 物体。仿真底座是自研 OmniGibson（构建于 NVIDIA Omniverse 之上），强调感知、物理、交互三维真实性：支持流体、可变形体、透明体、热效应（加热/冷却）以及物体状态转变（切开、混合、烹饪）。任务用 **BDDL** 声明式定义，含谓词逻辑式的成功条件。

它的"残忍"之处在于难度：长时程移动操作任务对当前 VLA/RL 策略极难，**榜单成绩普遍偏低**——这既是局限，也是它作为"下一代难度标尺"的价值：当 LIBERO 被各家刷到 95%+ 时，BEHAVIOR-1K 仍能分辨强弱。局限：依赖 NVIDIA GPU 与 Omniverse 栈，配置门槛高；已实例化可直接评测的任务约占 1/4。

## 第 17 章 SAPIEN：铰接物体仿真与 ManiSkill 的地基

> 862★ ｜ UCSD/斯坦福/SFU ｜ 2026-10 活跃

SAPIEN 是"物理丰富、面向铰接物体"的仿真环境——门、抽屉、水龙头、家电这些带运动关节的物体，恰恰是家庭操作任务的主角。其数据集 PartNet-Mobility 提供数千个带运动学标注的铰接物体。

3.0 版完成核心重构：**实体-组件系统**（ECS）+ **PhysX 5 GPU 并行仿真**——后者是其上层项目 ManiSkill 构建大规模并行机器人学习环境的基础。渲染侧 SapienRenderer 支持光线追踪与 GPU 加速立体深度传感器仿真。

SAPIEN 的最大影响力通过下游放大：**ManiSkill** 系列基准与 **RoboTwin 2.0**（ICML 2026，双臂操作数据合成平台，也是 Embodied-AI-Guide 入门教程的底座）都构建其上，使它成为国内具身操作研究的事实基础设施之一。相比 Genesis 的"多物理大而全"，SAPIEN 定位更聚焦"铰接物体 + 高效并行"——功能面较窄，但工程成熟度高。

## 第 18 章 SPEAR：把整个 Unreal Engine 变成仿真器

> 620★ ｜ ECCV 2026 ｜ MIT + CC0 ｜ 作者含 Vladlen Koltun、German Ros

SPEAR 的设计反常规：它**不是又一个封装好的仿真器，而是一个 Python 库**——通过模块化插件架构连接并程序化控制任意 Unreal Engine 应用，直接继承 UE 全生态的照片级资产与渲染能力。

三个核心技术：

1. **超大规模可编程性**：通过 UE 反射系统向 Python 暴露 **14,000+ 个 UE 函数**，比现有 UE 系仿真器高一个数量级；
2. **高速传感器渲染**：单实例将 1920×1080 照片级图像以 **73 FPS** 直接写入 NumPy 数组，并提供独有 ground-truth 模态：非漫反射本征图像分解、材质 ID、物理着色参数；
3. **事务式编程模型**：`begin_frame()/end_frame()` 定义事务，帧内 Python 代码构成确定性执行的 UE 工作图，每个函数有 `call_async` 异步变体。

应用演示横跨 Epic 官方样例：控制 6 种不同动作空间的具身智能体（CitySample 行人与汽车、StackOBot 飞行机器人、跑酷角色、四足）、MetaHuman 多视角人脸渲染、与 MuJoCo 实时协同仿真，甚至让 AI 编码助手用自然语言写 SPEAR 程序编辑场景。

局限：依赖完整 UE 工程，构建链重；物理仍需外接（UE Chaos 对机器人精度场景不够用）。但"把任意 UE 应用变成具身 AI 仿真器"的思路，为照片级 Sim2Real 数据生成提供了独特的开源路径。

## 第 19 章 Cosmos：世界模型即仿真器

> ~12.2k★ ｜ NVIDIA ｜ OpenMDW-1.1 许可

NVIDIA Cosmos 是面向物理 AI 的开放世界基础模型平台，当前家族 **Cosmos 3**（2026-05）是构建于统一 **Mixture-of-Transformers** 架构上的 omnimodal 世界模型，口号是 "one model family that sees, reasons, simulates, and acts"。

**双塔架构**：

- **Reasoner**（文本+视觉 → 文本）：世界理解、接地、任务规划与具身推理——例如"滚来的球意味着后面可能有孩子"这类危险预判；
- **Generator**（文本/视觉/音频/动作 → 视觉/音频/动作）：世界仿真、未来预测、合成数据生成与策略学习。

按部署规模分三档：**Super 64B**（H200/B200，最高质量）、**Nano 16B**（RTX Pro 6000/H100，最常用后训练底座）、**Edge 4B**（跑 Jetson AGX Orin/Thor，实时机器人策略）——**把世界模型推到机器人端侧是 2026 年的标志性进展**。

官方 checkpoint 展示了后训练方向：Nano-Sim-Auto（闭环驾驶仿真，多视角+LiDAR）、Nano-Sim-Bimanual（闭环机器人策略仿真）、Nano-Policy-DROID（开源 SOTA 机器人策略）等。平台工具链完备：Cosmos Framework（SFT/LoRA/RL/蒸馏）、Cosmos Curator（数据清洗）、Cosmos Evaluator（评测）。推理后端支持 Diffusers、vLLM、SGLang、TensorRT-LLM 与 NIM，并可暴露 OpenAI 兼容 API。

局限（官方明示）：长时程、高分辨率、物理复杂输出仍有时间不一致、物体形变、动力学失真等伪影，安全关键应用需额外系统级验证。

## 第 20 章 LongLive：从实时长视频到可部署机器人策略

> 2.8k★ ｜ Apache-2.0 ｜ NVIDIA 研究院 ｜ 四个子项目

LongLive 仓库串起了一条从"实时长视频生成"到"机器人策略世界模型"的完整技术演进线，四个自包含子项目：

1. **LongLive 1.0**（ICLR 2026）：实时交互式长视频生成。三项核心技术：attention sink（稳定流式自回归）、KV-recache、streaming long tuning；LongLive-1.3B 在单张 H100 上达 20.7 FPS（扩散帧率，不含 VAE 解码——引用时注意口径）。
2. **LongLive 2.0**：NVFP4 量化 + 序列并行的长视频训练/推理基础设施；LongLive-2.0-5B 在单张 GB200 上 24.8–45.7 FPS，VBench 最高 85.06。
3. **LongLive-Plug**："Once-for-All" 蒸馏——把 CFG、少步化等可复用能力训成功能 LoRA，一次训练可挂载到任意兼容下游模型（覆盖 FantasyWorld、DreamZero、Matrix-Game 3.0 等世界模型）而无需重训。
4. **Long-WAM**（2026-10，arXiv:2610.10528）：**长上下文世界动作模型**——核心是流式视觉记忆，在机器人视频上预训练，支持流式逆动力学推理；基准覆盖 LIBERO、RoboTwin 2.0、RoboCasa（上下文从 0s 扩展到 9.6s）；真机部署覆盖 YAM、Franka、Unitree G1，提供 RTX 5090、DGX Spark、AGX Thor 运行档位。

Long-WAM 的意义：它是"视频世界模型 → 可部署机器人策略"路线（对标 Genie 系、DreamerV3 传统）在 2026 年**最完整的开源工程化样本**。衍生生态庞大：17+ 个后续工作。与 Cosmos 的分工——Cosmos 做"基础模型平台化"，LongLive 做"流式生成与 WAM 的研究前沿"，两者共同指向 World Action Model 这个具身智能与视频生成的交汇点。

---

# 第四篇 人形机器人与真机硬件：给 AI 一副身体

## 篇首语：硬件开源化的 2026

如果说 2025 年开源社区还在争论"人形机器人能不能开源"，2026 年的答案已经是"从 CAD 图纸到 RL 策略全链路开源"。本篇八个项目构成一条完整的身体供应链叙事：**整机（asimov-1、roboto_origin）→ 部件（openarm）→ 遥操作与数据采集（xr_teleoperate）→ 动作数据管线（GMR、soma-retargeter）→ 训练与部署（ProtoMotions、rl_sar）**。

## 第 21 章 OpenArm：6500 美元的人形双臂

> 3.7k★ ｜ CERN-OHL-S（硬件）/ Apache-2.0（软件）｜ enactic 出品

OpenArm 是全开源 7 自由度人形机械臂，核心卖点是**高反向可驱动性（backdrivability）与柔顺性**——专为安全的人机交互设计，同时保留实用负载。双臂完整系统售价 **6500 美元**，定位类似"机械臂界的 SO-100，但达到人形尺度和准力控水平"。

技术栈采用"一硬件、多仓库"的模块化组织：`openarm_hardware`（CAD/BOM）、`openarm_description`（URDF）、`openarm_can`（CAN 总线控制）、`openarm_ros2`（ros2_control + MoveIt）、`openarm_mujoco`/`openarm_isaac_lab`（仿真）、`openarm_dataset`（含 LeRobot 转换）。

2.0 版两个创新值得记录：**OpenArm KER**——无电机、与 2.0 运动学一致的"领导者臂"，用于低成本遥操作数据采集；**OpenArm Cell**——统一背景、光照与相机位姿的标准化实验环境，使全球不同实验室的操纵结果可复现、可对比——这是对"具身智能评测不可比"痛点的直接回应。

软件编排基于 dora-rs 数据流框架：每个功能是一个节点，用 dataflow YAML 连成应用，可在真臂与 MuJoCo、KER 与 VR 遥操作之间替换节点。局限：仅覆盖上肢（无腿无全身移动）；Isaac Lab 2.0 支持仍在进行中；"Cell 标准环境"的推广依赖社区采用。

## 第 22 章 Asimov-1：大厂级开源人形

> 1.4k★ ｜ CERN-OHL-S-2.0 + GPL-2.0 ｜ Menlo Research（LMArena 幕后团队）

Asimov 1 提供了 1 米级人形中罕见的完整开源度：**完整机械 CAD、电气 CAD（原理图 + PCB）、MuJoCo 仿真模型与机载软件**。两种获取方式：2 万美元的未组装 DIY 套件（已发货），或完全自购料（提供 BOM 与装配手册）。

规格：身高 1.2m、体重 35kg、25 个驱动自由度（双腿 6×2、双臂 5×2、腰 1、颈 2）。架构务实：5 条 CAN 总线；计算双板分工——树莓派 5 管媒体网络，Radxa CM5 专责运动控制；7075 铝合金 + MJF PA12 尼龙结构。负载指标按真实动作标注而非空泛"额定负载"：深蹲 5kg、双臂弯举 5kg（峰值 15kg）——反映较认真的工程态度。

关键现状：路线图上机械 CAD、MuJoCo 模型、线束与 PCB 已完成，但**运动控制策略（locomotion policy）尚在开发中**——即当前阶段是"硬件开源先行，大脑后补"。$20k 套件标出了全尺寸双足人形的现实成本基线，而行走策略的缺失也提醒我们：**具身智能的瓶颈从来不只是硬件**。

## 第 23 章 ROBOTO_ORIGIN：淘宝 + 嘉立创就能复现的人形

> 2.8k★ ｜ 上海 RoboParty ｜ 2025-04 启动，4 个月出能跑能跳的原型

这是中文圈最彻底的"全民 DIY 人形"尝试：README 明确承诺**所有零件可经淘宝采购 + 嘉立创打样独立复现**，配合开源训练部署代码即可实现行走奔跑——整个研发过程（结构、电子、训练、部署）全部开源。

仓库定位是"快照聚合仓"：`modules/` 下聚合 8 个子仓库——`rpo_hardware`（CAD/PCB/BOM）、`rpo_description`（URDF/MJCF）、`roboparty_train`（Isaac Lab RL 训练，支持 Sim2Sim 到 MuJoCo）、`roboparty_deploy`（ROS2 Humble 部署框架）、`roboparty_firmware`（嵌入式固件）、`roboparty_xr_teleop`（XR 遥操作）、`roboparty_navigation`（导航）。技术栈：ROS2 Humble + Isaac Lab 2.1.1 + rsl_rl。


它的价值在于**证明了中国供应链下人形复现的可行性**——把全尺寸人形机器人的入场券从"百万元级实验室预算"拉到"创客级预算"。局限：公司成立仅一年，长期维护存疑；性能上限受 DIY 执行器约束；"能跑能跳"的原型与量产可靠性之间距离未知。与 Asimov-1 对照很有意思：一个走"精密工程 + 高价套件"，一个走"供应链民主化 + 低门槛"——两条路线都在试探开源人形的边界。

## 第 24 章 xr_teleoperate：戴上头显教机器人干活

> 1.7k★ ｜ 宇树官方 ｜ 衍生自 OpenTeleVision

宇树官方的开源遥操作框架，用 Apple Vision Pro、PICO 4 Ultra、Meta Quest 3 遥操作宇树人形机器人——是 G1/H1/H2 用户做数据采集的事实标准工具。三层系统：

1. **视觉回传**：机器人头部相机经 WebRTC 把第一视角流推到头显浏览器；
2. **姿态输入**：头显手部/手柄追踪经 Vuer WebSocket 上行；
3. **运动解算**：Host 端 arm IK（pinocchio），经 CycloneDDS 下发到真机或 Isaac Lab 仿真。

覆盖 G1 23/29 DOF、H1、H1_2、H2、R1 全系，末端支持 Dex3-1 灵巧手、Inspire、BrainCo 手部控制器。典型流程：

```bash
python teleop_hand_and_arm.py --input-mode=hand --arm=G1_29 --ee=dex3 --record
# 浏览器打开 https://<host-ip>:8012 → 点击 Virtual Reality → 按 r 开始遥操，s 起止录制
```

录制数据配套 `unitree_IL_lerobot` 仓库转 LeRobot 格式训练——**遥操作 → 数据 → 模仿学习**的链路完全打通，这正是 VLA 时代数据金字塔最底层的采集工具。局限：部署摩擦大（证书、防火墙、固定网段 192.168.123.x）；仅支持宇树自家机型；无线丢包影响体验。

## 第 25 章 GMR：人体动作到任意人形的通用翻译官

> 2.8k★ ｜ ICRA 2026 ｜ MIT ｜ 斯坦福 Karen Liu 组与 Yanjie Ze 等

"人形机器人从人类动作中学习"是当前最重要的数据放大路线——人类视频无限，机器人遥操作数据稀缺，中间需要一座桥：**运动重定向（Motion Retargeting）**。GMR 就是这座桥的事实标准之一。

任务定义：输入每帧人类动作（各部位全局平移 + 旋转四元数），输出每帧机器人状态（基座位姿 + 关节角）。内部求解器基于 mink + MuJoCo 的解析+数值混合 IK。三个关键设计使其广受欢迎：

1. **实时性**：i9-13900K 上 35–45 FPS，足以支撑实时全身遥操作；
2. **针对 RL 跟踪策略的精心调参**——重定向结果分布直接影响下游 motion tracker 训练质量；
3. **广覆盖**：已支持 **17+ 种人形机器人**（Unitree G1/H1/H2、Booster K1/T1、KUAVO、Talos、Galaxea R1 Pro 轮式等）与 5 类人类数据源（SMPL-X/AMASS、LAFAN1 BVH、Xsens BVH、OptiTrack FBX、PICO VR 流）。

```bash
uv run scripts/smplx_to_robot.py --smplx_file <...> --robot unitree_g1 --save_path <...> --rate_limit
# 视频→机器人：先用 GVHMR 从单目视频提取人体姿态，再 gvhmr_to_robot.py
```

作者自己承认的局限：为所有人体设计单一 IK 配置并不容易（TEST_MOTIONS.md 收录失败案例）；非标准 BVH 骨架会加载失败。维护极其活跃（几乎每周更新），NVIDIA MimicKit、BeyondMimic 等生态均已接入其格式。

## 第 26 章 SOMA Retargeter：NVIDIA 式的工业化重定向

> 690★ ｜ Apache-2.0 ｜ NVIDIA 生态

与 GMR 的"多数据源实时重定向"定位不同，NVIDIA 的 SOMA Retargeter 专注 **BVH 离线批量转换 + 工业化调参工具链**。五步管线：读 SOMA 骨架 BVH → 按机器人比例缩放 → 逐帧 IK → **接触稳定化与关节限位钳制**（含足地接触检测与 foot-plant 校正）→ 写出 CSV。

附带的两个进阶工具体现了工业级用心：**Robot Configurator**（引导式添加新机器人）与 **IK 权重优化器**（用 15 条动作自动调 IK 目标权重）。内置 unitree_g1/h2、booster_t1 及 AGIBot 国产机型等 5 个目标。

它是 NVIDIA SOMA 生态的关键一环：SOMA-X 身体模型 → GEM-X 视频动捕 → Kimodo 动作生成 → **SOMA Retargeter** → ProtoMotions 训练 → SONIC 全身控制——形成从人体数据到机器人策略的完整闭环。局限：只接受 SOMA 基准骨架命名的 BVH；输出是纯运动学动画，须在仿真验证后才可上真机。

## 第 27 章 ProtoMotions：从 AMASS 到真机 G1 的零样本迁移

> 2.4k★ ｜ Apache-2.0 ｜ NVIDIA ｜ 作者含 Xue Bin Peng 与 Erwin Coumans（PyBullet 作者）

ProtoMotions 是"物理仿真数字人 + 人形机器人"运动技能训练的工程化整合，四大支柱：

1. **多仿真器抽象**：同一策略一键在 IsaacGym / Isaac Lab 3.0 / NVIDIA Newton / MuJoCo 间切换（sim2sim 测试）；
2. **大规模动作学习**：4 张 A100 约 12 小时学完 AMASS 全库（40+ 小时人类动作）；支持 24×A100、每卡 1.3 万条动作的扩展；
3. **内置重定向**：v3 起基于 PyRoki 的优化器，一条命令把整个 AMASS 重定向到任意机器人；
4. **模块化 MDP**：任务由 Control/Observation/Reward/Experiment 四层组件拼装，新 RL 算法约 50 行代码接入。

最亮眼的成绩：在 BONES-SEED（约 14.2 万条动作）上训练单一 General Tracking Policy，**零样本迁移到真机 Unitree G1**。部署管线导出内嵌观测计算的 ONNX 模型，实机侧只需提供原始传感器信号；换机器人只改一个参数：`--robot-name=h1_2`。

局限：大配方以 A100 集群为基线；部署 runtime 标注为"原型参考代码"；对 SMPL/SOMA 人体资产有许可依赖。

## 第 28 章 rl_sar：从仿真策略到真机跑步的最后一公里

> 1.5k★ ｜ Apache-2.0 ｜ fan-ziqi

rl_sar（simulation and real）解决的是具身 RL 最实际的问题：**legged_gym 里训练好的策略，怎么跑到真机上？** 它的答案是把 `.pt` 策略经 libtorch 或 onnxruntime 推理，在 ROS/ROS2 或纯 CMake 环境中驱动 Gazebo/MuJoCo 仿真器或真机。

工程成熟度是其最大优点：双训练栈（IsaacGym/Isaac Lab）、双仿真器（Gazebo/MuJoCo）、双推理后端（libtorch/onnx）、游戏手柄安全映射（A 键初始姿态、LB+RB 急停、RB+方向键切换技能槽）、复位/急停机制齐备。已验证 12+ 款机器人，其中 Unitree A1/Go2/Go2W/G1、智元 D1 等有真机验证记录。

```bash
./cmake_build/bin/rl_sim_mujoco g1 scene_29dof     # MuJoCo 仿真
./cmake_build/bin/rl_real_go2 <NETWORK_INTERFACE>  # Go2 真机（192.168.123 网段）
```

局限：只处理"已训练好的策略推理"（训练需外接 legged_gym/robot_lab）；Gazebo Classic 已停止上游演进；安全免责声明明确——无线连接有失控风险。它是第四篇的合适终点：**从数据采集（遥操作）到动作翻译（重定向）到策略训练（ProtoMotions）再到真机部署（rl_sar），具身智能的身体供应链已经全部开源。**

---

# 第五篇 训练/推理基础设施：具身智能的"修路人"

## 篇首语

当模型和硬件都开源之后，剩下的问题是如何把它们高效地训练、部署、评估、进化。本篇六个项目恰好构成一个闭环：**训练（RLinf、wall-x）→ 用与进化（RPent、Zetta）→ 数据与评测（rerun、inspect-robots）**。它们共同标志着具身智能正在获得自己的"vLLM 时刻"。

## 第 29 章 RLinf：具身 RL 基础设施的事实领跑者

> ~5.5k★ ｜ OSDI 2026 ｜ Isaac Lab v3.0 官方采用

RLinf（Infrastructure + Infinite）是面向具身智能与 Agentic AI 的大规模强化学习基础设施，学术背书密度极高：主论文《Flexible and Efficient Large-Scale Reinforcement Learning via Macro-to-Micro Flow Transformation》发表于 **OSDI 2026**，DynaRL 同会，RLinf-VLA 与 RLinf-USER 入选 RSS 2026，FUSCO 入选 NSDI 2027。产业采用方包括智元、X Square、PsiBot、Dexmal、摩尔线程。

核心思想是"**宏到微流变换**"：用户以数据流图（macro flow）声明 RL 工作流（rollout ↔ 推理引擎 ↔ 训练器），框架自动拆分为微任务并做弹性调度。三大能力：

1. **灵活性**：PPO/GRPO/SAC/DAPO 等算法 × 16+ 仿真器（LIBERO、ManiSkill、RoboTwin、BEHAVIOR、Genesis…）× 11+ VLA 模型（π₀/π₀.₅ 原生重实现、OpenVLA-OFT、GR00T、Cosmos3、FastWAM…）自由组合；
2. **性能**：具身 RL 吞吐最高 **2.434×**；对 BEHAVIOR 的系统级优化把 rollout 延迟从 1028.7 ms/step 降到 41.2 ms/step（**25×**）；
3. **异构硬件抽象**：同一套栈跑 CUDA、AMD ROCm、华为昇腾、摩尔线程 MUSA 等**八种加速器后端**，换机器零改代码。

v0.4（2026.10）把 RL 从仿真带入真机：`rlinf.robotics` 统一机器人接口，真机后训练算法 RLT/STEAM/OPD/RTC，以及 **CEL——96 台真机器人、5 个真实任务的集体经验学习**。

局限：概念多、月度大版本迭代快，文档追代码有滞后；环境依赖链庞大（推荐 Docker）；VLA RL 通常需多卡，个人开发者门槛高。

## 第 30 章 RPent：冻结的 VLA + 记忆引导的 Agent

> 1.6k★ ｜ RLinf 团队姊妹项目 ｜ arXiv:2607.08448

RPent（Recursive Physical Agent）代表 2026 年的热方向：**LLM Agent 包裹冻结 VLA**。核心命题："冻结的 VLA + 记忆引导的 Agent 组合 > 单独的 VLA"。

架构分工：Agentic Planner（Claude Code / Codex，即用前沿 LLM 做高层规划）调用 Action Primitive 层（π₀.₅、RLDX-1、LingBot-VLA 等 VLA 作为可操纵原语），辅以 SAM 3.0 做视觉 grounding；执行结果沉淀为**任务记忆**（GLOBAL_MEMORY.md + 任务级记忆文件），供后续任务检索——记忆蒸馏闭环。

```bash
rpent --robot libero --suite libero_object_swap --task 2 --seed 0 \
  --planner claude_code --model claude-opus-4-8 --dashboard
```

成绩：LIBERO-PRO 全八套件 **92.63% 总成功率（741/800）**。它不动模型权重、纯上下文/记忆层提分，部署成本极低，且与 RLinf 互补——**RPent 负责"用"，RLinf 负责"训"**。

局限需冷静看待：依赖商用 LLM API（成本、延迟、可复现性）；92.63% 的高分建立在记忆与评测集的关系上，跨域鲁棒性需谨慎解读；长时程任务的错误级联（规划错 → VLA 执行错 → 记忆污染）仍缺系统性防护。

## 第 31 章 Wall-X：国产具身基础模型的全栈开源

> 1.3k★ ｜ 星海图 X-Square Robot ｜ arXiv:2509.11766 / 2605.30877 / 2606.01955

Wall-X 是 WALL 系列具身基础模型的全栈开源（权重 + 训练 + 推理 + serving）。模型谱系三步走：WALL-OSS（Igniting VLMs toward the Embodied Space）→ Wall-OSS-0.5（梯度桥接预训练，可直部署）→ **WALL-WM 世界动作模型**（在语义事件边界处耦合未来视频想象与动作预测——又一个 VLA×世界模型合流的实例）。

架构特色是"**VLM 骨干 + 双动作头**"：以 Qwen2.5-VL MoE 为底座，两条动作解码分支——**Flow Matching 头**（连续平滑，高精度闭环操纵）与 **FAST 头**（离散 token 自回归，高响应率），同一模型兼顾两种需求。训练侧亮点：DMuon 优化器与自研 CUDA 算子随 `pip install` 编译；数据格式对齐 LeRobot。

```bash
python -m wall_x.trainer.fsdp_trainer.train_fsdp --config <config.yml>   # FSDP 微调
bash scripts/run_serving.sh --checkpoint-path ... --port 32195           # WebSocket 服务
```

局限：README 对模型设计细节着墨少（重定向到三篇论文）；FAST 分支 tokenizer 训练文档薄弱；社区以微信群为主，英文生态弱于 π0/OpenVLA。

## 第 32 章 Zetta：不训模型，只进化 harness

> 1.3k★ ｜ Apache-2.0 ｜ arXiv:2608.16590 ｜ LIBERO-Pro 90.8% + 11.1× 推理加速

Zetta 是"test-time 自进化"路线最严谨的开源实现：**冻结基础策略，在线进化"代码化运行时批评家（critic）与恢复技能（recovery skills）"**——本质是用 LLM 写"补丁代码"而非微调权重。

进化协议的严谨度令人印象深刻：50 条开发 rollout（种子避开 1–20）→ 失败轨迹聚类 → 因果诊断 → 生成 Critic-Recovery 候选包 → 影子回放 → **同种子配对门控** → 留出种子 1–20 终测 → 晋升或打回。配预注册种子、内容哈希存证、基础设施故障与任务失败分账等防作弊纪律——把评测协议内建为系统约束，学术上站得住。

推理加速同样亮眼：FlashRT 后端以 FP8 + CUDA Graph 服务 π₀.₅——单 RTX 4090 上单样本延迟 230.4 → 23.3 ms（**9.9×**），与基线动作余弦相似度 0.9989。

局限：恢复技能是"代码包"，泛化上界受 LLM 写代码能力限制，对感知层系统性缺陷无能为力；进化流程重度依赖前沿 LLM API；harness 代码复杂度高（manifest/queue/gate 多层机制），上手曲线陡。

RPent 与 Zetta 代表了同一思想的两个变奏：**当基础模型冻结时，智能来自周围的系统**——这是 2026 年最值得关注的架构哲学转向之一。

## 第 33 章 Rerun：物理 AI 的数据层

> ~11.7k★ ｜ MIT / Apache-2.0 ｜ Python/Rust/C++ 三 SDK

Rerun 自称"物理 AI 的数据层"（The data layer for physical AI），已超越"可视化工具"定位：在共享列式存储上统一完成**记录、查询、可视化、流向训练**的多模态数据流水线。

三个关键设计：

1. **实体路径 + 时间线数据模型**：一切数据以 `rr.log("path/to/entity", rr.Points3D(...))` 记录在实体树上，多速率多模态流（图像、点云、变换、关节状态）按任意时间线同步索引——因而能"回到过去"逐帧检查任意传感器组合；
2. **列块存储引擎（Rust 实现）**：同一份数据既喂给内置 Viewer 实时渲染，也可用 SQL 查询、直接流式进训练——"no export jobs, no stale copies"；
3. **格式包容**：原生支持 MCAP、rrd、LeRobot 等录制格式。

```python
rr.init("app"); rr.spawn()
rr.set_time("frame", sequence=42)
rr.log("cam/rgb", rr.Image(...))
```

典型叙事：扫地机器人撞墙，文本日志无解——需要把 RGB/深度/激光/检测框/3D 地图全部对齐到同一时间轴上回放定位根因。Rerun 就是这个场景的标准答案，LeRobot 等生态已默认集成。局限：实体数过多时 viewer 卡顿；API 仍在演进、预期 breaking changes；对纯 RViz 需求而言过于重量。

## 第 34 章 inspect-robots：给物理 AI 装上"考试系统"

> 717★ ｜ MIT ｜ alpha 状态

如果说 rerun 解决"看清过程"，inspect-robots 解决"量化结果"。它的口号是"If you know Inspect AI, this is that for robotics"——把英国 AI 安全研究所的 LLM 评测框架范式（可审计日志、评分器、HTML 报告）搬到机器人领域：**基准定义一次，任意策略（LLM Agent、VLA）× 任意本体（真机/仿真器）**组合评测。

核心抽象三件套：**Task**（场景 + scorer + max_steps）、**Policy**（策略槽：VLA、LLM agent、脚本皆可）、**Embodiment**（本体槽：经插件注册）。真机插件覆盖 I2RT YAM、Franka FR3/Panda、AgiBot A2 Ultra、Unitree G1、SO-100/101 等；策略插件包括前沿 LLM 经工具调用驱臂（每步动作块需 approver 检查）、code-as-policy（LLM 写 Python 调 SAM3 + Pyroki IK）与语音插件。

独有功能：`inspect-robots summarize logs/failed-run.json` 生成 learnings 文件喂给下次运行——**失败摘要驱动的学习闭环**；全程 Rerun 实时流式可视化，产出含 LLM 完整对话记录的可审计日志。

它补齐了 VLA 时代"结果不可比、过程不可查"的短板。局限：alpha 状态 API 不稳定；真机控制 Linux-only；评分器较朴素（长时程/部分成功/安全指标待扩展）。

---

# 第六篇 趋势与展望

## 第 35 章 2026 具身智能技术版图

把全书 35+ 项目放回一张图：

```
学习地图    Embodied-AI-Guide ｜ Octoday ｜ every-embodied ｜ dive-into ｜ awesome-VLA
──────────────────────────────────────────────────────
VLA 框架    starVLA（广度）｜ dexbotic（保真）｜ FluxVLA（部署）
VLA 模型    Psi0（人形）｜ DM0.5（综合最强）｜ LingBot-VLA2（数据工程）
            GigaBrain-0.7（世界模型数据）｜ Mano-P（GUI）｜ Alpamayo（驾驶）
──────────────────────────────────────────────────────
仿真平台    Genesis（多物理）｜ SAPIEN（铰接）｜ SPEAR（照片级）｜ BEHAVIOR-1K（基准）
开源生态    LeRobot（数据+模型+工具事实标准）
世界模型    Cosmos（平台化）｜ LongLive/Long-WAM（流式 WAM）
──────────────────────────────────────────────────────
本体整机    Asimov-1 ｜ ROBOTO_ORIGIN ｜ OpenArm（部件）
数据采集    xr_teleoperate（遥操作）｜ GMR / SOMA Retargeter（重定向）
训练部署    ProtoMotions ｜ rl_sar
──────────────────────────────────────────────────────
RL 基础设施 RLinf（OSDI 2026）
Agent 层    RPent（记忆 Agent）｜ Zetta（自进化 harness）
具身模型    Wall-X（WALL-OSS/WM）
数据与评测  rerun（数据层）｜ inspect-robots（评测）
```

## 第 36 章 趋势观察与选型建议

### 36.1 五大趋势

**趋势一：VLA 从论文走向平台化工程。** starVLA/dexbotic/FluxVLA 三家框架 + LeRobot 生态，让"训练一个 VLA"从课题组手搓变为标准化流程。垂直分化同步发生：GUI（Mano-P）、驾驶（Alpamayo）、人形全身（Psi0）。

**趋势二：世界模型与具身智能正式合流。** GigaBrain 用世界模型造数据，WALL-WM 在语义事件边界耦合想象与动作，Cosmos 把世界模型推到 Jetson 端侧，Long-WAM 让视频世界模型直接产出机器人策略——**"在脑内排练，在现实中行动"正在成为标准范式**。

**趋势三：智能从权重迁移到系统。** RPent（记忆 Agent）与 Zetta（自进化 harness）证明冻结模型 + 系统层进化可以大幅提分；三系统架构（System-2 语义 + System-1 动作 + System-0 反射控制）成为共识。"模型只是智能的一部分"是 2026 年最重要的架构认知。

**趋势四：基础设施与评测工程化。** RLinf 上 OSDI、Isaac Lab 官方采用、rerun 自称"数据层"、inspect-robots 引入可审计评测——具身智能正在补齐"系统与工程"这门课，这正是 LLM 领域 2023–2024 年走过的路。


**趋势五：中国开源力量的全链条崛起。** 从学习资源（Embodied-AI-Guide、every-embodied）到模型（DM0.5、LingBot、GigaBrain、Wall-X）到硬件（宇树、OpenArm、roboto_origin）到基础设施（RLinf 的八种国产加速器后端），中文项目在每个层级都有头部存在。

### 36.2 选型速查

| 你的需求 | 推荐 |
|---|---|
| 系统入门 | Embodied-AI-Guide → every-embodied 动手 |
| 复现/微调 VLA | dexbotic（求稳）/ starVLA（求全） |
| 要开箱好成绩的 VLA | DM0.5（opendm） |
| 人形全身操作研究 | Psi0 + GMR + Unitree G1 |
| 遥操作采数据 | xr_teleoperate（宇树）/ OpenArm KER |
| 四足/人形真机部署 | rl_sar |
| 大规模 RL 后训练 | RLinf |
| 冻结模型提分 | RPent（Agent 路线）/ Zetta（进化路线） |
| 数据记录与调试 | rerun |
| 标准化评测 | inspect-robots |
| 世界模型平台 | NVIDIA Cosmos |

### 36.3 给不同读者的行动建议

- **学生**：跟 Embodied-AI-Guide 的一周动手路径（RoboTwin 2.0 + ACT）入门，再用 every-embodied 第 6 章复现一个 VLA，你就超过了大多数只读论文的同龄人。
- **研究者**：三篇必读——Psi0（数据配方）、RLinf 主论文（系统视角）、Long-WAM（WAM 路线）；关注"冻结模型 + 系统进化"（RPent/Zetta）这个新战场。
- **工程师/创业者**：数据是真瓶颈——先搭遥操作采集管线（xr_teleoperate/OpenArm + LeRobot），再考虑模型；部署侧关注 FluxVLA 的 Jetson 路径与 Cosmos Edge。
- **创客**：roboto_origin 或 Asimov-1 的 CAD + rl_sar + SO-101 起步，2026 年个人拥有一台会走路的机器人已经可行。

**2027 年值得盯的三个信号**：① BEHAVIOR-1K 全量任务是否被任何模型攻破；② 世界模型端侧（Cosmos Edge / Long-WAM on Thor）能否跑出可用真机策略；③ "Agent 包裹 VLA"（RPent 式）与"端到端 VLA"（DM0.5 式）两条路线的正面交锋结果。

---

# 附录

## 附录 A：项目速查表

| 项目 | 方向 | Stars | 更新 | 一句话定位 |
|---|---|---|---|---|
| TianxingChen/Embodied-AI-Guide | 指南 | 16.7k | 2026-10 | 中文世界最热具身智能技术指南 |
| Octoday-Hub/Embodied-AI | 指南 | 3.0k | 2026-10 | 星期八社区资源导航 |
| datawhalechina/every-embodied | 教程 | 4.3k | 2026-10 | 从 0 构建 VLA 学习库 |
| datawhalechina/dive-into-embodied-ai | 教程 | 648 | 2026-10 | 从零搭建完整机器人课程 |
| jonyzhang2023/awesome-embodied-vla-va-vln | 论文列表 | 3.6k | 2026-09 | 700+ 篇 VLA/VA/VLN 论文雷达 |
| starVLA/starVLA | VLA 框架 | 3.9k | 2026-10 | 乐高式 VLA 研究平台 |
| dexmal/dexbotic | VLA 框架 | 3.2k | 2026-10 | 复现保真的一站式工具箱 |
| physical-superintelligence-lab/Psi0 | VLA 模型 | 2.9k | 2026-10 | 人形全身操作学术标杆 |
| Mininglamp-AI/Mano-P | GUI-VLA | 2.8k | 2026-06 | OSWorld 第一的边缘 GUI 智能体 |
| open-gigaai/giga-brain-0 | VLA 模型 | 2.7k | 2026-09 | 世界模型数据驱动三系统 VLA |
| dexmal/opendm | VLA 模型 | 2.2k | 2026-10 | DM0.5：综合成绩最强开源 VLA |
| Robbyant/lingbot-vla(-v2) | VLA 模型 | 1.9k+1.1k | 2026-09 | 务实路线 6 万小时数据配方 |
| NVlabs/alpamayo | 驾驶 VLA | 2.1k | 2026-09 | NVIDIA 10B 推理 VLA（CoC） |
| FluxVLA/FluxVLA | VLA 框架 | 767 | 2026-10 | 部署侧最深的统一运行时 |
| Genesis-Embodied-AI/genesis-world | 仿真 | ~30k | 2026-10 | 多物理统一 + 跨平台编译 |
| huggingface/lerobot | 生态 | ~28.6k | 2026-10 | 机器人界的 Hugging Face |
| StanfordVL/BEHAVIOR-1K | 基准 | 1.8k | 2026-10 | 1000 项人类需求任务标尺 |
| haosulab/SAPIEN | 仿真 | 862 | 2026-10 | 铰接物体仿真 + ManiSkill 地基 |
| spear-sim/spear | 仿真 | 620 | 2026-10 | 把任意 UE 应用变成仿真器 |
| NVIDIA/cosmos | 世界模型 | ~12.2k | 2026-10 | 世界模型平台（Super/Nano/Edge） |
| NVlabs/LongLive | 世界模型 | 2.8k | 2026-10 | 流式长视频 + Long-WAM |
| enactic/openarm | 硬件 | 3.7k | 2026-10 | $6500 全开源人形双臂 |
| menloresearch/asimov-1 | 硬件 | 1.4k | 2026-10 | $20k 全开源人形整机 |
| Roboparty/roboto_origin | 硬件 | 2.8k | 2026-10 | 淘宝+嘉立创可复现 DIY 人形 |
| unitreerobotics/xr_teleoperate | 遥操作 | 1.7k | 2026-09 | 宇树 XR 遥操作标准工具 |
| YanjieZe/GMR | 重定向 | 2.8k | 2026-10 | 17+ 人形的通用动作重定向 |
| NVIDIA/soma-retargeter | 重定向 | 690 | 2026-10 | 工业化 BVH 重定向管线 |
| NVlabs/ProtoMotions | 运动学习 | 2.4k | 2026-10 | AMASS→真机 G1 零样本迁移 |
| fan-ziqi/rl_sar | 部署 | 1.5k | 2026-08 | RL 仿真验证与实机部署 |
| RLinf/RLinf | RL 基础设施 | ~5.5k | 2026-10 | OSDI 2026 具身 RL 训练infra |
| RLinf/RPent | Agent | 1.6k | 2026-10 | 记忆引导 Agent 包裹 VLA |
| X-Square-Robot/wall-x | 具身模型 | 1.3k | 2026-09 | WALL-OSS/WM 双动作头 |
| air-embodied-brain/Zetta-Embodiment | harness | 1.3k | 2026-10 | 冻结模型的自进化 harness |
| rerun-io/rerun | 数据层 | ~11.7k | 2026-10 | 物理 AI 多模态数据底座 |
| robocurve/inspect-robots | 评测 | 717 | 2026-10 | 物理 AI 可审计评测框架 |

## 附录 B：术语表

- **具身智能（Embodied AI）**：依赖身体与环境物理交互获得智能的 AI 系统。
- **VLA（Vision-Language-Action）**：以视觉-语言模型为骨干、输出机器人动作的多模态模型。
- **动作分块（Action Chunk）**：一次推理预测多步动作序列，降低控制频率要求。
- **FAST 分词器**：Physical Intelligence 提出的动作序列离散化方法，使动作可用自回归方式解码。
- **Flow Matching**：学习概率流速度场的生成方法，VLA 动作头的当前默认选择。
- **双/三系统架构**：System-2（VLM 慢推理）+ System-1（快速动作头）+ System-0（RL 反射控制）的分层控制范式。
- **世界模型（World Model）**：能预测环境未来状态的生成模型。
- **WAM（World Action Model）**：直接产出动作的世界模型，视频生成与机器人控制的交汇点。
- **运动重定向（Motion Retargeting）**：把人体动作数据映射为机器人关节轨迹的技术。
- **遥操作（Teleoperation）**：人通过主从臂/XR 设备远程控制机器人以采集示教数据。
- **Sim2Real / Sim2Sim**：仿真训练的策略向真实世界/另一仿真器的迁移。
- **Loco-manipulation**：移动操作——边走边操作的全身协调任务。
- **Harness**：包裹基础模型的外部系统（规划、记忆、批评、恢复），可在不改权重的情况下提升表现。
- **数据金字塔**：互联网视频 → 人类视频 → 遥操作数据 → 精修微调数据的具身智能数据层级。

---

*本书数据截至 2026 年 10 月 29 日，基于各项目 GitHub 仓库、技术报告与论文的公开信息整理。开源世界日新月异，请以各仓库最新状态为准。*




