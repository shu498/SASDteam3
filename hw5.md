## 1. 🏗️ UML 類別圖 (Class Diagram) - 中文詳細版
> 此圖表展示資料庫模型 (Models) 的欄位定義，以及 API 與服務層的主要功能。

``` mermaid
classDiagram
    %% --- 樣式定義 ---
    classDef model fill:#fff9c4,stroke:#fbc02d,stroke-width:1px;
    classDef view fill:#e1f5fe,stroke:#0277bd,stroke-width:1px;
    classDef service fill:#e8f5e9,stroke:#2e7d32,stroke-width:1px;

    %% --- 1. 資料模型 (Django Models) ---
    class AudioFile {
        <<音訊檔案資料表>>
        +int 編號_id
        +String 檔案名稱
        +String 存放路徑
        +DateTime 上傳時間
        +float 音訊長度
        +儲存檔案()
        +刪除檔案()
    }

    class Transcript {
        <<逐字稿資料表>>
        +int 編號_id
        +int 關聯音訊ID
        +Text 轉寫內容
        +String 語言代碼
        +JSON 時間戳記資料
        +DateTime 建立時間
    }

    class Summary {
        <<重點摘要資料表>>
        +int 編號_id
        +int 關聯逐字稿ID
        +Text 摘要內容
        +String 使用模型版本
    }

    class Translation {
        <<翻譯結果資料表>>
        +int 編號_id
        +int 關聯逐字稿ID
        +String 目標語言
        +Text 翻譯內容
    }

    %% --- 2. API 視圖 (Django Views) ---
    class AudioUploadView {
        <<上傳介面>>
        +接收上傳請求(request)
    }
    class SummaryView {
        <<摘要介面>>
        +建立摘要請求(request)
        +取得摘要結果(request)
    }
    class TranslationView {
        <<翻譯介面>>
        +建立翻譯請求(request)
    }

    %% --- 3. 業務邏輯服務 (Services) ---
    class AudioService {
        <<音訊處理服務>>
        +驗證檔案格式(file)
        +預處理與降噪(file) : 回傳路徑
    }

    class AIService {
        <<AI 核心服務>>
        +執行Whisper轉寫(path) : 回傳文字
        +執行Transformer摘要(text) : 回傳摘要
        +執行多語言翻譯(text, lang) : 回傳翻譯
    }

    %% --- 關係連結 ---
    %% Models 關係
    AudioFile "1" -- "1" Transcript : 產生
    Transcript "1" -- "0..1" Summary : 生成摘要
    Transcript "1" -- "0..*" Translation : 翻譯成多種語言

    %% Views 調用 Services
    AudioUploadView ..> AudioService : 使用
    AudioUploadView ..> AIService : 使用
    SummaryView ..> AIService : 使用
    TranslationView ..> AIService : 使用

    %% Views 操作 Models
    AudioUploadView --> AudioFile : 建立
    AudioUploadView --> Transcript : 建立
    SummaryView --> Summary : 建立
    TranslationView --> Translation : 建立

    %% --- 套用樣式 (改為逐行設定以避免錯誤) ---
    class AudioFile model
    class Transcript model
    class Summary model
    class Translation model
    
    class AudioUploadView view
    class SummaryView view
    class TranslationView view
    
    class AudioService service
    class AIService service
```
