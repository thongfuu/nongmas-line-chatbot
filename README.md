# NongMas - AI Financial Assistant Chatbot

![n8n](https://img.shields.io/badge/Workflow-n8n-FF6D5A?style=for-the-badge&logo=n8n&logoColor=white)
![LINE](https://img.shields.io/badge/Platform-LINE_OA-00C300?style=for-the-badge&logo=line&logoColor=white)
![Gemini](https://img.shields.io/badge/AI-Google_Gemini-8E44AD?style=for-the-badge&logo=google-gemini&logoColor=white)
![MongoDB](https://img.shields.io/badge/Database-MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white)
![Google Drive](https://img.shields.io/badge/Storage-Google_Drive-34A853?style=for-the-badge&logo=google-drive&logoColor=white)

> **"Financial tracking made conversational."**
> NongMas is an advanced LINE Chatbot powered by **Generative AI** that automates expense tracking via Text, Image (Slips), and Voice.

## Key Features

### 1. Multi-Modal Expense Tracking
* **Text:** "Lunch 50 baht" -> AI extracts `Category: Food`, `Amount: 50`.
* **Voice:** Send a voice message -> Gemini Transcribes -> AI Process -> Save to DB.
* **Image (Slips):** Upload a bank slip -> Gemini Vision extracts `Date`, `Amount`, `TransacID` -> Checks for duplicates -> Saves automatically.

### 2. Smart Duplicate Protection
* Before saving a slip, the system checks **MongoDB** for the unique `transaction_id`.
* **If duplicate:** Responds with "Duplicate slip detected!" and rejects the entry.
* **If new:** Uploads the slip image to **Google Drive** (organized by UserID) and logs the transaction.

### 3. Interactive Dashboard
* **Daily/Monthly Summary:** Calculates income, expense, and balance in real-time using MongoDB Aggregation pipelines.
* **Rich UI:** Responds with a beautiful **LINE Flex Message** dashboard.
* **Actionable:** Users can click to view details or delete specific transactions directly from the chat.

## Workflow Diagram

This diagram represents the actual logic flow from the n8n workflow:

```mermaid
graph TD
    %% Event Trigger
    Start(["Webhook (LINE Events)"]) --> Router{"Check Message Type"}

    %% --- Image Handling (Slips) ---
    Router -- "Image" --> GetImg["Get Image Content"]
    GetImg --> Vision["Gemini Vision (Analyze Slip)"]
    Vision --> Extract["Extract JSON (ID, Amount, Date)"]
    Extract --> DupCheck{"Check MongoDB\n(Duplicate ID?)"}
    
    DupCheck -- "Yes (Duplicate)" --> ReplyDup["Reply: 'Slip Duplicated!'"]
    DupCheck -- "No (New)" --> UploadDrive["Upload Image to Google Drive"]
    UploadDrive --> SaveMongoImg["Insert into MongoDB"]
    SaveMongoImg --> ReplyFlex["Reply: Success Flex Message"]

    %% --- Text & Voice Handling ---
    Router -- "Voice" --> Transcribe["Gemini Transcribe (Audio->Text)"]
    Router -- "Text" --> TextProc["Process Text"]
    Transcribe --> AI_NLP["AI (OpenAI/Gemini) -> JSON"]
    TextProc --> AI_NLP
    AI_NLP --> SaveMongoText["Insert into MongoDB"]
    SaveMongoText --> ReplyFlex

    %% --- Postback Actions (Dashboard) ---
    Router -- "Postback (Summary)" --> MongoAgg["MongoDB Aggregate (Sum Income/Expense)"]
    MongoAgg --> GenDash["Generate Flex Message"]
    GenDash --> ReplyDash["Reply: Dashboard"]

    Router -- "Postback (Delete)" --> MongoDel["Delete Document"]
    MongoDel --> ReplyDel["Reply: 'Deleted'"]

    %% Styles
    style Vision fill:#8e44ad,stroke:#9b59b6,color:white
    style AI_NLP fill:#8e44ad,stroke:#9b59b6,color:white
    style SaveMongoImg fill:#47A248,stroke:#2ecc71,color:white
    style SaveMongoText fill:#47A248,stroke:#2ecc71,color:white
    style MongoAgg fill:#47A248,stroke:#2ecc71,color:white
    style UploadDrive fill:#34A853,stroke:#27ae60,color:white
```

## Tech Stack
- Core Engine: n8n (Self-hosted/Cloud)
- Messaging: LINE Messaging API (Webhook, Flex Message)
- Database: MongoDB (Transactions, Logs)
- Storage: Google Drive (Slip Images)
- AI Models:
    - Google Gemini 2.5 Flash: For Image Analysis (OCR) & Voice Transcription.
    - OpenAI / Gemini: For Natural Language Processing (Intent Classification).

## Usage Examples
| Input Type | Example | Result (Database) |
| :--- | :--- | :--- |
| **Text** | "Paid electricity bill 1,200 baht" | `Class: Bill`, `Amount: 1200`, `Type: Expense` |
| **Text** | "Salary came in 30,000" | `Class: Salary`, `Amount: 30000`, `Type: Income` |
| **Voice** | (Speaking) "Bought coffee 60 baht" | `Class: Beverage`, `Amount: 60`, `Type: Expense` |
| **Image** | [Upload K-Plus Slip] | Extracts `TransacID`, `Time`, saves image to Drive, logs entry. |

## Demo & Screenshot

<table>
  <tr>
    <th width="33%">ส่งข้อความ (Text)</th>
    <th width="33%">ส่งสลิป (Slip OCR)</th>
    <th width="33%">ส่งเสียง (Voice)</th>
  </tr>
  <tr>
    <td valign="top">
      <img src="https://github.com/user-attachments/assets/3d49d9ec-8944-40dc-8c28-676b9489bae7" width="100%" alt="ส่งข้อความ" />
    </td>
    <td valign="top">
      <img src="https://github.com/user-attachments/assets/dc6f388a-4dc7-49c8-9516-8d4fcc9a56c0" width="100%" alt="ส่งสลิป" />
    </td>
    <td valign="top">
      <img src="https://github.com/user-attachments/assets/eea61152-801b-4e0d-9db0-3ea499251647" width="100%" alt="ส่งเสียง" />
    </td>
  </tr>
  <tr>
    <th>สรุปรายเดือน (Dashboard)</th>
    <th>ดูรายการวันนี้ (Daily View)</th>
    <th></th> </tr>
  <tr>
    <td valign="top">
      <img src="https://github.com/user-attachments/assets/5474c44b-5126-492c-ab55-413e5f73bd14" width="100%" alt="สรุปรายเดือน" />
    </td>
    <td valign="top">
      <img src="https://github.com/user-attachments/assets/6321f274-c2cb-4f32-9032-e190925c8655" width="100%" alt="ดูรายการวันนี้" />
    </td>
    <td></td> </tr>
</table>


## Installation & Setup
1. Import Workflow: Import NongMas_line_chatbot.json into n8n.
2. Credentials: Set up the following credentials in n8n:
   - LINE Developer: Access Token & Channel Secret.
   - Google Cloud: OAuth2 for Google Drive & Gemini API.
   - MongoDB: Connection String (Atlas/Local).
   - OpenAI: API Key (if enabled).
3. Drive Folder: Replace the Folder ID in the "Search files and folders" node with your specific Google Drive Folder ID.
4. Webhook: Connect the n8n Webhook URL to your LINE Developers Console.
