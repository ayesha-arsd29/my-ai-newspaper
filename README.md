
# 📰 AI News Aggregator & Curator

An automated, end-to-end data pipeline and AI engineering project that aggregates the latest news from top AI companies (OpenAI, Anthropic) and YouTube creators, enriches the content, and uses LLMs to summarize, personalize, and rank a daily email digest.

## 🎯 Project Overview
Staying up-to-date with the fast-moving AI landscape is difficult. This project automates the entire workflow: it scrapes raw RSS feeds and YouTube channels, extracts full-text articles and video transcripts, generates AI summaries, and ranks the content based on a personalized user profile. Finally, it formats the top-ranked articles into a beautifully styled HTML email and delivers it directly to the inbox.

This project demonstrates a production-ready approach to **LLM Orchestration**, **Data Engineering**, and **Cloud Database Management**.

## 🏗️ System Architecture & Pipeline

The system is built as a modular, 4-stage pipeline:

```text
[Stage 1: Ingestion]
RSS Feeds (OpenAI, Anthropic) & YouTube Channels
      │
      ▼
[Stage 2: Enrichment] (Docling & YouTube Transcript API)
Full-text Markdown & Video Transcripts extraction
      │
      ▼
[Stage 3: AI Summarization] (Groq API / Llama-3)
Generating concise digests for each piece of content
      │
      ▼
[Stage 4: Personalization & Curation] (Groq API / Llama-3)
Ranking digests based on user profile & generating email intro
      │
      ▼
[Stage 5: Delivery] (SMTP)
HTML Email Digest sent to user
```

## ✨ Key Features

- **Multi-Source Ingestion:** Scrapes RSS feeds (OpenAI, Anthropic) and YouTube channel feeds using `feedparser` and the YouTube RSS API.
- **Intelligent Enrichment:** Uses IBM's **Docling** to convert complex web articles into clean Markdown, and `youtube-transcript-api` to extract video captions.
- **Stateful Database Tracking:** Uses SQLAlchemy to track the state of every article (e.g., "needs transcript," "needs summary," "needs ranking") to avoid duplicating work and wasting API credits.
- **AI Summarization:** Leverages **Groq (Llama-3.3-70B)** for high-speed, low-latency summarization, enforcing strict JSON outputs using Pydantic.
- **Personalized Curation:** An AI Curator Agent scores and ranks content (0.0-10.0) based on a customizable user profile (interests, expertise level, preferences).
- **Beautiful Email Delivery:** Dynamically generates a responsive HTML email template with the AI-written introduction, article summaries, and direct links.

## 🛠️ Tech Stack & Skills Demonstrated

**Languages & Frameworks:**
- **Python 3.14** (Primary language)
- **uv** (Modern, ultra-fast Python package and project manager)
- **SQLAlchemy** (ORM for database interactions)
- **Pydantic** (Data validation and strict LLM output schemas)

**AI & Data Extraction:**
- **Groq API** (Llama-3.3-70B & GPT-OSS models)
- **Docling** (Advanced document parsing and Markdown conversion)
- **feedparser** (RSS parsing)
- **youtube-transcript-api** (Caption extraction)

**Infrastructure & DevOps:**
- **PostgreSQL** (Relational Database)
- **Supabase** (Cloud Database Hosting)
- **SMTP / Gmail** (Email delivery)
- **Docker** (Local database setup via Docker Compose)
- **Git / GitHub** (Version control)

## 📂 Project Structure

```text
my-ai-newspaper/
├── app/
│   ├── agent/               # AI Agents (DigestAgent, CuratorAgent, EmailAgent)
│   ├── database/            # SQLAlchemy models, connection, and Repository pattern
│   ├── profiles/            # User profile definitions for personalization
│   ├── scrapers/            # Data ingestion (YouTube, OpenAI, Anthropic)
│   ├── services/            # Pipeline runners (process_digests, process_email, etc.)
│   └── config.py            # Configuration and YouTube channel IDs
├── .env.example             # Environment variable template
├── pyproject.toml           # Project dependencies and metadata
├── docker-compose.yml       # Local Postgres setup (optional)
├── README.md
└── runner.py                # Main orchestration script for Stage 1
```

## 🚀 Getting Started

### 1. Prerequisites
- Python 3.10+
- [uv](https://docs.astral.sh/uv/) installed
- A Supabase account (or local PostgreSQL)
- A Groq API Key (Free tier available at console.groq.com)
- A Gmail account with an App Password

### 2. Installation
Clone the repository and install dependencies:
```bash
git clone https://github.com/yourusername/my-ai-newspaper.git
cd my-ai-newspaper
uv sync
```

### 3. Environment Variables
Copy the `.env.example` file to `.env` and fill in your credentials:
```bash
cp .env.example .env
```

```env
# Database (Supabase)
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your_supabase_password
POSTGRES_HOST=db.your-project.supabase.co
POSTGRES_PORT=5432
POSTGRES_DB=postgres

# AI
GROQ_API_KEY=gsk_your_groq_api_key

# Email
MY_EMAIL=your.email@gmail.com
APP_PASSWORD=your_16_letter_app_password
```

### 4. Initialize the Database
Create the necessary tables in your Supabase database:
```bash
python -m app.database.create_table
```

## 🏃‍♂️ Running the Pipeline

Because the pipeline involves heavy processing and API calls, it is split into distinct, runnable stages.

**Stage 1: Fetch new metadata**
```bash
python -m app.runner
```

**Stage 2: Enrich content (Fetch transcripts & markdown)**
```bash
python app/services/process_youtube_transcripts.py
python app/services/process_anthropic_markdown.py
```

**Stage 3: Generate AI Digests**
```bash
python app/services/process_digests.py
```

**Stage 4: Rank and Send Email**
```bash
python app/services/process_email.py
```

## 🧠 Engineering Decisions & Challenges

- **The "RSS 15-Video Limit":** YouTube's RSS feeds only provide the 15 most recent videos. The pipeline handles this by running frequently enough to catch new uploads before they fall off the feed, while the PostgreSQL database acts as a permanent historical archive to prevent re-scraping.
- **Cost and Rate-Limit Optimization:** Processing full-text articles via LLMs can get expensive. The system uses a `__UNAVAILABLE__` marker for failed transcripts to avoid retrying dead links, and truncates content to 8,000 characters per LLM call to optimize context windows and cost.
- **Reliable JSON Parsing:** Instead of relying on prompt engineering alone to get structured data from the LLM, the system uses Pydantic models and Groq's JSON mode to guarantee that the AI returns exact schemas, eliminating parsing errors.

---

### Author
**Ayesha** - *AI Engineer*
- [LinkedIn](https://linkedin.com/in/yourprofile)
- [GitHub](https://github.com/yourusername)

*(Feel free to connect with me to discuss AI engineering, LLM orchestration, or this project!)*