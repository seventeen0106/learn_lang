<div align="center">

# ✨ 三、輸出解析器（StrOutputParser）

</div>

這是一個簡單的解析器，它從 AIMessageChunk 中提取內容字段，給我們模型返回的 token。

## 01 函式庫

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
```

## 02 解析器使用範例
```python
prompt = ChatPromptTemplate.from_template("講一個早安問候語關於 {topic}")

# 2. 準備「輸出解析器」(Output Parser)
# AI 回傳的東西是一個包含很多雜訊的物件（例如：AIMessage(content="早安...")），所以你印出來時必須寫 chunk.content 才能拿到純文字。
# StrOutputParser 就像是一個濾水器。它可以自動幫你把 AI 回傳的複雜物件過濾掉，只把最純淨的「文字內容」萃取出來往後送。
parser = StrOutputParser()
chain = prompt | model | parser

async for chunk in chain.astream({"topic": "父母親"}):
    print(chunk, end="|", flush=True)
```
