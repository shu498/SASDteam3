## 1.UML 類別圖 
> 此圖表展示資料庫模型的欄位定義，以及 API 與服務層的主要功能。

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
```
## 2. 🔄 循序圖與活動圖

### 📌 UC-01: 音訊上傳與轉寫

#### 2.1 循序圖
``` mermaid
sequenceDiagram
    autonumber
    participant User as 使用者
    participant FE as 前端 (React)
    participant API as 後端 API (Django)
    participant FF as AudioService (FFmpeg)
    participant AI as AIService (WhisperX)
    participant DB as 資料庫

    User->>FE: 1. 上傳音訊檔 / 錄音
    FE->>API: 2. POST /api/upload (File)
    
    activate API
    API->>FF: 3. 驗證與預處理 (Preprocess)
    activate FF
    FF-->>API: 4. 回傳處理後路徑 (WAV 16k)
    deactivate FF
    
    API->>AI: 5. 請求轉寫 (Transcribe)
    activate AI
    AI-->>API: 6. 回傳逐字稿 (JSON + Timestamps)
    deactivate AI
    
    API->>DB: 7. 儲存 AudioFile 與 Transcript
    API-->>FE: 8. 回傳轉寫結果 (200 OK)
    deactivate API
    
    FE->>User: 9. 顯示轉寫文字
```
### 2.2 活動圖
``` mermaid
graph TD
    %% --- 樣式定義 ---
    classDef start_end fill:#f5f5f5,stroke:#333,stroke-width:2px,rx:10,ry:10,color:#000000;
    classDef proc fill:#e3f2fd,stroke:#1565c0,stroke-width:1px,color:#000000;
    classDef decision fill:#fff9c4,stroke:#fbc02d,stroke-width:2px,rx:5,ry:5,color:#000000;

    Start((開始)):::start_end
    End((結束)):::start_end

    %% 節點
    Input[使用者上傳音訊]:::proc
    CheckFmt{格式是否支援?}:::decision
    Error1[顯示格式錯誤訊息]:::proc
    Preprocess[FFmpeg 降噪與轉碼]:::proc
    CheckLen{長度是否 < 限制?}:::decision
    Error2[顯示檔案過大訊息]:::proc
    AI_Process[WhisperX 進行轉寫]:::proc
    SaveDB[儲存至資料庫]:::proc
    Display[前端顯示逐字稿]:::proc

    %% 流程
    Start --> Input
    Input --> CheckFmt
    
    CheckFmt -- 否 --> Error1
    Error1 --> End
    
    CheckFmt -- 是 --> CheckLen
    CheckLen -- 否 --> Error2
    Error2 --> End
    
    CheckLen -- 是 --> Preprocess
    Preprocess --> AI_Process
    AI_Process --> SaveDB
    SaveDB --> Display
    Display --> End
```
### 📌 UC-02: AI 重點摘要生成
### 循序圖
``` mermaid
sequenceDiagram
    autonumber
    participant User as 使用者
    participant FE as 前端 (React)
    participant API as 後端 API (Django)
    participant DB as 資料庫
    participant AI as AIService (Transformer)

    User->>FE: 1. 點擊「生成摘要」
    FE->>API: 2. POST /api/summary (TranscriptID)
    
    activate API
    API->>DB: 3. 查詢原始逐字稿
    DB-->>API: 4. 回傳 Text Content
    
    API->>AI: 5. 請求生成摘要 (Summarize)
    activate AI
    AI-->>API: 6. 回傳摘要文本
    deactivate AI
    
    API->>DB: 7. 儲存 Summary 紀錄
    API-->>FE: 8. 回傳摘要 JSON
    deactivate API
    
    FE->>User: 9. 顯示重點摘要區塊
```
### 2.4 活動圖
``` mermaid
graph TD
    %% --- 樣式定義 ---
    classDef start_end fill:#f5f5f5,stroke:#333,stroke-width:2px,rx:10,ry:10,color:#000000;
    classDef proc fill:#e3f2fd,stroke:#1565c0,stroke-width:1px,color:#000000;
    classDef decision fill:#fff9c4,stroke:#fbc02d,stroke-width:2px,rx:5,ry:5,color:#000000;

    Start((開始)):::start_end
    End((結束)):::start_end

    %% 節點
    Req[請求生成摘要]:::proc
    Fetch[讀取原始文本]:::proc
    CheckCount{字數 > 50字?}:::decision
    MsgError[提示: 內容過短無法摘要]:::proc
    AI_Sum[Transformer 分析語意]:::proc
    GenList[生成條列式重點]:::proc
    Save[儲存摘要結果]:::proc
    Show[顯示於介面]:::proc

    %% 流程
    Start --> Req
    Req --> Fetch
    Fetch --> CheckCount
    
    CheckCount -- 否 --> MsgError
    MsgError --> End
    
    CheckCount -- 是 --> AI_Sum
    AI_Sum --> GenList
    GenList --> Save
    Save --> Show
    Show --> End
```
### 📌 UC-03: 多語言翻譯
### 2.5 循序圖
``` mermaid
sequenceDiagram
    autonumber
    participant User as 使用者
    participant FE as 前端 (React)
    participant API as 後端 API (Django)
    participant AI as AIService (Translation Model)

    User->>FE: 1. 選擇目標語言 (如: English)
    FE->>API: 2. POST /api/translate (Text, TargetLang)
    
    activate API
    API->>AI: 3. 請求翻譯文本
    activate AI
    note right of AI: 載入對應語言模型
    AI-->>API: 4. 回傳翻譯結果
    deactivate AI
    
    API-->>FE: 5. 回傳翻譯 JSON
    deactivate API
    
    FE->>User: 6. 顯示翻譯對照結果
```
### 2.6 活動圖
``` mermaid
graph TD
    %% --- 樣式定義 ---
    classDef start_end fill:#f5f5f5,stroke:#333,stroke-width:2px,rx:10,ry:10,color:#000000;
    classDef proc fill:#e3f2fd,stroke:#1565c0,stroke-width:1px,color:#000000;
    classDef decision fill:#fff9c4,stroke:#fbc02d,stroke-width:2px,rx:5,ry:5,color:#000000;


    Start((開始)):::start_end
    End((結束)):::start_end

    %% 節點
    SelectLang[選擇目標語言]:::proc
    ClickTrans[點擊翻譯按鈕]:::proc
    CheckCache{是否已有快取?}:::decision
    GetCache[讀取資料庫翻譯紀錄]:::proc
    CallAI[呼叫翻譯模型]:::proc
    SaveDB[儲存翻譯結果]:::proc
    Display[雙欄顯示原文與譯文]:::proc

    %% 流程
    Start --> SelectLang
    SelectLang --> ClickTrans
    ClickTrans --> CheckCache
    
    CheckCache -- 是 --> GetCache
    GetCache --> Display
    
    CheckCache -- 否 --> CallAI
    CallAI --> SaveDB
    SaveDB --> Display
    
    Display --> End
```

