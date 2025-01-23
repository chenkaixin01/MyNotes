+# 初始化环境

```
conda create -n onnx python==3.10
conda activate onnx
set HF_ENDPOINT=https://hf-mirror.com
pip install optimum[exporters-tf]
pip install optimum[onnxruntime]
```

# 1 通过optimum-cli转换onnx

[Export a model to ONNX with optimum.exporters.onnx](https://hf-mirror.com/docs/optimum/main/en/exporters/onnx/usage_guides/export_a_model#exporting-a-model-to-onnx-using-the-cli)

```
optimum-cli export onnx --model infgrad/stella-large-zh-v2 --task feature-extraction infgrad/stella-large-zh-v2-onnx/# python代码转换onnx
```

# 2 通过代码转换

## 2.1 确认模型输入参数

```
import os

os.environ["HF_ENDPOINT"] = "https://hf-mirror.com"  # 设置为hf的国内镜像网站
from transformers import AutoTokenizer
tokenizer = AutoTokenizer.from_pretrained(r"D:\code\transfomers\stella")
inputs = tokenizer("This framework generates embeddings for each input sentence", return_tensors="pt")
print(inputs)
```

## 2.2 自定义模型包装，将onnx的输入输出包装成我们想要的

```
class MyEmbeddingModel(torch.nn.Module):
    def __init__(self, pretrained_model_name):
        super(MyEmbeddingModel, self).__init__()
        # self.model = AutoModel.from_pretrained(pretrained_model_name)
        self.model = SentenceTransformer(pretrained_model_name)
        self.fts = self.model[0]
        self.pls = self.model[1]


    # 定义onnx的模型入参
    def forward(self, input_ids, token_type_ids, attention_mask):
        # inputs 模型实际的入参
        inputs = {
            "input_ids":input_ids,
            "token_type_ids": token_type_ids,
            "attention_mask":attention_mask,
        }
        # outputs = self.pls(self.fts(inputs))
        outputs = self.model(inputs)
        # 从模型的实际输出中取出目标输出
        return outputs['sentence_embedding']
```

```
import torch
import os
from sentence_transformers import SentenceTransformer

os.environ["HF_ENDPOINT"] = "https://hf-mirror.com"  # 设置为hf的国内镜像网站


torch.set_printoptions(precision=32)
class MyEmbeddingModel(torch.nn.Module):
    def __init__(self, pretrained_model_name):
        super(MyEmbeddingModel, self).__init__()
        # self.model = AutoModel.from_pretrained(pretrained_model_name)
        self.model = SentenceTransformer(pretrained_model_name)
        self.fts = self.model[0]
        self.pls = self.model[1]


    # 定义onnx的模型入参
    def forward(self, input_ids, token_type_ids, attention_mask):
        # inputs 模型实际的入参
        inputs = {
            "input_ids":input_ids,
            "token_type_ids": token_type_ids,
            "attention_mask":attention_mask,
        }
        # outputs = self.pls(self.fts(inputs))
        outputs = self.model(inputs)
        # 从模型的实际输出中取出目标输出
        return outputs['sentence_embedding']

onnx_path = r'D:\code\transfomers\stella-large-zh-v2\model.onnx'
model = MyEmbeddingModel(r"D:\code\transfomers\stella")

model.eval()  # 设置模型为评估模式#准备输入数据
input_ids = torch.ones([1, 2], dtype=torch.int32)
token_type_ids = torch.ones([1, 2], dtype=torch.int32)
attention_mask = torch.ones([1, 2], dtype=torch.int32)


# 导出为ONNX模型
torch.onnx.export(
    model,  # 被导出的模型
    (input_ids, token_type_ids, attention_mask),  # 示例输入
    f=onnx_path,  # 导出文件的路径
    opset_version=18,  # 0NNX操作集版本
    verbose=True,
    input_names=["input_ids", "token_type_ids", "attention_mask"],  # 输入名
    output_names=["final_encodes:0"],  # 输出名
    dynamic_axes={"input_ids": [0, 1],
                  "token_type_ids": [0, 1],
                  "attention_mask": [0, 1],
                  "final_encodes:0": [0]}, # 动态轴设置
)
```

# 3 模型验证

```
from transformers import AutoTokenizer
from optimum.onnxruntime import ORTModelForCustomTasks
from sentence_transformers import SentenceTransformer
import numpy as np

# 原始模型
model = SentenceTransformer(r"D:\code\transfomers\stella")
# Our sentences we like to encode
sentences = [
    "This framework generates embeddings for each input sentence",
]

# Sentences are encoded by calling model.encode()
sentence_embeddings = model.encode(sentences)

tokenizer = AutoTokenizer.from_pretrained(R"D:\code\transfomers\stella-large-zh-v2") # 使用 Pytorch 模型的字典
onnxModel = ORTModelForCustomTasks.from_pretrained(R"D:\code\transfomers\stella-large-zh-v2")

inputs = tokenizer("This framework generates embeddings for each input sentence", return_tensors="pt")

outputs = onnxModel(**inputs)
# print(outputs)
sentence_embedding = outputs["final_encodes:0"]
# print("Embedding:", sentence_embedding)
embedding = sentence_embedding.cpu().numpy()
# print("Embedding:", embedding)

for x,y in zip(sentence_embedding.cpu().numpy()[0],embedding[0]):
    print(x)
    print(y)
    print('--------------------------------')
diffList = [abs(x - y) for x,y in zip(sentence_embedding.cpu().numpy()[0],embedding[0])]

sum_diff = sum(diffList)

print(sum_diff)
```

# 4 模型参数调整示例

```
import torch
import os
from sentence_transformers import SentenceTransformer

os.environ["HF_ENDPOINT"] = "https://hf-mirror.com"  # 设置为hf的国内镜像网站
from transformers import AutoModel,AutoConfig

config = AutoConfig.from_pretrained(r"D:\dl\pre_trained_model\m3e-base-onnx")

# torch.set_printoptions(precision=64)
class MyEmbeddingModel(torch.nn.Module):
    def __init__(self, pretrained_model_name):
        super(MyEmbeddingModel, self).__init__()
        # self.model = AutoModel.from_pretrained(pretrained_model_name)
        self.model = SentenceTransformer(pretrained_model_name)
        self.fts = self.model[0]
        self.pls = self.model[1]


    def forward(self, input_ids, attention_mask):
        inputs = {
            "input_ids":input_ids,
            "token_type_ids": torch.zeros_like(input_ids),
            "attention_mask":attention_mask,
        }
        outputs = self.pls(self.fts(inputs))
        # outputs = self.model(inputs)
        return outputs['sentence_embedding']
        # outputs = self.model(input_ids=input_ids, token_type_ids=token_type_ids, attention_mask=attention_mask)
        # return outputs[0], outputs[1]


onnx_path = r'D:\dl\pre_trained_model\m3e-base-onnx\model.onnx'
model = MyEmbeddingModel(r"D:\dl\pre_trained_model\m3e-base-onnx")
# model = AutoModel.from_pretrained(r"D:\dl\pre_trained_model\m3e-base-onnx", config=config)
# model = SentenceTransformer(r"D:\dl\pre_trained_model\m3e-base-onnx")

model.eval()  # 设置模型为评估模式#准备输入数据
input_ids = torch.ones([1, 2], dtype=torch.int32)
token_type_ids = torch.ones([1, 2], dtype=torch.int32)
attention_mask = torch.ones([1, 2], dtype=torch.int32)


# 导出为ONNX模型
torch.onnx.export(
    model,  # 被导出的模型
    (input_ids, attention_mask),  # 示例输入
    f=onnx_path,  # 导出文件的路径
    opset_version=11,  # 0NNX操作集版本
    verbose=False,
    input_names=["input_ids", "attention_mask"],  # 输入名
    output_names=["out1"],  # 输出名
    dynamic_axes={"input_ids": [0, 1],
                  # "token_type_ids": [0, 1],
                  "attention_mask": [0, 1],
                  "out1": [0]},
    # 动态轴设置
)
```

```
import torch

# Load model directly
from transformers import AutoTokenizer, AutoModelForSequenceClassification
import logging
logging.basicConfig(level=logging.DEBUG)

tokenizer = AutoTokenizer.from_pretrained(r"D:\code\transfomers\bge-reranker-large")
model = AutoModelForSequenceClassification.from_pretrained(r"D:\code\transfomers\bge-reranker-large")

model.eval()

device = torch.device('cuda:0' if torch.cuda.is_available() else 'cpu')
print(device)
# 加载保存的模型
onnx_path = r'D:\code\transfomers\bge-reranker-large-onnx\model.onnx'
def make_train_dummy_input(seq_len):
    org_input_ids = torch.tensor(
        [[i for i in range(seq_len)]], dtype=torch.int32)
    org_input_mask = torch.tensor([[1 for i in range(int(
        seq_len/2))] + [1 for i in range(seq_len - int(seq_len/2))]], dtype=torch.int32)
    return (org_input_ids.to(device), org_input_mask.to(device))


with torch.no_grad():
    model=model.to(device)
    org_dummy_input = make_train_dummy_input(64)
    # print(org_dummy_input)
    output = torch.onnx.export(model,
                               org_dummy_input,
                               onnx_path,
                               verbose=True,
                               opset_version=11,
                               # 需要注意顺序！不可随意改变, 否则结果与预期不符
                               input_names=[
                                   'input_ids', 'attention_mask'],
                               # 需要注意顺序, 否则在推理阶段可能用错output_names
                               output_names=['logits'],
                               do_constant_folding=True,
                               dynamic_axes={"input_ids": {0: "batch_size", 1: "sequence_length"},
                                             "attention_mask": {0: "batch_size", 1: "sequence_length"},
                                             "logits": {0: "batch_size"}
                                            }
                               )
```
# 5 onnx简化

```
pip install onnx-simplifier  
python -m onnxsim stella_v2.onnx stella_v2_sim.onnx
```