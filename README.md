<div align="center">
<h1>
  星辰语义大模型-TeleChat2
</h1>
</div>


<p align="center">
	🤗 <a href="https://huggingface.co/Tele-AI" target="_blank">Hugging Face</a> • 🤖 <a href="https://modelscope.cn/organization/TeleAI" target="_blank">ModelScope</a> • 🏔 <a href="https://gitee.com/mindspore/mindformers/tree/dev/research/telechat" target="_blank">MindSpore</a> • 🐾 <a href="https://gitee.com/Tele-AI/tele-chat" target="_blank">gitee</a>️ • 💬 <a href="https://github.com/Tele-AI/Telechat/blob/master/images/wechat.jpg" target="_blank">WeChat</a>
</p>

# 目录

- [模型介绍](#模型介绍)
- [效果评测](#效果评测)
- [模型推理和部署](#模型推理和部署)
- [模型微调](#模型微调)
- [声明、协议、引用](#声明协议引用)

# 最新动态

- 2024.9.20 开源TeleChat2-115B模型，该模型是**首个完全国产算力训练并开源的千亿参数模型**。

# 模型介绍

### 星辰语义大模型-TeleChat2

- 星辰语义大模型**TeleChat2**是由中国电信人工智能研究院研发训练的大语言模型，该系列模型**完全基于国产算力**训练。
- 本次开源**TeleChat2-115B**模型采用10万亿 Tokens中英文高质量语料进行训练，同步开源对话模型**TeleChat2-115B**的多格式、多平台权重文件。
- **TeleChat2**在训练数据、训练方法等方面进行了改进，在通用问答和知识类、代码类、数学类榜单上相比**TeleChat1**均有大幅提升。
    - **TeleChat2**完全基于国产算力和国产深度学习框架进行训练，算力和算法框架更自主可控。优化MP、PP、SP实现方式提升模型性能，优化算子来提升训练速度。
    - 我们使用大量小模型实验来验证scaling law规律，在不同模型结构、不同数据配比和数据清洗方式中寻找最优设计。
    - 采用RingAttention及其他序列切分方式，实现长文训练性能提升；通过ntk-aware+attention-scaling的方式保证训练长度切换时的平稳过渡，以此来保证模型在不同长度数据下的训练效果。
- 在微调数据方面，我们进行了指令复杂性提升与多样性扩充，通过数据合成和人工标注生成高质量数据，并使用拒绝采样生成多样的推理路径；通过研究一套基于base模型反向选择偏好对齐数据方案，基于适配数据最大限度提升模型效果。
    - 通用能力较TeleChat系列模型提升超过29%，在逻辑推理、总结摘要、长文写作和数学计算上均有大幅提升。

### 模型结构

我们采用标准的 `Decoder-only` 结构设计了 **TeleChat2** 模型，使用 [Rotary Embedding](https://arxiv.org/pdf/2104.09864.pdf)
的位置编码方法、使用 [SwiGLU](https://arxiv.org/pdf/2002.05202.pdf)
激活函数来替代GELU激活函数、使用基于 [RMSNorm](https://arxiv.org/abs/1910.07467) 的 Pre-Normalization进行层标准化操作。我们将**TeleChat2**的词嵌入层和输出lm
head层参数分开，有助于增强训练稳定性和收敛性。我们选择了GQA以节约attention部分的参数量和计算量、提升训练和推理速度。

**TeleChat2**的模型结构配置如下表所示：

|      | layer_num | hidden_size | ffn_hidden_size | head_num | tie_word_embeddings | GQA  |
| ---- | --------- | ----------- | --------------- | -------- | ------------------- | ---- |
| 115B | 96        | 8192        | 40960           | 64       | 否                  | 是   |

---

	我们开源的TeleChat模型：

- 支持deepspeed微调，开源了基于deepspeed的训练代码，支持Zero并行显存优化，同时集成了FlashAttention2
- 多轮能力支持。开源了多轮数据构建方式，针对多轮模型训练集成了针对多轮的mask loss训练方式，更好的聚焦多轮答案，提升问答效果。

本次发布版本和下载链接见下表

| 模型版本       | 下载链接 |
| -------------- | -------- |
| telechat2-115B-FP32 |   [modelscope](https://modelscope.cn/models/TeleAI/TeleChat2-115B)|

# 效果评测

TeleChat模型相比同规模模型在评测效果方面也有较好的表现，我们的评测集涵盖了包括MMLU、C-Eval、CMMLU、
GSM8K、MATH、HumanEval、BBH等数据集，评测能力包括了指令遵循、考试能力、数学计算和推理、代码生成等

## 评测集介绍

### 通用能力

- MMLU 数据集是一个全面的英文评测数据集，涵盖了 57 个学科，包括人文学科、社会科学、自然科学、初等数学、美国历史、计算机科学、法律等等。

- CEVAL 数据集是一个全面的中文评估测试集，包括初中、高中、大学和专业难度级别的多项选择题，涵盖了 52 个不同的学科领域。

- CMMLU 数据集同样是一个全面的中文评估测试集，涵盖了从基础学科到高级专业水平的67个主题。

### 推理和代码能力

- GSM8K 数据集包含了8.5K高质量的小学数学题，能够评估语言模型在数学推理能力上的表现。

- HumanEval 数据集是一个由openai提供的代码能力测试数据集，它由 164 个编程问题组成，要求根据给定的问题和代码模板，生成正确的代码片段。

	- BBH 数据集全名为BIG-Bench Hard（BBH），包含23个具有挑战性的BIG-Bench任务，均为之前的语言模型评估中没有超过平均人类评审者表现的任务。

- MBPP 数据集包含大约1000个众包的Python编程问题，涵盖编程基础知识、标准库功能等。每个问题包括任务描述、代码解决方案和3个自动化测试用例。

### 主观题能力

- [AlignBench](https://github.com/THUDM/AlignBench)是一个多维度全面评估中文大模型对齐水平的评测基准，包含638道单轮主观评测题。

- [MT-bench](https://github.com/lm-sys/FastChat/blob/main/fastchat/llm_judge/README.md)是一个用于评估聊天助手的具有挑战性的多轮开放式问题集，包含80通多轮主观评测题。

### 指令遵循能力

- [IFEval](https://github.com/EleutherAI/lm-evaluation-harness/blob/main/lm_eval/tasks/ifeval/README.md)旨在评估语言模型对指令的精确遵循能力，它包含了500条可精确验证的指令，是Open
  LLM Leaderboard中使用的核心基准测试之一。

## 评测结果如下

| Dataset    | Llama-3.1-70B | Qwen1.5-110B | Qwen2-72-instruct | DeepSeek-v2 | TeleChat2-115B |
|:----------:|:-------------:|:------------:|:-----------------:|:-----------:|:--------------:|
| C-Eval     | -             | -            | 83.8              | 78          | **86.9**       |
| MMLU       | **86**        | 80.4         | 82.3              | 77.8        | 80.9           |
| CMMLU      | 69.01         | 87.64        | 87.47             | 81.6        | **89.94**      |
| GSM8K      | **95.1**      | 85.4         | 91.1              | 92.2        | 92.2           |
| HumanEval  | 80.5          | 52.4         |**86**               | 81.1        | 75             |
| BBH        | -             | 74.8         | -                 | 79.7        | **89.04**      |
| MBPP       | **86**        | 58.1         | 80.2              | 72          | 78             |
| AlignBench | -             | 7.86         | **8.27**          | 7.91        | 8.03           |
| MT-bench   | 8.79          | 8.88         | **9.12**          | 8.97        | 8.89           |
| IFEval     | **87.5**      | -            | 77.6              | 63.8        | 82.81          |

# 模型推理和部署

### 模型推理

当前模型推理兼容了单卡和多卡推理，以及针对长文推理做了部分优化工作。

**模型推理方法示范**

```python
>> > import os
>> > import torch
>> > from transformers import AutoModelForCausalLM, AutoTokenizer, GenerationConfig
>> > tokenizer = AutoTokenizer.from_pretrained('../models/115B', trust_remote_code=True)
>> > model = AutoModelForCausalLM.from_pretrained('../models/115B', trust_remote_code=True, device_map="auto",
                                                  torch_dtype=torch.float16)
>> > generate_config = GenerationConfig.from_pretrained('../models/115B')
>> > question = "生抽与老抽的区别？"
>> > answer, history = model.chat(tokenizer=tokenizer, question=question, history=[], generation_config=generate_config,
                                  stream=False)
>> > print(answer)
生抽和老抽是两种不同的酱油，它们在风味、色泽和用途上都有所区别。

1.颜色：生抽的颜色比较淡，而老抽的颜色较深。生抽的颜色呈红褐色或棕红色，而老抽的颜色则呈棕黑色。

2.味道：生抽具有鲜美的咸味和微甜的味浅，而老抽浓郁，颜色较深。根据个人口味和烹饪需求选择不同的酱油类型可以获得更好的口感和菜肴效果。
```

### 模型部署

TeleChat目前提供了API、Web两种部署方式。目前仅提供简单的单卡单并发场景，用于演示和效果测试。

API: 分为流式接口和json接口，支持传入推理参数

Web: 支持流式生成、多轮对话

[现已支持vllm推理](./vllm_inf)

# 模型微调

## 数据处理

为了方便数据配比，解耦了数据处理和模型训练，数据权重配比文件如**data.json**所示，json字典中key为读取数据的路径，value为训练时数据的权重。单轮、多轮数据格式如样例数据所示

```shell
{
  "datas/single_turn_example.jsonl": 2.0,
  "datas/multi_turn_example.jsonl": 1.0
}
```

运行**process_data.py**即可将文件处理成tokens，并保存。其中**data_output_path/train_data_{i}.pt**保存处理后的文件，**i的范围是0~num_workers**
。训练时会加载路径下所有**train_data_{i}.pt**文件

* 数据通过**data_path**读取，最终拼接生成**num_samples**个**max_seq_len**长度的sample进行训练。如样例所示，假设**datas/single_turn_example.jsonl**和**
  datas/multi_turn_example.jsonl**
	  各有1000条samples，配比过后数据池中则总共包含3000条samples。在数据拼接过程中，程序会不断遍历数据池，尽可能将数据拼接到4096长度（不够就左padding），直至生成到num_samples的个数。因此，每个sample中会包含多条拼接而成的数据。
* process_method选择**single**或**multiple**单进程或多进程处理数据。

```python
python3 -u process_data.py \
   --data_path data.json \ # 数据配比文件路径
   --tokenizer_path ../../models/12B \ # 模型/tokenzier路径
   --data_output_path $DATA_OUTPUT_PATH \ # 处理后数据保存地址
   --max_seq_len $MAX_LEN \ # 数据长度
   --num_samples $NUM_SAMPLES \ # 最终生成拼接后的数据数量
   --num_workers 10 \ # 多进程个数
   --process_method multiple \ # 多进程&单进程处理
   --seed 42
```

## 全参训练

```shell
deepspeed --master_port 29500 --hostfile=my_hostfile main.py \
   --data_path $DATA_OUTPUT_PATH \ # tokenzie后的数据文件存放地址
   --model_name_or_path $model_path \
   --with_loss_mask \
   --per_device_train_batch_size 1 \
   --max_seq_len 4096 \
   --learning_rate 3e-5 \
   --weight_decay 0.0001 \
   --num_train_epochs 1 \
   --gradient_accumulation_steps 4 \
   --lr_scheduler_type cosine \
   --precision fp16 \ # 训练精度，fp16或bf16
   --warmup_proportion 0.1 \ 
   --gradient_checkpointing \
   --offload \
   --seed 1233 \
   --zero_stage $ZERO_STAGE \ 
   --save_steps 10 \
   --deepspeed \ 
   --output_dir $OUTPUT # 输出路径 
```

# 国产化适配

### 昇腾Atlas 800T A2训练服务器+昇思MindSpore框架:  训练、推理适配

当前星辰语义大模型TeleChat2支持昇腾Atlas 800T A2训练服务器，可基于昇思MindSpore框架进行模型训练和评测。

- 性能方面，具体对比如下：
-
| NAME                 | performance(samples/p/s) | Epochs | AMP_Type |
  |--------------------------| ---------------------: | ------ | -------: |
| 115B  |  0.0192            | 1      |        O1 |
| 115B           | 0.0174                | 1      |       O2 |

	说明：建议采用8台一组进行训练

- TeleChat支持昇腾Atlas 800T A2训练服务器，可基于昇思MindSpore框架进行模型训练，训练所需的modeling、README、
  脚本已发布：[TeleChat-MindSpore](https://gitee.com/mindspore/mindformers/tree/dev/research/telechat)

# 声明、协议、引用

### 声明

我们在此声明，不要使用TeleChat模型及其衍生模型进行任何危害国家社会安全或违法的活动。同时，我们也要求使用者不要将TeleChat模型用于没有安全审查和备案的互联网服务。我们希望所有使用者遵守上述原则，确保科技发展在合法合规的环境下进行。

我们已经尽我们所能，来确保模型训练过程中使用的数据的合规性。然而，尽管我们已经做出了巨大的努力，但由于模型和数据的复杂性，仍有可能存在一些无法预见的问题。因此，如果由于使用TeleChat开源模型而导致的任何问题，包括但不限于数据安全问题、公共舆论风险，或模型被误导、滥用、传播或不当利用所带来的任何风险和问题，我们将不承担任何责任。

### 协议

社区使用 TeleChat 模型需要遵循《[TeleChat模型社区许可协议](./TeleChat模型社区许可协议.pdf)》。TeleChat模型支持商业用途，如果您计划将 TeleChat
模型或其衍生品用于商业目的，您需要通过以下联系邮箱
tele_ai@chinatelecom.cn，提交《TeleChat模型社区许可协议》要求的申请材料。审核通过后，将特此授予您一个非排他性、全球性、不可转让、不可再许可、可撤销的商用版权许可。

### 引用

如需引用我们的工作，请使用如下 reference:

```
@misc{wang2024telechat,
      title={TeleChat Technical Report}, 
      author={Zihan Wang and Xinzhang Liu and Shixuan Liu and Yitong Yao and Yuyao Huang and Zhongjiang He and Xuelong Li and Yongxiang Li and Zhonghao Che and Zhaoxi Zhang and Yan Wang and Xin Wang and Luwen Pu and Huihan Xu and Ruiyu Fang and Yu Zhao and Jie Zhang and Xiaomeng Huang and Zhilong Lu and Jiaxin Peng and Wenjun Zheng and Shiquan Wang and Bingkai Yang and Xuewei he and Zhuoru Jiang and Qiyi Xie and Yanhan Zhang and Zhongqiu Li and Lingling Shi and Weiwei Fu and Yin Zhang and Zilu Huang and Sishi Xiong and Yuxiang Zhang and Chao Wang and Shuangyong Song},
      year={2024},
      eprint={2401.03804},
      archivePrefix={arXiv},
      primaryClass={cs.CL}
}
```