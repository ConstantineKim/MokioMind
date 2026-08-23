# MiniMind 学习记录

最后更新：2026-08-23

## 学习目标

- 从理论走到实操，真正理解小型语言模型从数据到训练、微调和推理的完整流程。
- 第一遍以 MokioMind 为主线；它是 MiniMind 的教学复刻。掌握主线后，再与新版 MiniMind 对照。
- 不只会运行命令：能解释输入、标签、损失、梯度、训练数据和模型输出之间的关系。

## 学习者画像与教学约定

- 已接触过大模型基础理论，但几乎没有代码实操经验。
- 不应假定已经理解术语、终端、Python 环境、Git、JupyterLab、SSH、训练数据格式等概念。
- 术语首次出现时先给白话定义，再说明其在当前项目中的位置和作用。
- 不要求直接阅读整份代码；每次只指定一个类或函数，以及这次需要回答的少量问题。
- 优先采用：`一个具体例子 -> 白话解释 -> 对应代码位置 -> 最小实验 -> 回顾`。
- 终端 `sed` 输出不适合阅读；Mac 上用编辑器阅读，云端只用于执行和训练。
- 每次只安排有限步骤；解释应以理解为目标，而不是让学习者获得模糊印象。
- 学习者已能用白话说明 logits、SFT loss 和 temperature 的关系，但当前不要求读懂对应 Python 代码；先建立概念与实验输出的对应关系，再逐步引入语法。
- 在每次实验前必须明确区分：环境验证、概念演示、真实模型训练；说明该实验是否创建模型、是否更新参数、是否使用 GPU，以及预期应观察什么。不能把“命令成功运行”当作理解完成。
- 优先用 Jupyter Notebook 分格运行教学实验：每格只承载一个概念或少量代码，输出显示在格子下方；终端主要保留给安装、Git、文件操作和正式训练。
- 每个新知识点开头标注：行业状态（基础标准 / 常见工程实践 / 前沿或研究性 / 教学简化）、现实用途、学习深度，说明它与企业常见 LLM 工作流的关系和局限。
- Python 语法基础不足：解释项目代码时要先拆开 Python 语法（变量、赋值、切片、函数调用、缩进块等），不假设能直接读懂表达式。

## 环境与版本

### Mac（阅读与小实验）

- 设备：MacBook Air M2，16GB 内存。
- 不用于正式或长时间训练。
- 本地代码目录：`/Users/mrwong/Documents/Program/AI/Minimind/eg/MokioMind-master`
- 本地代码版本：Git commit `125dc2d`，工作区应保持干净。
- 此版本是从 GitHub 直接克隆并 checkout 到指定 commit，不再使用旧压缩包目录。

### AutoDL（运行与训练）

- 平台：AutoDL。
- 实例：北京 B 区，vGPU-32GB * 1；实际识别为 NVIDIA GeForce RTX 4080 SUPER，约 32GB 显存。
- CPU：12 核；内存：62GB；数据盘：50GB。
- 云端代码目录：`/root/autodl-tmp/MokioMind`
- 云端代码版本：Git commit `125dc2d`，工作区干净。
- Python：3.12；PyTorch：2.5.1+cu124；CUDA 可用；已安装 `transformers`。
- 使用 JupyterLab 内置终端操作。SSH 可作为备用，不是当前教学入口。
- 实例按量计费。每次云端实验结束后应在 AutoDL 控制台点击“关机”，不要点击“释放”。
- 正常关机后当前实例的 `/root/autodl-tmp` 通常保留；释放实例或换一台新实例时不会自动迁移，因此 GitHub 是跨实例的主备份。

## 已完成

### 云端环境验证

执行 `nvidia-smi`，GPU 正常识别且无运行进程。PyTorch 检查结果：`2.5.1+cu124`，CUDA 可用，GPU 为 `NVIDIA GeForce RTX 4080 SUPER`。

### 第一个最小语言模型训练步骤

用 MokioMind 创建教学微型模型：hidden size 64、2 层 Transformer、8 个 attention heads、词表大小 128、batch size 2、序列长度 12。

完成一次 `input_ids -> logits -> loss -> backward()`：

```text
input_ids: (2, 12)
logits:    (2, 12, 128)
params:    102,720
loss:      4.8524
grad_norm: 2.6576
```

### 结构变化小实验

```text
layers=2, seq_len=12, masked=False, params=102,720, loss=5.1532
layers=2, seq_len=24, masked=False, params=102,720, loss=5.0210
layers=4, seq_len=12, masked=False, params=197,184, loss=5.1149
layers=2, seq_len=12, masked=True, params=102,720, loss=4.7744
```

已观察到：序列长度增加时参数量不变；Transformer 层数增加时参数量增加。

### Token 化实验

文本 `你好，世界`：

```text
文本 token id: [5134, 270, 2219]
input_ids: [1, 5134, 270, 2219, 2, 0, ..., 0]
labels:    [1, 5134, 270, 2219, 2, -100, ..., -100]
mask:      [1, 1, 1, 1, 1, 0, ..., 0]
```

BOS id 为 1，EOS id 为 2，PAD id 为 0。PAD 在 labels 中改为 `-100`，不参与 loss；mask 为 1 的位置是真实文本，0 是补齐位置。

### Logits、loss 与参数更新

- 三选一交叉熵：正确且自信时 loss 为约 0.0247，错误且自信时约 5.0247。
- 微型 MokioMind logits shape 为 `(2, 12, 128)`；三维依次是 batch 中样本数、序列位置数、词表候选数。
- 最小参数更新：10 次更新后，loss 从 `1.0986` 降至 `0.1690`；正确候选的分数升高，其他候选分数下降。
- 需牢记：`target=0` 的 0 是候选编号；`logits=[5,1,0]` 最后一个 0 是第 2 个候选的分数。

### Notebook 本地备份

- 云端 Notebook：`/root/autodl-tmp/MokioMind/01_logits_and_training.ipynb`
- 本地分支：`learning-notes`
- 已提交：`48bbf38 Add logits learning notebook`，其父提交为 `125dc2d`。
- 本地提交包含 Notebook 与 `.omc/project-memory.json`；当前工作区干净。

## 术语白话表

- **token**：模型处理文字时切出的最小片段，不一定等于一个汉字或英文单词。
- **token id**：每个 token 在词表中的数字编号。
- **input_ids**：一条文本转换得到的一串 token id，是模型真正接收的数字输入。
- **labels**：训练时用于判定模型预测是否正确的目标答案。
- **pretrain / 预训练**：用大量原始文本，让模型反复练习“根据前文预测下一个 token”。
- **SFT / 有监督微调**：用“问题和标准回答”训练，让模型更会遵循指令和回答问题。
- **assistant**：对话数据中模型要学习模仿的回答方；user 是提问方，system 是规则或背景方。
- **loss / 损失**：模型预测与目标答案之间的差距，训练目标是让它逐渐变小。
- **gradient / 梯度**：告诉参数应该往哪个方向调整，才能让 loss 下降的信号。
- **logits**：模型针对下一个位置的全部候选 token 给出的原始分数；分数越高，模型越倾向该 token。训练时 loss 推动正确 token 的 logit 相对更高；推理时 temperature 调节 logits 的差距。logits 不是概率。
- **候选编号与分数**：`target=0` 中的 0 是第 0 个候选的编号；`logits=[5,1,0]` 中的 0 是第 2 个候选的分数。
- **PAD（补齐）**：为让同一批文本长度一致而添加的空位 token；由 attention mask 和 `-100` 在模型计算中忽略。
- **`-100`**：PyTorch 交叉熵损失中的忽略标记；该位置不计入 loss。

## 下次学习

继续以 Notebook 逐格学习：将 next-token prediction 的“错开一位”与真实 `labels` 对齐；之后再回到 `PretrainDataset.__getitem__`。

