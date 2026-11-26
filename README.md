# 📧 Automated HR Email Outreach Workflow (n8n)

An end-to-end **n8n automation workflow** designed to streamline job and internship applications. By filling out a single web form, this workflow automatically searches the web for target companies in your preferred city, uses AI to extract verified HR contact details, attaches your resume, and prepares personalized email drafts directly in your Gmail account.

## ✨ Key Features

* **Interactive Form Trigger:** Clean web interface to input candidate details, target role, location preferences, resume link, and email templates.

* **Automated Resume Fetching:** Automatically downloads your PDF resume directly from a public Google Drive link.

* **Smart Search Query Generation:** Dynamically builds up to 150 targeted search queries combining your desired role, experience level, city/area, and industry sectors.

* **AI-Powered Data Extraction:** Uses **Firecrawl** for web searching and an **OpenAI LLM** (`Information Extractor`) to parse company names, HR emails (public or domain-inferred), stipend details, and work-week policies while filtering out consultancies and job boards.

* **Deduplication & Ranking:** Validates email syntax, removes duplicates, filters by paid/unpaid preferences, and prioritizes leads with verified emails and stipends.

* **CSV Lead Export:** Compiles all extracted company leads into a downloadable `.csv` report.

* **Safe Gmail Draft Creation:** Creates up to 100 personalized Gmail drafts with your resume attached—giving you full control to review before hitting send.

* **Executive Summary Report:** Generates a final HTML summary draft sent to your own email containing a status table of all created drafts alongside the attached CSV file.

## 🛠️ Tech Stack & Prerequisites

To run this workflow, you will need an active **n8n** instance (Cloud or Self-Hosted) and credentials for the following services:

1. **Google Drive:** Publicly accessible link (`Anyone with the link`) to your PDF resume.

2. **Firecrawl API:** Used by the `Search Web` node to perform deep web searches.

3. **OpenAI API:** Used by the `Extract Companies and HR Emails` node to structure unstructured search data.

4. **Gmail OAuth2:** Used to generate personalized outreach drafts and the final summary draft.

## 🔄 Workflow Architecture (Step-by-Step)

```
[Start Form] ➔ [Download Resume PDF] ➔ [Build Search Queries] ➔ [Search Web (Firecrawl)]
                                                                         │
[Create Summary Draft]  [Build Summary]  [Create Gmail Drafts]  [Extract & Clean Leads (AI)]

```

### 1. Form Submission (`Start Form`)

Captures user inputs via an n8n Form Trigger:

* **Candidate Info:** Full Name, Email ID, Experience Level (`Fresher` / `Experienced`), and Google Drive Resume Link.

* **Job Preferences:** Target Role (e.g., *AI ML Intern*), Opportunity Type (`Internship` / `Job`), Stipend Preference (`Paid` / `Unpaid` / `Any`), City, and Preferred Location (e.g., *Whole City* or specific neighborhoods).

* **Email Template:** Custom Subject and Message body supporting dynamic `{company-name}` placeholders.

### 2. Resume Retrieval (`Download Resume PDF`)

Extracts the File ID from your Google Drive URL and downloads the PDF binary into the workflow memory so it can be attached to every email draft.

### 3. Query Generation & Web Search (`Build Search Queries` ➔ `Search Web`)

* **Code Node:** Combines your role, city, and sector keywords (IT, startups, agencies, etc.) into targeted search strings.

* **Firecrawl Node:** Executes batched web searches to gather live hiring pages, career portals, and company directories.

### 4. AI Extraction & Filtering (`Prepare Search Text` ➔ `Extract Companies` ➔ `Keep Valid Emails`)

* **Information Extractor (OpenAI):** Reads search snippets and extracts structured JSON containing `company_name`, `hr_email`, `email_type` (`public` vs `guessed`), `paid_internship`, `stipend_info`, `five_day_week`, and `source_url`.

* **Split & Filter:** Splits the array into individual company records and uses Regex validation to discard invalid email formats.

### 5. Data Ranking & CSV Generation (`Prepare Rows with Resume` ➔ `Create CSV File`)

* Removes duplicate email addresses and filters results based on your stipend preference.

* Ranks leads so verified public emails and paid internships appear first (capped at 100 leads per run).

* Generates a structured `.csv` file containing all lead details.

### 6. Personalized Draft Creation (`Create Internship Draft`)

Loops through each qualifying company, replaces `{company-name}` in your subject and body with the actual company name, attaches your resume PDF, and saves the email in your **Gmail Drafts** folder.

### 7. Summary Report (`Build Summary` ➔ `Create Summary Draft`)

Compiles an HTML table showing every company processed, stipend details, and draft status (`DRAFTED` or `FAILED`). Creates one final draft addressed to your own email with the `.csv` lead list attached.

## 🚀 Setup & Installation

1. **Download the Workflow:**
   Clone this repository or download the `Email Outreach to HR Manager.json` file.

2. **Import into n8n:**

   * Open your n8n dashboard and click **Add Workflow**.

   * Click the **⋮** (three dots) menu in the top right and select **Import from File...**

   * Select `Email Outreach to HR Manager.json`.

3. **Configure Credentials:**

   * Open the **Search Web** node and link your **Firecrawl API** credentials.

   * Open the **OpenAI Chat Model** node and link your **OpenAI API** credentials.

   * Open both **Create Internship Draft** and **Create Summary Draft** nodes and connect your **Gmail OAuth2** account.

4. **Run the Workflow:**

   * Click **Test Workflow** (or activate the workflow and open the Production Form URL).

   * Fill out your details, submit the form, and check your Gmail **Drafts** folder once execution completes!

|  |  | 

## ⚠️ Best Practices & Notes

* **Review Before Sending:** This workflow intentionally creates **Drafts** rather than sending emails automatically. Always review guessed email addresses (`hr@domain.com` or `careers@domain.com`) and personalize your pitch before sending.

* **Google Drive Permissions:** Ensure your resume link sharing setting is set to **"Anyone with the link"**; otherwise, the HTTP Request node will download an HTML login page instead of your PDF.

* **API Rate Limits:** Searching up to 150 queries consumes Firecrawl and OpenAI tokens. Adjust the `.slice(0, 150)` limit in the `Build Search Queries` node if you want faster, smaller runs.