# SASD team3
System Analysis and Design Team No.3 project

## 即時語音轉文字系統
本系統能讓使用者上傳音訊或即時錄音，並自動將語音轉成文字。<br>
系統也會整理長篇內容、抓出重點做成摘要，並提供多國語言翻譯，方便原文與譯文一起查看。<br>
![Python](https://img.shields.io/badge/Python-3.11.9-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Django](https://img.shields.io/badge/Django-Backend-092E20?style=for-the-badge&logo=django&logoColor=white)
![DRF](https://img.shields.io/badge/Django_REST_Framework-API-A30000?style=for-the-badge&logo=django&logoColor=white)
![React](https://img.shields.io/badge/React-Frontend-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![WhisperX](https://img.shields.io/badge/AI-WhisperX-FF6F00?style=for-the-badge&logo=openai&logoColor=white)
![FFmpeg](https://img.shields.io/badge/Tool-FFmpeg-007808?style=for-the-badge&logo=ffmpeg&logoColor=white)
![Git](https://img.shields.io/badge/Version_Control-Git-F05032?style=for-the-badge&logo=git&logoColor=white)

## 成員
| 學號 | 姓名 |
| :---: | :---: |
| C112118226 | 林東毅 |
| C112118227 | 張恩豪 |
| C112118234 | 蘇子皓 |

``` mermaid
sequenceDiagram
    participant User as 使用者
    participant FE as 前端 (React)
    participant BE as 後端 (Django)
    participant AI as AI 模型 (WhisperX/Transformer)

    User->>FE: 上傳音訊/錄音
    FE->>BE: POST /api/upload (音訊檔)
    activate BE
    BE->>BE: FFmpeg 格式轉換與預處理
    BE->>AI: 發送音訊進行 STT (語音轉文字)
    activate AI
    AI-->>BE: 返回原始文本 (Transcript)
    deactivate AI
    
    par 平行處理
        BE->>AI: 請求重點摘要 (Summarization)
        BE->>AI: 請求翻譯 (Translation)
    end
    
    activate AI
    AI-->>BE: 返回摘要與翻譯結果
    deactivate AI

    BE-->>FE: 返回 JSON (文本 + 摘要 + 翻譯)
    deactivate BE
    FE->>User: 顯示結果圖表與文字
```
