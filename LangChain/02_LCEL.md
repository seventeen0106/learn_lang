<div align="center">

# ✨ 二、LangChain 表達語言（LCEL）

</div>

 LCEL 是一種聲明式方法，用於通過鏈接不同的 LangChain 原語來指定"程序"。使用 LCEL 創建的鏈條會自動實現 stream 和 astream，允許串流最終輸出。

## 01 函式庫

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
```

## 02 使用範例
 
 ```python
# 1. 準備動態的「提示詞模板」(Prompt Template)，不用把問題寫死，可以挖一個洞 {topic} 當作變數。
prompt = ChatPromptTemplate.from_template("講一個早安問候語關於 {topic}")

# 2. 準備「輸出解析器」(Output Parser)
parser = StrOutputParser()

# 3. ✨組裝流水線 (Chain)：LCEL 的精華！利用 |（管線符號）把三個獨立的元件串在一起。
chain = prompt | model | parser

# 4. 執行與非同步串流輸出 (Async Stream)
async for chunk in chain.astream({"topic": "父母親"}):
    print(chunk, end="|", flush=True)
```
