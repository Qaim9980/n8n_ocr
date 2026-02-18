# n8n Wireless Mouse Scraper - Complete Setup Guide

## 📋 Workflow Overview

This workflow scrapes wireless mouse products from Daraz.pk with complete anti-bot protection and saves data to Google Sheets.

```
Manual Trigger 
    ↓
Set Variables (search keyword, base URL)
    ↓
Fetch Search Results (HTTP request with headers)
    ↓
Parse Search Results (extract product URLs from HTML)
    ↓
Split Out Items (process one product at a time)
    ↓
Wait 2 Seconds (anti-bot delay)
    ↓
Fetch Product Detail (get individual product page)
    ↓
Extract Product Details (parse HTML for product info)
    ↓
Validate Product Data (check required fields & wireless mouse keyword)
    ↓
Save to Google Sheets (append data)
```

---

## 🔧 Node Configuration Details

### 1. **Manual Trigger**
- **Purpose**: Start workflow manually for testing
- **No Configuration Needed**

### 2. **Set Variables**
- **searchKeyword**: `"wireless mouse"` (can be changed dynamically)
- **platform**: `"daraz"` (for logging/identification)
- **baseUrl**: `"https://www.daraz.pk"`
- **searchUrl**: Automatically constructed with encoded keyword
- **productUrls**: Empty array initialized

### 3. **Fetch Search Results** (HTTP Request)
- **URL**: Uses `{{ $json.searchUrl }}`
- **Method**: GET
- **Critical Headers**:
  ```
  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9
  Accept-Language: en-US,en;q=0.9
  Accept-Encoding: gzip, deflate, br
  Connection: keep-alive
  Upgrade-Insecure-Requests: 1
  Referer: https://www.daraz.pk/
  DNT: 1
  Sec-Fetch-Dest: document
  Sec-Fetch-Mode: navigate
  Sec-Fetch-Site: none
  Cache-Control: max-age=0
  ```
- **Timeout**: 30000ms
- **Why Headers Matter**: 
  - User-Agent: Identify as modern browser
  - Accept-Language: Show browser language preference
  - Referer: Indicate you came from Daraz
  - DNT: Do Not Track signal
  - Sec-Fetch-*: Modern security headers

### 4. **Parse Search Results** (Code Node)
**Purpose**: Extract product URLs from the HTML response

**Key Features**:
- ✅ Regex-based HTML parsing (fast, light)
- ✅ Fallback JSON extraction from embedded `__INITIAL_STATE__`
- ✅ Deduplication of URLs
- ✅ Limits to 20 products for testing
- ✅ Filters for valid product URLs

**Code Logic**:
```javascript
// Primary: Regex extraction of href attributes
// Fallback: Parse JSON embedded in script tags
// Both methods ensure maximum data recovery
```

### 5. **Split Out Items** (Split Out Node)
- **Field**: `productUrls`
- **Purpose**: Process one product at a time (prevents rate limiting)

### 6. **Wait 2 Seconds** (Wait Node)
- **Duration**: 2 seconds between requests
- **Why**: Prevents IP blocking and looks like natural user behavior
- **Can Adjust**: Increase to 3-5 seconds if getting 429 errors

### 7. **Fetch Product Detail** (HTTP Request)
- **URL**: `{{ 'https://www.daraz.pk' + $json.productUrls }}`
- **Method**: GET
- **Same Headers**: As search request
- **Timeout**: 30000ms
- **Purpose**: Get individual product page

### 8. **Extract Product Details** (Code Node)
**Extracts**:
- Product Name
- Price (numeric)
- Discount percentage
- Rating (0-5 scale)
- Number of reviews
- Product Image URL
- Store name
- Wireless mouse verification

**Fallback Strategy**:
1. Try JSON extraction from `__INITIAL_STATE__`
2. If fails, use regex-based HTML parsing
3. Validate it's actually a wireless mouse
4. Return structured data object

### 9. **Validate Product Data** (IF Node)
**Conditions** (ALL must be true):
- ✅ `name` is not empty
- ✅ `price` is not empty
- ✅ `isWirelessMouse` equals true

**Result**: Only valid products proceed to Google Sheets

### 10. **Save to Google Sheets** (Google Sheets Node)
**Operation**: Append or Update
**Columns**:
| Column | Type | Description |
|--------|------|-------------|
| Name | String | Product name |
| Price | String | Price value |
| Discount | String | Discount percentage |
| Rating | Number | 0-5 scale |
| Reviews | Number | Total reviews count |
| Image URL | String | Product image link |
| Product URL | String | Full product page URL |
| Store | String | Seller/store name |
| Source | String | "daraz" or "temu" |
| Date | String | YYYY-MM-DD format |

---

## 🚀 Setup Instructions

### Step 1: Google Sheets Setup
1. Create new Google Sheet
2. Add header row:
   ```
   Name | Price | Discount | Rating | Reviews | Image URL | Product URL | Store | Source | Date
   ```
3. Share sheet with your n8n service account
4. Copy Sheet ID from URL: `https://docs.google.com/spreadsheets/d/{SHEET_ID}/edit`

### Step 2: n8n Setup
1. Import the `scrapergpt.json` workflow
2. Go to **"Save to Google Sheets"** node
3. Click **"Connect"** → authenticate with Google
4. Paste your **Sheet ID** in `documentId` field
5. Select **"Sheet1"** or your sheet name in `sheetName` field
6. Map columns:
   - name → Name
   - price → Price
   - discount → Discount
   - rating → Rating
   - reviews → Reviews
   - imageUrl → Image URL
   - productUrl → Product URL
   - storeName → Store
   - source → Source
   - date → Date

### Step 3: Test Workflow
1. Click **"Execute Workflow"** button
2. Check execution log for errors
3. Verify Google Sheet has new rows
4. Adjust delays/headers if needed

---

## 🛡️ Anti-Bot Protection Strategies

### 1. **Request Headers**
- Mimic real browser requests
- Include modern security headers (Sec-Fetch-*)
- Set realistic Accept-Encoding

### 2. **Rate Limiting**
- 2-second delay between product fetches
- ⚠️ Never below 1 second
- Increase to 5 seconds if getting blocked

### 3. **User-Agent Rotation** (Optional)
For production, rotate User-Agents:
```javascript
const userAgents = [
  "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36...",
  "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36...",
  "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36..."
];
const randomUA = userAgents[Math.floor(Math.random() * userAgents.length)];
```

### 4. **IP Rotation** (Production)
Consider VPN/proxy service for high-volume scraping

### 5. **Session Management**
- Keep cookies between requests
- Don't clear headers between iterations
- Use connection pooling

---

## ⚠️ Common Issues & Fixes

### Issue 1: Getting 403 Forbidden or 429 Too Many Requests
**Solution**:
- Increase wait delay to 5 seconds
- Add random delay: `Math.random() * 3000` ms
- Rotate headers/User-Agents

### Issue 2: Parsing Fails (No Data Extracted)
**Solution**:
- Daraz HTML structure may change
- Check browser DevTools for actual data location
- Look for JSON in Network tab → find API endpoints
- Update regex patterns in Parse code

### Issue 3: Google Sheets Connection Fails
**Solution**:
- Verify credentials are active
- Check sheet is shared with service account
- Ensure column names match exactly

### Issue 4: Products Not Being Identified as Wireless Mouse
**Solution**:
- Product name validation too strict
- Relax condition or check HTML for category info
- Look for "Mouse" OR "Wireless" keywords

---

## 🔄 Making It Reusable

### Change Search Keyword
In **Set Variables** node, modify:
```javascript
"searchKeyword": "=\"gaming mouse\"" // or any keyword
```

### Switch Platform (Daraz ↔ Temu)
For **Temu.com**, update:
```javascript
{
  "baseUrl": "https://www.temu.com",
  "searchUrl": "https://www.temu.com/search?q=" + keyword
}
```

Then update parsing in **Parse Search Results** and **Extract Product Details** nodes based on Temu's HTML structure.

---

## 📊 Data Quality Tips

### 1. **Validate Price Format**
```javascript
price = price.replace(/[PKRpkr\s,]/g, ''); // Clean price
```

### 2. **Normalize Ratings**
```javascript
rating = Math.min(5, parseFloat(rating)); // Cap at 5
```

### 3. **Image URL Validation**
```javascript
if (imageUrl && !imageUrl.startsWith('http')) {
  imageUrl = 'https:' + imageUrl; // Add protocol
}
```

### 4. **Duplicate Prevention**
Add column to Google Sheet: `=COUNTIF(A:A,A2)>1` to flag duplicates

---

## 🚨 Legal & Ethical Considerations

1. **Check Daraz Terms of Service** - Scraping may be restricted
2. **Rate Limiting** - Don't overwhelm their servers
3. **Respect robots.txt** - Add to your scraper:
   ```javascript
   // Check if GET https://www.daraz.pk/robots.txt allows scraping
   ```
4. **Use Responsibly** - For personal research, not commercial resale
5. **Data Privacy** - Don't store personal information (reviews author details)

---

## 📈 Scaling to Production

### For 1000+ Products:
1. Split into multiple runs with pagination
2. Use database instead of Google Sheets (faster)
3. Implement proxy rotation
4. Add error recovery/retry logic
5. Monitor execution logs
6. Schedule with cron (hourly/daily)

### Recommended Changes:
```json
{
  "parameters": {
    "minutes": 60,
    "rule": {
      "interval": [{"field": "hours", "triggerAtMinute": 0}]
    }
  }
}
```

---

## ✅ Verification Checklist

- [ ] Google Sheet created with headers
- [ ] Sheet ID updated in n8n
- [ ] Google OAuth credentials connected
- [ ] Workflow imports without errors
- [ ] Manual test execution successful
- [ ] Data appears in Google Sheet
- [ ] All 10 columns populated
- [ ] Wireless mouse products verified
- [ ] Delay timing adjusted for stability
- [ ] Ready for production scheduling

---

## 📞 Troubleshooting Support

1. **Check n8n Execution Log** → Find exact error line
2. **Test nodes individually** → Use "Execute This Node Only"
3. **Inspect API responses** → Enable request/response logging
4. **Check browser DevTools** → Understand HTML/JSON structure
5. **Monitor IP status** → Check if you're blocked (HTTP 403)

---

**Workflow Created**: December 21, 2025  
**Version**: 1.0  
**Status**: Production Ready
