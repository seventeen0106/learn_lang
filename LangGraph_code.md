<div align="center">

# 1️⃣ 環境建置

</div>

# 2️⃣ 打造對話機器人

## 01 定義狀態

```python
### 定義一個StateGraph物件以將聊天機器人建構成狀態機。
from typing import Annotated
from typing_extensions import TypedDict
from langgraph.graph import StateGraph
from langgraph.graph.message import add_messages

class State(TypedDict): # State是一個用 List 類型的單一鍵messages定義的類別對象
    messages: Annotated[list, add_messages] # 使用add_messages()函數附加新訊息而不是覆寫它們。
```

## 02 定義語言模型

### colab

> 從 `secret` 檔案中讀取

```python
### 定義 OpenAI 語言模型
import os
from langchain_openai import ChatOpenAI

# 在colab中抓API key
os.environ["OPENAI_API_KEY"] = userdata.get('OPENAI_API_KEY')

# 定義模型函數（apertis）
llm = ChatOpenAI(
    model="gpt-4.1-nano",
    openai_api_base="https://api.apertis.ai/v1",
)
```

### jupyter

> 從 `.env` 檔案中讀取

```python
### 定義 OpenAI 語言模型
import os
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI

# 在 .env 中抓 API key
load_dotenv()

# 定義模型函數（apertis）
llm = ChatOpenAI(
    model="gpt-4.1-nano",
    openai_api_base="https://api.apertis.ai/v1",
)
```

> [!NOTE]
> 補充：定義模型函數（openAI）
```python
llm = ChatOpenAI(
    model="gpt-4-0613",
    temperature=0,
    max_tokens=None,
    timeout=None,
    max_retries=2,
)
```


## 03 添加語言模型節點

```python
### 此聊天機器人功能會作為名為「chatbot」的節點加入圖(State)中
def chatbot(state: State):
    return {"messages": [llm.invoke(state["messages"])]}
```

## 04 構建圖並且編譯

```python
graph_builder = StateGraph(State) #設定"圖"

graph_builder.add_node("chatbot", chatbot) #設定節點
graph_builder.set_entry_point("chatbot") #設定起點
graph_builder.set_finish_point("chatbot") #設定終點

graph = graph_builder.compile()
```

## 05 ✨✨✨ 可以視覺化，查看圖的流向

```python
### 可視化建構的圖
from IPython.display import Image, display

try:
    display(Image(graph.get_graph(xray=True).draw_mermaid_png()))
except Exception:
    # This requires some extra dependencies and is optional
    pass
```

## 06 實現聊天界面

```python
while True:
    user_input = input("使用者: ") #使用者輸入
    if user_input.lower() in ["quit", "exit", "q"]: #當使用者輸入"quit" 、 "exit"或"q"時，循環退出。
        print("掰啦！")
        break
    for event in graph.stream({"messages": [("user", user_input)]}):
        for value in event.values():
            print("AI 助理:", value["messages"][-1].content) #AI回覆
```









