<div align="center">

# ✨ 三、節點(Node)：工作流程的基石

</div>

> 每個節點都是一個工作站，負責接收原材料（輸入），進行加工（處理），然後產出成品（輸出）。
>
> 這些「工作站」可以執行各種操作，從簡單的資料轉換到複雜的下達決策。

## 01 函式庫

```
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
```

## 02 設定函數（等等轉化成節點）

### 設定 AI 模型

```
### 設定 AI 模型

## apertis AI 節點、模型
model = ChatOpenAI(
    model="gpt-4.1-nano",
    openai_api_base="https://api.apertis.ai/v1",
)

## System Prompt 設定
prompt_str = """
You are given one question and you have to extract city name from it
Don't respond anything except the city name and don't reply anything if you can't find city name
Only reply the city name if it exists or reply 'no_response' if there is no city name in question

  Here is the question:
  {user_query}
"""

## 把前面打好的詞塞進去 langGraph 的程式中
prompt = ChatPromptTemplate.from_template(prompt_str)

## 結合
chain = prompt | model

## 
result = chain.invoke({"user_query": "請問高雄天氣如何?"})
print(result.content)
```
