# MemT-1.0 消融与系统实验：一手数据 ＋ DeepSeek 读出的补充（2026-10-21）

> **用途**：给外部强模型（ChatGPT 镜像站）读的**评审材料**——只放"已核对过的结论与数字"，不放我的结论。
> **数据来源两路**：
> ① **一手**：arXiv HTML 全文（`https://arxiv.org/html/2610.26825v2`）中我**直接读到**的段落与表格；
> ② **DeepSeek 官网**读同一份 HTML 得到的转述（**§4/§5/§6.4**）——其数字与①在窗口化实验上**逐个吻合**（见 §四）。

---

## 一、§4.2 Efficient Operators（窗口化）——**两路数字完全一致**

32K pack、2048-token step、单 GPU 合成输入：

| 窗口 | 峰值显存 | F+B 中位时间 | 代价 |
| :-- | :-- | :-- | :-- |
| whole-pack（不切） | **47.83 GiB** | 462.7 ms | — |
| 4096-token window（2 step） | **9.49 GiB**（降 80.1%） | **470.6 ms**（增 1.6%） | 多 2 个 step 的读出融合 |
| 2048-token window（1 step） | **6.03 GiB**（降 87.4%） | **493.5 ms**（增 6.6%） | 时间换显存 |

**一手补充（§4.2 原文，DeepSeek 未读到）**：*"As pack length increases from 8K to 32K, whole-pack peak memory grows from
**12.37 GiB → 47.83 GiB**, while **4096-token windows grow from 8.36 GiB → 9.49 GiB**. Windowing bounds the lifetime of
projections and intermediates, while retained outputs and boundary states still grow with length."*
⇒ **窗口化把"随历史线性增长的显存"压成近常数**（8.36→9.49 GiB，8K→32K）。

**一手补充（附录 F 表 16，波动性）**（32K/4096）：whole-pack 中位 **462.96 ms**（σ 0.76）vs windowed **470.58 ms**（σ 38.01）
⇒ **窗口化的中位时间只多 1.6%，但单次波动大 50 倍**（σ 0.76 → 38.01）——**代价是尾延迟变差**。

## 二、§4.3 有状态推理与服务（DeepSeek 读出）

- backbone 与 hyper memory 权重固定、跨轨迹共享；**每条轨迹各自维护动态记忆状态**；
  状态元素数 `N_state = Σ_l E_l · H_kv,l · d_k,l · d_v,l`（无 MoS 时 E_l=1）⇒ **持久状态大小可由配置直接估算，不随历史增长**。
- 每个 step 内生成读 incoming memory snapshot，当前文本用该 step 的 KV cache；交互结束后 storage 把输入与回复写入记忆。
- 数字：Evaluation 的两 GPU、单请求 BEAM 实验，**64K token 历史下，Nano 的每轨迹状态大小相比 full context 约降 33×**。

## 三、§5 记忆缩放行为（DeepSeek 读出）

- **§5.2 协议**：全部基于 Nano，backbone/训练目标/评测协议固定；数据缩放＝沿一条连续 post-training 轨迹扫 checkpoint；
  容量缩放＝改变 MoS 的 memory expert 数量，其余不变。
- **§5.3 数据与训练缩放（Figure 6）**：从 CPT checkpoint 出发，同一 post-training 语料跑 2 个 epoch：
  平均分 **36.65 → 46.80**；增益**早期最大**（第 1/4 epoch 内陡升，第一 epoch 末 45.40）；
  同期 LoCoMo **40.39 → 55.39**；第二 epoch 末采样 **46.80 / 57.53**。
  ⇒ **固定容量下继续训练仍有增益，且跨 epoch 仍涨**。
- **§5.4 记忆容量缩放（Figure 7）**：400 步 CPT 后，post-training 中比 **2 个 vs 4 个 memory expert**（200／3400／5500 步评测）：
  200 步：+0.41 平均／+1.42 LoCoMo；3400 步：**+2.01／+3.25**；5500 步：**+2.69／+2.20**；
  4-expert 从 3400→5500 继续升：平均 44.29→44.68，LoCoMo 53.51→55.58。
  ⇒ **扩 expert 有益，且后期 checkpoint 增益更大**。

## 四、§6.4 消融（★ 最有价值的一节，DeepSeek 读出，Table 4）

### 6.4.1 架构消融（baseline 平均 46.97）

| 变体 | 平均分 | 读法 |
| :-- | :-- | :-- |
| **baseline（token-level 更新）** | **46.97** | — |
| **w/ TL：turn-level 更新粒度** | **40.98** | **降幅最大（−5.99）** |
| w/o 稳定机制（去掉 GDN-2 的稳定化） | 46.43 | 略降 |
| w/ GDN（换成旧 GDN） | 46.72 | 接近 baseline |
| w/ BE（双向编码） | 45.43 | **逐 benchmark 不一致**：MB/IM/PM/PMS 提升，其余退化，总体平均反而降 |

⇒ **真正带来收益的是"更新的时间粒度"（token-level ≫ turn-level），而不是记忆读写规则的具体形式**
（去掉稳定机制、换 GDN 都只是小幅下降）；**双向编码总体无益**。
⚠️ 本节**未**把 local/hyper memory 比例、MoS、门控作为消融项（DeepSeek 明确标"未找到"）。

### 6.4.2 训练消融

- **post-training 贡献更大**：相对 CPT checkpoint，post-training 把平均分提高约 **+10.69 分**；
  CPT 作为初始化也带来额外总体收益，并能改善 post-training。
- 具体"哪段训练数据起作用"**未找到**（本节没给细分数字）。

## 五、§4.1 分布式训练（DeepSeek 读出）

- 训练 pack 把多条独立轨迹拼在一起；attention 只在每个 step 内计算，**memory 在同一轨迹的 step 之间传递、在轨迹边界重置**；
  逐 decoder layer 处理整个 pack，层内 attention 与 memory 投影按 token 批量计算，**只有 memory block 沿轨迹推进状态**。
- 框架：Megatron-Core ＋ Bridge；backbone MoE 层用 expert parallelism。**GPU 卡数：未找到**。

## 六、明确"未找到"的（不做推测补全）

- **附录 C（GDN-2 稳定机制 / C.2 对称化修正 symmetrization fix）**：DeepSeek 未读到（页面在 §6.5 截断）。
  我这边一手 HTML 也只看到目录与 §2.2.2 的一句"stable mechanism 的详细分析见附录 C"。
- **附录 E/F 完整结果**：同上；但 §4.2 Table 1 的核心数字两路一致（见 §一）。
- **local/hyper memory 比例、MoS、门控各自的消融贡献**：论文 §6.4 未列（DeepSeek 与我一致确认未找到）。

## 七、请评审的问题（针对一个"个人 AI 记忆系统"的设计者）

背景：我要把这类机制落到一个**个人 Agent 记忆系统**上（明文记忆条目 ＋ 规则文件 ＋ 检索层 ＋ 一个"每天一条资讯精选"的自动流程），
它与论文的区别是：**不能训练基座、不能改权重**，只能改**写入策略、字段与触发时机**。

1. **§6.4.1 的"粒度 > 规则形式"这个结论**，对"明文记忆系统"应该怎么翻译？
   论文的 token-level 更新在明文系统里对应什么可操作的做法？
2. **§6.4.2 的 post-training 贡献 +10.69 分**，对"记忆该放规则文件还是放记忆条目"有什么指导？
   论文的 post-training 相当于"内化"，在明文系统里对应什么？
3. **§5.4 扩 expert 有益（+2.69）**，对一个不能改模型、只能改文件与流程的系统，
   等价的"扩容量"手段是什么？
4. **§4.3 的"每轨迹状态固定大小、可由配置直接估算"**这个设计，
   对我的"每天一条资讯 ＋ 规则文件 ＋ 可检索条目"的结构意味着什么？
5. 这篇论文里**哪些机制是我这种明文系统用不上的**（别硬套）？哪些是真正能落地的？
6. 最后：给我一个**能在一周内落地**的最小改造清单，按收益排序。

**要求**：只针对上面六问作答，结论先行，能落到具体做法就落到具体做法；
不要复述论文原文；若某问在论文里没有依据，明说"论文未给依据"，不要编。
