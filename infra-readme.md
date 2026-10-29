# 大模型基础设施前沿 2026

**——训练/推理基础设施的开源版图、技术原理与选型指南**

> 数据截至 2026 年 10 月 29 日。全部素材来自各项目 GitHub README、官方论文与技术报告的一手调研，star 数为 2026-10 实时抓取。

---

## 前言

如果说大模型是这个时代的"电力"，那么基础设施就是发电厂、电网与变压器。

2023—2024 年，社区的目光集中在模型本身：参数规模、benchmark 分数、下一个惊艳的 demo。而 2026 年，竞争的主战场已经悄然转移——**同样的模型，谁能用更低的成本、更高的吞吐、更稳的工程把它训练出来、服务出去，谁就赢**。这是所有计算平台走向成熟的标志：PC 时代最后的赢家是操作系统和编译器，移动互联网时代最后的赢家是云和容器，而大模型时代，最后的赢家正在此刻被书写——就是本书要讲的基础设施层。

这一年发生了几件标志性事件：

- **TGI 归档**。Hugging Face 官方推理栈 Text Generation Inference 于 2026 年 3 月 21 日归档（只读），官方建议迁移到 vLLM/SGLang。一个时代的落幕，推理引擎"四强格局"正式确立。
- **RL 后训练爆发**。verl（24k★）成为 GRPO 时代的事实标准，slime、AReaL、SkyRL、EasyR1、RAGEN、Miles 各有侧重，训练重心从 SFT 全面转向 RL。
- **KV Cache 基础设施化**。Mooncake、LMCache、Dynamo、OpenLake 把"预填充-解码分离 + KV 复用"做成了数据中心标准架构，KV Cache 从引擎内部资源升格为数据中心级共享资源。
- **端侧推理多元化**。llama.cpp、MNN、web-llm、Cactus、uzu 各占浏览器/手机/穿戴生态位，Unsloth 转型"本地训练+运行 UI"成为消费级入口。

本书基于对 35+ 个活跃开源项目 README 与论文的逐个精读写成，每章遵循统一体例：**定位背景 → 核心概念 → 技术原理 → 典型用法 → 评价与局限**。它既是一本"项目解读合集"，也试图回答一个更大的问题：**2026 年的大模型基础设施，技术版图长什么样，正在往哪里去。**

---

## 导论：四层基础设施栈

在展开具体项目之前，先给出本书的全局分析框架。2026 年的大模型训练/推理基础设施可以划分为四层：

```
┌─────────────────────────────────────────────────────┐
│  第四层：生态与工作流                                  │
│  transformers v5 · peft · trl · accelerate · kernels │
│  optimum · TEI ·（TGI 已归档 ✝）                      │
├─────────────────────────────────────────────────────┤
│  第三层：服务与 KV 基础设施                            │
│  Dynamo · Mooncake · LMCache · production-stack      │
│  OpenLake · Kubeflow Trainer                         │
├─────────────────────────────────────────────────────┤
│  第二层：训练与推理引擎                                │
│  训练：Megatron-LM · torchtitan · verl · slime ...   │
│  推理：vLLM · SGLang · llama.cpp · TensorRT-LLM ...  │
├─────────────────────────────────────────────────────┤
│  第一层：硬件与算子                                    │
│  NVIDIA/AMD/昇腾/摩尔线程 · AMX/NEON 指令集           │
│  量化格式（GGUF/FP8/MXFP4/NVFP4）· 手写内核           │
└─────────────────────────────────────────────────────┘
```

三条贯穿全书的观察线索：

1. **解耦（Disaggregation）**：预填充与解码分离（P/D）、控制器与执行器分离、控制面与执行面分离——凡是热路径上的异质工作负载，2026 年都在被拆开。
2. **复用（Reuse）**：KV Cache 跨请求复用、跨实例复用、跨集群复用；权重跨后端转换（Megatron Bridge）；内核从 Hub 分发（huggingface/kernels）——"不要重复计算"成为最高准则。
3. **降精度（Low Precision）**：从 FP16 到 BF16 到 FP8 再到 MXFP8/NVFP4，量化不再是推理的专利，RL 训练配方也全面低精度化。

带着这三条线索，我们进入正文。

---

# 第一篇 LLM 推理引擎

> 2026 年的推理引擎格局：数据中心看 vLLM（吞吐）与 SGLang（结构化+前缀复用），端侧看 llama.cpp（事实标准）与 MNN/Cactus（垂直生态），异构与消费级看 KTransformers，本地工作台看 Unsloth。新变量是 TokenSpeed 与 uzu。

## 第 1 章 llama.cpp：无处不在的端侧底座

### 1.1 定位与背景

llama.cpp 由 Georgi Gerganov 于 2023 年 3 月创建，最初目标朴素得近乎浪漫：在一台 MacBook 上跑通 LLaMA。三年后，它已演化为 ggml-org 组织维护的端侧/本地推理**事实标准**（132.7k★，MIT 许可），构建在自研的 ggml 张量库之上，核心哲学是"纯 C/C++、零依赖、最小安装代价下覆盖尽可能广的硬件"。

它的生态影响力不在于自身多快，而在于它成为了**别人脚下的地基**：GGUF 模型格式、llama-server 的 OpenAI 兼容 API，是 Ollama、LM Studio、Unsloth Desktop 等上层产品的共同底座。

### 1.2 核心技术原理

- **GGUF 格式**：单文件封装权重、tokenizer、元数据与聊天模板，支持 mmap 零拷贝加载；配套 Python 转换/量化脚本，是量化权重分发的事实标准。
- **量化体系**：覆盖 1.5-bit 到 8-bit 全谱系整数量化（Q2_K~Q8_0、IQ 系列超低位宽、K-quants 分块量化），在 CPU 内存带宽受限场景下以低位宽换吞吐。
- **ggml 后端抽象**：统一计算图 + 可插拔后端，17+ 种——CUDA、HIP、Metal、Vulkan、SYCL、CANN（昇腾）、MUSA（摩尔线程）、OpenCL、OpenVINO、Hexagon DSP、IBM zDNN、WebGPU、RPC 等。Apple Silicon 通过 NEON/Accelerate/Metal 获一等公民待遇；x86 支持 AVX/AVX2/AVX512/AMX；RISC-V 支持 RVV/ZVFH。
- **CPU+GPU 混合推理**：超出总显存的模型可按层部分 offload（`-ngl` 参数），是"消费级硬件跑大模型"的关键机制。

2026 年新进展：工具链收敛为 `llama cli` / `llama serve`；v0.5.0（2026-09）带来 CUDA conv2d implicit GEMM 加速、Metal MoE 算子融合、llama-server 多地址绑定（TCP/UNIX socket）、DFlash 投机解码、Nemotron MTP 与 Gemma 4 DSpark draft 支持。

### 1.3 性能与用法

参考性能：RTX 4090 上 7B Q4_K_M 约 178 tok/s；M3 Max 跑 70B Q4_K_M 约 11.2 tok/s；Ryzen 9 9950X 纯 CPU 7B 约 23 tok/s。

```bash
curl -LsSf https://llama.app/install.sh | sh
llama cli -hf ggml-org/Qwen3.5-0.8B-GGUF      # 直接从 HF 拉取运行
llama serve -hf ggml-org/Qwen3.5-0.8B-GGUF    # OpenAI 兼容 API + 内置 Web UI
```

### 1.4 评价与局限

llama.cpp 的价值在于"无处不在"：任何硬件、任何平台都有可用的构建。局限同样清晰：面向单机低并发设计，服务端吞吐与集群调度能力弱于 vLLM/SGLang；超低位宽（IQ1/IQ2）量化有损；C/C++ 贡献门槛使新架构支持通常滞后于 PyTorch 生态数天。定位：**端侧与个人本地推理的公共底座，而非数据中心吞吐王者**。

## 第 2 章 vLLM：吞吐之王与 Model Runner V2

### 2.1 定位与背景

vLLM 源自 UC Berkeley Sky Computing Lab（SOSP'23 论文），现由 2000+ 贡献者、数十所高校与企业共同维护（95.8k★），是自托管 LLM 推理的事实标准服务引擎，支持 200+ Hugging Face 模型架构（含 MoE、混合注意力/SSM、多模态、嵌入、奖励模型）。

### 2.2 核心技术原理

- **PagedAttention**：借鉴操作系统虚拟内存分页管理 KV Cache，将注意力 KV 切成固定大小 block 按需分配，消除内部/外部碎片，是高并发吞吐的基石。
- **连续批处理 + chunked prefill + prefix caching**：请求在迭代粒度动态进出批次；长 prompt 切块填空；相同前缀 KV 复用。
- **P/D 分离 → E/P/D 分离**：将 Prefill（计算密集）与 Decode（访存密集）拆到独立 GPU 集群独立扩缩容；2026 年 v0.28 进一步演进为 Model Runner V2 的 **E/P/D 分离**（encoder/prefill/decode 三段解耦，opt-in）。
- **Model Runner V2**：重写 worker——每个请求在固定大小 GPU 状态表中占稳定一行，每步用 Triton gather kernel 构建输入张量，实现异步调度与投机解码的零 CPU-GPU 同步，解决了 V1 单文件 6700 行、异步+投机解码需要同步点的问题。
- **分层 KV Cache**：v0.28 引入 GPU→CPU→磁盘（mmap）三级溢出分层，可插拔 tier manager。
- 其他：投机解码（n-gram、EAGLE、DFlash、MTP）、torch.compile、FP8/MXFP4/NVFP4/INT4/AWQ/GPTQ/GGUF 量化、CUTLASS/TRTLLM-GEN 优化的 MoE GEMM 内核、TP/PP/EP/CP 并行、多 LoRA、Rust gRPC 控制面前端。

### 2.3 性能与用法

v0.28（2026-08，584 commits/270 贡献者）新增 Kimi-K3（Decode Context Parallel + 融合 FlashKDA）与 DeepSeek-V4（稀疏 MLA 端到端 + MTP + DSpark）支持。硬件覆盖 NVIDIA（含 Blackwell FP4）、AMD、Intel GPU、x86/ARM CPU，插件式支持 TPU、Gaudi、昇腾、Apple Silicon 等。

```bash
uv pip install vllm
vllm download-kernels   # 预编译内核加速启动
vllm serve Qwen/Qwen3-32B --tensor-parallel-size 2
```

### 2.4 评价与局限

数据中心级吞吐服务首选、生态与硬件兼容性最强。局限：Python 主路径的调度开销（正被 V2/Rust 前端消解）；版本升级破坏性变更频繁（0.28→0.30 移除约十几个 flag）；分层 KV 的磁盘层延迟代价显著，不适合延迟敏感在线业务。

## 第 3 章 SGLang：RadixAttention 与 Agent 时代的推理引擎

### 3.1 定位与背景

SGLang 由 LMSYS 孵化（38.3k★），从"结构化生成前端 DSL + RadixAttention 运行时"起家，现明确面向**Agentic 工作负载、大规模服务和 RL rollout** 三大场景——verl、slime、AReaL 等主流 RL 训练框架均以其为 rollout 引擎。生产采用广泛，全球支撑超 40 万 GPU。

### 3.2 核心技术原理

- **RadixAttention**：以基数树（压缩前缀树）跨请求管理 KV Cache——树的边是 token 序列，节点指向 GPU 物理页；新请求到达时做最长前缀匹配，命中部分零计算复用；LRU 淘汰 + 缓存感知调度。多轮对话/Agent 场景命中率 70-95%，TTFT 可从秒级降到几十毫秒，原论文基准较基线提速 6.4 倍。
- **HiCache 分层缓存**：GPU HBM→主机 DRAM→外部存储（mooncake/hf3fs/nixl 等）三级扩展，支持零拷贝布局、异构 TP 共享、预取策略与运行时热插拔存储后端，可与 P/D 分离组合。
- **结构化约束解码**：正则→FSM、JSON schema、EBNF 约束在解码循环内逐 token 施加 logit mask，保证输出可解析。
- 零开销 CPU 调度器、投机解码（配套 SpecForge 训练 draft 模型）、内置 SGLang Diffusion 图像/视频生成引擎。

### 3.3 性能与用法

典型基准较 vLLM 有 10-30% 吞吐优势（存在前缀重叠的对话类负载）；多轮 Agent 场景实测吞吐提升至 3.5 倍、TTFT 降 80%。硬件矩阵极宽：NVIDIA 全系（B200/B300/GB200/GB300、DGX Spark、Jetson）、AMD MI300X~MI355X、Google TPU、Intel、Apple Silicon、华为昇腾、摩尔线程 S5000。

```bash
docker pull lmsysorg/sglang:latest
python -m sglang.launch_server --model Qwen/Qwen3-32B --tp 4
# HiCache：--enable-hierarchical-cache --hicache-ratio 2 --hicache-io-backend kernel
```

### 3.4 评价与局限

Agent/多轮/RAG 等前缀共享密集型场景的吞吐最优解、RL rollout 基础设施事实标准。局限：单轮无前缀复用场景优势归零；HiCache 跨实例复用仅限 L3 存储后端、配置面复杂。与 vLLM 的竞争正演化为"各有所长+相互借鉴"。

## 第 4 章 KTransformers：CPU+GPU 异构跑超大 MoE

### 4.1 定位与背景

KTransformers 由清华大学 MADSys 实验室、Approaching.AI 与 9#AISoft 联合维护（19.6k★），2025 年 2 月以"24GB 显存 + 382GB 内存跑 DeepSeek-R1/V3，3~28 倍加速"出圈，SOSP'25 发表推理论文。2026 年项目重组：原框架归档至 `archive/`，聚焦 **kt-kernel** 的两大能力——异构推理与 MoE 微调（KT-FT，预印本 2026-10 发布），并深度集成进 SGLang。

### 4.2 核心技术原理

- **异构算子分发**：MoE 模型的注意力等稠密算子放 GPU，路由专家（参数量占 95%+ 但激活稀疏）放 CPU；CPU 侧用 Intel AMX/AVX512 手写内核执行 INT4/INT8 反量化 GEMM。
- **专家调度与 NUMA 感知**：热专家驻留 GPU、冷专家在 CPU；三层（GPU-CPU-Disk）前缀缓存复用。
- **KT-FT 微调**：layout-aware + schedule-optimized 的 CPU-GPU 协同 MoE 微调，较 ZeRO-Offload 提速 6-12 倍、CPU 内存减半；支持 LoRA 与全参。
- **Day0 支持**是显著特色：2026 年内 GLM-5 系列、Kimi-K2.5/K2.6、MiniMax-M2.5/M3、DeepSeek-V4-Flash 均发布当日或次日支持。

### 4.3 性能与用法

DeepSeek-R1-0528 (FP8) 在 8×L20 + Xeon 上总吞吐 227.85 tok/s；DeepSeek-V3 LoRA 微调 4×RTX 4090 达 3.7 it/s；Qwen3-30B-A3B 单卡 4090 达 8+ it/s。

```bash
cd kt-kernel && pip install .                      # 推理内核
pip install "ktransformers[sft]==0.7.0" "sglang-kt==0.7.0"   # 微调（LLaMA-Factory 集成）
```

### 4.4 评价与局限

"消费级硬件跑超大 MoE + 本地微调超大 MoE"细分市场的独特存在。局限：主要面向 MoE 架构；极致性能依赖 AMX 指令集；并发服务能力弱于纯 GPU 集群。路线图显示其长期定位是 SGLang 的异构内核插件。

## 第 5 章 Unsloth：从微调加速库到本地 AI 工作台

### 5.1 定位与背景

Unsloth 由 Daniel Han 与 Michael Han 兄弟创建（78.7k★），以"2 倍速、省 70% 显存、零精度损失的微调"闻名。2026 年完成形态跃迁：从 pip 库进化为**三形态产品**——Unsloth Desktop（桌面 App）、Unsloth Studio（Web UI）、Unsloth Core（代码库），自称"第一个既能跑模型又能训模型的桌面应用"，与 Ollama/LM Studio 的"只推理不训练"形成差异。

### 5.2 核心技术原理

- **手写 Triton/CUDA 训练内核**：对 RoPE、RMSNorm、交叉熵等算子做融合重写，配合梯度检查点优化，实现 2× 训练加速与 70% 显存节省。
- **推理侧基于 llama.cpp**：桌面端运行模型走 llama.cpp 后端，支持 GGUF/MLX/扩散/嵌入/音频模型。
- **Agent 生态接入**是 2026 年重点：`unsloth start` 一条命令将本地模型接入 Claude Code、Codex、VSCode 等 10 种编码 Agent 与 MCP 工具链；提供私有无限 web search、自动上下文压缩、RAG。

### 5.3 用法与评价

```bash
curl -fsSL https://unsloth.ai/install.sh | sh
unsloth studio                       # 启动 Web UI
unsloth start claude --model unsloth/Qwen3.8-27B-GGUF:UD-Q4_K_XL
```

评价："训练+推理+Agent 托管"三合一的本地 AI 工作站形态在开源界独树一帜。局限：核心 Apache 2.0 与 Studio UI AGPL-3.0 双许可，商用需注意；长尾模型回退 HF 路径；深度服务化非其定位。

## 第 6 章 端侧三杰：MNN、web-llm、Cactus

### 6.1 alibaba/MNN（16.3k★）

阿里淘系技术团队出品的轻量级深度学习框架，2016 年起在淘宝/天猫/钉钉等 30+ App 落地，MLSys'20 发表核心论文。**MNN-LLM** 是端侧大模型运行时，另有 MNN-Diffusion、MNN Chat 多模态 App、TaoAvatar 3D 数字人。技术特点：极致轻量（Android 核心 so 约 800KB）；深度手写汇编优化（ARM v8.2 FP16 提速 2 倍、VNNI 提速 2.5 倍）；后端覆盖 CPU/OpenCL/Vulkan/Metal/CUDA 及 CoreML/NNAPI/QNN NPU；2026-07 发布 3.6.1 新增 Qualcomm Hexagon DSP 后端。定位：国内端侧工业级部署首选之一。

### 6.2 mlc-ai/web-llm（19.3k★）

CMU Catalyst/UW 团队（陈天奇组）出品，借助 **WebGPU + TVMjs（WASM）**&#x5B9E;现完全在浏览器内、无服务器的 LLM 推理。核心技术：模型编译为 WASM 库 + 权重由 Cache API/IndexedDB/OPFS 缓存；提供**完全兼容 OpenAI API** 的 TypeScript 接口（流式、JSON mode）；支持 Web Worker 卸载计算；2026 年新增 SRI 完整性校验（对 config/WASM/tokenizer 做 sha256 哈希验证，防供应链篡改）。局限：WebGPU 仅覆盖 Chromium 系；8B+ 模型首次加载耗时与内存压力大。定位：隐私敏感、零服务端成本浏览器 AI 的标杆。

### 6.3 cactus-compute/cactus（6.2k★）

面向手机、可穿戴与智能家居的**端云混合 AI 引擎**，覆盖从量化、内核、计算图到运行时的完整自研栈：自研 CQ 量化（基于旋转+码本，4-bit 至 1-bit，CQ4 精度接近 FP16——Gemma-4-E2B 上 MMLU 仅降 3 分）、ARM NEON 手写算子、零拷贝计算图、OpenAI 兼容引擎。独有特性：**置信度驱动的云端接力**——本地模型置信度低于阈值自动 handoff 到云端；内置 26M 参数工具调用小模型 Needle；绑定覆盖 Swift/Kotlin/Flutter/React Native/Python/Rust。性能：iPhone 17 Pro 上 Gemma-4-E2B-CQ4 prefill 729 tok/s、1k 上下文峰值内存 644MB。定位：端侧垂直整合度最高的新锐方案。

## 第 7 章 新变量：TokenSpeed 与 uzu

### 7.1 lightseekorg/tokenspeed（2.8k★）

2026 年 5 月由非营利 **LightSeek Foundation** 发布，自称"TensorRT-LLM 级性能 + vLLM 级易用性"，专注生产级 Agentic 工作负载，8 月被纳入 PyTorch 官方 Ecosystem Landscape。三大架构差异点：① **控制面/执行面分离**——控制面用 C++ 有限状态机实现，请求生命周期与 KV Cache 所有权由类型系统在**编译期**保证安全；执行面保留 Python 便于快速迭代；② **内核一级公民化**——可插拔分层内核系统，Blackwell 上的 MLA 实现号称业界最快；③ **Rust 入口驱动**，请求路径上无 Python。性能：Qwen3.5-397B-A17B agentic 负载 580 tok/s（PyTorch 官博）；GB300 上单用户 685 tok/s。局限：项目极新，生产案例尚少；强绑定最新代际数据中心硬件。定位：值得重点跟踪的"中立基金会 + 顶级厂商赞助"新变量。


### 7.2 trymirai/uzu（3.0k★）

Mirai 公司出品的 MIT 许可 Rust 推理引擎，主打"在 App 内直接部署 AI：零延迟、完全隐私"。Rust 原生内核经 uniffi-rs/pyo3/napi-rs 提供 Swift/Python/TypeScript 三语言绑定，深度优化 Apple 统一内存架构；自带模型格式（配套 lalamo 转换工具）、模型仓库标识符、工具调用、结构化输出、OpenAI 兼容 HTTP 服务器。局限：硬件生态封闭于 Apple 优先；模型格式私有（非 GGUF）。定位：**Apple 平台应用内嵌推理的商业级 SDK**。

### 7.3 第一篇小结：推理引擎选型速查

| 项目 | 语言/栈 | 主战场 | 差异化核心 |
|---|---|---|---|
| llama.cpp | C/C++ | 端侧/本地 | GGUF 生态、17+ 后端、量化谱系最全 |
| vLLM | Python(+Rust) | 数据中心服务 | PagedAttention、E/P/D 分离、吞吐与生态第一 |
| SGLang | Python | Agentic/RL rollout | RadixAttention、HiCache、结构化解码 |
| KTransformers | C++/Python | 超大 MoE 消费级 | CPU-GPU 异构专家分发、AMX、MoE 本地微调 |
| Unsloth | Python/llama.cpp | 本地训练+推理 | 微调内核加速、Desktop 工作台、Agent 接入 |
| MNN | C++ | 手机/IoT（国内） | 极致轻量、Hexagon DSP、阿里生产验证 |
| Web-LLM | TS/WASM | 浏览器 | WebGPU+TVMjs、无服务器 |
| uzu | Rust | Apple 应用内嵌 | 统一内存优化、商业 SDK |
| TokenSpeed | C++/Rust | 数据中心 Agentic | 编译期 KV 安全、MLA 极速 |
| Cactus | C/C++ | 手机/穿戴 | 自研 CQ 量化、云接力 |

---

# 第二篇 RL 后训练框架

> 2026 年最热的基础设施赛道。训练重心从 SFT 全面转向 RL，八个框架各自卡位：verl 是生态核心，slime→Miles 构成 SGLang 系生产谱系，AReaL/SkyRL 领跑异步化，RAGEN 代表算法诊断研究，Oumi 是上层工作流平台，EasyR1 卡位多模态垂类。

## 第 8 章 verl：GRPO 时代的事实标准

### 8.1 定位与背景

verl（Volcano Engine RL for LLMs）由字节跳动 Seed 团队发起、社区共同维护（24k★），其开源版本即 EuroSys 2025 论文 **HybridFlow: A Flexible and Efficient RLHF Framework**（arXiv:2409.19256）的实现。贡献方阵容极广：Anyscale、LMSys、阿里 Qwen、清华、UC Berkeley、Moonshot、NVIDIA Research 等数十家机构。Doubao-1.5-pro 的 RL Scaling 模型、Seed-Thinking-v1.5（AIME 2024 86.7 分）均由 verl 训练；字节 DAPO 算法（AIME 2024 上 50 分）也在 verl 上复现。

### 8.2 核心技术原理

核心是 **HybridFlow 混合控制器编程模型**——用 Ray 编排 RL 数据流（单控制器负责算法逻辑），把计算与数据依赖解耦，模型执行交给各引擎（多控制器）。关键设计：

- 训练后端支持 **FSDP / FSDP2 / Megatron-LM**，rollout 支持 **vLLM / SGLang / HF Transformers**。
- 标志性创新 **3D-HybridEngine**：在训练与生成阶段切换时做 actor 模型 resharding，消除显存冗余、降低通信开销。
- 灵活的设备映射可将不同模型（actor/critic/reward/reference）放置到不同 GPU 组。
- 社区版本在 experimental 目录推进 fully_async_policy（全异步）、one_step_off_policy、transfer_queue 等方向；支持专家并行扩展到 671B 模型、数百 GPU。

**算法覆盖最全**：PPO、GRPO、GSPO、DAPO、Dr.GRPO、ReMax、REINFORCE++、RLOO、PRIME、PF-PPO（ICML 2025）、VAPO 等；含可验证奖励与模型奖励、VLM 多模态 RL、多轮工具调用 agent loop、LoRA RL。硬件支持 NVIDIA、AMD ROCm、华为昇腾。

### 8.3 用法与评价

```bash
pip install verl
# 配置切换后端：actor_rollout_ref.actor.strategy=fsdp2
# 示例：examples/ppo_trainer、examples/grpo_trainer
# 算法配方已迁移至独立仓库 verl-recipe（dapo、gspo、prime 等）
```

评价：生态最庞大——TinyZero、Search-R1、PRIME、rStar2-Agent、verl-agent、EasyR1、SkyRL 等数十个项目基于它构建，扩展子项目覆盖 VLA、扩散模型（VeRL-Omni）、Tinker API。适合作为通用基座与算法研究平台。局限：代码量大、抽象层次多，上手门槛高于精简派；同步范式为主，异步能力尚在 experimental。

## 第 9 章 slime：GLM 系的生产验证

### 9.1 定位与背景

slime 是智谱（GLM/THUDM）开源的 **RL Scaling 训练框架**（8.8k★），无独立系统论文，但实战背书极强——GLM-4.5/4.6/4.7、GLM-5 全系的 RL 后训练均由 slime 完成，自称"经过最完整生产验证的开源 RL 框架"。SGLang 团队深度参与（官方博客 "An SGLang-Native Post-Training Framework for RL Scaling"）。

### 9.2 核心技术原理

架构极简而克制，三大模块：**training（Megatron）** 负责主训练；**rollout（SGLang + router）** 生成数据写入 Data Buffer；**data buffer** 作为桥梁管理 prompt 初始化、自定义数据与生成方法。设计哲学是"训练/rollout/数据走同一条通路"，不堆砌互相割裂的 trainer/rollout 服务/agent 框架。

- 参数传递默认走 Ray object-store，可选 **straw**（基于共享 JuiceFS 存储的持久化载荷传输，Ray 只传控制消息）。
- 高级部署支持 P/D 分离、delta 权重同步（只传变化字节）、外部 rollout 引擎（跨 GPU 集群磁盘更新权重）。
- Agentic 能力通过 `--custom-generate-function-path` 自定义生成函数实现多轮/工具/多智能体；支持 fully-async rollout 应对长尾样本。

### 9.3 评价与局限

生态繁荣——Miles、vime（vLLM 官方基于 slime 换 rollout 后端）、Dressage（阿里 Accio）、Relax（红杉/RedAI 全模态）、AWS AgentCore RL Toolkit 等均构建其上，已形成 "RL substrate" 地位。局限：算法种类不如 verl 丰富（官方食谱偏 GRPO 系）、训练后端仅 Megatron、对 AMD/昇腾支持不及 verl/AReaL。

## 第 10 章 AReaL：全异步 RL 的代表

### 10.1 定位与背景

AReaL 由清华 IIIS 与蚂蚁集团联合开发（5.9k★），定位"大规模**全异步** RL 系统"。论文：**AReaL**（arXiv:2505.24298），前身 ReaLHF 发表于 MLSys 2025；AReaL 2.0 微服务架构另有技术报告 arXiv:2607.01120。

### 10.2 核心技术原理

核心卖点是**全异步 RL 训练范式**——生成与训练解耦，通过可控的 off-policyness（`max_head_offpolicyness`）在保证算法正确性的前提下消除同步等待；同一算法设 `max_head_offpolicyness=0` 即退回同步模式。boba² 版本实测全异步带来 **2.77× 加速**且训练效果相当或更优。

2026 年 7 月的 **AReaL 2.0** 重构为**微服务架构**：独立服务（training / inference / agent / weight-update），配套 Hermes 在线 RL 回路与端到端 SWE RL 训练示例。对黑盒 agent 应用只需替换 `base_url`/`api_key` 即可接入 RL 服务。训练后端 Megatron（DP/TP/SP/CP/PP/EP 全支持）、FSDP2 与新的 **Archon** 后端；推理后端 SGLang（默认）与 vLLM。算法创新活跃：KPop（双向二值 KL token 掩码）、IcePop、SAPO、M2PO 等。硬件支持 NVIDIA 与**华为昇腾**。

### 10.3 评价与局限

异步 RL + 微服务方向最有代表性的系统，agentic/在线 RL 场景（搜索、客服 τ²-bench、SWE、终端 agent）案例齐全；AReaL-SEA 数据合成引擎在 235B MoE 上超过 GPT-5（τ²-bench）。局限：微服务架构部署复杂度高于同步框架；社区规模与 verl 有差距；Archon 新后端生态尚浅。

## 第 11 章 EasyR1 与 SkyRL：两条新锐路线

### 11.1 EasyR1：多模态 RL 的 Quick Start（5.2k★）

LLaMA-Factory 作者郑耀威（hiyouga）团队出品，是 verl 的**精简 fork**，专注**多模态（VLM）RL 训练**。继承 verl 的 HybridEngine 与 Ray 编排，结合 vLLM 最新 **SPMD 模式**实现高效可扩展 rollout。相比 verl 做了大量减法：去掉 Megatron 后端，主打 FSDP + vLLM 组合，配置扁平。支持 padding-free 训练、LoRA、断点续训。提供完整硬件需求估算表：Qwen2.5-VL-7B GRPO 全参 BF16 需 4×40GB，72B 需 16×80GB，LoRA 显著降低门槛。

```bash
pip install -e .
bash examples/qwen2_5_vl_7b_geo3k_grpo.sh   # 三步跑通 Qwen2.5-VL-7B GRPO
```

评价：R1-V、Vision-R1、Seg-Zero、GUI-R1、NoisyRollout、Long-RL（NVIDIA）等大量多模态 RL 论文用其复现，是垂类默认基线。局限：明确不做 SFT 与推理；VLM 暂不兼容 Ulysses 序列并行；功能始终滞后 verl 一个身位。

### 11.2 SkyRL：Berkeley 的模块化全栈（2.5k★）

UC Berkeley Sky Computing Lab（Ion Stoica、Joseph Gonzalez 团队）出品，与 Anyscale 深度合作。全家桶分四件套：**skyrl-train**（模块化训练框架）、**skyrl-tx**（跨平台库，实现 **Tinker API 的开源后端**）、**skyrl-agent**（长程真实环境 agent 层）、**skyrl-gym**（Gymnasium API 的工具使用环境库）。前沿特性激进：**全异步 RL + 飞行中权重更新（in-flight weight updates）**、vLLM 原生 RL API（与 vLLM 官方合作）、Multi-LoRA 持续学习、on-policy distillation；2026/10 对 Reflection AI 的 501B-A23B MoE 提供 Day-0 支持。代表作：SkyRL-SQL（653 样本训练的 7B 模型在 Text2SQL 上比肩 GPT-4o）。局限：星数相对低、组件拆分带来认知成本。

## 第 12 章 RAGEN：Agent RL 的算法显微镜

### 12.1 定位与背景

RAGEN（2.8k★）源自斯坦福（李飞飞、吴佳俊团队参与）与西北大学 MLL Lab 的学术项目，专注**多轮 Agent RL 的算法研究与方法论诊断**。两篇论文：**RAGEN**（arXiv:2504.20073）与 **RAGEN-2: Reasoning Collapse in Agentic RL**（arXiv:2604.06268），提出 StarPO 算法框架。

### 12.2 核心技术原理

**StarPO（State-Thinking-Actions-Reward Policy Optimization）** 将 agent-环境交互建模为 MDP（状态与动作均为 token 序列），分 rollout/update 两阶段：rollout 阶段模型生成 `<think>...</think><ans>action</ans>` 引导的交互轨迹；update 阶段以重要性采样整体优化轨迹，兼容 PPO（token 级 GAE 优势）与 GRPO（轨迹级归一化奖励）。基础设施基于 veRL 子模块，环境采用 Gym 兼容接口（内置 Sokoban、WebShop、DeepCoder、Lean 等 10 个环境）。

RAGEN-2 的理论贡献是诊断**模板坍缩（Echo Trap）**：仅靠熵无法检测"单输入内多样、跨输入雷同"的坍缩，需分解为条件熵 H(Z|X)（输入内多样性）与互信息 I(X;Z)（跨输入可区分性）两个轴；并提出 **SNR-Adaptive Filtering**——用奖励方差作轻量代理挑选高信号 prompt，缓解噪声梯度，在多算法、多尺度、多模态上稳定涨点。

### 12.3 评价与局限

贡献在于把 agent RL 的失败模式变成可度量、可干预的对象，深刻影响了 ROLL、Search-R1 等后续工作。局限：工程投入有限，环境偏 toy/game 型，直接用于大规模生产训练不现实。

## 第 13 章 Miles 与 Oumi：生产工程派与平台派

### 13.1 Miles：万亿参数级的生产工程（3.2k★）

RadixArk（SGLang 核心团队商业化公司）推出，直接 fork 自 slime，技术报告 **Miles v0.1: Production-Level Post-Training**（arXiv:2609.08368）。技术栈为 **SGLang（rollout）+ Megatron-LM（训练）**，强化了 slime 的企业级短板：

- **TITO（token-in-token-out）**：rollout 与训练间不做 detokenize/retokenize，保证 token 级一致性。
- **R3（Rollout Routing Replay）**：rollout 时记录 MoE 专家路由并在训练前向中重放，消除路由不一致导致的大规模训练不稳。
- **低精度训练**：MXFP8 / NVFP4 端到端 RL 配方（Blackwell 原生）。
- **快速权重更新**：万亿参数模型（Kimi-K2.6）权重在环内秒级送达，disaggregated 部署走 P2P RDMA。
- **容错**：SGLang 引擎宕机原地恢复续训；同一脚本可跑 Ray 或 Kubernetes。

硬件覆盖 NVIDIA GB300→A100 与 AMD MI355X 全系。评价：TITO+R3+低精度+容错都是 trillion 级生产训练的一线痛点解法，AMD 生态最全。局限：起步晚、受 RadixArk 商业节奏影响。

### 13.2 Oumi：全生命周期平台（9.4k★）

美国初创公司 oumi.ai 推出的开源平台，覆盖数据准备→训练→评测→部署，统一 API。训练侧封装 FSDP/DeepSpeed/DDP 与 LoRA/QLoRA；推理侧封装 vLLM/SGLang；RL 能力双轨：TRL 的 GRPO/DPO/KTO + 集成 **verl** 实现"任意 Oumi 环境上的 GRPO"。特色是 agentic 训练闭环（tool-calling SFT、可执行工具环境）与 `oumi launch` 统一调度 AWS/Azure/GCP/Lambda/Slurm，另有 `oumi-mcp` 可从 Claude/Cursor 驱动。局限：RL 吞吐依赖底层引擎，自身无 HybridEngine 级优化；官方标注仍处 beta。

### 13.3 第二篇小结：八框架对比

| 维度 | verl | slime | AReaL | EasyR1 | RAGEN | SkyRL | Miles | Oumi |
|---|---|---|---|---|---|---|---|---|
| 出身 | 字节 Seed | 智谱 GLM | 清华+蚂蚁 | hiyouga | 斯坦福+西北 | Berkeley | RadixArk | oumi.ai |
| 论文 | HybridFlow EuroSys'25 | 实战背书 | 2505.24298 | 引用 HybridFlow | StarPO | 2511.16108 | 2609.08368 | 无 |
| 训练后端 | FSDP/Megatron | Megatron | Megatron/FSDP2/Archon | FSDP | veRL 子模块 | veRL 系/tx | Megatron/FSDP2/torchtitan | FSDP/DeepSpeed 封装 |
| Rollout | vLLM/SGLang | SGLang | SGLang/vLLM | vLLM SPMD | veRL 内 | vLLM 原生 RL | SGLang | vLLM/SGLang |
| 同步范式 | 同步为主 | 异步可选 | **全异步原生 2.77×** | 同步 | 同步 | 全异步+飞行中权重更新 | 全异步+RDMA 秒级 | 平台层 |
| 模态 | 文本+VLA/Omni 扩展 | 文本 | 文本+VLM | **VLM 主打** | Agent/游戏 | Agent 工具 | 文本+VLM+扩散 | 文本+VLM |
| 硬件 | NV/AMD/昇腾 | NV | NV/昇腾 | NV | NV | NV/AMD | NV/**AMD 全系** | 随策略 |
| 最佳适用 | 通用基座/超大 MoE | GLM 系生产 | 异步 agentic RL | 多模态快速实验 | 算法诊断 | Tinker/长程 agent | 万亿级/低精度/AMD | 全流程不想拼装 |

共性趋势：训练/rollout 解耦（Megatron/FSDP + SGLang/vLLM 组合）、全异步化、MoE 路由一致性（R3/router replay）、token 级一致性（TITO）、低精度 RL（MXFP8/NVFP4）与 agentic 环境标准化（Gymnasium/OpenEnv/Tinker API）。

---

# 第三篇 分布式训练与数据中心基础设施

> 训练侧的主线是 Megatron（极致性能）与 torchtitan/Automodel（PyTorch 原生 DTensor 路线）的路线之争；服务侧的主线是 KV Cache 从"引擎内部资源"升格为"数据中心级共享资源"——Dynamo（路由+KVBM）、Mooncake（Store/TE）、LMCache、OpenLake 构成"KV 池 + 路由 + 分层存储"的新中间层。

## 第 14 章 Megatron-LM：大规模训练的研究基座

### 14.1 定位与背景

Megatron-LM（18.3k★）是 NVIDIA 的大规模训练旗舰，仓库包含两个组件：**Megatron-LM** 是参考实现（Megatron Core + 预配置训练脚本），适合研究团队与快速实验；**Megatron Core** 是可组合库，提供 GPU 优化的 transformer 构建块与高级并行策略，适合框架开发者构建自定义训练管线。几乎所有主流 RL 框架（verl、slime、AReaL、Miles）的"性能后端"选项都是它。

### 14.2 核心技术原理

- **五维并行**：TP（张量）、PP（流水线）、DP（数据）、EP（专家）、CP（上下文）可自由组合成 N-D 并行布局；混合精度支持 FP16/BF16/FP8/**FP4**。
- **Megatron Bridge**：提供 HF ↔ Megatron checkpoint 双向转换与生产级配方，是连接开源模型生态与 NVIDIA 私有栈的桥梁。
- 2026 年进展：Core 0.19（2026-08）引入 HybridModel、分位数 MoE router 均衡、packed-sequence MoE dispatch、CUDA Graph 兼容激活；2026/10 官方博客发布**位级确定性预训练**指南——大规模训练下复现训练、验证 checkpoint 恢复、降低确定性开销。

### 14.3 用法与评价

```bash
uv pip install megatron-core   # Python 3.12+ / PyTorch 2.6.0+
```

评价：千卡级训练的事实基座，性能上限最高。局限：学习曲线陡峭、与 NVIDIA 硬件深度绑定、代码改动成本高——这正是 torchtitan 等 PyTorch 原生路线的切入点。

## 第 15 章 torchtitan：PyTorch 官方的原生路线

### 15.1 定位与背景

torchtitan（5.9k★，ICLR 2025 论文）是 PyTorch 官方的 **clean-room、PyTorch 原生**大规模训练实现。核心主张：一份可读的代码库，让模型从预训练走到监督微调再到强化学习，用 FSDP + 张量/流水线/上下文/专家并行组合，**无需重写模型**。它是 PyTorch 官方路线的代表——用 DTensor/DeviceMesh 把并行能力下沉到 PyTorch 本体。

### 15.2 核心技术原理

- **可组合并行**：模型声明自己的分片方式，FSDP2、TP、PP、CP、EP 通过 `DeviceMesh` 和 DTensor 堆叠成 N-D 布局。
- **TitanRL（2026/08）**：可 hack 的 RL 栈，在训练器与 vLLM 生成之间复用相同的模型定义与内核，支持**位级可复现的 batch-invariant 模式**。
- 前沿技术：DeepSeek、Qwen、Kimi、gpt-oss、Llama、FLUX 模型支持；MXFP8/NVFP4 低精度、Dist-MoE、CUDA Graphs。
- 生态：AMD 发布了优化 fork（torchtitan-amd）；SkyPilot 提供官方教程。

### 15.3 评价与局限

评价：可读性与可改性远胜 Megatron，是研究复现与中等规模训练的理想选择，代表了"并行策略下沉到框架本体"的产业方向。局限：仍在快速开发（建议 nightly），单点性能极限仍逊于 Megatron 调优后的极致配置。


## 第 16 章 Automodel 与 Kubeflow Trainer：训练的工程配套

### 16.1 NVIDIA-NeMo/Automodel（1.1k★）

NVIDIA 的 **DTensor 原生 SPMD 库**，主打"Day-0 集成 Hugging Face 模型"：直接拿 HF 模型代码做微调/预训练，自动化并行（FSDP2/TP/CP/PP），内置自定义内核与优化配方。2026/10 示例：Nanbeige4.3-33B 在 128K packed sequences 上用 context parallelism 微调。定位：Megatron 与 HF 生态之间的"易用桥"。

### 16.2 kubeflow/trainer（2.3k★）

Kubernetes 上的分布式训练标准。2026/08 发布 v2.3.0：runtime snapshot 机制（解耦 runtime 生命周期）、增强 MPI 支持、Megatron-LM 示例；v2.2 支持 JAX/XGBoost runtime、指标传播到 TrainJob 状态；v2.1 提供 Distributed Data Cache、Kueue/Volcano 拓扑感知调度。配套 Kubeflow SDK（CustomTrainer/BuiltinTrainer/本地 PyTorch 执行）。定位：企业 K8s 集群上跑训练/微调的通用入口。

### 16.3 stas00/ml-engineering（19.6k★）

"机器学习工程开源书"——作者在训练 BLOOM-176B（2022）、IDEFICS-80B（2023）中积累的实战经验，覆盖三部分：Insights（选云厂商、GPU 升级决策框架，含真实 H200→B200 基准测算）、Hardware（计算/存储/网络）、Orchestration。特色是大量可复制粘贴的脚本与命令。它不是软件，而是万亿级训练的"战场手册"，与其他章节的系统项目互为表里。

## 第 17 章 NVIDIA Dynamo：数据中心推理的调度中枢

### 17.1 定位与背景

Dynamo（8.4k★，160+ 社区贡献者）是 NVIDIA 的**数据中心级分布式推理服务框架**（Rust 编写），定位是"推理服务的 Kubernetes"——不替代 vLLM/TensorRT-LLM 等引擎，而是把它们编排成集群。

### 17.2 核心技术原理

- **KVBM（KV Cache Block Manager）**：跨引擎的 KV 缓存管理层，支持分层 offload。
- **KV-aware 路由**：调度器感知各 worker 的前缀缓存内容，把请求路由到命中率最高的实例——与 SGLang 的缓存感知调度同源，但做在集群层。
- **P/D 分离编排**：原生支持预填充/解码 worker 分离部署与弹性扩缩容，官方 recipes 覆盖主流模型。
- 支持 vLLM、SGLang、TensorRT-LLM 多引擎后端，gRPC/NATS 控制面。

### 17.3 评价

评价：NVIDIA 官方押注的"推理中间层"，与 Mooncake/LMCache 生态深度互操作。局限：强绑定 NVIDIA 技术栈；概念层多（frontend/router/worker/planner），运维心智负担不小。

## 第 18 章 Mooncake：KV Cache 中心化的先驱

### 18.1 定位与背景

Mooncake（6.9k★）是 Kimi（月之暗面）的**生产级 serving 平台**，FAST'25 **最佳论文** *A KVCache-centric Disaggregated Architecture for LLM Serving* 的开源实现。官方数据：在该架构下 Kimi 真实负载可**多处理 75% 的请求**同时遵守 SLO。

### 18.2 核心技术原理

两大组件：**Mooncake Transfer Engine**（基于 RDMA/零拷贝的高性能数据传输层，PyPI 包覆盖 CUDA/NPU/MUSA/EFA/ROCm 全硬件）与 **Mooncake Store**（中心化的 KV Cache 池，内容寻址、跨 serving 实例共享前缀）。核心思想是**以 KV Cache 为中心做资源解耦**：调度器不再只看 GPU 空闲，而是看"哪个节点的缓存池里已有这个前缀"，Prefill 与 Decode 各自独立扩缩。

2026 年生态爆发的标志：**10 月，TensorRT-LLM 官方合并了 Mooncake Store connector**（内容寻址 KV 池化、跨实例前缀复用）；TokenSpeed、SGLang HiCache 等也都把 Mooncake 列为一等存储后端。Mooncake 正在成为"KV Cache 界的 S3"。

### 18.3 评价

评价：学术（FAST'25 最佳论文）与生产（Kimi 全量流量）双重验证，是 KV Cache 基础设施化的开创者。局限：部署需要 RDMA 网络配套；对中小规模用户而言收益不如大集群显著。

## 第 19 章 LMCache 与 production-stack：vLLM 官方集群方案

### 19.1 LMCache（12.2k★）

"最快的 KV Cache 层"，vLLM 生态的官方 KV 管理组件。核心能力：KV Cache 的**跨请求、跨实例复用**——把KV 从 GPU 溢出到 CPU/远端存储，多 vLLM 实例间共享前缀缓存。快速上手：`docker run apostacyh/vllm:lmcache ... --lmcache-config-file example-local.yaml`，第二轮 QA 的 TTFT 大幅下降。典型场景：RAG（共享文档前缀）、多轮对话、多实例负载均衡下的缓存命中。与 Mooncake 的区别：LMCache 偏 vLLM 单机/小集群内的 KV 分层，Mooncake 做的是跨集群的 KV 池化基础设施。

### 19.2 vllm-project/production-stack（2.7k★）

vLLM 官方的**生产部署参考实现**：Helm chart 一键在 K8s 上部署多实例 vLLM，内置请求路由（KV-aware）与 LMCache 集成、Web 仪表盘监控。核心承诺："从单实例到分布式部署，不改一行应用代码"。配套 Lambda Labs/AWS EKS/GCP/Azure 云部署教程与双周社区会。2026 年路线图已发布。定位：不想自建运维体系又想上集群的团队的最短路径。

## 第 20 章 OpenLake：GPU 饥饿的存储引擎

### 20.1 定位与背景

OpenLake（2.6k★）是面向 LLM 推理与 GPU 训练的**高性能分布式存储引擎**，Rust 编写、构建于 `io_uring` 之上。痛点很直白：GPU 越来越快，存储成了瓶颈——checkpoint 恢复、KV 溢出、训练数据吞吐都需要"喂饱 GPU"的存储。

### 20.2 核心技术原理

- 官方宣称 **百万+ IOPS、1ms 内延迟、600µs 读**，推理吞吐 8×。
- **ExANS（2026/08）**：BF16 KV Cache 的**无损 GPU 编解码器**，1.51× 成本节省。
- **Deferred Materialization（2026/07）**："突破 KV 之墙"——管理 100TB 级 KV cache，按需物化而非全量落盘。
- 2026/09 登顶 **MLPerf Storage v3.0** 对象 checkpointing 榜单（领先 NVIDIA 与 Nebius）。

### 20.3 评价与局限

评价：把"存储"拉进了 LLM 基础设施的叙事中心，MLPerf 榜单背书。局限：性能数字全部来自官方博客，**未经独立复现**（引用需注明）；项目较新，生产案例少。

### 20.4 第三篇小结：KV 基础设施的新中间层

| 项目 | 层级 | 核心抽象 | 一句话定位 |
|---|---|---|---|
| Mooncake | 跨集群 | KV Store + Transfer Engine | KV Cache 界的 S3 |
| LMCache | 实例间 | KV 分层缓存 | vLLM 官方 KV 层 |
| Dynamo | 集群 | KVBM + KV-aware 路由 | 推理服务的调度中枢 |
| production-stack | 集群 | Helm 参考实现 | vLLM 上集群最短路径 |
| OpenLake | 存储 | io_uring 存储引擎 + KV 编解码 | 喂饱 GPU 的存储层 |

---

# 第四篇 Hugging Face 生态：模型定义层与它的时代转折

> HF 生态仍是全球模型的"定义层"——但它正在重新定位：2026 年发布 transformers v5，官方推理栈 TGI 归档退场，重心转向训练工具链与新的 Kernel Hub。

## 第 21 章 时代的转折点：TGI 归档事件

### 21.1 事件时间线（经多源交叉核实）

- **2025-12-11**：TGI 官方文档宣布进入 maintenance mode，此后仅接受小 bug 修复与文档改进；官方明确推荐改用 vLLM、SGLang（本地场景 llama.cpp/MLX）。
- **2026-03-21**：GitHub 仓库归档（只读）。归档后新模型（以 Qwen3 为代表）在 TGI 上健康检查失败。
- **2026-06**：sagemaker-python-sdk PR #5960 合并，ModelBuilder 按任务自动路由：text-generation→vLLM DLC，多模态→vLLM-Omni，嵌入/重排→TEI；v3 SDK 彻底移除 TGI 镜像接口。

### 21.2 官方迁移指引

HF Inference Endpoints 文档给出四步法：同模型新建 vLLM 端点（硬件配置通常不变）→ 测试验证 → 切换流量 → 下线旧端点。K8s 自托管迁移要点：镜像 `ghcr.io/huggingface/text-generation-inference` → `vllm/vllm-openai`；入口参数 `--model-id` → `--model`；环境变量 `HUGGING_FACE_HUB_TOKEN` → `HF_TOKEN`。两者均为 OpenAI 兼容 API，客户端代码基本零改动。

### 21.3 历史注脚

TGI 曾于 2023-07 改为 HFOIL 1.0 源可用许可证（禁止托管转售），2024-04 回归 Apache 2.0；最后功能版本 v3.3.7（2025-12）。TGI 的历史功绩在于率先推动"推理引擎依赖 transformers 模型架构定义"的路线——这条路线上被下游继承，只是承接者换成了 vLLM 与 SGLang。

### 21.4 格局影响

TGI 退场后，推理引擎四强格局正式确立：**vLLM（吞吐首选）、SGLang（结构化+前缀复用）、TensorRT-LLM（NVIDIA 极致性能）、llama.cpp（端侧）**。这是一个深刻的行业信号：**通用大厂的全家桶模式敌不过专注引擎的社区模式**。

## 第 22 章 transformers v5 与 peft：模型定义层

### 22.1 transformers（168k★）

2026 年发布 **v5 大版本**——这个从 2020 年沿用至今的模型库，仍是全球最通用的"模型定义层"：所有推理引擎（vLLM/SGLang）与训练框架（TRL/verl 的 FSDP 路径）都以"能加载 transformers 模型"为兼容基线。v5 的方向：清理遗留 API、强化与 kernels 库的集成（从 Hub 拉取优化内核）、持续第一时间接入新架构。它不再是性能担当，而是**兼容性的公共货币**。

### 22.2 peft（21.9k★）

参数高效微调（PEFT）标准库，LoRA/QLoRA/DoRA/IA3/Adapter 全家桶，与 transformers Trainer、TRL、accelerate 无缝衔接。实用亮点：单卡 80GB 上可全参微调大量模型；README 实测 mt0-xxl 全参 OOM → LoRA+DeepSpeed CPU offload 仅需 22GB GPU/52GB CPU。局限：部分架构需手动指定 target_modules；合并进基座后不可逆。

## 第 23 章 TRL 与 Accelerate：训练工具链

### 23.1 trl（19.6k★）

后训练标准库：SFT/GRPO/DPO/KTO/Reward 全家桶，均是对 Transformers Trainer 的轻封装，原生支持 DDP/DeepSpeed ZeRO/FSDP。**GRPOTrainer** 实现 DeepSeek-R1 所用算法（比 PPO 省内存），内置 `accuracy_reward`（math-verify）；`trl.experimental` 为快速演进特性的孵化区。2026 年扩展：AMD Instinct（ROCm）与 AWS Trainium 原生支持；8×B300 单节点 SFT 501B-A23B 的 Beam 模型；集成 Unsloth 加速内核。零代码 CLI：`trl sft --model_name_or_path ... --dataset_name ...` 即完成微调。

### 23.2 accelerate（9.9k★）

PyTorch 训练循环的"薄包装"分布式启动器——约 5 行代码（`Accelerator()` + `prepare()`）让同一份脚本跑单机/多机/TPU/fp16/bf16/fp8。是 transformers Trainer、TRL、PEFT 的公共底层。`accelerate config` 问答式生成配置 → `accelerate launch` 启动。局限：不做并行策略创新（无 TP/PP/EP），千卡场景需上 FSDP/DeepSpeed 或转 torchtitan/Megatron。

## 第 24 章 新基建：kernels 与 TEI、optimum

### 24.1 huggingface/kernels：Kernel Hub（2026 重点新库）

让 Python 库直接从 HF Hub 加载计算 kernel，把"算子"变成和模型权重一样可分发的 Hub 资产——这是 HF 生态 2026 年最值得关注的新基建。设计目标：**可移植**（可从 PYTHONPATH 外加载）、**多版本共存**（同进程加载同一 kernel 的多个版本）、**兼容**（跨 Python/CUDA 版本与 C++ ABI）。三组件：`kernels` Python 包（用户侧）、`kernel-builder`（作者侧构建工具）、官方维护的 kernels-community。

```python
from kernels import get_kernel
activation = get_kernel("kernels-community/activation", version=1)
activation.gelu_fast(y, x)
```

与 vLLM 的 `vllm download-kernels` 相互印证——**"内核即资产、从 Hub 分发"正在成为行业共识**。

### 24.2 TEI 与 optimum

- **text-embeddings-inference（TEI）**：Rust 编写的嵌入/重排序专用推理服务器，主打小镜像、秒级启动（true serverless）；token 动态批处理 + Flash Attention + Candle 自研内核；紧跟 MTEB 前排（Qwen3-Embedding 0.6B–8B、Gemma3、Nomic MoE）。局限：仅覆盖 encoder 类架构，decoder-only 生成式 reranker（如 Qwen3-Reranker）需走 vLLM。
- **optimum（3.5k★）**：硬件优化工具集，2026 年架构重大调整：**ONNX 集成已拆分到独立的 optimum-onnx 仓库**（README 用三重警告强调新安装方式）。现仍收编：OpenVINO+NNCF（Intel）、ExecuTorch（边缘）、Quanto（PyTorch 原生量化）、optimum-habana（Gaudi）、optimum-neuron（Trainium/Inferentia）。子项目碎片化是明显信号。

### 24.3 第四篇小结

HF 生态的战略收缩与聚焦清晰可见：**放弃推理引擎（TGI），守住模型定义层（transformers v5）与训练工具链（trl/peft/accelerate），开辟内核分发（kernels）新战场**。

---

# 第五篇 趋势与展望

## 第 25 章 2026 年基础设施版图全景

```
        ┌─────────────── 模型定义层 ───────────────┐
        │   transformers v5 · GGUF · safetensors   │
        └──────────────────┬───────────────────────┘
                           │
     ┌─────────────────────┼──────────────────────┐
     │ 训练侧              │                       │ 推理侧
     │ Megatron-LM(极致)   │                       │ vLLM/SGLang(引擎)
     │ torchtitan/Automodel│                       │ llama.cpp/MNN(端侧)
     │ (PyTorch 原生)      │                       │ TokenSpeed/uzu(新变量)
     │        │            │                       │        │
     │   RL 框架层         │                       │   服务编排层
     │   verl(标准)        │                       │   Dynamo(调度)
     │   slime→Miles(生产) │                       │   production-stack
     │   AReaL(异步)       │                       │        │
     │        │            │                       │   KV 基础设施层
     └────────┼────────────┘                       │   Mooncake(LMCache/OpenLake)
              │                                    │        │
              └────────────┬───────────────────────┘
                           │
                  ┌────────┴────────┐
                  │  硬件与算子层     │
                  │  NV/AMD/昇腾/摩尔 │
                  │  kernels(Hub 分发)│
                  │  FP8/MXFP4/NVFP4 │
                  └──────────────────┘
```

## 第 26 章 五大趋势判断

**趋势一：TGI 时刻的启示——专注者胜。** Hugging Face 全家桶模式在推理引擎上败给了 vLLM/SGLang 的社区模式。基础设施层没有"生态绑架"，只有性能与工程质量的持续竞争。这对所有"平台型"玩家是警示：基础设施层的护城河是性能曲线，不是先发优势。

**趋势二：RL 后训练的"vLLM 时刻"。** 训练重心从 SFT 转向 RL 已不可逆。verl 的 HybridFlow 成为事实标准，围绕它分化出五条路线（生产派 slime/Miles、异步派 AReaL/SkyRL、垂类派 EasyR1、算法派 RAGEN、平台派 Oumi）。共性技术收敛于：训练/rollout 解耦、全异步化、MoE 路由一致性（R3）、token 级一致性（TITO）、低精度 RL（MXFP8/NVFP4）。

**趋势三：KV Cache 成为数据中心的一等公民。** 从 PagedAttention（引擎内分页）到 RadixAttention/HiCache（跨请求复用）到 Mooncake Store（跨集群池化）再到 OpenLake（存储层编解码），KV Cache 管理的边界在持续外扩。TensorRT-LLM 官方合并 Mooncake connector 是标志性事件：**KV 池化已成行业共识，"KV 界的 S3"之争刚刚开始。**

**趋势四：两条训练路线的长期对峙。** Megatron（极致性能、NVIDIA 私有栈、学习曲线陡峭）vs torchtitan/Automodel（PyTorch 原生 DTensor、可读可改、上限稍低）。前者是超大规模的现在，后者是中长期的潜在赢家——就像 CUDA C++ 与 Triton 的关系。Megatron Bridge 的出现说明两条路线正在互相靠近。

**趋势五：解耦-复用-降精度的三重奏。** P/D 分离、控制面/执行面分离（TokenSpeed）、训练/rollout 分离（RL 框架）——解耦无处不在；KV 复用、权重转换（Bridge）、内核分发（kernels）——复用无处不在；FP8→MXFP8→NVFP4 从推理渗入训练——降精度无处不在。这三条工程主旋律，比任何单一项目都更能预示 2027 年的方向。

## 第 27 章 选型速查与 2027 年信号

### 27.1 按需求选型

| 你的需求 | 推荐组合 |
|---|---|
| 数据中心生产推理 | vLLM + LMCache + production-stack（或 SGLang + Dynamo） |
| Agent/多轮/RAG 高命中场景 | SGLang（RadixAttention + HiCache） |
| 端侧/个人本地 | llama.cpp（+ Unsloth Desktop 工作台） |
| 手机/IoT 国内工业级 | MNN；穿戴/云接力场景看 Cactus |
| 消费级跑超大 MoE | KTransformers |
| RL 后训练（通用） | verl（基座）+ SGLang（rollout） |
| RL 后训练（GLM 系/生产） | slime 或 Miles |
| RL 后训练（agentic/异步） | AReaL 或 SkyRL |
| 多模态 RL 快速实验 | EasyR1 |
| 万卡预训练 | Megatron-LM（或 torchtitan 研究路线） |
| K8s 集群训练 | Kubeflow Trainer |
| 嵌入/重排服务 | TEI（生成式 reranker 走 vLLM） |
| 微调入门/单卡 | Unsloth + peft + TRL |


### 27.2 2027 年值得盯的三个信号

1. **TokenSpeed 能否撬动 vLLM 的地位**——中立基金会 + 顶级厂商赞助 + 编译期安全，如果它在生产案例上站稳，将是推理引擎格局五年来最大的变数。
2. **KV 池化的标准化**——Mooncake Store 已被 TensorRT-LLM/SGLang/TokenSpeed 三方接入，若 vLLM 官方（LMCache 之外）也原生接入，KV 界的"S3 协议"将提前到来。
3. **PyTorch 原生路线的临界点**——torchtitan 的 TitanRL 若在超大规模 RL 训练上追平 Megatron 后端的性能，训练侧将迎来像 TGI 归档一样的标志性洗牌。

---

# 附录

## 附录 A：项目速查表（35+ 项目）

### 推理引擎

| 项目 | Stars | 更新 | 一句话 |
|---|---|---|---|
| ggml-org/llama.cpp | 132.7k | 2026-10-29 | C/C++ 端侧事实标准，GGUF 生态核心 |
| vllm-project/vllm | 95.8k | 2026-10-29 | PagedAttention、E/P/D 分离，吞吐之王 |
| unslothai/unsloth | 78.7k | 2026-10-29 | 微调加速 + 本地训练/运行 UI 三形态 |
| sgl-project/sglang | 38.3k | 2026-10-29 | RadixAttention，Agent/RL rollout 首选 |
| kvcache-ai/ktransformers | 19.6k | 2026-10-28 | CPU+GPU 异构跑超大 MoE + MoE 微调 |
| mlc-ai/web-llm | 19.3k | 2026-10 | 浏览器内 WebGPU 推理 |
| alibaba/MNN | 16.3k | 2026-10-28 | 阿里端侧引擎，Hexagon DSP |
| cactus-compute/cactus | 6.2k | 2026-09 | 手机/穿戴端云混合全家桶 |
| trymirai/uzu | 3.0k | 2026-10-29 | Rust + Apple 统一内存 SDK |
| lightseekorg/tokenspeed | 2.8k | 2026-10-29 | 编译期 KV 安全的新引擎 |

### RL 后训练框架

| 项目 | Stars | 更新 | 一句话 |
|---|---|---|---|
| verl-project/verl | 24k | 2026-10-29 | HybridFlow，RL 事实标准（EuroSys'25） |
| oumi-ai/oumi | 9.4k | 2026-10-27 | SFT/RL/评测/部署全生命周期平台 |
| THUDM/slime | 8.8k | 2026-10-27 | GLM 全系 RL 生产验证，SGLang 原生 |
| areal-project/AReaL | 5.9k | 2026-10-28 | 全异步 RL + 微服务 2.0（2.77× 加速） |
| hiyouga/EasyR1 | 5.2k | 2026-09 | 多模态 RL Quick Start（verl 精简 fork） |
| radixark/miles | 3.2k | 2026-10-29 | TITO+R3+NVFP4 企业级生产派 |
| mll-lab-nu/RAGEN | 2.8k | 2026-10-28 | StarPO 与 Echo Trap 算法诊断 |
| NovaSky-AI/SkyRL | 2.5k | 2026-10-29 | Berkeley 全栈四件套 + Tinker 开源后端 |

### 训练与数据中心

| 项目 | Stars | 更新 | 一句话 |
|---|---|---|---|
| NVIDIA/Megatron-LM | 18.3k | 2026-10-29 | TP/PP/EP/CP 五维并行基座，Core 0.19 |
| stas00/ml-engineering | 19.6k | 2026-10-27 | 万亿级训练实战开源书 |
| ai-dynamo/dynamo | 8.4k | 2026-10-29 | NVIDIA 推理调度中枢（KVBM+路由） |
| LMCache/LMCache | 12.2k | 2026-10-29 | vLLM 官方 KV 分层缓存层 |
| kvcache-ai/Mooncake | 6.9k | 2026-10-28 | Kimi 生产平台，FAST'25 最佳论文 |
| pytorch/torchtitan | 5.9k | 2026-10-29 | PyTorch 原生训练 + TitanRL |
| kubeflow/trainer | 2.3k | 2026-10-28 | K8s 分布式训练标准（v2.3） |
| vllm-project/production-stack | 2.7k | 2026-10-27 | vLLM 官方 K8s 参考实现 |
| openlake-project/openlake | 2.6k | 2026-10-24 | io_uring 存储引擎，MLPerf Storage 榜首 |
| NVIDIA-NeMo/Automodel | 1.1k | 2026-10-29 | DTensor 原生 HF 模型微调 |

### Hugging Face 生态

| 项目 | Stars | 说明 |
|---|---|---|
| huggingface/transformers | 168k | v5 大版本，模型定义公共货币 |
| huggingface/peft | 21.9k | LoRA/QLoRA 标准库 |
| huggingface/trl | 19.6k | SFT/DPO/GRPO 训练标准库 |
| huggingface/accelerate | 9.9k | 分布式启动器 |
| huggingface/kernels | — | Kernel Hub，内核从 Hub 分发 |
| huggingface/TEI | — | 嵌入/重排 Rust 服务器 |
| huggingface/optimum | 3.5k | 硬件优化导出（ONNX 已拆分） |
| TGI（已归档） | — | 2026-03-21 归档，官方建议迁 vLLM/SGLang |

## 附录 B：术语表

- **PagedAttention**：vLLM 的核心机制，借鉴 OS 虚拟内存分页管理 KV Cache，消除显存碎片。
- **RadixAttention**：SGLang 的核心机制，用基数树跨请求复用 KV 前缀。
- **P/D 分离（PD Disaggregation）**：预填充与解码部署到不同 GPU 集群；vLLM v0.28 扩展为 E/P/D（编码器/预填充/解码三段）。
- **Model Runner V2**：vLLM 重写的 worker 层，固定 GPU 状态表 + Triton gather kernel 实现零同步调度。
- **HybridFlow**：verl 的混合控制器编程模型（EuroSys'25），单控制器编排算法、多控制器执行引擎。
- **3D-HybridEngine**：verl 中训练/生成切换时的 actor 模型 resharding 机制。
- **TITO（Token-in-Token-out）**：Miles 的 token 级一致性机制，rollout 与训练间不做重分词。
- **R3（Rollout Routing Replay）**：Miles 的 MoE 专家路由重放，消除训练/推理路由不一致。
- **StarPO**：RAGEN 提出的多轮 Agent RL 算法框架（State-Thinking-Actions-Reward PO）。
- **Echo Trap / 模板坍缩**：RAGEN-2 诊断的 Agent RL 失败模式——跨输入输出雷同，需用条件熵与互信息双轴检测。
- **GGUF**：llama.cpp 的单文件模型格式，封装权重/tokenizer/元数据/聊天模板。
- **KVBM**：Dynamo 的 KV Cache Block Manager，跨引擎 KV 管理层。
- **ExANS**：OpenLake 的 BF16 KV Cache 无损 GPU 编解码器。
- **MXFP8/NVFP4**：Blackwell 原生低精度格式，2026 年 RL 训练配方已端到端支持。
- **Kernel Hub**：huggingface/kernels 提出的"算子即 Hub 资产"分发模式。
- **HFOIL**：TGI 曾短暂使用的源可用许可证（禁止托管转售），2024-04 已回归 Apache 2.0。

---

*全书完。数据截至 2026-10-29，基于各项目 README、官方论文（SOSP'23/SOSP'25/EuroSys'25/FAST'25/OSDI'26 等）与官方博客的一手调研。*

