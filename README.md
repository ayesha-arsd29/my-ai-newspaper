# 📰 AI Newspaper

An automated, end-to-end data pipeline and AI engineering project that aggregates the latest news from top AI companies (OpenAI, Anthropic) and YouTube creators, enriches the content, and uses LLMs to summarize, personalize, and rank a daily email digest.

## 🎯 Project Overview
Staying up-to-date with the fast-moving AI landscape is difficult. **AI Newspaper** automates the entire workflow: it scrapes raw RSS feeds and YouTube channels, extracts full-text articles and video transcripts, generates AI summaries, and ranks the content based on a personalized user profile. Finally, it formats the top-ranked articles into a beautifully styled HTML email and delivers it directly to the inbox every morning.

This project demonstrates a production-ready approach to **LLM Orchestration**, **Data Engineering**, **Cloud Database Management**, and **CI/CD Automation**.

## 🏗️ System Architecture & Pipeline

The system is built as a modular, 5-stage pipeline orchestrated by `main.py`:

```text
[Stage 1: Ingestion]
RSS Feeds (OpenAI, Anthropic) & YouTube Channels
      │
      ▼
[Stage 2: Enrichment] (Docling & YouTube Transcript API)
Full-text Markdown & Video Transcripts extraction
      │
      ▼
[Stage 3: AI Summarization] (Groq API)
Generating concise digests for each piece of content
      │
      ▼
[Stage 4: Personalization & Curation] (Groq API )
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
- **AI Summarization:** Leverages **Groq (openai/gpt-oss-120b)** for high-speed, low-latency summarization, enforcing strict JSON outputs using Pydantic.
- **Personalized Curation:** An AI Curator Agent scores and ranks content (0.0-10.0) based on a customizable user profile (interests, expertise level, preferences).
- **Beautiful Email Delivery:** Dynamically generates a responsive HTML email template with the AI-written introduction, positive daily affirmations, article summaries, and direct links.
- **Fully Automated CI/CD:** Deployed via **GitHub Actions** to run on a daily cron schedule with zero manual intervention.

## 🛠️ Tech Stack & Skills Demonstrated

**Languages & Frameworks:**
- **Python 3.14** (Primary language)
- **uv** (Modern, ultra-fast Python package and project manager)
- **SQLAlchemy** (ORM for database interactions)
- **Pydantic** (Data validation and strict LLM output schemas)

**AI & Data Extraction:**
- **Groq API** (openai/gpt-oss-120b)
- **Docling** (Advanced document parsing and Markdown conversion)
- **feedparser** (RSS parsing)
- **youtube-transcript-api** (Caption extraction)

**Infrastructure & DevOps:**
- **PostgreSQL** (Relational Database)
- **Supabase** (Cloud Database Hosting)
- **GitHub Actions** (CI/CD and Scheduled Cron Jobs)
- **SMTP / Gmail** (Email delivery)
- **Docker** (Local database setup via Docker Compose)

## 📂 Project Structure

```text
AI-Newspaper/
├── .github/
│   └── workflows/
│       └── daily-pipeline.yml  # GitHub Actions schedule configuration
├── app/
│   ├── agent/                  # AI Agents (DigestAgent, CuratorAgent, EmailAgent)
│   ├── database/               # SQLAlchemy models, connection, and Repository pattern
│   ├── profiles/               # User profile definitions for personalization
│   ├── scrapers/               # Data ingestion (YouTube, OpenAI, Anthropic)
│   ├── services/               # Pipeline stage runners (process_digests, process_email, etc.)
│   └── config.py               # Configuration and YouTube channel IDs
├── .env.example                # Environment variable template
├── main.py                     # Master orchestrator/scheduler script
├── pyproject.toml              # Project dependencies and metadata
├── docker-compose.yml          # Local Postgres setup (optional)
├── README.md
└── uv.lock
```

## 🚀 Getting Started

### 1. Prerequisites
- Python 3.10+
- [uv](https://docs.astral.sh/uv/) installed
- A Supabase account (or local PostgreSQL)
- A Groq API Key (Free tier available at [console.groq.com](https://console.groq.com))
- A Gmail account with an App Password

### 2. Installation
Clone the repository and install dependencies:
```bash
git clone https://github.com/yourusername/AI-Newspaper.git
cd AI-Newspaper
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

## 🏃‍♂️ Running the Pipeline Locally

The entire process is orchestrated by the `main.py` file in the root directory. To run the full end-to-end pipeline locally:

```bash
python main.py
```

Alternatively, if you want to test individual stages, you can run the specific workers inside the `app/services/` folder.

## ☁️ Deployment & Automation (GitHub Actions)

This project is fully automated using GitHub Actions. 

The workflow file (`.github/workflows/daily-pipeline.yml`) is configured to:
1. Spin up a fresh Ubuntu runner (with 7GB of RAM to handle Docling's memory needs).
2. Install `uv` and sync all dependencies.
3. Securely inject environment variables using GitHub Repository Secrets.
4. Execute `main.py` on a daily cron schedule (e.g., `0 8 * * *` for 8:00 AM UTC).

To configure this for your own fork:
1. Go to your repository on GitHub.
2. Navigate to **Settings > Secrets and variables > Actions**.
3. Add all the variables from your local `.env` file as **New repository secrets**.
4. The pipeline will now run automatically every day without any manual intervention.

## 🧠 Engineering Decisions & Challenges

- **The "RSS 15-Video Limit":** YouTube's RSS feeds only provide the 15 most recent videos. The pipeline handles this by running frequently enough to catch new uploads before they fall off the feed, while the PostgreSQL database acts as a permanent historical archive to prevent re-scraping.
- **Cost and Rate-Limit Optimization:** Processing full-text articles via LLMs can get expensive. The system uses a `__UNAVAILABLE__` marker for failed transcripts to avoid retrying dead links, and truncates content to 8,000 characters per LLM call to optimize context windows and cost.
- **Reliable JSON Parsing:** Instead of relying on prompt engineering alone to get structured data from the LLM, the system uses Pydantic models and Groq's JSON mode to guarantee that the AI returns exact schemas, eliminating parsing errors.
- **Overcoming Free-Tier Memory Limits:** Deploying Docling (which requires 2-4GB RAM) on free web hosting platforms (like Render or Hugging Face Spaces) resulted in Out-Of-Memory crashes. The architecture was migrated to GitHub Actions, which provides 7GB of RAM for free, ensuring reliable daily executions.

