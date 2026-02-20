# learn_lang

## 虛擬環境

### 激活虛擬環境
```
lang_env\Scripts\activate
```

> [!CAUTION]
> 記得運行的時候 kernel 要變

### 刪除指定 kernel：
```
jupyter kernelspec list
```
```
jupyter kernelspec uninstall 要刪除的kernal
```

## 環境變數設定

### 1️⃣ 安裝：
```
pip install python-dotenv
```

### 2️⃣ 建立 `.env` 檔：
> [!CAUTION]
> 記得開啟副檔名修改，檔名叫做 `.env`，不是 `.env.txt`
```
OPENAI_API_KEY=你的API_KEY
```

### 3️⃣ Notebook 改成：
```
from dotenv import load_dotenv

load_dotenv()
```

## Day3

1. 修正了格式
2. API 格式修改為 apertis

## 
