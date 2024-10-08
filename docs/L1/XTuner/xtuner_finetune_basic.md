# XTuner微调前置基础

## 1 基本概念

在进行微调之前，我们需要了解一些基本概念。

### 1.1 Finetune简介

微调（fine-tuning）是一种基于预训练模型，通过少量的调整（fine-tune）来适应新的任务或数据的方法。

微调是在预训练模型的基础上，将模型中一些层的权重参数进行微调，以适应新的数据集或任务。

预训练模型部分已经在大规模数据上得到了训练，它们通常是较为通用且高性能的模型，因此可以很好地作为新任务的起点。微调可以加快模型的收敛速度，降低模型过拟合的风险，并在不消耗过多计算资源的情况下获取较好的模型性能。

#### 1.1.1 Finetune的两种范式

在大模型的下游应用中，经常会用到两种微调模式：**增量预训练** 和 **指令跟随** 。

1. **增量预训练**

增量预训练是一种在已有预训练模型（比如：InternLM基座模型）的基础上，利用特定领域的数据进行进一步训练的方法。它的目的是在保持模型原有能力的同时，注入新的领域知识，进一步优化现有的预训练模型，从而提升模型在特定领域任务中的表现（比如：InternLM垂类基座模型）。增量预训练模型能够接受少量的新数据进行更新并适应新的任务，而不需要重新训练整个模型，这种方式可以很好地利用现有的预训练模型的知识，并在新数据上获得更好的性能。

2. **指令跟随**

指令跟随是指让模型根据用户输入的指令来执行相应的操作。模型通过对大量自然语言指令和相应操作的数据进行训练，学习如何将指令分解为具体的子任务，并选择合适的模块来执行这些任务（比如：InternLM垂类对话模型）。

### 1.2 微调技术

大多数大型语言模型（LLM）的参数规模巨大，且规模日益增大，导致模型的训练和微调成本高昂，直接训练需要耗费大量计算资源和费用。近年来，如何高效地对大模型进行微调成为了研究热点，而LoRA和QLoRA两种微调技术因其高效性和实用性受到了广泛关注。

#### 1.2.1 LoRA简介

LoRA（Low-Rank Adaptation）是一种使用低精度权重对大型预训练语言模型进行微调的技术，它的核心思想是在不改变原有模型权重的情况下，通过添加少量新参数来进行微调。这种方法降低了模型的存储需求，也降低了计算成本，实现了对大模型的快速适应，同时保持了模型性能。

然而，由于使用了低精度权重，LoRA的一个潜在的缺点是在微调过程中可能会丢失一些原始模型的高阶特征信息，因此可能会降低模型的准确性。

#### 1.2.2 QLoRA简介

QLoRA（Quantized LoRA）微调技术是对LoRA的一种改进，它通过引入高精度权重和可学习的低秩适配器来提高模型的准确性。并且在LoRA的基础上，引入了量化技术。通过将预训练模型量化为int4格式，可以进一步减少微调过程中的计算量，同时也可以减少模型的存储空间，这对于在资源有限的设备上运行模型非常有用。最终，可以使我们在消费级的显卡上进行模型的微调训练。

###  1.3 XTuner简介

XTuner 的官方仓库是：https://github.com/InternLM/xtuner （欢迎Star）！

XTuner 一个大语言模型&多模态模型微调工具箱。*由* *MMRazor* *和* *MMDeploy* *联合开发。*

- 🤓 **傻瓜化：** 以 配置文件 的形式封装了大部分微调场景，**0基础的非专业人员也能一键开始微调**。
- 🍃 **轻量级：** 对于 7B 参数量的LLM，**微调所需的最小显存仅为 8GB** ： **消费级显卡✅，colab✅**

#### 1.3.1 功能亮点

- 适配多种生态
  - 支持多种微调算法
  - 适配多种开源生态（HuggingFace、ModelScope等）
  - 自动优化加速器
- 适配多种硬件

#### 1.3.2 常用命令

以下是一些常用的命令。

- 查看帮助


```bash
xtuner help
```

- 查看版本


```bash
xtuner version
```

- 列出所有预定义配置文件


```bash
xtuner list-cfg
```

- 列出包含指定名称的预定义配置文件

> `xtuner list-cfg` 命令用于列出内置的所有配置文件。参数 `-p` 或 `--pattern` 表示模式匹配，后面跟着的内容将会在所有的配置文件里进行模糊匹配搜索，然后返回最有可能得内容。


```bash
xtuner list-cfg -p $NAME
```

- 复制配置文件

> `xtuner copy-cfg` 命令用于复制一个内置的配置文件。该命令需要两个参数：`CONFIG` 代表需要复制的配置文件名称，`SAVE_PATH` 代表复制的目标路径。


```bash
xtuner copy-cfg $CONFIG $SAVE_PATH
```

- 执行微调训练

> `xtuner train` 命令用于启动模型微调进程。该命令需要一个参数：`CONFIG` 用于指定微调配置文件。


```bash
xtuner train $CONFIG
```

- 将 pth 格式的模型文件转换成 HuggingFace 格式的模型

> `xtuner convert pth_to_hf` 命令用于进行模型格式转换。该命令需要三个参数：`CONFIG` 表示微调的配置文件， `PATH_TO_PTH_MODEL` 表示微调的模型权重文件路径，即要转换的模型权重， `SAVE_PATH_TO_HF_MODEL` 表示转换后的 HuggingFace 格式文件的保存路径。

除此之外，我们其实还可以在转换的命令中添加几个额外的参数，包括：

| 参数名                | 解释                                         |
| --------------------- | -------------------------------------------- |
| --fp32                | 代表以fp32的精度开启，假如不输入则默认为fp16 |
| --max-shard-size {GB} | 代表每个权重文件最大的大小（默认为2GB）      |


```bash
xtuner convert pth_to_hf $CONFIG $PATH_TO_PTH_MODEL $SAVE_PATH_TO_HF_MODEL
```

- 将原始模型与微调结果进行合并

> `xtuner convert merge`命令用于合并模型。该命令需要三个参数：`LLM` 表示原模型路径，`ADAPTER` 表示 Adapter 层的路径， `SAVE_PATH` 表示合并后的模型最终的保存路径。

在模型合并这一步还有其他很多的可选参数，包括：

| 参数名                 | 解释                                                         |
| ---------------------- | ------------------------------------------------------------ |
| --max-shard-size {GB}  | 代表每个权重文件最大的大小（默认为2GB）                      |
| --device {device_name} | 这里指的就是device的名称，可选择的有cuda、cpu和auto，默认为cuda即使用gpu进行运算 |
| --is-clip              | 这个参数主要用于确定模型是不是CLIP模型，假如是的话就要加上，不是就不需要添加 |

> CLIP（Contrastive Language–Image Pre-training）模型是 OpenAI 开发的一种预训练模型，它能够理解图像和描述它们的文本之间的关系。CLIP 通过在大规模数据集上学习图像和对应文本之间的对应关系，从而实现了对图像内容的理解和分类，甚至能够根据文本提示生成图像。


```bash
xtuner convert merge $LLM $ADAPTER $SAVE_PATH
```

## 2 创建开发机

我们需要前往 [InternStudio](https://studio.intern-ai.org.cn/) 中创建一台开发机进行使用。

步骤1：登录InternStudio后，在控制台点击 “创建开发机” 按钮可以进入到开发机的创建界面。

![](https://raw.githubusercontent.com/wux-labs/ImageHosting/main/XTuner/image-01.png)

步骤2：在 “创建开发机” 界面，选择开发机类型：个人开发机，输入开发机名称：XTuner微调，选择开发机镜像：Cuda12.2-conda。

![](https://raw.githubusercontent.com/wux-labs/ImageHosting/main/XTuner/image-02.png)

步骤3：在镜像详情界面，点击 “使用” 链接，确认使用该镜像。

![](https://raw.githubusercontent.com/wux-labs/ImageHosting/main/XTuner/image-03.png)

步骤4：资源配置可以选择 10% （如果有更高资源可以使用，也可以选择更高的资源配置），然后点击 “立即创建” 按钮创建开发机。

![](https://raw.githubusercontent.com/wux-labs/ImageHosting/main/XTuner/image-04.png)

步骤5：创建完成后，在开发机列表中可以看到刚创建的开发机，点击 “进入开发机” 链接可以连接进入到开发机。

![](https://raw.githubusercontent.com/wux-labs/ImageHosting/main/XTuner/image-05.png)

当我们有了这些前置知识和服务器之后，就可以进行下一步的微调任务了。

