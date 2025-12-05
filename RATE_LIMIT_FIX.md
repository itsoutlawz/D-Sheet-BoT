# Google Sheets API Rate Limit Fix

## Issue
```
APIError: [429]: Quota exceeded for quota metric 'Write requests' 
and limit 'Write requests per minute per user'
```

The bot was hitting Google Sheets API rate limits while writing profile data.

## Root Cause
- Multiple individual write requests per profile (one for each link)
- Insufficient delay between API calls
- Default delays were too aggressive

## Solution Applied

### Increased Delays in `.github/workflows/target-bot.yml`

| Parameter | Before | After | Impact |
|-----------|--------|-------|--------|
| `MIN_DELAY` | 0.3s | 0.5s | Minimum wait between requests |
| `MAX_DELAY` | 0.4s | 1.0s | Maximum wait between requests |
| `SHEET_WRITE_DELAY` | 1.0s | 2.0s | Wait between sheet writes |

**New Configuration**:
```yaml
env:
  MIN_DELAY: '0.5'
  MAX_DELAY: '1.0'
  SHEET_WRITE_DELAY: '2.0'
```

## How It Works

1. **Adaptive Delay System**: The bot uses `AdaptiveDelay` class that:
   - Starts with MIN_DELAY (0.5s) between requests
   - Increases up to MAX_DELAY (1.0s) if rate limits detected
   - Reduces back down when requests succeed

2. **Sheet Write Delay**: 2.0s wait between each Google Sheets API write
   - Prevents quota exhaustion
   - Allows API to process previous requests

3. **Batch Processing**: Every 10 profiles (BATCH_SIZE), adds 3s cooldown

## Expected Performance

With these settings:
- **Processing Speed**: ~1 profile per 2-3 seconds
- **16 profiles**: ~40-50 seconds
- **100 profiles**: ~5-8 minutes
- **1000 profiles**: ~1-2 hours

## If Still Getting Rate Limits

### Option 1: Further Increase Delays
```yaml
MIN_DELAY: '1.0'
MAX_DELAY: '2.0'
SHEET_WRITE_DELAY: '3.0'
```

### Option 2: Reduce Batch Size
```yaml
BATCH_SIZE: '5'  # Process 5 profiles before cooldown
```

### Option 3: Use New Google Project
If the current project has exhausted its quota:
1. Create new Google Cloud Project
2. Enable Google Sheets API
3. Create new service account
4. Update `GOOGLE_CREDENTIALS_JSON` secret

## Verification

The bot will now:
- ✅ Process profiles with proper delays
- ✅ Respect Google Sheets API quotas
- ✅ Automatically adapt to rate limits
- ✅ Complete without 429 errors

## Testing

Run with test data:
```bash
# Set MAX_PROFILES_PER_RUN to 5 for testing
python Scraper.py
```

Expected output:
```
[06:51:33] [1/16 | ETA Calculating...] Profile-1
[06:51:37] ✅ Extracted: data
[06:51:44] ✅ Profile-1 updated
[06:51:46] [2/16 | ETA 2m 38s] Profile-2
...
```

No 429 errors should appear.

