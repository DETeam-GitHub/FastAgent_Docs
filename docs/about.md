# 关于 FastAgent

## 为什么会有这个项目

Debug967 在某一天使用LLM大模型API时，发现函数调用太 ~~TM~~ 难用了，为了简化函数调用流程，便开发了这个项目。

## 为什么要用这个项目

不说废话，上对比

如果我们要通过函数调用实现web搜索，则有以下两种写法：

### 原生

> 写的很烂，大家轻点喷

zpws.py

```python
import requests
import uuid
from rich import print


api_key = "YOUR_API_KEY"

def run_v4_sync(q="中国队奥运会拿了多少奖牌"):
    msg = [
        {
            "role": "user",
            "content":q
        }
    ]
    tool = "web-search-pro"
    url = "https://open.bigmodel.cn/api/paas/v4/tools"
    request_id = str(uuid.uuid4())
    data = {
        "request_id": request_id,
        "tool": tool,
        "stream": False,
        "messages": msg
    }

    resp = requests.post(
        url,
        json=data,
        headers={'Authorization': api_key},
        timeout=300
    )
    return resp



if __name__ == '__main__':
    print(run_v4_sync().json())
```

main.py

```python
from zhipuai import ZhipuAI
from rich import print
import json
from zpws import run_v4_sync as zps

tools = [
    {
        "type": "function",
        "function": {
            "name": "search",
            "description": "进行Web搜索并返回JSON格式的结果",
            "parameters": {
                "type": "object",
                "properties": {"q": {"type": "string", "description": "问题"}},
                "required": ["q"],
            },
        },
    }
]


def rpy(e):
    print(e)
    try:
        return zps(e)
    except:
        return f"Error For {e}"


client = ZhipuAI(api_key="YOUR_API_KEY")

msg = [
    {
        "role": "system",
        "content": """你可以进行Web搜索""",
    }
]


def czp(om):
    response = client.chat.completions.create(
        model="glm-4-flash",
        messages=om,
        tools=tools,
    )

    if response.choices[0].finish_reason == "tool_calls":
        # print(response.choices[0])
        om.append(
            {
                "role": "tool",
                "content": rpy(
                    json.loads(
                        response.choices[0].message.tool_calls[0].function.arguments
                    )["q"]
                ).content.decode(),
            }
        )
        print("tool!")
        return czp(om)
    elif response.choices[0].finish_reason == "stop":
        om.append({"role": "assistant", "content": response.choices[0].message.content})
        return om


while True:
    msg.append({"role": "user", "content": input("User> ")})
    print(czp(msg)[-1]["content"])

```

### 使用本项目

```python
from fast_agent import FastAgent
import requests,uuid
from zhipuai import ZhipuAI
import 


KEY = "YOUR_API_KEY"

c = FastAgent(KEY)

@c.tool(q="搜索内容")
def s(q:str):
    """网络搜索器，返回json"""
    api_key = KEY
    print("正在搜索...",q)
    msg = [
        {
            "role": "user",
            "content":q
        }
    ]
    tool = "web-search-pro"
    url = "https://open.bigmodel.cn/api/paas/v4/tools"
    request_id = str(uuid.uuid4())
    data = {
        "request_id": request_id,
        "tool": tool,
        "stream": False,
        "messages": msg
    }

    resp = requests.post(
        url,
        json=data,
        headers={'Authorization': api_key},
        timeout=300
    )
    return resp.content.decode()

while True:
    l , p= c.chat(l,input("User> "))
    print(l[-1]["content"])
```

> 明显看出：
> 使用本项目的代码量减少很多，且可读性更高
