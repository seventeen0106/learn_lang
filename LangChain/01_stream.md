<div align="center">

# ✨ 一、串流功能

</div>

## Chat Model 的互動機制

> LangChain 組件都實現"可運行"（Runnable）協議，這個標準接口使得定義和調用自定義鏈變得簡單直接。包括以下方法：

- `stream`：串流回應的片段
- `invoke`：對輸入調用鏈
- `batch`：對輸入列表調用鏈

所有的 Runnable 物件都實作了一個同步方法 `stream` 和一個異步變體 `astream`。

這些方法旨在以分塊(Chunk)形式串流最終輸出，並在每個塊可用時立即生成。

### 01 invoke：同步、一次性給全

> 在模型生成完整回應後才返回完整結果。

```python
from langchain_openai import ChatOpenAI

model = ChatOpenAI(
    model="gpt-4.1-nano",
    openai_api_base="https://api.apertis.ai/v1", 
)

model.invoke("請你講一個早安問候語")
```

### 02 stream：串流

相比之下，stream() 方法允許我們在生成過程中逐步獲取回應，這對提升用戶體驗至關重要。`stream()` 方法返回一個生成器，能夠在生成過程中逐步產生 token。

> [技術要點] 串流的關鍵在於程序中的每個步驟都能夠處理輸入流，即一次處理一個輸入塊，並生成相應的輸出塊。它要求開發者重新思考數據處理流程，從整體處理轉向增量處理。

```python
# 呼叫 model.stream 時，它回傳的不是文字，而是一個「產生器物件（Generator Object）」。
# 你可以把它想像成一個「已經接好水管，但還沒轉開水龍頭」的狀態。
print(model.stream("請你講一個早安問候語"))

# 這樣寫，字就會一個一個印出來了！
for chunk in model.stream("請你講一個早安問候語"):
    print(chunk.content, end="")
```

### 03 astream：非同步/異步的串流（須配合 async for 或 await 使用）

> **什麼是「異步（Asynchronous）」？**
> 
> 想像一下你在下載一個 10GB 的大檔案。如果是「同步」，你的電腦畫面會卡死，滑鼠不能動，直到下載完畢。但如果是「異步」，你可以一邊下載，一邊繼續上網看 YouTube。
>
> 使用 async for，代表程式在等待網路傳送下一個字的微小空檔，可以暫時去做其他運算，等到字送達了，再來執行 print。

```python
chunks = []
async for chunk in model.astream("請你告訴我 k8s 跟 docker 區別"):
    chunks.append(chunk)
    print(chunk.content, end="|", flush=True) # 使用 end="|" 和 flush=True 可以實現平滑的輸出效果，讓用戶感受到實時生成的過程。
```

```python
# 將所有字組合後印出來
for chunk in model.stream("寫一首關於 k8s 的兒歌"):
    print(chunk.content, end="", flush=True)
```

> [!TIP]
> 使用 end="|" 和 flush=True 可以實現平滑的輸出效果，讓用戶感受到實時生成的過程。
