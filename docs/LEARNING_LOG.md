# MiniMind 学习记录

最后更新：2026-08-31

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
- 成本约束：普通 Python、概念演示、Notebook 阅读和小实验默认在 Mac 本地完成；只有需要项目依赖、GPU、较大模型或正式训练时才启动远程机器。
- 每个新知识点开头标注：行业状态（基础标准 / 常见工程实践 / 前沿或研究性 / 教学简化）、现实用途、学习深度，说明它与企业常见 LLM 工作流的关系和局限。
- Python 语法基础不足：解释项目代码时要先拆开 Python 语法（变量、赋值、切片、函数调用、缩进块等），不假设能直接读懂表达式。
- 记录门槛：用户仍在追问或只说“大概理解”时，不把该知识点写入“已掌握”；先保留为待确认/待巩固。只有用户明确确认理解，或通过简短复述检查后，才记录为已掌握。教学方法和用户偏好本身可立即记录。
- 易混淆知识的记录方式：答疑完成并确认掌握后，优先用简单图形、表格、空间关系、对齐图或流程图作为学习日志中的记忆锚点；大段文字只保留必要的白话解释，避免阅读疲劳和后续遗忘。对于人为约定的机制，记录时要明确标注“这是工程设计规则”，并与模型通过训练学到的内容分开。
- 对照 MokioMind 真实代码时：必须指出具体文件和行号，并提供可点击的绝对路径链接；先解释简化实验，再说明它在项目训练循环中的对应位置。
- 真实代码教学展示方式：先给最小实验建立概念，再截取对应的 5-15 行真实代码，逐行解释变量、条件和调用；跨文件时先给伪代码流程，再分别链接入口和关键实现。不要要求先通读长文件。
- 用户疑问记录规则：不仅记录已掌握结论，也记录学习过程中出现的关键疑问、误解和澄清；疑问解决后用“问题 -> 直观解释 -> 代码对应”保存，避免复习时重复卡住。包括 `torch.optim.SGD([w], lr=0.1)` 的含义、`optimizer.step()` 与 `zero_grad()` 的区别等。
- 新术语教学规则：首次引入任何缩写、特殊数值或 API 行为时，必须先给白话定义、产生原因、对当前流程的影响，再使用术语；不能只把术语放进验收题或结论。

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
- 已理解 next-token prediction：训练样本中的完整句子同时提供输入和标准答案；计算 loss 时错开一位，把第 `t` 个位置的预测与第 `t+1` 个已知 token 对比。训练推动正确 token 相对于其他候选的概率提高。推理时没有现成的后续答案，模型会逐 token 生成并把自己的结果继续作为输入。

### 已掌握的理解（截至 2026-08-31）

- 能用白话解释 token、token id、`input_ids`、BOS、EOS、PAD，以及 tokenizer 将文本转换为数字序列的作用。
- 能解释 padding：为了让同一批样本长度一致而补 PAD；`attention_mask=0` 的位置不应被注意力使用；labels 中的 PAD 改为 `-100`，不参与 loss。
- 能解释 logits 是每个位置对词表候选的原始分数，loss 衡量预测与标准答案的差距，梯度用于调整参数。
- 能解释 next-token prediction：完整句子同时作为输入来源和标准答案来源，计算 loss 时整体错开一位；训练时答案已知，推理时模型逐 token 生成并使用自己的输出。
- 能通过图示理解 `shift_logits` 与 `shift_labels` 的对齐目的：保留可预测位置与对应的下一个 token。
- 已用三维示意理解张量形状：`torch.zeros((2, 4, 10))` 可看作 2 张表，每张表 4 行、每行 10 个数字；通用读法是 `(批量中的样本数, 每条样本的位置数, 每个位置的候选数)`。因此模型 logits 的 `(2, 12, 128)` 可读为 2 条文本、每条 12 个位置、每个位置对 128 个候选 token 打分。这里的“张量”先按可批量计算的数字容器理解。

#### 三维张量的空间记忆锚点

把 `(2, 4, 10)` 想成沿“深度”叠放的两张数字表：

```text
                         横向：10 个候选 token 的分数
                    ┌──────────────────────────────┐
                    │ 0  0  0  0  0  0  0  0  0  0 │  ← 位置 0
                    │ 0  0  0  0  0  0  0  0  0  0 │  ← 位置 1
                    │ 0  0  0  0  0  0  0  0  0  0 │  ← 位置 2
                    │ 0  0  0  0  0  0  0  0  0  0 │  ← 位置 3
                    └──────────────────────────────┘
                         ↑ 竖向：4 个 token 位置

              深度方向：第 0 条文本 ───────→ 第 1 条文本

记忆口诀：向后翻页看“第几条文本”，向下看“第几个位置”，向右看“这个位置的候选分数”。
```

因此模型 logits 的 `(2, 12, 128)` 仍按同一空间关系理解：两张表、每张 12 行、每行 128 个候选分数。

### 仍需巩固

- Python 基础语法：列表、切片（`[:-1]`、`[1:]`）、循环、函数调用和变量赋值。
- PyTorch 张量的维度、索引写法和 `clone()`；目前先理解用途，不要求记忆内部实现细节。
- `PretrainDataset.__getitem__` 如何把真实 JSON 文本加工成三项返回值。
- 已能说明 PAD 的三个必要处理：`input_ids` 用 PAD 补齐 batch 内长度；labels 中的 PAD 改为 `-100`，使其不参与 loss；`attention_mask` 中的 PAD 改为 `0`，使注意力机制忽略这些占位位置。能够区分 `-100`（控制是否计入 loss）与 `0`（控制注意力是否关注）。
- 已能按步骤解释 `PretrainDataset.__getitem__` 的核心代码：tokenizer 将文本转换为 token id；加 BOS/EOS 并用 PAD 补齐得到 `input_ids`；复制 `input_ids` 得到独立的 `labels`，后续可将 PAD 改为 `-100` 而不影响输入。
- 已澄清 `__getitem__(index)` 的返回含义：`index` 只选择一条样本；`return input_ids, labels, attention_mask` 返回的是这条样本的三个字段，不是三条样本。后续 `DataLoader` 才会把多条样本分别堆叠成 batch。
- 能判断 batch 中模型输出 logits 的形状；本次需继续区分输入与输出：`batch_size=4`、序列长度 12 时，`input_ids` 是 `(4, 12)`，词表大小 128 的 `logits` 才是 `(4, 12, 128)`。
- 已能用形状解释输入与输出的区别：`input_ids` 是 `(4, 12)` 的 token 编号表；`logits` 是 `(4, 12, 128)`，因为每个位置都包含 128 个候选 token 的分数。`labels` 与 `attention_mask` 通常和 `input_ids` 形状相同。
- 已理解 `vocab_size` 会决定 logits 的最后一维：每个位置对词表中的每个 token 输出一个分数。但需牢记它必须与 tokenizer 词表匹配；变大意味着候选集合更大，不等于模型必然更聪明，还会增加输出层计算和参数成本。
- 已澄清词表大小关系：通常 `model.vocab_size == len(tokenizer)`，而不是简单要求小于或等于。若模型词表小于 tokenizer，较大的 token id 会导致 embedding/索引越界；若模型词表大于 tokenizer，通常可运行但多出的候选没有对应 token，属于浪费并可能造成配置不一致。
- 已复习 shift 对齐：`shift_logits = logits[:, :-1, :]` 去掉最后一个“预测位置”（它要预测序列结束后的内容，故没有答案）；`shift_labels = labels[:, 1:]` 去掉 BOS。对齐后，模型在输入位置 `t` 输出的 logits 与真实的下一个 token `labels[t+1]` 比较。注意 logits 是该位置的一整排候选分数，不是 token 本身。
- 需特别巩固候选编号与分数位置的对应：`label=3` 是候选编号，不是分数；在 Python 从 0 开始编号的列表 `[0.1, 0.2, 0.3, 2.5, 0.4]` 中，编号 3 对应第 4 个元素 `2.5`。交叉熵据此从整排 logits 中取出正确候选的分数进行比较。
- 已澄清：`labels` 中的值本来就是 tokenizer 的 token id，不是另一套额外编号；由于 token id 被设计为 `0` 到 `vocab_size-1` 的整数，它同时也是 logits 候选维度的数组下标。例如 token id 3 代表某个具体 token，loss 会读取 logits 的第 3 列。`-100` 是特殊忽略标记，不是 token id。
- 已掌握 logits 与 label 的精确配对：`logits[样本][位置]` 是该位置的一整排候选分数，`labels[样本][位置]` 是由 tokenizer 给出的正确下一个 token id；该 id 同时定位 logits 的对应列。交叉熵不只判断正确项是否最高，而是按正确 token 的相对概率连续计分；训练通过降低 loss 推动正确 token 的分数相对升高，通常使其成为最高分。
- 已进一步澄清接口规则：token id 被人为设计为 `0..vocab_size-1` 的整数，并约定 logits 最后一维按同样顺序排列候选分数。因此若正确 token id 是 5566，loss 就读取 `shift_logits[样本, 位置, 5566]`；这是 tokenizer、模型输出层和 loss 之间预先约定的映射规则。训练真正学到的是给定上下文时各候选分数的相对大小，不是学习这个下标规则。
- 交叉熵的“label 查找正确分数 -> softmax 得到概率 -> `-log(正确概率)`”目前仅为大概理解，尚未确认掌握；后续需要用一个更小的数值例子复核后再升级为已掌握。
- 用户已确认掌握交叉熵前半段：`label` 是正确 token id，用来定位 logits 的对应列；logits 是一排候选分数；softmax 将整排分数转为概率；再对正确答案概率计算 `-log(x)` 得到该位置的 loss。模型如何通过梯度让正确概率上升仍待学习。
- 已理解交叉熵的基本计算链：`label` 是正确 token id，用作查表下标；先从整排 logits 取出该下标对应的分数，再对整排 logits 做 softmax 得到概率，最后计算 `loss = -log(正确 token 的概率)`。token id 与分数不是直接相减的两个量；id 只负责定位，分数/概率才参与误差计算。已用表格记忆锚点巩固这一点。
- 已进一步澄清接口规则：token id 被人为设计为 `0..vocab_size-1` 的整数，并约定 logits 最后一维按同样顺序排列候选分数。因此若正确 token id 是 5566，loss 就读取 `shift_logits[样本, 位置, 5566]`；这是 tokenizer、模型输出层和 loss 之间预先约定的映射规则。训练真正学到的是给定上下文时各候选分数的相对大小，不是学习这个下标规则。
- 已进一步澄清接口规则：token id 被人为设计为 `0..vocab_size-1` 的整数，并约定 logits 最后一维按同样顺序排列候选分数。因此若正确 token id 是 5566，loss 就读取 `shift_logits[样本, 位置, 5566]`；这是 tokenizer、模型输出层和 loss 之间预先约定的映射规则。训练真正学到的是给定上下文时各候选分数的相对大小，不是学习这个下标规则。
- 已理解单条样本的结构：`(input_ids, labels, attention_mask)` 是一个包含三个字段的元组；示例中每个 `torch.tensor([..])` 都是一维、长度为 3 的张量，单条样本三个字段的形状都是 `(3,)`。`torch.tensor([...])` 会按列表中的实际数字创建内容；全 0 张量应使用 `torch.zeros(...)`，不能把未填写值理解成自动全 0。
- 已能说明 `torch.tensor([0, 10, 11])`：先有普通 Python 列表 `[0, 10, 11]`，再转换为 PyTorch 张量；内容仍是这 3 个数字，形状为 `(3,)`，转换的目的是使用 PyTorch 的批量计算和模型接口。

### 本次已确认掌握（2026-09-09）

- 能完整追踪一条真实样本 `你好，世界`：项目 tokenizer 产生 `[5134, 270, 2219]`；加入 `BOS=1`、`EOS=2` 并补 `PAD=0`，形成 `input_ids`；复制为 `labels` 并将 PAD 改为 `-100`；生成 `attention_mask`。
- 能解释 next-token 对齐：模型看到 `BOS` 预测 `你好`，看到 `BOS 你好` 预测 `，`，依次类推；`shift_logits = logits[:, :-1, :]` 与 `shift_labels = labels[:, 1:]` 让每个预测位置和已知的下一个 token 对齐。
- 能区分真实模型输出与教学示例：项目 `vocab_size=6400`，每个位置的 logits 是 6400 个原始分数；示例中只放大其中几项，不能把示例分数当成仓库权重的实测结果。
- 能完整解释单个位置的 loss：模型产生整排 logits；softmax 对整排分数做 `exp(logit) / 所有 exp(logit) 之和` 得到概率；`label` 是 tokenizer 的 token id，同时作为列号取出正确 token 的概率；`loss = -log(正确概率)`。token id 只负责定位，不参与数值乘法。
- 能区分推理和训练：推理通常用 `argmax` 选择概率最高的 token 作为输出；训练不必先选一个 token，而是用 label 计算 loss，再通过梯度更新模型参数，使正确 token 的概率总体提高。
- 已使用记忆图 `complete-next-token-loss.html` 串起“原文 → 编号 → 对齐 → logits → softmax → loss”的空间关系。
- 已记住交叉熵对 logits 的梯度结论：`gradient = p - y`，即“模型概率向量减去正确答案的独热目标向量”。正确 token 的分量通常为负，错误候选的分量通常为正，因此梯度下降会提高正确 logit、降低错误 logits。这里的 `p - y` 是 loss 对 logits 的梯度，不是直接存储的模型参数梯度；其完整高数推导仍待巩固。

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
- **optimizer / 优化器**：掌管“怎么根据梯度改参数”的对象。它保存一份参数清单（`optimizer.param_groups`），拿着梯度和学习率算出每个参数改多少，在 `step()` 时执行修改。梯度只表示坡度，真正改参数的是优化器；本项目用的是 AdamW。

## 下次学习

继续学习真实训练循环中的梯度累积：为什么项目不会每个小批次都执行 `optimizer.step()`，以及 `accumulation_steps` 如何影响更新时机；仍使用本地极小实验，不启动远程 GPU。

### 本次已确认掌握（2026-09-28）

- 能区分训练循环的三个动作：`backward()` 根据当前 loss 计算梯度并写入参数的 `.grad`；`optimizer.step()` 读取这些梯度并修改模型参数；`zero_grad()` 清空优化器管理的参数梯度。
- 理解 PyTorch 默认会累积梯度：如果第二次 `backward()` 前不清空，第二轮梯度会叠加到第一轮，影响下一次参数更新。
- 能手算单参数更新：`w=1.0`、`w.grad=-4`、`lr=0.1` 时，`w_new = 1.0 - 0.1 × (-4) = 1.4`。
- 能区分参数与输出：`optimizer.step()` 修改的是模型内部参数，不是已经算出来的 `logits`；下一次前向传播会用新参数重新生成 logits。

### 梯度累积：本次已确认掌握（2026-09-28）

- `accumulation_steps=2` 时，第 1 个小批次只执行 `backward()`，不会更新参数；第 2 个小批次的梯度累积完成后，才执行一次参数更新。
- 两次 `backward()` 之间不能 `zero_grad()`，因为 PyTorch 默认会把新梯度加到旧梯度上；中间清空就无法累积两个小批次。
- 每个小批次的 `loss` 除以 `accumulation_steps`，使累积后的梯度近似两个小批次梯度的平均值。不除时，梯度通常约为 2 倍，等价于把有效学习率放大，可能导致更新过猛。
- 训练代码中的精确对应关系：
  - [trainer/train_pretrain.py:63](/Users/mrwong/Documents/Program/AI/Minimind/eg/MokioMind-master/trainer/train_pretrain.py:63)：`loss = loss / args.accumulation_steps`
  - [trainer/train_pretrain.py:65](/Users/mrwong/Documents/Program/AI/Minimind/eg/MokioMind-master/trainer/train_pretrain.py:65)：`backward()` 写入梯度，不改参数
  - [trainer/train_pretrain.py:67](/Users/mrwong/Documents/Program/AI/Minimind/eg/MokioMind-master/trainer/train_pretrain.py:67)：累积到指定次数才进入更新分支
  - [trainer/train_pretrain.py:76](/Users/mrwong/Documents/Program/AI/Minimind/eg/MokioMind-master/trainer/train_pretrain.py:76)：`scaler.step(optimizer)` 执行真实参数更新（混合精度包装下的 `optimizer.step()`）
  - [trainer/train_pretrain.py:79](/Users/mrwong/Documents/Program/AI/Minimind/eg/MokioMind-master/trainer/train_pretrain.py:79)：更新后清空梯度
- 第 5 课验收：`step=3` 时余数为 1，所以只做 `backward()`；5 个小批次按当前条件更新 2 次（第 2、4 批），第 5 批的梯度会留在参数的 `.grad` 中，但不会由当前 epoch 末尾自动触发 `step()`。
- 这个取余判断的作用是控制更新时机，不改变小批次形状；它把“每批更新”变成“累积若干批后更新一次”。
- 真实代码的 epoch 边界位于 [trainer/train_pretrain.py:332](/Users/mrwong/Documents/Program/AI/Minimind/eg/MokioMind-master/trainer/train_pretrain.py:332)。当前循环在 [trainer/train_pretrain.py:67](/Users/mrwong/Documents/Program/AI/Minimind/eg/MokioMind-master/trainer/train_pretrain.py:67) 只按整组触发更新，因此最后不足一组时不会自动补一次 `step()`；这是当前实现的边界行为，暂不修改。

#### 梯度累积记忆锚点

```text
小批次 A: loss/2 -> backward() ─┐
                                ├─ 梯度相加 -> step() -> 参数更新一次
小批次 B: loss/2 -> backward() ─┘                    -> zero_grad()
```

```text
没有除以 2：     gA + gB       -> 更新约 2 倍大
除以 2：         gA/2 + gB/2   -> 等于两个小批次梯度的平均
```

#### 三动作记忆锚点

```text
当前参数 w
    │
    ├─ backward()       算当前 loss 的坡度，写入 w.grad；w 不变
    │
    ├─ optimizer.step() 按坡度更新 w；真正改变参数
    │
    └─ zero_grad()      清掉旧 w.grad；为下一轮准备
```

```text
参数：      w = 1.0  ──step()──>  w = 1.4
坡度便条：  grad     ──step()──>  被读取
旧便条：    grad     ──zero_grad()──> 清空
```

### 梯度裁剪：当前答疑澄清（2026-09-29）

- 用户发现最初 Notebook 只展示 `-8 -> -1`，没有解释转换过程；正确记忆是：当梯度范数 `8` 超过阈值 `1` 时，统一乘以正比例 `1/8`，所以 `-8 × (1/8) = -1`。正比例只改变大小，不改变正负方向。
- `clip_grad_norm_` 修改的是参数的 `.grad`，不修改参数本身；参数要等 `optimizer.step()` 或 `scaler.step(optimizer)` 才改变。
- `optimizer.zero_grad()` 只清空梯度，不更新参数。项目的 `scaler.step(optimizer)` 是混合精度训练对 `optimizer.step()` 的包装。
- `torch.optim.SGD([w], lr=0.1)` 表示创建一个 SGD 优化器，让它管理参数 `w`，学习率是 `0.1`；它本身不立即改变 `w`，只有执行 `step()` 才更新。
- 浮点误差是计算机数值计算的通用现象，不是机器学习独有；普通模型训练通常接受极小误差，而金额等严格场景会改用整数分、十进制定点或高精度数值。
- L2 范数是梯度整体长度：单个梯度 `[-8]` 的范数是 `|-8|=8`；多个梯度 `[3,4]` 的范数是 `sqrt(3²+4²)=5`；四个梯度 `[1,2,2,4]` 的范数是 `sqrt(1²+2²+2²+4²)=5`。
- 第 7 课澄清：Notebook 中的 `loss = 1.5 * w1.square() + 2.0 * w2.square()` 是人为设计的教学函数，不是 MokioMind 的真实 loss。因为 `w1=w2=1` 时，导数分别为 `2*1.5*1=3` 和 `2*2.0*1=4`，所以故意构造出梯度 `[3,4]`，便于画二维箭头和演示整体范数。真实模型中的 loss 来自模型输出、labels 和交叉熵等计算，梯度由 `backward()` 自动求出，不会手工指定成 `[3,4]`。
- 第 8 课答疑中确认：`scaler.scale(loss)` 只放大 loss，从而让 backward 得到放大后的梯度，不直接修改参数或 tokenizer；`unscale_` 在裁剪和更新前把梯度还原，否则裁剪会把“人为放大的梯度”当成真实梯度。
- `scaler.step(optimizer)` 与 `optimizer.step()` 都负责使用梯度更新参数；前者额外检查 `inf/nan`，发现本轮梯度溢出就跳过更新，避免坏数值污染模型。
- `scaler.update()` 管理的是下一轮的缩放因子，不是固定常数：没有溢出时可逐步增大，发生溢出时减小；`optimizer.zero_grad()` 仍只负责清空参数梯度。
- `inf` / infinity：表示“无限大”的特殊浮点值，通常来自数值超过当前数据类型能表示的最大范围；它不是正常的有限数字。`nan` / Not a Number：表示“不是一个有效数字”的特殊浮点值，常见于 `0/0`、`inf-inf` 等非法或失控运算。二者都会让梯度或参数失去正常意义，因此 `GradScaler` 检测到它们时会跳过本次参数更新并调小缩放因子。

```text
梯度 [3,4] 的箭头：
       ↑ 4
       │      ● (3,4)
       │     /
       │    /  L2 长度 = sqrt(3²+4²) = 5
       │   /
───────┼────────→ 3

更多参数：仍然把所有分量平方后相加，再开平方；只是箭头进入更高维空间。
```

### 学习率与优化器：本次答疑澄清（2026-09-29）

第 09 课验收题结果：第 1、2、3 题正确；第 4 题落在正确区域但不精确。第 2 题曾被误判为错，原因见下文。

- 教学失误一（用户当场指出，成立）：`优化器` 一词在第 03、04、08 课就已随 `optimizer.step()`、`zero_grad()`、`scaler.step(optimizer)` 出现，第 09 课验收题又直接问“哪一行把学习率写入优化器”，但从未按约定第 31 条先给白话定义。此后每引入一个新 API 名，先给白话定义、产生原因、对当前流程的影响，再放进题目或结论。
- 教学失误二（用户反驳后确认，成立）：第 2 题题干问“一次更新量会怎样变化”，但没有写明基准是“相对于学习率 0.1 时的更新量”。用户答“会增加 0.04”，指参数从 1.0 增加到 1.04，算术正确，却被判错。问“怎样变化”的题必须写明基准值。

#### 优化器（optimizer）是什么

- 白话定义：优化器是掌管“怎么根据梯度改参数”这件事的对象。梯度只说明坡度，不会自己改参数；优化器拿着梯度，按某个规则算出每个参数改多少，再执行修改。
- 为什么要单独有这么一个对象，而不直接写 `w = w - lr * w.grad`：
  - 真实模型有几百万个参数、分散在各层，需要一份参数清单来统一遍历，这份清单就是 `optimizer.param_groups`。
  - 更新规则不止一种；本项目用的是 AdamW。换公式只换优化器，训练循环不用改。
  - AdamW 要为每个参数额外保存状态（动量、方差估计等），这些状态必须与参数一一对应。
  - 学习率是优化器的属性，按参数组存放；要改学习率就改优化器里的值，而不是改一个普通变量。

```text
梯度（坡度便条）
      │
      ▼
  ┌────────────┐  拿着梯度 + 学习率，逐一算出每个参数改多少
  │   优化器    │  AdamW：在梯度基础上加了动量和自适应缩放
  └────────────┘
      │
      ▼
   参数 w 被改写 ──> 下一次前向传播产生新的 logits
```

#### 学习率从哪来、写到哪去（逐行）

| 行号 | 代码 | 作用 |
| --- | --- | --- |
| [train_pretrain.py:312](/Users/mrwong/Documents/Program/AI/Minimind/eg/MokioMind-master/trainer/train_pretrain.py:312) | `optimizer = optim.AdamW(model.parameters(), lr=args.learning_rate)` | 创建优化器，接管全部参数，并写入初始学习率 |
| [train_pretrain.py:48](/Users/mrwong/Documents/Program/AI/Minimind/eg/MokioMind-master/trainer/train_pretrain.py:48) | `lr = get_lr(epoch * iters + step, args.epochs * iters, args.learning_rate)` | 算出“这一步”该用多大学习率 |
| [train_pretrain.py:50](/Users/mrwong/Documents/Program/AI/Minimind/eg/MokioMind-master/trainer/train_pretrain.py:50) | `for param_group in optimizer.param_groups:` | 遍历优化器保存的参数组（本项目只有一组） |
| [train_pretrain.py:51](/Users/mrwong/Documents/Program/AI/Minimind/eg/MokioMind-master/trainer/train_pretrain.py:51) | `param_group["lr"] = lr` | **真正写入**：把新学习率覆盖进优化器的配置 |
| [train_pretrain.py:76](/Users/mrwong/Documents/Program/AI/Minimind/eg/MokioMind-master/trainer/train_pretrain.py:76) | `scaler.step(optimizer)` | 优化器此时才用“当前学习率 × 梯度”真正改参数 |

- `param_groups` 是优化器内部的参数清单，列表每一项是一组参数的配置字典，里面有 `"params"`（这组参数）和 `"lr"` 等设置。因为学习率每步都变，所以每一步都要把新值写回去；写进去之后，优化器在 `step()` 时才读它。
- 第 4 题的精确答案：写入动作是第 51 行的赋值，第 50 行只是它的循环头；两行都点出来才算答全。

#### 第 2 题：绝对量与相对变化（2026-09-29 澄清，原判错已撤回）

更新量 = `-学习率 × 梯度`，梯度固定为 `-4` 时：

| 学习率 | 计算 | 更新量 | 参数变化 |
| --- | --- | --- | --- |
| 0.1 | `-0.1 × (-4)` | `+0.4` | 1.0 → 1.4（参数增加 0.4） |
| 0.01 | `-0.01 × (-4)` | `+0.04` | 1.0 → 1.04（参数增加 0.04） |

- 原答“会增加 0.04”指的是“参数从 1.0 增加到 1.04”，**正确**。误判来自把“更新量”读成了“这一增量相对上一格的变化”，而用户一直在说参数的绝对增量。
- 两种说法答的是不同问题，都成立：
  - 参数这一步增加多少 —— 绝对量，`0.04`。
  - 这一增量相对学习率 0.1 时怎么变 —— 相对量，`0.4 → 0.04`，缩到 `1/10`。
- 正负号由梯度决定，学习率不改变方向，只改变大小；这与第 07 课“正比例只改变大小，不改变正负方向”是同一条规则。
