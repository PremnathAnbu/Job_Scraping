# 🕵️ Job Scraper — Automated IT Job Alert System

![Python](https://img.shields.io/badge/Python-100%25-blue?style=for-the-badge&logo=python&logoColor=white)
![Selenium](https://img.shields.io/badge/Selenium-Web%20Scraping-43B02A?style=for-the-badge&logo=selenium&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-Database-47A248?style=for-the-badge&logo=mongodb&logoColor=white)
![Gmail](https://img.shields.io/badge/Gmail-Email%20Alerts-EA4335?style=for-the-badge&logo=gmail&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-CI%2FCD-2088FF?style=for-the-badge&logo=githubactions&logoColor=white)

---

## 📌 Overview

**Job Scraper** is a fully automated Python bot that scrapes IT/software job listings from [FreshersWorld](https://www.freshersworld.com/jobs/category/it-software-job-vacancies), stores only new (unique) jobs into **MongoDB**, and sends an **email alert** summarizing how many new jobs were found — all triggered automatically via **GitHub Actions** on a schedule.

The pipeline works in three clean stages:
1. **Collect** job listing URLs across multiple pages using Selenium
2. **Scrape** detailed job info from each URL (title, company, salary, location, etc.)
3. **Alert** via email with the count of newly added jobs

---

## ✨ Features

- 🔍 **Multi-page Link Collection** — Crawls up to 10 pages of IT job listings from FreshersWorld
- 🧑‍💻 **Headless Selenium Scraping** — Browser automation runs in headless mode, fully compatible with GitHub Actions Ubuntu runners
- 🗄️ **MongoDB Deduplication** — Only inserts new jobs; skips already-stored listings using URL as unique key
- 📧 **Gmail Email Alerts** — Sends an SMTP email report with the count of new jobs added each run
- ⏰ **Scheduled Automation** — GitHub Actions runs the bot on a cron schedule automatically
- 🔐 **Environment Variable Security** — Secrets (MongoDB URI, Gmail credentials) managed via `.env` and GitHub Secrets

---

## 🗂️ Project Structure

```
Job_Scraping/
├── .github/
│   └── workflows/          # GitHub Actions workflow (scheduled trigger)
├── scrap1.py               # Stage 1: Collects job URLs from FreshersWorld (10 pages)
├── scraper.py              # Stage 2: Scrapes job details from each URL via Selenium
├── db.py                   # MongoDB connection & unique job insertion logic
├── email_alert.py          # Gmail SMTP email notification sender
├── main.py                 # Orchestrator: runs all 3 stages in sequence
├── job_url.txt             # Text file storing collected job URLs
├── requirements.txt        # Python dependencies
└── .gitignore
```

---

## 🔄 How It Works

```
┌─────────────────────────┐
│  GitHub Actions (cron)  │
│  Triggers main.py       │
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│  scrap1.py              │
│  Crawl FreshersWorld    │
│  Collect job URLs       │
│  Save to job_url.txt    │
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│  scraper.py             │
│  Open each URL in       │
│  headless Chrome        │
│  Extract job details    │
│  → title, company,      │
│    salary, location,    │
│    education, etc.      │
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│  db.py                  │
│  Check MongoDB for      │
│  duplicate job_url      │
│  Insert only NEW jobs   │
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│  email_alert.py         │
│  Send Gmail with count  │
│  of new jobs added      │
└─────────────────────────┘
```

---

## 📦 Scraped Job Data Fields

Each job document stored in MongoDB contains the following fields:

| Field | Description |
|---|---|
| `job_url` | Unique job listing URL (used as deduplication key) |
| `title` | Job role / title |
| `company` | Company name |
| `salary` | Salary range |
| `location` | Primary job location |
| `education` | Minimum education requirement |
| `experience` | Required experience |
| `hiring_process` | Hiring process type |
| `employment_type` | Full-time / Part-time / Contract |
| `locality` | City/district |
| `state` | State |

---

## 🛠️ Tech Stack

| Category | Technology |
|---|---|
| Language | Python 3.8+ |
| Web Scraping | Selenium (headless Chrome) |
| Database | MongoDB Atlas (via PyMongo) |
| Email | Gmail SMTP (smtplib) |
| CI/CD | GitHub Actions |
| Env Management | python-dotenv |

---

## ⚙️ Installation & Setup

### 1. Clone the Repository

```bash
git clone https://github.com/PremnathAnbu/Job_Scraping.git
cd Job_Scraping
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

### 3. Configure Environment Variables

Create a `.env` file in the root directory:

```env
MONGO_URI=your_mongodb_connection_string
EMAIL_USER=your_gmail_address@gmail.com
EMAIL_PASS=your_gmail_app_password
EMAIL_TO=recipient_email@gmail.com
```

> **Note:** For `EMAIL_PASS`, use a [Gmail App Password](https://support.google.com/accounts/answer/185833), not your regular Gmail password.

### 4. Install ChromeDriver (for local runs)

Make sure Google Chrome and ChromeDriver are installed and their versions match. The scraper uses `/usr/bin/chromedriver` by default (Ubuntu path).

For local use on other OS, update the `Service` path in `scrap1.py` and `scraper.py`:

```python
service = Service("/path/to/your/chromedriver")
```

---

## 🚀 Usage

### Run the Full Pipeline Locally

```bash
python main.py
```

This will:
1. Collect job links from FreshersWorld (up to 10 pages) → saved to `job_url.txt`
2. Scrape job details from each link using headless Chrome
3. Insert only new jobs into MongoDB
4. Send a Gmail email with the count of new jobs added

---

## ⏰ GitHub Actions — Scheduled Automation

The `.github/workflows/` directory contains a workflow that triggers `main.py` automatically on a cron schedule (e.g., daily).

### Setting Up GitHub Secrets

Go to your repo → **Settings → Secrets and variables → Actions** and add:

| Secret Name | Value |
|---|---|
| `MONGO_URI` | Your MongoDB Atlas connection string |
| `EMAIL_USER` | Your Gmail address |
| `EMAIL_PASS` | Your Gmail App Password |
| `EMAIL_TO` | Recipient email address |

The workflow installs Chrome + ChromeDriver on the Ubuntu runner automatically, then runs `main.py`.

---

## 🗄️ MongoDB Structure

- **Database:** `jobdata`
- **Collection:** `jobs`
- **Unique key:** `job_url` (prevents duplicate entries on every run)

Each document in the `jobs` collection follows the scraped job data schema described above.

---

## 📧 Email Alert Sample

When new jobs are found, an email is sent like:

```
Subject: Job Scraper Update
Body:    42 New jobs were added to the database.
```

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m 'Add your feature'`
4. Push to the branch: `git push origin feature/your-feature`
5. Open a Pull Request

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

---

## 👤 Author

**Premnath Anbu**
- GitHub: [@PremnathAnbu](https://github.com/PremnathAnbu)

---

> ⭐ If you found this project useful, please give it a star!
