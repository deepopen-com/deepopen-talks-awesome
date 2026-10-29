# Awesome  AI video  VLLM and Embodied AI

by DeepOpen

## AI Video

AI video 前沿   [AI video 前沿](https://github.com/deepopen-com/deepopen-talks-awesome/blob/main/ai-video-readme.md)



## VLLM  基础设施

基础设施 [基础设施前沿](https://github.com/deepopen-com/deepopen-talks-awesome/blob/main/infra-readme.md)

## 具身智能

具身智能、世界模型  [具身智能、世界模型](https://github.com/deepopen-com/deepopen-talks-awesome/blob/main/embodied-readme.md)





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


