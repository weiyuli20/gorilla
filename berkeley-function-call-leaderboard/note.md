bfcl 评估

一、pip insatll 方式
1.安装

```
pip install bfcl-eval

```

2.创建目录保存结果

```
mkdir bfcl_eval_results

复制.envexample 到.env

cp $(python -c "import bfcl_eval; print(bfcl_eval.__path__[0])")/.env.example $BFCL_PROJECT_ROOT/.env
# Fill in necessary values in `.env`
```

3.运行

```
 bfcl generate --model qwen3.8-flash-FC --test-category simple_python
```

qwen3.8-flash-FC 是模型名称,需要看模型是否在bfcl默认支持的模型列表里，如果不在，需要注册模型

/home/dyyuser/.conda/envs/vllm_env/lib/python3.10/site-packages/bfcl_eval/constants/model_config.py

```
# 自定义百炼模型
MODEL_CONFIG_MAPPING["qwen3.8-flash-FC"] = ModelConfig(
    model_name="qwen3.8-flash",
    display_name="Qwen3.8-Flash (FC)",
    url="https://bailian.console.aliyun.com/",
    org="Qwen",
    license="Unknown",
    model_handler=OpenAICompletionsHandler,
    input_price=None,
    output_price=None,
    is_fc_model=True,
    underscore_to_dot=True, # 有些模型会将函数名的.替换为_,导致评测时出错
)

```

在.env里配置openai的key和url

```
OPENAI_API_KEY=sk-ws-H.YLHLXMY.923q.MEUCIQDy43LKXhvh4B3ZeY0p7gazJBLCpVOfWL6MuFlrjUnGiQIgVEcAi5vMLDScaHhpLBSLTGs88NuSQzK5N2SuEnIZcdQ
OPENAI_BASE_URL=https://llm-mvatcq7dlr50w886.cn-beijing.maas.aliyuncs.com/compatible-mode/v1
```

同时修改
/home/dyyuser/.conda/envs/vllm_env/lib/python3.10/site-packages/bfcl_eval/model_handler/api_inference/qwen.py

增加
```
from dotenv import load_dotenv
load_dotenv()
```
同时修改该类的 get_chat_model({...}) 中改为：

```
'model_server': os.environ["OPENAI_BASE_URL"],
'api_key': os.environ["OPENAI_API_KEY"],

```

另外，generate命令启动报错时可能将类的初始化需要修改为
```
class QwenAgentNoThinkHandler(QwenAgentThinkHandler):

    def __init__(
        self,
        model_name,
        temperature,
        registry_name,
        is_fc_model,
        **kwargs,
    ) -> None:
        super().__init__(
            model_name, temperature, registry_name, is_fc_model, **kwargs
        )
```


bfcl generate 会将生成结果保存在 bfcl_eval_result/result目录

4.计算评分

```
bfcl evaluate --model qwen3.8-flash-FC --test-category simple_python
```

这个操作会使用已经生成的结果，计算评分，将结果保存在 bfcl_eval_result/score 目录



二、 源码模式：
1.下载源码
```
git clone https://github.com/ShishirPatil/gorilla.git

# Change directory to the `berkeley-function-call-leaderboard`
cd gorilla/berkeley-function-call-leaderboard

# Install the package in editable mode
pip install -e .
```


2.环境变量配置
```
cp bfcl_eval/.env.example .env
# Fill in necessary values in `.env`
```

3. 生成结果（在berkeley-function-call-leaderboard路径下运行）
```
bfcl generate --model qwen3.8-flash-FC --test-category simple_python
```

可能需要注册模型。

4.计算评分
```
bfcl evaluate --model qwen3.8-flash-FC --test-category simple_python
```
