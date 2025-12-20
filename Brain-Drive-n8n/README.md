# 🧠 Brain-Drive: n8n RAG Automation System

<p align="center">
  <img src="https://img.shields.io/badge/n8n-Automation-orange?style=for-the-badge&logo=n8n" alt="n8n">
  <img src="https://img.shields.io/badge/Supabase-Database-3ECF8E?style=for-the-badge&logo=supabase" alt="Supabase">
  <img src="https://img.shields.io/badge/OpenAI-GPT--4o-412991?style=for-the-badge&logo=openai" alt="OpenAI">
  <img src="https://img.shields.io/badge/Google%20Drive-API-4285F4?style=for-the-badge&logo=googledrive" alt="Google Drive">
</p>

<p align="center">
  <strong>Transform your documents into an intelligent, searchable AI-powered knowledge base</strong>
</p>

---

## 📖 Overview

**Brain-Drive** is an intelligent automation system built on **n8n** that transforms your **Google Drive** documents into a searchable, AI-powered knowledge base. It leverages **Retrieval-Augmented Generation (RAG)** to provide accurate, context-aware answers from your personal document library.

### ✨ Key Features

- 🔄 **Automatic Document Ingestion** - Monitors Google Drive and processes new files automatically
- 🧩 **Smart Text Chunking** - Splits documents into optimal chunks (200 chars with 20 char overlap)
- 🔍 **Semantic Search** - Uses vector embeddings for intelligent similarity matching
- 🤖 **AI-Powered Responses** - GPT-4o-mini generates accurate answers from your knowledge base
- 💾 **Persistent Memory** - Conversation history maintained across sessions
- 📄 **PDF Support** - Native extraction and processing of PDF documents

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           BRAIN-DRIVE ARCHITECTURE                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                    📤 INGESTION PIPELINE                            │   │
│  │                    (Doc-Vector-Uploader)                            │   │
│  │                                                                     │   │
│  │   📁 Google Drive  ──►  📄 Download  ──►  📝 Extract PDF            │   │
│  │        Trigger              File              Text                  │   │
│  │                                                   │                 │   │
│  │                                                   ▼                 │   │
│  │   💾 Supabase  ◄──  🔢 OpenAI  ◄──  ✂️ Recursive                   │   │
│  │   Vector Store     Embeddings      Text Splitter                    │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                    🤖 INTELLIGENCE INTERFACE                        │   │
│  │                    (RAG-AI-Agent)                                   │   │
│  │                                                                     │   │
│  │   💬 Chat  ──►  🧠 AI Agent  ──►  🔍 Vector Search                  │   │
│  │   Trigger      (GPT-4o-mini)      (Supabase)                        │   │
│  │                      │                  │                           │   │
│  │                      ▼                  ▼                           │   │
│  │               📝 Generate  ◄──  📚 Retrieved                        │   │
│  │                  Answer          Context                            │   │
│  │                      │                                              │   │
│  │                      ▼                                              │   │
│  │               💭 Buffer Memory (Conversation History)               │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 🚀 Workflows

This project consists of two core workflows that work together seamlessly:

### 1. 📤 Doc-Vector-Uploader.json

The **Ingestion Pipeline** - Automatically processes and stores your documents.

| Node                        | Function                                     |
| --------------------------- | -------------------------------------------- |
| **Google Drive Trigger**    | Monitors folder for new files (every minute) |
| **Edit Fields**             | Extracts and stores file metadata            |
| **Download File**           | Retrieves the document from Google Drive     |
| **Extract from File**       | Parses PDF content into raw text             |
| **Recursive Text Splitter** | Chunks text (200 chars, 20 overlap)          |
| **OpenAI Embeddings**       | Generates 1536-dim vector embeddings         |
| **Supabase Vector Store**   | Stores embeddings with metadata              |

### 2. 🤖 RAG-AI-Agent.json

The **Intelligence Interface** - Your conversational AI assistant.

| Node                      | Function                               |
| ------------------------- | -------------------------------------- |
| **Chat Trigger**          | Receives user messages via webhook     |
| **AI Agent**              | Orchestrates the RAG pipeline          |
| **OpenAI Chat Model**     | GPT-4o-mini for response generation    |
| **Simple Memory**         | Buffer window for conversation context |
| **Supabase Vector Store** | Retrieves relevant document chunks     |
| **OpenAI Embeddings**     | Converts queries to vector space       |

#### 🎯 Agent System Prompt

The AI agent follows strict guidelines:

- ✅ Always searches the knowledge base before answering
- ✅ Bases answers ONLY on retrieved context
- ✅ Honestly states when information isn't available
- ✅ Maintains professional and concise tone

---

## 🛠️ Tech Stack

| Category             | Technology                                          | Purpose                |
| -------------------- | --------------------------------------------------- | ---------------------- |
| **Automation**       | [n8n.io](https://n8n.io/)                           | Workflow orchestration |
| **Vector Database**  | [Supabase](https://supabase.com/)                   | PostgreSQL + pgvector  |
| **Embeddings**       | [OpenAI](https://openai.com/)                       | text-embedding-ada-002 |
| **Language Model**   | [OpenAI GPT-4o-mini](https://openai.com/)           | Response generation    |
| **Document Storage** | [Google Drive](https://developers.google.com/drive) | Source documents       |

---

## 📋 Prerequisites

Before getting started, ensure you have the following:

- ✅ **n8n instance** (self-hosted or cloud)
- ✅ **Supabase account** with a project created
- ✅ **OpenAI API key** with access to embeddings and chat models
- ✅ **Google Cloud project** with Drive API enabled

### 1. 🗄️ Database Setup (Supabase)

Run the following SQL script in your Supabase SQL Editor to initialize the vector store:

```sql
-- Enable the vector extension
create extension if not exists vector;

-- Create the documents table
create table if not exists documents (
  id bigserial primary key,
  content text,
  metadata jsonb,
  embedding vector(1536)
);

-- Create an index for faster similarity searches
create index on documents using ivfflat (embedding vector_cosine_ops)
  with (lists = 100);

-- Create the similarity search function
create or replace function match_documents (
  query_embedding vector(1536),
  match_count int default null,
  filter jsonb default '{}'
) returns table (
  id bigint,
  content text,
  metadata jsonb,
  similarity float
)
language plpgsql
as $$
#variable_conflict use_column
begin
  return query
  select
    id,
    content,
    metadata,
    1 - (documents.embedding <=> query_embedding) as similarity
  from documents
  where metadata @> filter
  order by documents.embedding <=> query_embedding
  limit match_count;
end;
$$;
```

### 2. 🔐 Google Cloud Setup

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select an existing one
3. Enable the **Google Drive API**
4. Configure **OAuth2** credentials:
   - Application type: **Web application**
   - Add authorized redirect URI: `https://oauth.n8n.cloud/oauth2/callback`
5. Download the client credentials JSON

> ⚠️ **Important:** Add your email to "Test Users" in the OAuth Consent Screen if the app is in testing mode.

### 3. 🔑 Required Credentials

| Service          | Credential Type | Required Scopes                  |
| ---------------- | --------------- | -------------------------------- |
| **OpenAI**       | API Key         | `chat.completions`, `embeddings` |
| **Supabase**     | API Key + URL   | Database access                  |
| **Google Drive** | OAuth2          | `drive.readonly`                 |

---

## ⚙️ Installation

### Step 1: Import Workflows

1. Download the workflow files:
   - `Doc-Vector-Uploader.json`
   - `RAG-AI-Agent.json`
2. Open your n8n instance
3. Go to **Workflows** → **Import from File**
4. Import both JSON files

### Step 2: Configure Credentials

1. **OpenAI Account**

   - Navigate to **Credentials** → **New Credential** → **OpenAI API**
   - Paste your OpenAI API key

2. **Supabase Account**

   - Add your Supabase project URL
   - Add your service role API key

3. **Google Drive OAuth2**
   - Add your client ID and client secret
   - Complete the OAuth2 flow

### Step 3: Configure Workflow Settings

1. **Doc-Vector-Uploader:**

   - Update the Google Drive folder ID to your target folder
   - The folder ID can be found in the URL: `drive.google.com/drive/folders/{FOLDER_ID}`

2. **RAG-AI-Agent:**
   - Customize the system prompt if needed
   - Adjust the model selection (default: `gpt-4o-mini`)

### Step 4: Activate Workflows

1. Activate **Doc-Vector-Uploader** first
2. Upload a test PDF to your Google Drive folder
3. Verify the document appears in your Supabase `documents` table
4. Activate **RAG-AI-Agent**
5. Test the chat interface

---

## 🧪 Testing

### Test the Ingestion Pipeline

```bash
# Upload a PDF to your monitored Google Drive folder
# Check Supabase for new entries:
SELECT id, LEFT(content, 100) as preview, metadata
FROM documents
ORDER BY id DESC
LIMIT 5;
```

### Test the RAG Agent

Access the chat interface at your n8n webhook URL and try:

- _"What documents do you have access to?"_
- _"Summarize the main topics in my knowledge base"_
- _"Find information about [specific topic]"_

---

## 📊 Configuration Options

### Chunk Settings (Text Splitter)

| Parameter      | Default | Description                           |
| -------------- | ------- | ------------------------------------- |
| `chunkSize`    | 200     | Characters per chunk                  |
| `chunkOverlap` | 20      | Overlapping characters between chunks |

### Agent Settings

| Parameter | Default        | Description                |
| --------- | -------------- | -------------------------- |
| Model     | `gpt-4o-mini`  | OpenAI model for responses |
| Memory    | Buffer Window  | Conversation history type  |
| Tool      | Knowledge Base | Supabase vector search     |

---

## 🔧 Troubleshooting

| Issue                   | Solution                                                   |
| ----------------------- | ---------------------------------------------------------- |
| Documents not uploading | Check Google Drive trigger folder ID and OAuth permissions |
| Empty search results    | Verify embeddings are being generated (check Supabase)     |
| Agent not finding info  | Ensure the `match_documents` function exists in Supabase   |
| Slow responses          | Consider adding an IVFFlat index on the embedding column   |

---

## 🚀 Future Enhancements

- [ ] Support for additional file formats (DOCX, TXT, MD)
- [ ] Multi-folder monitoring
- [ ] Document deletion sync
- [ ] Custom embedding models
- [ ] Streaming responses
- [ ] Admin dashboard for document management

---

## 📝 License

MIT License - Feel free to use and modify for your own projects!

---

## 👤 Author

<p align="center">
  <strong>Ahmed Khalid Elshamy</strong>
</p>

<p align="center">
  <a href="https://github.com/Ak-Elshamy">
    <img src="https://img.shields.io/badge/GitHub-Ak--Elshamy-181717?style=for-the-badge&logo=github" alt="GitHub">
  </a>
  <a href="https://linkedin.com/in/a-elshamy">
    <img src="https://img.shields.io/badge/LinkedIn-a--elshamy-0A66C2?style=for-the-badge&logo=linkedin" alt="LinkedIn">
  </a>
</p>

---

<p align="center">
  ⭐ Star this repository if you found it helpful!
</p>
