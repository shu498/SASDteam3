## 🗄️ 資料庫實體關係圖 (ERD)

系統資料庫的邏輯設計

### 1. 組合實體說明
本系統包含以下兩個多對多或依賴型的組合實體：

1.  **翻譯紀錄**
    * **組成**：由「原始逐字稿」與「目標語言」共同定義。
    * **用途**：解決一份文件對應多種語言版本的需求。
2.  **音訊標籤關聯**
    * **組成**：由「音訊檔」與「標籤」組成。
    * **用途**：處理多對多關係，讓一個音訊可擁有多個標籤（如：會議、重要），一個標籤也可套用到多個音訊。

---

### 2. 系統 ERD 圖表 
> **圖例說明**：
> * `PK`：主鍵
> * `FK`：外鍵
> * `string`, `int`, `datetime`：資料型態

```mermaid
erDiagram
    %% --- 1. 使用者資料表 ---
    USER {
        int 編號_ID PK
        string 使用者名稱
        string 電子郵件
        string 密碼雜湊
        datetime 註冊日期
    }

    %% --- 2. 音訊檔案資料表 ---
    AUDIO_FILE {
        int 編號_ID PK
        int 上傳者ID FK
        string 檔案名稱
        string 存放路徑
        float 音訊長度_秒
        string 檔案格式_MIME
        datetime 上傳時間
    }

    %% --- 3. 逐字稿資料表 ---
    TRANSCRIPT {
        int 編號_ID PK
        int 關聯音訊ID FK
        text 轉寫內容_全文
        string 語言代碼_如zhTW
        json 字詞時間戳記
        datetime 建立時間
    }

    %% --- 4. 組合實體：翻譯紀錄 (依賴逐字稿) ---
    TRANSLATION {
        int 編號_ID PK
        int 來源逐字稿ID FK
        string 目標語言_如en
        text 翻譯後內容
        datetime 翻譯時間
    }

    %% --- 5. 標籤資料表 (分類用) ---
    TAG {
        int 編號_ID PK
        string 標籤名稱
        string 標籤顏色_Hex
    }

    %% --- 6. 組合實體：音訊與標籤的中間表 ---
    AUDIO_TAG {
        int 編號_ID PK
        int 音訊ID FK
        int 標籤ID FK
    }

    %% --- 7. 重點摘要資料表 ---
    SUMMARY {
        int 編號_ID PK
        int 關聯逐字稿ID FK
        text 摘要內容
        string 使用模型版本
        datetime 生成時間
    }

    %% --- 關聯線與中文動作說明 ---
    
    %% 使用者 上傳 音訊 (1對多)
    USER ||--o{ AUDIO_FILE : "上傳"
    
    %% 音訊 生成 逐字稿 (1對1)
    AUDIO_FILE ||--|| TRANSCRIPT : "生成"
    
    %% 逐字稿 擁有 多種翻譯 (1對多)
    TRANSCRIPT ||--o{ TRANSLATION : "翻譯"
    
    %% 逐字稿 整理出 摘要 (1對多)
    TRANSCRIPT ||--o{ SUMMARY : "整理"
    
    %% 音訊 包含 標籤 (透過中間表 AUDIO_TAG)
    AUDIO_FILE ||--o{ AUDIO_TAG : "標記"
    TAG ||--o{ AUDIO_TAG : "套用"
```
