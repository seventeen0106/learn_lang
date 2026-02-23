<div align="center">

# ✨ 二、狀態(state)：消息的共享中心

</div>

> 狀態是捕獲應用程序當前快照的共享資料結構，使節點能夠相互通信和交換訊息。它可以是任何 Python 類型,但通常是 TypedDict 或 Pydantic BaseModel。

**有兩種類型的圖可以在 LangGraph 中創建:**

- 基本圖(Basic Graph): 只能將一個節點的輸出傳遞給下一個節點。因為無法包含狀態
- 有狀態圖圖(Stateful Graph): 可以包含在節點之間傳遞的狀態,並且可以在任何節點訪問此狀態。

## 01 函式庫
```python
from typing import TypedDict, Annotated, Sequence
import operator
```

## 02 狀態設定

```python
### 首先先來建立整張圖的狀態，我們將創建一種稱為「Messages」的狀態，它將儲存整個工作流程中發生的所有對話。所以讓我們先創建它！
class AllState(TypedDict):
    messages: Annotated[Sequence[str], operator.add]
```



