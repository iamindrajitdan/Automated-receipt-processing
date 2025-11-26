# Automated Receipt Processing Tool - Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    AUTOMATED RECEIPT PROCESSING WORKFLOW                 │
└─────────────────────────────────────────────────────────────────────────┘

    ┌──────────┐
    │   User   │
    └────┬─────┘
         │ 1. Upload Receipt
         ▼
    ┌─────────────────┐
    │   Amazon S3     │
    │  (Bucket)       │
    │  /incoming/     │
    └────┬────────────┘
         │ 2. S3 Event Notification
         │    (ObjectCreated)
         ▼
    ┌─────────────────┐
    │  AWS Lambda     │◄──────┐
    │  (Python 3.9)   │       │
    │  - Orchestrator │       │ IAM Role
    └────┬────────────┘       │ (Permissions)
         │                    │
         │ 3. Extract Text    │
         ▼                    │
    ┌─────────────────┐       │
    │ Amazon Textract │       │
    │  - OCR Engine   │       │
    └────┬────────────┘       │
         │                    │
         │ 4. Store Data      │
         ▼                    │
    ┌─────────────────┐       │
    │  Amazon         │       │
    │  DynamoDB       │       │
    │  - receipts     │       │
    │  PK: receiptID  │       │
    │  SK: date       │       │
    └─────────────────┘       │
         │                    │
         │ 5. Send Summary    │
         ▼                    │
    ┌─────────────────┐       │
    │  Amazon SES     │◄──────┘
    │  - Email        │
    └────┬────────────┘
         │
         ▼
    ┌──────────┐
    │   User   │
    │  (Email) │
    └──────────┘


┌─────────────────────────────────────────────────────────────────────────┐
│                         IAM PERMISSIONS                                  │
├─────────────────────────────────────────────────────────────────────────┤
│  • S3: GetObject (read receipts)                                        │
│  • Textract: AnalyzeExpense, DetectDocumentText                         │
│  • DynamoDB: PutItem (write receipt data)                               │
│  • SES: SendEmail (send notifications)                                  │
│  • CloudWatch Logs: CreateLogGroup, PutLogEvents                        │
└─────────────────────────────────────────────────────────────────────────┘
```

## Data Flow

1. **Upload** → Receipt image uploaded to S3 /incoming/ folder
2. **Trigger** → S3 event automatically invokes Lambda function
3. **Extract** → Lambda sends receipt to Textract for OCR processing
4. **Parse** → Textract returns structured data (merchant, date, amount, items)
5. **Store** → Lambda writes extracted data to DynamoDB table
6. **Notify** → Lambda sends formatted email via SES with receipt details

## Key Components

- **Event Source**: S3 bucket with event notifications
- **Compute**: Lambda function (extended timeout, environment variables)
- **AI/ML**: Textract for intelligent document processing
- **Database**: DynamoDB for NoSQL storage
- **Notification**: SES for email delivery
- **Security**: IAM role with least-privilege permissions
