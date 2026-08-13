# Agent AI — RAG & MCP-Based Document Q&A Agent

A full-stack AI document question-answering application that combines **Retrieval-Augmented Generation (RAG)** with **Model Context Protocol (MCP)** based web search.

Users can upload PDF documents, ask questions about their contents, and receive:

* **RAG Answer** — generated from the uploaded PDF
* **MCP Answer** — generated using web search through an MCP server
* **Voice Interaction** — speech recognition and text-to-speech support

---

## Tech Stack

### Frontend

* React
* Ant Design
* Axios
* `react-speech-recognition`
* `speak-tts`

### Backend

* Node.js
* Express.js
* Multer
* CORS
* dotenv

### AI / RAG

* OpenAI API
* GPT-5
* OpenAI Embeddings
* LangChain
* PDFLoader
* RecursiveCharacterTextSplitter
* MemoryVectorStore

### MCP / Web Search

* Model Context Protocol (MCP)
* MCP Client / Server
* SerpAPI
* Zod

---

## Project Architecture

```text
React Frontend
      |
      | HTTP
      v
Express.js Backend
      |
      +----------------------+
      |                      |
      v                      v
   RAG Pipeline          MCP Client
      |                      |
      v                      v
Uploaded PDF            MCP Server
      |                      |
PDFLoader                   |
      |                  search_web
Text Splitter                |
      |                      v
Embeddings               SerpAPI
      |
Vector Store
      |
Retriever
      |
GPT-5
```

The `/chat` endpoint runs both pipelines and returns:

```json
{
  "ragAnswer": "...",
  "mcpAnswer": "..."
}
```

---

## Project Structure

```text
agentai/
│
├── package.json
├── package-lock.json
├── .gitignore
│
├── public/
│   └── ...
│
├── src/
│   ├── App.js
│   ├── index.js
│   │
│   └── components/
│       ├── ChatComponent.js
│       ├── PdfUploader.js
│       └── RenderQA.js
│
└── server/
    ├── package.json
    ├── package-lock.json
    ├── .env
    ├── chat.js
    ├── chat-mcp.js
    ├── mcp-server.js
    ├── server.js
    │
    └── uploads/
```

---

## How the RAG Pipeline Works

The RAG pipeline processes an uploaded PDF as follows:

```text
PDF
 ↓
PDFLoader
 ↓
Text
 ↓
RecursiveCharacterTextSplitter
 ↓
Document Chunks
 ↓
OpenAI Embeddings
 ↓
MemoryVectorStore
 ↓
Retriever
 ↓
Relevant Document Chunks
 ↓
Prompt
 ↓
GPT-5
 ↓
Answer
```

The document is split into chunks of approximately **500 characters**, embedded using OpenAI embeddings, and stored in an in-memory vector store.

When the user asks a question, relevant chunks are retrieved and supplied to GPT-5 as context.

---

## How the MCP Pipeline Works

The MCP architecture is:

```text
React
  ↓
Express server.js
  ↓
MCP Client (chat-mcp.js)
  ↓
MCP Server (mcp-server.js)
  ↓
search_web tool
  ↓
SerpAPI
  ↓
Web Search Results
  ↓
GPT-5
  ↓
MCP Answer
```

The MCP server exposes the tool:

```text
search_web
```

which performs Google searches through SerpAPI.

---

## Environment Variables

Create:

```text
server/.env
```

Add:

```env
OPENAI_API_KEY=YOUR_OPENAI_API_KEY
SERPAPI_KEY=YOUR_SERPAPI_KEY
```

Do not commit this file.

Make sure the root `.gitignore` contains:

```gitignore
server/.env
```

---

## Installation

### 1. Install frontend dependencies

From the project root:

```bash
npm install
```

### 2. Install backend dependencies

Move into the backend:

```bash
cd server
```

Run:

```bash
npm install --legacy-peer-deps
```

Then return to the root:

```bash
cd ..
```

---

## Running the Application

The root `package.json` contains a development command that starts both the React frontend and Express backend.

Run:

```bash
npm run dev
```

Alternatively, run them separately.

### Frontend

From the project root:

```bash
npm start
```

### Backend

From:

```text
server/
```

run:

```bash
npm start
```

The Express backend runs on:

```text
http://localhost:5001
```

---

## API Endpoints

### Upload PDF

```http
POST /upload
```

Local endpoint:

```text
http://localhost:5001/upload
```

The uploaded PDF is saved inside:

```text
server/uploads/
```

---

### Ask a Question

```http
GET /chat?question=<your question>
```

Example:

```text
http://localhost:5001/chat?question=What is this document about?
```

The response contains both the RAG and MCP answers:

```json
{
  "ragAnswer": "Answer based on the uploaded PDF",
  "mcpAnswer": "Answer based on web search"
}
```

---

## Voice Interface

The application also supports conversational voice interaction using:

```text
react-speech-recognition
```

for speech-to-text and:

```text
speak-tts
```

for text-to-speech.

The UI includes a **Chat Mode** that allows the user to:

1. Start recording
2. Convert speech into text
3. Send the recognized question to the backend
4. Receive the RAG response
5. Read the answer aloud
6. Resume listening

---

## Main Frontend Components

### `PdfUploader.js`

Handles PDF uploads and sends them to:

```text
POST /upload
```

using Axios and Ant Design's upload component.

### `ChatComponent.js`

Handles:

* User questions
* Backend requests
* Loading state
* Voice recognition
* Text-to-speech
* Chat Mode

### `RenderQA.js`

Displays:

* User question
* RAG answer from the uploaded document
* MCP answer from web search

### `App.js`

Combines the uploader, question interface, and answer display into the main application.

---

## Main Backend Files

### `chat.js`

Implements the RAG pipeline:

```text
PDF → Chunking → Embeddings → Vector Store → Retrieval → GPT-5
```

### `mcp-server.js`

Creates an MCP server and registers the:

```text
search_web
```

tool backed by SerpAPI.

### `chat-mcp.js`

Acts as the MCP client.

It launches/connects to the MCP server, calls `search_web`, and uses GPT-5 to summarize the search results.

### `server.js`

Runs the Express backend and exposes:

```text
POST /upload
GET /chat
```

The `/chat` endpoint executes both:

```js
chat(...)
```

and:

```js
chatMCP(...)
```

before returning both answers to the frontend.

---

## Optional Deployment

The original project design suggests:

```text
Frontend → AWS Amplify
Backend  → AWS App Runner
```

Deployment configuration is not included in the local project setup.

---

## Summary

Agent AI demonstrates a full-stack AI application combining:

* PDF document ingestion
* Retrieval-Augmented Generation
* Semantic retrieval
* OpenAI embeddings
* GPT-based question answering
* React frontend development
* Express REST APIs
* Voice interaction
* Model Context Protocol
* MCP tool calling
* Web search integration

The result is a document Q&A application capable of answering questions using both **uploaded documents** and **external web information**.
