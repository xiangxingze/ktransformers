<!-- 省略目录 -->
# 仅需24GB显存的桌面上运行GPT-4/o1级别的本地VSCode Copilot
- [概要](#概要)
	- [前提条件](#前提条件)
	- [基准测试结果](#基准测试结果)
		- [V0.2](#v02)
			- [设置](#设置)
			- [内存消耗](#内存消耗)
			- [基准测试结果](#基准测试结果)
		- [V0.3预览](#v03预览)
			- [设置](#设置-1)
			- [内存消耗](#内存消耗-1)
			- [基准测试结果](#基准测试结果-1)
	- [如何运行](#如何运行)
		- [V0.2演示](#v02演示)
			- [单插槽版本（32核心）](#单插槽版本32核心)
			- [双插槽版本（64核心）](#双插槽版本64核心)
		- [V0.3演示](#v03演示)
			- [双插槽版本（64核心）](#双插槽版本64核心)
	- [一些解释](#一些解释)
	- [常见问题](#常见问题)
		- [R1无思考](#r1无思考)
		- [更多常见问题](#更多常见问题)

# 概要

> **2025年2月10日**：支持在单卡（24GB显存）/多卡和382G内存上运行DeepseekR1和V3，速度提升最高可达3~28倍。<br>

大家好，我们是KTransformers团队（曾因本地CPU/GPU混合推理开源项目DeepSeek-V2而闻名）。

我们听到了大家对DeepSeek-R1/V3的请求——我们很高兴终于能为大家提供支持！
抱歉让大家久等，但我们一直在酝酿一些真正令人惊叹的东西！

今天，我们很自豪地宣布，我们不仅支持DeepSeek-R1/V3，正如以下视频所示：

https://github.com/user-attachments/assets/ebd70bfa-b2c1-4abb-ae3b-296ed38aa285

</p>

- **[新功能!!!] 本地671B DeepSeek-Coder-V3/R1**：仅使用14GB显存和382GB内存运行其Q4_K_M版本。
	- 预填速度（tokens/s）： 
 		- KTransfermor：54.21（32核心）→ 74.362（双插槽，2×32核心）→ 255.26（优化的基于AMX的MoE内核，仅V0.3）→ 286.55（选择性使用6个专家，仅V0.3）
 		- 相比llama.cpp在2×32核心下的10.31 tokens/s，速度提升最高可达**27.79倍**。
 	- 解码速度（tokens/s）：
 		- KTransfermor：8.73（32核心）→ 11.26（双插槽，2×32核心）→ 13.69（选择性使用6个专家，仅V0.3）
 		- 相比llama.cpp在2×32核心下的4.51 tokens/s，速度提升最高可达**3.03倍**。


我们还展示了即将到来的优化预览，包括英特尔AMX加速内核和选择性专家激活方法，这将显著提升性能。使用V0.3预览版，我们实现了最高286 tokens/s的预填速度，使得本地推理速度比llama.cpp快**28倍**。
二进制分发现已提供，源代码将尽快发布！请在此处查看wheel包：[点击这里](https://github.com/kvcache-ai/ktransformers/releases/download/v0.1.4/ktransformers-0.3.0rc0+cu126torch26fancy-cp311-cp311-linux_x86_64.whl)


## 前提条件
我们在以下配置上进行了最佳性能测试（V0.2）：<br>
CPU：Intel (R) Xeon (R) Gold 6454S 1T内存（2个NUMA节点）<br>
GPU：4090D 24G显存<br>
内存：标准DDR5-4800服务器内存（1 TB）
## 基准测试结果
### V0.2
#### 设置
- 模型：DeepseekV3-q4km（int4）<br>
- CPU：cpu_model_name: Intel (R) Xeon (R) Gold 6454S，每插槽32核心，2插槽，2个NUMA节点
- GPU：4090D 24G显存
- 我们在充分预热后进行测试
#### 内存消耗：
  - 单插槽：382G内存，至少14GB显存
  - 双插槽：1T内存，至少14GB显存

#### 基准测试结果

"6 experts"案例是V0.3预览的一部分

| 提示<br>(500 tokens) | 双插槽Ktrans（6个专家） | 双插槽Ktrans（8个专家） | 单插槽Ktrans（6个专家） | 单插槽Ktrans（8个专家）| llama.cpp（8个专家） | 
| --- | --- | --- | --- | --- | --- | 
| 预填token/s | 97.32 | 82.94 | 65.14 | 54.21 | 10.31 |
| 解码token/s | 13.69 | 12.208 | 10.303 | 8.73 |4.51 |

**解码速度提升最高可达<u>3.03倍</u>，预填速度提升最高可达<u>9.44倍</u>。**

### V0.3预览
#### 设置
- 模型：DeepseekV3-BF16（在线量化为int8用于CPU，int4用于GPU）
- CPU：cpu_model_name: Intel (R) Xeon (R) Gold 6454S，每插槽32核心，2插槽，2个NUMA节点
- GPU：（1~4）x 4090D 24G显存（较长的提示需要更多显存）

#### 内存消耗：
- 644GB内存，至少14GB显存

#### 基准测试结果
| 提示长度  | 1K  | 2K  | 4K  | 8K |
|---------------|-----|-----|-----|-----|
| KTrans（8个专家）预填token/s |   185.96  |  255.26   |  252.58   |  195.62   |
| KTrans（6个专家）预填token/s |   203.70  |  286.55   |  271.08   |  207.20   |

**KTrans V0.3的预填速度比KTrans V0.2快<u>3.45倍</u>，比llama.cpp快<u>27.79倍</u>。**
**解码速度与KTrans V0.2（6个专家版本）相同，因此省略**

主要加速来源：
- 英特尔AMX指令集和我们特别设计的缓存友好内存布局
- 基于离线域外数据配置文件选择较少专家的策略


*根据我们对DeepSeekV2、DeepSeekV3和DeepSeekR1的研究，当我们稍微减少推理中的激活专家数量时，输出质量不会改变。但解码和预填速度会加快，这是令人鼓舞的。因此，我们的演示利用了这一点*

## 如何运行
### V0.2演示
#### 单插槽版本（32核心）
我们的local_chat测试命令是：
``` shell
git clone https://github.com/kvcache-ai/ktransformers.git
cd ktransformers
git submodule init
git submodule update
numactl -N 1 -m 1 python ./ktransformers/local_chat.py --model_path <你的模型路径> --gguf_path <你的gguf路径>  --prompt_file <你的提示文件>  --cpu_infer 33 --max_new_tokens 1000
<当你看到聊天时，按回车加载提示文件>
```
`<你的模型路径>`可以是本地路径，也可以从在线Hugging Face设置，如deepseek-ai/DeepSeek-V3。如果在线连接遇到问题，请尝试使用镜像（hf-mirror.com）<br>
`<你的gguf路径>`也可以在线上，但由于文件较大，建议你下载并量化模型为所需格式（注意这是目录路径）<br>
`--max_new_tokens 1000`是最大输出token长度。如果你发现答案被截断，可以增加这个数字以获得更长的答案（但要注意OOM，增加这个数字会降低生成速度）。 
<br>
命令numactl -N 1 -m 1旨在避免NUMA节点之间的数据传输<br>
注意！如果你正在测试R1并且可能会跳过思考。你可以添加参数：`--force_think true`。这在[常见问题](#常见问题)部分有解释

#### 双插槽版本（64核心）
在安装（使用install.sh或`make dev_install`）之前，确保通过`export USE_NUMA=1`设置环境变量`USE_NUMA=1`（如果已经安装，请重新安装并设置此环境变量）<br>
我们的local_chat测试命令是：
``` shell
git clone https://github.com/kvcache-ai/ktransformers.git
cd ktransformers
git submodule init
git submodule update
export USE_NUMA=1
make dev_install # 或 sh ./install.sh
python ./ktransformers/local_chat.py --model_path <你的模型路径> --gguf_path <你的gguf路径>  --prompt_file <你的提示文件>  --cpu_infer 65 --max_new_tokens 1000
<当你看到聊天时，按回车加载提示文件>
```
参数的含义相同。但由于我们使用双插槽，我们将cpu_infer设置为65

### V0.3演示
#### 双插槽版本（64核心）
我们的local_chat测试命令是：
``` shell
wget https://github.com/kvcache-ai/ktransformers/releases/download/v0.1.4/ktransformers-0.3.0rc0+cu126torch26fancy-cp311-cp311-linux_x86_64.whl
pip install ./ktransformers-0.3.0rc0+cu126torch26fancy-cp311-cp311-linux_x86_64.whl
python -m ktransformers.local_chat --model_path <你的模型路径> --gguf_path <你的gguf路径>  --prompt_file <你的提示文件>  --cpu_infer 65 --max_new_tokens 1000
<当你看到聊天时，按回车加载提示文件>
```
参数的含义与V0.2相同。但由于我们使用双插槽，我们将cpu_infer设置为65

## 一些解释
1. 我们还希望进一步利用Xeon Gold CPU上的两个NUMA节点。为了避免节点之间数据传输的成本，我们在两个节点上“复制”了关键矩阵，这会消耗更多内存但会加速预填和解码过程。但这种方法在加载权重时会占用大量内存且速度较慢，因此在加载时要耐心并监控内存使用情况。我们将优化这一巨大的内存开销。敬请期待~<br>
2. 命令参数`--cpu_infer 65`指定使用多少核心（可以超过物理核心数量，但并不是越多越好。根据实际核心数量稍微调整到较低值）<br>

3. 为什么要进行CPU/GPU混合推理？
DeepSeek的MLA运算符计算量非常大。虽然完全在CPU上运行是可能的，但将繁重的计算卸载到GPU会带来巨大的性能提升。

4. 速度提升来自哪里？

   - 专家卸载：与传统的基于层或KVCache卸载（如llama.cpp所见）不同，我们将专家计算卸载到CPU，将MLA/KVCache卸载到GPU，完美契合DeepSeek的架构，实现最佳效率。
   - 英特尔AMX优化——我们的AMX加速内核经过精心调优，比现有的llama.cpp实现快几倍。我们计划在清理后开源该内核，并考虑将其贡献给llama.cpp。

5. 为什么选择英特尔CPU？
英特尔是目前唯一支持AMX指令集的CPU供应商，与仅支持AVX的替代品相比，性能显著提升。
## 常见问题
### R1无思考
注意！如果你正在测试R1并且可能会跳过思考。你可以添加参数：`--force_think true`。详细信息在[常见问题](./FAQ.md)部分<br>

### 更多常见问题
[查看详细信息](./FAQ.md)
