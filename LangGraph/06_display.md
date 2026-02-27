<div align="center">

# ✨ 六、將圖的流程視覺化

</div>

## 01 函式庫

```python
from IPython.display import Image, display
```

## 02 程式範例

```python
# 使用 try...except 區塊來捕捉可能的錯誤，避免程式因為畫不出圖而整個崩潰
try:
    # 1. app.get_graph(): 從剛剛編譯好的 app 中取得圖的結構
    # 2. draw_mermaid_png(): 將結構轉換成 Mermaid 格式的流程圖，並輸出成 PNG 圖片檔
    # 3. Image() 與 display(): 這是 Jupyter Notebook (或 Colab) 特有的功能，用來在畫面上直接顯示圖片
    display(Image(app.get_graph().draw_mermaid_png()))
    
except Exception: # 如果發生任何錯誤 (Exception)，就執行這裡
    pass
```
