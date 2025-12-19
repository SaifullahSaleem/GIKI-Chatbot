# GIKI Data Chatbot 🤖

A sophisticated AI-powered chatbot that scrapes data from the Ghulam Ishaq Khan Institute (GIKI) website, indexes it using vector embeddings, and provides intelligent question-answering capabilities using RAG (Retrieval-Augmented Generation) architecture.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Components](#components)
- [API Endpoints](#api-endpoints)
- [Technologies Used](#technologies-used)
- [Troubleshooting](#troubleshooting)
- [Future Enhancements](#future-enhancements)

## 🎯 Overview

The GIKI Data Chatbot is an end-to-end intelligent assistant that:
1. **Scrapes** institutional data from GIKI's website (faculty profiles, research projects, news, labs)
2. **Processes** and embeds the data using sentence transformers
3. **Stores** embeddings in Pinecone vector database for efficient semantic search
4. **Retrieves** relevant context based on user queries
5. **Generates** accurate answers using Groq's LLM API

This project demonstrates a complete RAG pipeline implementation with web scraping, vector search, and LLM integration.

## ✨ Features

- **Intelligent Web Scraping**: Custom Scrapy spider that crawls GIKI website and extracts structured data
- **Semantic Search**: Uses `all-MiniLM-L6-v2` embeddings for accurate similarity matching
- **Vector Database**: Pinecone integration for scalable, fast retrieval
- **LLM Integration**: Groq API (compound-mini model) for natural language generation
- **Robust Error Handling**: Implements retry logic with exponential backoff for rate limits
- **Context Optimization**: Smart truncation to avoid payload size issues
- **Web Interface**: Clean, responsive Flask-based chat UI
- **MCP Protocol Support**: Model Context Protocol wrapper for external integrations
- **Markdown Support**: Rich text formatting in bot responses

## 🏗️ Architecture

```
┌─────────────┐      ┌──────────────┐      ┌───────────────┐
│   Scrapy    │─────▶│  JSON Data   │─────▶│  Embedding    │
│   Spider    │      │   Storage    │      │   Generator   │
└─────────────┘      └──────────────┘      └───────┬───────┘
                                                    │
                                                    ▼
┌─────────────┐      ┌──────────────┐      ┌───────────────┐
│    User     │─────▶│    Flask     │─────▶│   Pinecone    │
│   Query     │      │   Backend    │      │ Vector Store  │
└─────────────┘      └──────┬───────┘      └───────────────┘
                            │
                            ▼
                     ┌──────────────┐
                     │  Groq LLM    │
                     │  (API Call)  │
                     └──────────────┘
```

### Data Flow

1. **Scraping Phase**: Scrapy crawls GIKI website → Extracts structured data → Saves to `giki_data.json`
2. **Indexing Phase**: Read JSON → Generate embeddings → Upsert to Pinecone index
3. **Query Phase**: User question → Embed query → Semantic search in Pinecone → Retrieve top-k matches → Send context + query to Groq → Return formatted answer

## 📁 Project Structure

```
giki-data-chatbot/
│
├── chatbot/                    # Flask web application
│   ├── __init__.py
│   └── app.py                  # Main Flask app with chat logic
│
├── embeddings/                 # Embedding and indexing logic
│   ├── __init__.py
│   └── embed.py                # Script to embed data and upsert to Pinecone
│
├── scraper/                    # Web scraping module
│   ├── __init__.py
│   ├── giki_spider.py          # Scrapy spider for GIKI website
│   └── run_spider.py           # Script to execute the spider
│
├── static/                     # Frontend assets
│   ├── giki-logo.png           # GIKI logo
│   ├── script.js               # Client-side JavaScript
│   └── style.css               # Styling
│
├── templates/                  # HTML templates
│   └── index.html              # Chat interface
│
├── config.py                   # API keys and configuration
├── create_index.py             # Script to create Pinecone index
├── mcp.py                      # MCP protocol wrapper server
├── requirements.txt            # Python dependencies
├── giki_data.json              # Scraped data (generated)
└── README.md                   # This file
```

## 🔧 Prerequisites

- Python 3.8+
- Pinecone account (free tier available)
- Groq API key
- Internet connection for scraping and API calls

## 📦 Installation

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/giki-data-chatbot.git
cd giki-data-chatbot
```

### 2. Create Virtual Environment

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1  # Windows PowerShell
# or
source venv/bin/activate     # Linux/Mac
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Download NLTK Data (if needed)

```python
python -c "import nltk; nltk.download('punkt')"
```

## ⚙️ Configuration

### 1. Update `config.py`

Replace the placeholder API keys with your actual credentials:

```python
# config.py
OPENAI_API_KEY = "your-openai-key"  # Not actively used, but present
PINECONE_API_KEY = "your-pinecone-api-key"
PINECONE_ENVIRONMENT = "us-east1-aws"  # or your region
PINECONE_INDEX_NAME = "giki-data-index"
```

### 2. Update Groq API Key in `chatbot/app.py`

```python
GROQ_API_KEY = "your-groq-api-key"
```

**⚠️ Security Note**: Never commit API keys to version control. Use environment variables or `.env` files in production.

## 🚀 Usage

### Step 1: Scrape GIKI Website

Run the Scrapy spider to collect data:

```bash
python scraper/run_spider.py
```

This creates `giki_data.json` with scraped content (faculty profiles, research projects, news, labs).

### Step 2: Create Pinecone Index

Initialize the vector database index:

```bash
python create_index.py
```

### Step 3: Generate and Upload Embeddings

Process the scraped data and upload to Pinecone:

```bash
python embeddings/embed.py --input giki_data.json
```

**Options**:
- `--input, -i`: Path to JSON file (default: `giki_data.json`)
- `--index, -x`: Pinecone index name (default: from `config.py`)
- `--batch, -b`: Batch size for upserts (default: 100)
- `--metric, -m`: Distance metric (`cosine` or `euclidean`)

### Step 4: Run the Chatbot

Start the Flask web server:

```bash
python chatbot/app.py
```

The chatbot will be available at `http://localhost:5000`

### Step 5 (Optional): Run MCP Server

For Model Context Protocol integration:

```bash
python mcp.py --port 5001
```

This exposes the `/mcp` endpoint for external systems.

## 🧩 Components

### 1. Web Scraper (`scraper/giki_spider.py`)

- **Framework**: Scrapy
- **Target**: GIKI website (faculty, research, news, labs)
- **Features**:
  - Intelligent page type detection
  - BeautifulSoup for HTML parsing
  - Follows internal links recursively
  - Respects `robots.txt`
  - Extracts structured data (name, research areas, titles, descriptions)

**Key Methods**:
- `parse()`: Entry point, routes to specialized parsers
- `parse_faculty_list()`: Extracts faculty profiles
- `parse_research_projects()`: Scrapes research projects
- `parse_news_listing()`: Collects news articles
- `parse_labs_and_research()`: Gets lab information

### 2. Embedding Engine (`embeddings/embed.py`)

- **Model**: `all-MiniLM-L6-v2` (384-dimensional embeddings)
- **Vector DB**: Pinecone
- **Features**:
  - Batch processing (default: 100 vectors/batch)
  - Metadata truncation to avoid size limits
  - Error handling with retry logic
  - Progress tracking

**Process**:
1. Load JSON data
2. Extract text and metadata for each entry
3. Generate embeddings using SentenceTransformers
4. Upsert vectors to Pinecone in batches

### 3. Flask Chatbot (`chatbot/app.py`)

- **Framework**: Flask
- **LLM**: Groq API (compound-mini model)
- **Search**: Pinecone semantic search

**Key Functions**:

#### `get_embedding(text)`
Generates 384-dimensional vector for text using SentenceTransformers.

#### `semantic_search(query, top_k)`
Retrieves top-k most similar documents from Pinecone.

#### `ask_groq_llm(context, question)`
Sends context and question to Groq API with:
- Automatic context truncation
- Retry logic for rate limits (429)
- Payload size error handling (413)
- Exponential backoff

#### `process_query(query, top_k)`
Main pipeline:
1. Semantic search for relevant chunks
2. Build context (truncated to 2000 chars)
3. Send to LLM
4. Format response as Markdown HTML

**Tunable Parameters** (in `app.py`):
```python
MAX_MATCH_TEXT_CHARS = 800   # Per-match truncation
MAX_CONTEXT_CHARS = 2000     # Total context sent to LLM
DEFAULT_TOP_K = 3            # Number of Pinecone matches
LLM_MAX_TOKENS = 250         # LLM response length
RETRY_MAX = 3                # Retry attempts
```

### 4. Web Interface

**Frontend Stack**:
- HTML5 + Vanilla JavaScript
- CSS3 with responsive design
- Fetch API for async requests

**Features**:
- Real-time chat interface
- Markdown rendering support
- Auto-scroll with "New messages" indicator
- Loading states
- Error handling

### 5. MCP Server (`mcp.py`)

A lightweight wrapper that:
- Accepts POST requests at `/mcp` endpoint
- Forwards queries to `process_query()`
- Returns MCP-compatible JSON responses

## 🌐 API Endpoints

### `GET /`
Renders the chat interface.

**Response**: HTML page

---

### `POST /chat`
Main chat endpoint for web interface.

**Request**:
```json
{
  "query": "What research areas does GIKI focus on?",
  "top_k": 3  // optional, default: 3
}
```

**Response**:
```json
{
  "answer": "<p>GIKI focuses on...</p>"  // Markdown-rendered HTML
}
```

**Error Response**:
```json
{
  "error": "Internal Server Error"
}
```

---

### `POST /mcp`
MCP protocol endpoint (when running `mcp.py`).

**Request**:
```json
{
  "input": "Tell me about GIKI faculty",
  "query": "alternative field name"
}
```

**Response**:
```json
{
  "status": "ok",
  "answer": "<p>GIKI has faculty members...</p>",
  "context": null
}
```

## 🛠️ Technologies Used

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Web Scraping** | Scrapy, BeautifulSoup4 | Extract data from GIKI website |
| **Embeddings** | SentenceTransformers | Generate semantic vectors |
| **Vector DB** | Pinecone | Store and search embeddings |
| **LLM** | Groq API | Generate natural language responses |
| **Backend** | Flask | Web server and API |
| **Frontend** | HTML/CSS/JS | User interface |
| **Text Processing** | Markdown, Python | Format responses |

## 🐛 Troubleshooting

### Issue: "Index does not exist"
**Solution**: Run `python create_index.py` before embedding data.

---

### Issue: 429 Rate Limit Errors
**Solution**: The app has built-in retry logic. If persistent:
- Check your Groq API quota
- Reduce request frequency
- Increase `RETRY_BASE_DELAY` in `app.py`

---

### Issue: 413 Payload Too Large
**Solution**: The app automatically reduces context size. If it persists:
- Decrease `MAX_CONTEXT_CHARS` in `app.py`
- Reduce `top_k` in queries
- Shorten `MAX_MATCH_TEXT_CHARS`

---

### Issue: Empty Search Results
**Causes**:
- Pinecone index is empty → Run embedding script
- Query embedding mismatch → Check model consistency
- Index name mismatch → Verify `config.py`

---

### Issue: Scraper Returns No Data
**Solution**:
- Check internet connection
- Verify GIKI website is accessible
- Update CSS selectors in `giki_spider.py` if website structure changed
- Check `robots.txt` compliance

---

### Issue: Import Errors
**Solution**:
```bash
pip install -r requirements.txt --upgrade
```

## 📈 Performance Tuning

### For Faster Responses
- Reduce `DEFAULT_TOP_K` (fewer chunks to process)
- Decrease `MAX_CONTEXT_CHARS` (smaller LLM payload)
- Use faster embedding model (trade-off: accuracy)

### For Better Accuracy
- Increase `top_k` (more context)
- Use larger `MAX_CONTEXT_CHARS` (more information)
- Fine-tune embedding model on domain data

### For Handling Large Datasets
- Increase `BATCH_SIZE` in `embed.py` (100-200 recommended)
- Use Pinecone's namespaces for organization
- Implement pagination for results

## 🔐 Security Best Practices

1. **Never commit API keys** to version control
2. Use **environment variables** for sensitive data:
   ```python
   import os
   PINECONE_API_KEY = os.getenv("PINECONE_API_KEY")
   ```
3. Add `.env` to `.gitignore`
4. Use **HTTPS** in production
5. Implement **rate limiting** on Flask endpoints
6. Sanitize user inputs to prevent injection attacks

## 🚧 Future Enhancements

- [ ] Add user authentication and session management
- [ ] Implement conversation history storage
- [ ] Add multi-turn conversation context
- [ ] Create admin dashboard for monitoring
- [ ] Implement A/B testing for different LLM models
- [ ] Add support for file upload (PDFs, docs)
- [ ] Integrate speech-to-text for voice queries
- [ ] Deploy to cloud (AWS, Azure, GCP)
- [ ] Add caching layer (Redis) for frequent queries
- [ ] Implement feedback mechanism for answer quality
- [ ] Create mobile app version
- [ ] Add multilingual support

## 📝 Development Notes

### Adding New Data Sources

1. Update `start_urls` in `giki_spider.py`
2. Add specialized parser method
3. Update heuristics in `parse()` method
4. Define metadata structure in `embed.py`

### Changing Embedding Model

1. Update model name in both `embed.py` and `app.py`
2. Check dimension compatibility with Pinecone index
3. Re-create index with new dimension if needed
4. Re-run embedding script

### Switching LLM Provider

1. Update API endpoint and authentication in `app.py`
2. Modify request/response format in `ask_groq_llm()`
3. Adjust prompt template if needed

## 📄 License

This project is for educational purposes. Ensure compliance with GIKI's `robots.txt` and terms of service when scraping.

## 👥 Contributing

Contributions are welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Make your changes with clear commit messages
4. Submit a pull request

## 📧 Contact

For questions or issues, please open a GitHub issue or contact the maintainers.

---

**Built with ❤️ for the GIKI community**