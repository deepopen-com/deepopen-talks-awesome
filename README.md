# Awesome  AI video  VLLM and Embodied AI

by DeepOpen

## AI Video

AI video 前沿   [AI video 前沿](https://github.com/deepopen-com/deepopen-talks-awesome/blob/main/ai-video-readme.md)



## VLLM  基础设施

基础设施 [基础设施前沿](https://github.com/deepopen-com/deepopen-talks-awesome/blob/main/infra-readme.md)

## 具身智能

具身智能、世界模型  [具身智能、世界模型](https://github.com/deepopen-com/deepopen-talks-awesome/blob/main/embodied-readme.md)


具身智能学习入门 [具身智能学习入门](https://github.com/deepopen-com/deepopen-talks-awesome/blob/main/embodied-learn.md)


# 世界模型的开源项目


我从 GitHub 和 Hugging Face 两个方向并行检索最新的世界模型项目：
GitHub 侧已抓到，再补 Hugging Face 模型库与几个细分方向：
再补 Hugging Face 最新发布维度（按创建时间）与趋势榜，以及 4D/驾驶方向：
快速核实几个头部项目的定位（避免只凭 description 判断）：
我在 GitHub（4 组关键词 + topic 检索）和 Hugging Face（模型检索、按创建时间排最新、趋势榜）都做了检索，筛选 2026 年内仍在更新的项目。先说一个重要提醒：**"world model"这个词现在被两类完全不同的东西共用**，检索时要小心——

- `deeplethe/utopia`（8.6k★，star 最高的"world model"项目）其实是**企业知识世界模型**（知识图谱+本体+时间感知，Rust 写，对标 Palantir 那一类），不是视频/物理世界模型；
- `Qwen/WorldPM-72B` 是"世界模型偏好模型"（用世界模型假设做奖励建模），也不是生成式世界模型。

排除这两类后，真正的世界模型项目按技术路线分如下。

## 一、交互式 / 可玩视频世界模型（最热）

| 项目 | Stars | 更新 | 亮点 |
|---|---|---|---|
| [NVIDIA/cosmos](https://github.com/NVIDIA/cosmos) | 12.2k | 10-28 | NVIDIA 官方开放平台：世界模型基础模型 + 数据集 + 工具链，整个生态的地基 |
| [SkyworkAI/Matrix-Game](https://github.com/SkyworkAI/Matrix-Game) | 2.8k | 10-29 | Matrix-Game 4.0：流式神经渲染，键鼠实时操控可玩游戏世界 |
| [Tencent-Hunyuan/HY-WorldPlay](https://github.com/Tencent-Hunyuan/HY-WorldPlay) | 1.6k | 06-10 | HY-World 1.5：强调**实时延迟**的交互世界建模系统框架 |
| [jd-opensource/JoyAI-Echo](https://github.com/jd-opensource/JoyAI-Echo) | 2.1k | 09-20 | 长时程**音视频联合**生成，面向持久叙事与交互内容 |
| [Tencent-Hunyuan/Hy-Nomas](https://github.com/Tencent-Hunyuan/Hy-Nomas) | 359 | **10-29** | 最新："自由形态交互式世界生成"，今天还在提交 |
| [NoizAI/WorldSonus](https://github.com/NoizAI/WorldSonus) | 237 | 10-21 | 世界模型的**实时空间音频**生成——这个生态位刚出现 |

## 二、生成式 3D / 4D 世界模型

| 项目 | Stars | 更新 | 亮点 |
|---|---|---|---|
| [Tencent-Hunyuan/HY-World-2.0](https://github.com/Tencent-Hunyuan/HY-World-2.0) | 2.8k | 08-12 | 多模态世界模型：3D 世界的**重建+生成+仿真**三合一（HF 上 721 赞，同类最高） |
| [nv-tlabs/lyra](https://github.com/nv-tlabs/lyra) | 2.8k | 07-20 | NVIDIA 空间智能实验室，ICLR'26 + SIGGRAPH Asia'26；Lyra 2.0 支持长时程 3D 一致的可探索生成，开源了 GUI 与训练代码 |
| [Tencent-Hunyuan/HunyuanWorld-Voyager](https://github.com/Tencent-Hunyuan/HunyuanWorld-Voyager) | 1.6k | 04 | 相机条件驱动的交互式 RGBD 视频生成，可实时漫游 |
| [Tencent-Hunyuan/HunyuanWorld-Mirror](https://github.com/Tencent-Hunyuan/HunyuanWorld-Mirror) | 1.2k | 05 | ICML 2026：快速通用 3D 重建 |
| [cvsp-lab/LivingWorld](https://github.com/cvsp-lab/LivingWorld) | 5 | 07 | ECCV 2026：带环境动态的交互式 4D 世界生成（学术新作） |

## 三、具身 World-Action Model（与 VLA 合流，增长最快）

| 项目 | Stars | 更新 | 亮点 |
|---|---|---|---|
| [DAXIAORobotics/kairos](https://github.com/DAXIAORobotics/kairos) | 3.1k | 09-15 | 4B 参数**原生跨本体**世界模型：统一理解/生成/动作预测，带混合线性时序记忆，可边缘实时部署（arXiv 2606.16533） |
| [thu-ml/Motus](https://github.com/thu-ml/Motus) | 1.3k | 10-09 | 清华：统一**潜动作**世界模型（arXiv 2512.13030） |
| [open-gigaai/giga-brain-0](https://github.com/open-gigaai/giga-brain-0) | 2.7k | 09 | 世界模型驱动的 VLA；HF 上配套发布了 Giga-World-1 |
| [NVlabs/LongLive](https://github.com/NVlabs/LongLive) | 2.8k | 10-28 | 长视频生成 / World Action Model 的推理基础设施 |
| 其他 | — | — | 复现类：X-WAM（统一 4D 世界动作建模）、Veri4D-WAM；HF 上 LaLa-World 的 policy 系列 10-28 刚上传 |

## 四、基础设施 · 评测 · 论文雷达（做研究的入口）

| 项目 | Stars | 更新 | 用途 |
|---|---|---|---|
| [knightnemo/Awesome-World-Models](https://github.com/knightnemo/Awesome-World-Models) | 3.5k | 10-26 | 最全的世界模型论文总表（三个 awesome 并存，说明领域尚未收敛） |
| [LMD0311/Awesome-World-Model](https://github.com/LMD0311/Awesome-World-Model) | 2.3k | 10-27 | 偏**自动驾驶**世界模型 |
| [galilai-group/stable-worldmodel](https://github.com/galilai-group/stable-worldmodel) | 2.3k | 10-02 | 可复现研究平台（arXiv 2605.21800）——世界模型界的"stable-baselines" |
| [OpenDCAI/OpenWorldLib](https://github.com/OpenDCAI/OpenWorldLib) | 888 | 10-20 | 统一代码库，把多个世界模型收进一套接口 |
| [EmbodiedCity/iWorld-Bench](https://github.com/EmbodiedCity/iWorld-Bench) | 22 | 07 | 以交互为中心的基准 + 统一动作生成框架 |
| [awslabs/world-models-inference](https://github.com/awslabs/world-models-inference) | 12 | 10-26 | 一条命令在 AWS 部署开源世界模型，每个模型自包含 |
| [danijar/dreamerv3](https://github.com/danijar/dreamerv3) | 3.9k | 10-04 | 经典 latent 世界模型基线（非生成式路线，但仍是必读） |

## 五、Hugging Face 上的权重（下载即用）

| 模型 | 赞/下载 | 更新 | 类型 |
|---|---|---|---|
| `tencent/HY-World-2.0` | 721L | 05-21 | image-to-3d |
| `tencent/HY-WorldPlay` | 526L | 03-06 | image-to-video |
| `robbyant/lingbot-world-base-cam` | 348L | 02-02 | image-to-video（LingBot 系列，**已迁移到 lingbot-world-v2 / Infinity**，老仓库不再维护） |
| `robbyant/lingbot-world-fast` | 62.6k 下载 | 04-02 | 加速版，实际使用最多的一个 |
| `open-gigaai/Giga-World-1` | 142L | 07-30 | robotics |
| `GD-ML/DreamX-World-5B` | 14.6k 下载 | 06-16 | image-to-video |
| `acvlab/ABot-World-0-5B-LF` | 79L | 08-21 | image-to-video |
| `TencentARC/WorldCrafter-Fast` | 24L | **10-26** | 最新发布 |

## 趋势判断

1. **从"生成视频"转向"可交互、可玩、低延迟"**——HY-WorldPlay 把"实时延迟"写进标题、Matrix-Game 强调键鼠操控，评价标准正从 FVD 转向"交互响应延迟 + 状态一致性"。
2. **3D/4D 原生成为主线**——Lyra、HY-World 2.0 不再把世界当 2D 视频流，而是直接生成带几何一致性的可探索空间。
3. **与具身智能彻底合流**——Kairos 和 Motus 这类"World-Action Model"直接输出可执行动作，世界模型从"看起来对"变成"能拿来规划"。
4. **音频/多模态补位**——WorldSonus（空间音频）、JoyAI-Echo（音视频长时程）说明"世界"正在从视觉扩展到完整感知。
5. **评测与部署基础设施开始出现**——stable-worldmodel、iWorld-Bench、OpenWorldLib、awslabs 部署工具，这是一个领域从"论文爆发"走向"工程化"的典型信号。

**想动手的话建议路径**：先看 `stable-worldmodel` 了解评测范式 → 用 `lingbot-world-fast`（下载量最高、权重最易跑）做实验 → 追 `Matrix-Game 4.0` 和 `HY-WorldPlay` 看交互实时性 → 关注 `Kairos`/`Motus` 看具身方向的演进。



# 具身智能的开源项目


我把 GitHub 和 Hugging Face 两个源一起查，覆盖「VLA 模型 / 人形机器人 / 仿真与世界模型 / 数据集与权重」四条线：
具身智能的生态分布有个很清晰的分工：**GitHub 出代码与框架，Hugging Face 出权重与数据**。我两边的数据都重新抓了一遍（2026-10-29 实时），按四条线整理如下。

## 一、GitHub：学习地图（入门先看这里）

| 项目 | Stars | 更新 | 说明 |
|---|---|---|---|
| [TianxingChen/Embodied-AI-Guide](https://github.com/TianxingChen/Embodied-AI-Guide) | 16.7k | 10-25 | Lumina 社区中文指南，最完整的入门地图 |
| [Octoday-Hub/Embodied-AI](https://github.com/Octoday-Hub/Embodied-AI) | 3.0k | 10-26 | 星期八社区：论文/项目/课程/数据集/招聘聚合 |
| [HCPLab-SYSU/Embodied_AI_Paper_List](https://github.com/HCPLab-SYSU/Embodied_AI_Paper_List) | 2.2k | 10-21 | 中山大学：Embodied AI Survey 配套论文库 |
| [zchoi/Awesome-Embodied-Robotics-and-Agent](https://github.com/zchoi/Awesome-Embodied-Robotics-and-Agent) | 1.9k | 10-27 | LLM/VLM 驱动的机器人研究清单 |
| [datawhalechina/dive-into-embodied-ai](https://github.com/datawhalechina/dive-into-embodied-ai) | 650 | 10-28 | 从零搭一台机器人：RL、World-Model、VLA 工程落地 |
| [leofan90/Awesome-World-Models](https://github.com/leofan90/Awesome-World-Models) | 2.0k | 10-26 | 世界模型定义与生成式应用的论文总表 |

## 二、GitHub：VLA 与基础模型

| 项目 | Stars | 更新 | 亮点 |
|---|---|---|---|
| [FluxVLA/FluxVLA](https://github.com/FluxVLA/FluxVLA) | 768 | 10-28 | 从数据到真机部署的一体化 VLA 工程平台 |
| [open-gigaai/giga-brain-0](https://github.com/open-gigaai/giga-brain-0) | 2.7k | 09-09 | 世界模型驱动的 VLA（配套 Giga-World-1 权重） |
| [DAXIAORobotics/kairos](https://github.com/DAXIAORobotics/kairos) | 3.1k | 09-15 | 4B 跨本体 World-Action Model，边缘实时部署 |
| [physical-superintelligence-lab/Psi0](https://github.com/physical-superintelligence-lab/Psi0) | 2.9k | 10 | RSS'26 人形 VLA |
| [dexmal/dexbotic](https://github.com/dexmal/dexbotic) | 3.2k | 10 | 星尘智能 VLA 工具箱；同门 [opendm](https://github.com/dexmal/opendm) 做开放世界基座 |
| [starVLA/starVLA](https://github.com/starVLA/starVLA) | 3.9k | 10 | 乐高式模块化 VLA 框架 |
| [NVlabs/alpamayo](https://github.com/NVlabs/alpamayo) | 2.1k | 09 | NVIDIA 10B 推理 VLA（自动驾驶），HF 上已有 R1/1.5/2-Super 三代权重 |
| [Mininglamp-AI/Mano-P](https://github.com/Mininglamp-AI/Mano-P) | 2.8k | 06 | GUI-VLA 边缘智能体，OSWorld 58.2% |

## 三、GitHub：仿真、数据、训练基础设施

| 项目 | Stars | 更新 | 亮点 |
|---|---|---|---|
| [Genesis-Embodied-AI/genesis-world](https://github.com/Genesis-Embodied-AI/genesis-world) | 30.1k | 10-28 | 生成式物理引擎仿真平台，生态第一 |
| [huggingface/lerobot](https://github.com/huggingface/lerobot) | 28.6k | **10-29** | 端到端机器人 AI 全家桶，GitHub↔HF 的桥梁 |
| [RLinf/RLinf](https://github.com/RLinf/RLinf) | 5.5k | 10-28 | 具身/Agentic RL 基础设施（OSDI 2026） |
| [StanfordVL/BEHAVIOR-1K](https://github.com/StanfordVL/BEHAVIOR-1K) | 1.8k | 10-29 | 1000 项日常任务基准，2026 挑战赛数据已在 HF |
| [NVIDIA/cosmos](https://github.com/NVIDIA/cosmos) | 12.2k | 10-28 | 世界模型开放平台（数据集+模型+工具） |
| [haosulab/SAPIEN](https://github.com/haosulab/SAPIEN) | 862 | 10-09 | 老牌具身仿真（PartNet-Mobility） |
| [spear-sim/spear](https://github.com/spear-sim/spear) | 620 | 10-26 | UE5 照片级真实感仿真 |
| [rerun-io/rerun](https://github.com/rerun-io/rerun) | 11.7k | 10-28 | 多模态机器人数据可视化/流式记录 SDK |

## 四、GitHub：人形与真机硬件

| 项目 | Stars | 更新 | 亮点 |
|---|---|---|---|
| [enactic/openarm](https://github.com/enactic/openarm) | 3.7k | 10 | 全开源人形机械臂（$6500 级） |
| [Roboparty/roboto_origin](https://github.com/Roboparty/roboto_origin) | 2.8k | 10 | 淘宝可复现的 DIY 人形机器人 |
| [YanjieZe/GMR](https://github.com/YanjieZe/GMR) | 2.8k | 10 | ICRA'26 人体动作→任意人形重定向 |
| [NVlabs/ProtoMotions](https://github.com/NVlabs/ProtoMotions) | 2.4k | 10 | NVIDIA 物理仿真运动学习 |
| [pollen-robotics/microduck](https://github.com/pollen-robotics/microduck) | 13.0k | 10-28 | 开源双足小鸭子机器人（入门硬件首选，star 意外地高） |
| [unitreerobotics/xr_teleoperate](https://github.com/unitreerobotics/xr_teleoperate) | 1.7k | 09 | 宇树 XR 遥操作 |
| [menloresearch/asimov-1](https://github.com/menloresearch/asimov-1) | 1.4k | 10 | 开源人形整机 v1 |

## 五、Hugging Face：可直接下载的权重（按赞排序）

浏览入口：`pipeline_tag=robotics` 或 `filter=lerobot`。

| 模型 | 赞 / 下载 | 说明 |
|---|---|---|
| [lerobot/smolvla_base](https://huggingface.co/lerobot/smolvla_base) | 481L / **177k** | 下载量第一，SmolVLA 基座，入门必跑 |
| [nvidia/Alpamayo-R1-10B](https://huggingface.co/nvidia/Alpamayo-R1-10B) | 446L | 自动驾驶推理 VLA |
| [microsoft/Magma-8B](https://huggingface.co/microsoft/Magma-8B) | 425L | 微软多模态 agent 基座（含机器人动作） |
| [nvidia/GR00T-N1-2B](https://huggingface.co/nvidia/GR00T-N1-2B) → [N1.7-3B](https://huggingface.co/nvidia/GR00T-N1.7-3B) | 359L→177L / 66.8k | NVIDIA 人形基础模型主线，迭代到 N1.7 |
| [openvla/openvla-7b](https://huggingface.co/openvla/openvla-7b) | 264L / 84k | 最经典的开源 VLA 基线 |
| [openbmb/MiniCPM-RobotManip](https://huggingface.co/openbmb/MiniCPM-RobotManip) | 191L | 面壁端侧操作模型（另有 RobotTrack） |
| [lerobot/pi05_base](https://huggingface.co/lerobot/pi05_base) | 114L / 19.6k | Physical Intelligence π0.5 开源版 |
| [black-forest-labs/flux-3-action-base](https://huggingface.co/black-forest-labs/flux-3-action-base) | 161L | **BFL 入局动作模型**，另有 so101 / droid 微调版 |
| [BAAI/RoboBrain2.0-7B](https://huggingface.co/BAAI/RoboBrain2.0-7B) | 128L | 智源具身大脑 |
| [unitreerobotics/UnifoLM-VLA-Base](https://huggingface.co/unitreerobotics/UnifoLM-VLA-Base) | 20L | 宇树统一具身模型（另有 WMA 世界动作模型） |
| [tencent/Hy-Embodied-0.5-VLA-UMI](https://huggingface.co/tencent/Hy-Embodied-0.5-VLA-UMI) | 16L | 腾讯混元具身 VLA（UMI / RoboTwin 两版） |

## 六、Hugging Face：数据集（比模型更稀缺的资源）

| 数据集 | 下载 | 更新 | 说明 |
|---|---|---|---|
| [agibot-world/AgiBotWorld2026](https://huggingface.co/datasets/agibot-world/AgiBotWorld2026) | **212.8k** | 10-24 | 智元百万级真机数据，当前下载量第一 |
| [simple-world-lab/HiFi-UMI-2K](https://huggingface.co/datasets/simple-world-lab/HiFi-UMI-2K) | 148k | 07-29 | UMI 手持采集高质量子集 |
| [Sharpa-Robotics/SharpaDex-v1.0](https://huggingface.co/datasets/Sharpa-Robotics/SharpaDex-v1.0) | 130k | 10-27 | 灵巧手数据 |
| [behavior-1k/2026-challenge-demos](https://huggingface.co/datasets/behavior-1k/2026-challenge-demos) | 72k | 08-05 | BEHAVIOR-1K 2026 挑战赛示范 |
| [lerobot/abc_130k_v3_train](https://huggingface.co/datasets/lerobot/abc_130k_v3_train) | 30k | 06-29 | LeRobot 官方大规模训练集 |
| [nvidia/video-to-data-...dexterity](https://huggingface.co/datasets/nvidia/video-to-data-robot-dexterity-task-library-and-dataset) | 23k | 08-21 | 视频转机器人数据的任务库 |
| [OpenGalaxea/Galaxea-Open-World-Dataset](https://huggingface.co/datasets/OpenGalaxea/Galaxea-Open-World-Dataset) | 25k | 04-17 | 开放世界场景数据 |

## 跨源观察

1. **HF 上出现一个新品类：World-Action Model 权重**——`unitreerobotics/UnifoLM-WMA-0-Base`、`open-gigaai/Giga-World-1`、`NU-World-Model-Embodied-AI/phyworld`，说明"世界模型直接出动作"已从论文走到可下载权重。
2. **GR00T 是迭代最快的一条线**：N1 → N1.5 → N1.6 → N1.7，半年四代，且下载量集中在最新版，生态跟随明显。
3. **意外玩家入场**：Black Forest Labs 用 FLUX 的生成先验做 `flux-3-action-*`，并直接给了 SO-101、DROID 两个真机/数据集微调版本——生成模型公司正在切入机器人动作空间。
4. **数据比模型更卷**：AgiBotWorld2026 单数据集 21 万下载，超过任何单个模型；`lerobot` 格式事实上已成为具身数据的"JPEG"。
5. **国内团队在 HF 侧存在感很强**：智元、宇树、腾讯混元、面壁、智源、星尘、蚂蚁灵波（lingbot-vla）都有正式权重发布。

**动手路径建议**：`lerobot/smolvla_base`（跑通推理）→ `lerobot/abc_130k_v3_train` 或 `AgiBotWorld2026`（数据）→ `nvidia/GR00T-N1.7-3B`（人形基座）→ `Genesis` 或 `BEHAVIOR-1K`（仿真评测）。

 

