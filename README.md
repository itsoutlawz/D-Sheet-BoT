# DamaDam Target Bot (Single-File) — v3.2.1

Automated bot to scrape DamaDam user profiles and store results in Google Sheets. Runs locally on Windows 10 or via GitHub Actions (scheduled every 1 hour).

## Features

- ✅ Scrapes DamaDam profiles (gender, city, posts, followers, joined date, etc.)
- ✅ Appends new profiles to the last row in Google Sheets (no overwriting)
- ✅ Batch processing with adaptive delays to avoid API rate limits
- ✅ Handles suspended/unverified accounts gracefully
- ✅ Cookie-based session persistence
- ✅ Quantico font formatting applied to all data
- ✅ Windows 10 compatible (no emoji encoding issues)
- ✅ Comprehensive logging with timestamps and progress tracking

## Quick Start (Local)

### Prerequisites

- Python 3.10+
- Chrome/Chromium browser
- Google service account credentials (JSON file)

### Setup

1. **Install dependencies:**

   ```bash
   pip install -r requirements.txt
   ```

2. **Set environment variables (PowerShell):**

   ```powershell
   $env:GOOGLE_SHEET_URL = "https://docs.google.com/spreadsheets/d/<1jn1DroWU8GB5Sc1rQ7wT-WusXK9v4V05ISYHgUEjYZc/edit"
   $env:GOOGLE_APPLICATION_CREDENTIALS = "path/to/credentials.json"
   ```

3. **Run the bot:**

   ```bash
   python Scraper.py
   ```

### Local Defaults

- Username: `0utLawZ` (can override with `DAMADAM_USERNAME`)
- Password: `asdasd` (can override with `DAMADAM_PASSWORD`)

## GitHub Actions Setup

### Prerequisites

1. **Fork this repository** to your GitHub account
2. **Enable GitHub Actions** in your repository settings
3. **Set up required secrets** in your repository settings (Settings → Secrets and variables → Actions → New repository secret):

### Required Secrets

Add these secrets to your GitHub repository settings (Settings → Secrets and variables → Actions):

| Secret Name | Description | Example |
|-------------|-------------|---------|
| `DAMADAM_USERNAME` | Your DamaDam username | `your_username` |
| `DAMADAM_PASSWORD` | Your DamaDam password | `your_password` |
| `DAMADAM_USERNAME_2` | (Optional) Secondary account username | `backup_username` |
| `DAMADAM_PASSWORD_2` | (Optional) Secondary account password | `backup_password` |
| `GOOGLE_SHEET_URL` | Full URL to your Google Sheet | `https://docs.google.com/...` |
| `GOOGLE_CREDENTIALS_JSON` | Service account JSON (entire file content) | `{"type": "service_account", ...}` |

### Google Cloud Setup

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select an existing one
3. Enable the Google Sheets API and Google Drive API
4. Create a service account and download the JSON key file
5. Copy the contents of the JSON file to the `GOOGLE_CREDENTIALS_JSON` secret

### Schedule

The bot is configured to run automatically every hour. You can modify the schedule in [.github/workflows/run_scraper.yml](.github/workflows/run_scraper.yml).

- **Cron schedule:** `0 * * * *` (at minute 0 of every hour)
- **Manual trigger:** Available via GitHub Actions UI

### Workflow Features

- **Automatic Chrome/ChromeDriver setup** - No manual installation needed
- **Secure credentials** - All sensitive data is stored as GitHub Secrets
- **Error handling** - Automatic retries and error logging
- **Artifact collection** - Logs are saved if the run fails
- **Timeout protection** - Job automatically stops after 59 minutes (GitHub Free limit)

### Monitoring

- Check the **Actions** tab in your repository to monitor workflow runs
- Failed runs will be marked with a red X
- Click on a workflow run to see detailed logs
- Logs are automatically deleted after 90 days (GitHub Free limit)

## Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `DAMADAM_USERNAME` | `0utLawZ` | Primary account username |
| `DAMADAM_PASSWORD` | `asdasd` | Primary account password |
| `DAMADAM_USERNAME_2` | `` | Secondary account username |
| `DAMADAM_PASSWORD_2` | `` | Secondary account password |
| `GOOGLE_SHEET_URL` | `` | Google Sheet URL (required) |
| `GOOGLE_APPLICATION_CREDENTIALS` | `` | Path to service account JSON |
| `GOOGLE_CREDENTIALS_JSON` | `` | Service account JSON string (alternative) |
| `MAX_PROFILES_PER_RUN` | `0` | Max profiles to scrape (0 = unlimited) |
| `BATCH_SIZE` | `10` | Profiles per batch before cool-off |
| `MIN_DELAY` | `0.3` | Minimum delay between requests (seconds) |
| `MAX_DELAY` | `0.5` | Maximum delay between requests (seconds) |
| `PAGE_LOAD_TIMEOUT` | `30` | Page load timeout (seconds) |
| `SHEET_WRITE_DELAY` | `1.0` | Delay between sheet writes (seconds) |

## Google Sheets Structure

### ProfilesTarget Sheet

Columns: IMAGE, NICK NAME, TAGS, LAST POST, LAST POST TIME, FRIEND, CITY, GENDER, MARRIED, AGE, JOINED, FOLLOWERS, STATUS, POSTS, PROFILE LINK, INTRO, SOURCE, DATETIME SCRAP

### Target Sheet

Columns: Nickname, Status, Remarks, Source

### Dashboard Sheet

Tracks run statistics: Run#, Timestamp, Profiles, Success, Failed, New, Updated, Unchanged, Trigger, Start, End

## Troubleshooting

### "GOOGLE_SHEET_URL is not set"

- Ensure environment variable is set correctly
- Check for typos in the URL

### "Profile scrape failed" (timeout)

- User account may be banned or suspended
- Check if the profile is accessible manually
- Increase `PAGE_LOAD_TIMEOUT` if needed

### API Rate Limit (429 errors)

- Bot automatically increases delays on rate limits
- Reduce `BATCH_SIZE` or increase `MAX_DELAY` if persistent
- Spread runs across different times

## Development

### Code Structure

- **IMPORTS & CONFIG:** Dependencies and configuration
- **HELPERS:** Utility functions (date conversion, text cleaning, etc.)
- **BROWSER & LOGIN:** Selenium setup and DamaDam login
- **GOOGLE SHEETS:** Google Sheets API integration
- **TARGET PROCESSING:** Fetching pending targets
- **PROFILE SCRAPING:** Profile data extraction
- **MAIN ENTRY:** Main execution loop

### Testing

```bash
# Test with limited profiles
MAX_PROFILES_PER_RUN=5 python Scraper.py

# Test with custom delays
MIN_DELAY=1.0 MAX_DELAY=2.0 python Scraper.py
```

## Version History

- **v3.2.1** (Current)
  - Single-file architecture
  - Append mode (new data to last row)
  - Quantico font formatting
  - Windows 10 compatible (no emoji)
  - 1-hour schedule on GitHub Actions
  - Adaptive delay system
  - Batch processing with cool-off
