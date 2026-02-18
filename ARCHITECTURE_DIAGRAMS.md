# Workflow Architecture & Diagrams

## 🏗️ Complete Workflow Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    N8N WORKFLOW: WIRELESS MOUSE SCRAPER      │
└─────────────────────────────────────────────────────────────┘

START
  │
  ├─ [🎯 Manual Trigger]
  │  └─ User clicks "Execute"
  │
  ├─ [⚙️ Set Variables]
  │  ├─ searchKeyword: "wireless mouse"
  │  ├─ platform: "daraz"
  │  ├─ baseUrl: "https://www.daraz.pk"
  │  └─ searchUrl: Constructed URL
  │
  ├─ [🌐 Fetch Search Results] 
  │  ├─ HTTP GET with Headers
  │  │  ├─ User-Agent (Chrome)
  │  │  ├─ Accept-Language
  │  │  ├─ Referer
  │  │  ├─ DNT
  │  │  └─ Sec-Fetch headers
  │  └─ Response: HTML
  │
  ├─ [🔍 Parse Search Results]
  │  ├─ Method 1: Extract JSON from __INITIAL_STATE__
  │  ├─ Method 2: Regex extract href attributes
  │  ├─ Deduplication & validation
  │  └─ Output: [ "/p/item1", "/p/item2", ... ]
  │
  ├─ [📦 Split Out Items]
  │  └─ Process ONE product at a time
  │     (Prevents overwhelming the server)
  │
  ├─ [⏱️ Wait 2 Seconds]
  │  └─ Delay between requests (anti-bot)
  │
  └─ [LOOP: For each product URL]
     │
     ├─ [🌐 Fetch Product Detail]
     │  ├─ HTTP GET individual product page
     │  ├─ Headers: Same as search
     │  └─ Response: Product HTML
     │
     ├─ [🔍 Extract Product Details]
     │  ├─ JSON extraction from embedded data
     │  ├─ Regex fallback parsing
     │  ├─ Extract fields:
     │  │  ├─ name
     │  │  ├─ price
     │  │  ├─ discount
     │  │  ├─ rating
     │  │  ├─ reviews
     │  │  ├─ imageUrl
     │  │  ├─ storeName
     │  │  └─ isWirelessMouse (validation)
     │  └─ Error handling
     │
     ├─ [✅ Validate Product Data]
     │  ├─ Check: name NOT empty
     │  ├─ Check: price NOT empty
     │  ├─ Check: isWirelessMouse == true
     │  └─ IF all pass → Continue
     │       ELSE → Skip product
     │
     └─ [💾 Save to Google Sheets]
        ├─ Append row with columns:
        │  ├─ Name
        │  ├─ Price
        │  ├─ Discount
        │  ├─ Rating
        │  ├─ Reviews
        │  ├─ Image URL
        │  ├─ Product URL
        │  ├─ Store
        │  ├─ Source
        │  └─ Date
        └─ Success ✓

END (All products processed)

```

---

## 🔄 Data Flow Diagram

```
SEARCH REQUEST
    │
    ├─→ [Set Variables]
    │    └─ searchKeyword → searchUrl
    │
    ├─→ [HTTP Request]
    │    └─ searchUrl → HTML Response
    │
    ├─→ [Parse HTML]
    │    └─ Extract Product URLs
    │         │
    │         ├─ Method 1: JSON parsing
    │         │  (window.__INITIAL_STATE__)
    │         │
    │         ├─ Method 2: Regex extraction
    │         │  (href="/p/...")
    │         │
    │         └─ Deduplicate URLs
    │
    ├─→ [Split for Batch]
    │    └─ Process 1 URL at a time
    │
    └─→ PRODUCT DETAIL LOOP
         │
         ├─→ [Wait] → [Fetch Product]
         │    │
         │    └─ productUrl → Product HTML
         │
         ├─→ [Extract Fields]
         │    │
         │    ├─ From JSON
         │    │  (if available)
         │    │
         │    ├─ From HTML Regex
         │    │  (fallback)
         │    │
         │    └─ Validate:
         │       - name exists
         │       - price exists
         │       - is wireless mouse
         │
         ├─→ [If Validation Pass]
         │    │
         │    └─→ [Save to Sheet]
         │         │
         │         └─ Google Sheets API
         │            └─ Append row
         │
         └─→ [Next Product]

```

---

## 📊 Data Structure Diagram

### Input Data (From Workflow Trigger)
```json
{
  "searchKeyword": "wireless mouse",
  "platform": "daraz",
  "baseUrl": "https://www.daraz.pk"
}
```

### After Parse Search Results
```json
{
  "productUrls": [
    "/p/LM12345",
    "/p/LM12346",
    "/p/LM12347",
    ...
  ],
  "totalFound": 20,
  "timestamp": "2025-12-21T14:30:00Z"
}
```

### After Extract Product Details
```json
{
  "name": "Wireless Mouse USB Receiver",
  "price": "1500",
  "discount": "20%",
  "rating": 4.5,
  "reviews": 150,
  "imageUrl": "https://...",
  "productUrl": "https://www.daraz.pk/p/LM12345",
  "storeName": "Store Name",
  "source": "daraz",
  "date": "2025-12-21",
  "isWirelessMouse": true,
  "extractionMethod": "json_state"
}
```

### After Google Sheets
```
Row 1: Headers
Row 2: Wireless Mouse USB Receiver | 1500 | 20% | 4.5 | 150 | https://... | https://... | Store Name | daraz | 2025-12-21
Row 3: [Next product...]
...
```

---

## 🔐 Anti-Bot Protection Layers

```
Layer 1: REQUEST HEADERS
┌──────────────────────────────┐
│ User-Agent: Chrome 91        │ ← Identify as real browser
│ Accept-Language: en-US       │ ← Show language preference
│ Referer: Daraz homepage      │ ← Show traffic source
│ DNT: 1                       │ ← Privacy signal
│ Sec-Fetch-*: Modern headers  │ ← Security compliance
└──────────────────────────────┘
       ↓
Layer 2: TIMING DELAYS
┌──────────────────────────────┐
│ 2+ seconds between requests  │ ← Human-like behavior
│ Random delay variation        │ ← Not bot-like patterns
│ Long connections kept alive  │ ← Session continuity
└──────────────────────────────┘
       ↓
Layer 3: DATA EXTRACTION
┌──────────────────────────────┐
│ JSON extraction first         │ ← Clean structured data
│ HTML regex fallback          │ ← Adaptable to changes
│ Error handling               │ ← Graceful degradation
└──────────────────────────────┘
       ↓
Layer 4: VALIDATION
┌──────────────────────────────┐
│ Check required fields        │ ← Quality assurance
│ Verify wireless mouse        │ ← Product relevance
│ Skip invalid products        │ ← Clean data only
└──────────────────────────────┘

RESULT: Low block risk + High quality data
```

---

## 🔄 Error Handling Flow

```
HTTP Request
    │
    ├─ Success (200)
    │  └─ Continue to parsing
    │
    ├─ Rate Limited (429)
    │  └─ ⚠️ Increase delay to 5-10 seconds
    │      Then retry
    │
    ├─ Forbidden (403)
    │  └─ ⚠️ IP might be blocked
    │      Wait 2 hours or use VPN
    │
    └─ Server Error (500+)
       └─ ⚠️ Website issue
           Retry with longer delay

Parse Search Results
    │
    ├─ JSON Found
    │  └─ Parse as JSON
    │
    ├─ JSON Not Found
    │  └─ Use Regex extraction (fallback)
    │
    └─ Both Fail
       └─ Return empty array, skip this batch

Extract Product Details
    │
    ├─ All fields found ✓
    │  └─ Proceed to validation
    │
    ├─ Some fields missing
    │  └─ Use fallback methods (HTML regex)
    │
    ├─ Price missing
    │  └─ Mark as "N/A"
    │
    └─ Parse error
       └─ Log error, skip product

Validate Product Data
    │
    ├─ All conditions pass ✓
    │  └─ Save to Google Sheets
    │
    └─ Validation fails
       └─ Skip product (don't save)
```

---

## 🎯 Request Header Flow

```
CLIENT (n8n)
    │
    ├─ User-Agent Header
    │  └─ Mozilla/5.0 (Windows NT 10.0; Win64; x64)...
    │     └─ Identifies as Chrome browser
    │
    ├─ Accept Header
    │  └─ text/html, application/xhtml+xml...
    │     └─ Show preference for HTML
    │
    ├─ Accept-Language Header
    │  └─ en-US, en;q=0.9
    │     └─ English language
    │
    ├─ Accept-Encoding Header
    │  └─ gzip, deflate, br
    │     └─ Compression support
    │
    ├─ Referer Header
    │  └─ https://www.daraz.pk/
    │     └─ Came from Daraz homepage
    │
    ├─ DNT Header
    │  └─ 1
    │     └─ Do Not Track signal
    │
    ├─ Sec-Fetch Headers
    │  ├─ Sec-Fetch-Dest: document
    │  ├─ Sec-Fetch-Mode: navigate
    │  └─ Sec-Fetch-Site: none
    │     └─ Modern browser security headers
    │
    └─ Connection Header
       └─ keep-alive
          └─ Persistent connection

       ↓
       
SERVER (Daraz.pk)
    │
    └─ Response
       ├─ 200 OK
       ├─ HTML content
       └─ Cookies (handled automatically)

```

---

## 🔍 Code Node Logic Flow

### Parse Search Results Logic:
```
JavaScript Execution
    │
    ├─ Read HTML response
    │
    ├─ Step 1: JSON Extraction
    │  ├─ Find pattern: window.__INITIAL_STATE__ = {...}
    │  ├─ Extract JSON block
    │  ├─ Parse JSON
    │  └─ Navigate: listModule → data → items
    │
    ├─ Step 2: Regex Extraction (if JSON fails)
    │  ├─ Find pattern: href="..." data-qa-locator="product-item"
    │  ├─ Extract all matching URLs
    │  └─ Clean and validate
    │
    ├─ Step 3: Deduplication
    │  ├─ Remove duplicate URLs
    │  └─ Check for valid product paths
    │
    └─ Return: Array of product URLs
       ├─ totalFound: number
       ├─ timestamp: ISO date
       └─ productUrls: [...]
```

### Extract Product Details Logic:
```
JavaScript Execution
    │
    ├─ Read Product HTML
    │
    ├─ Extraction Method 1: JSON
    │  ├─ Find: window.__INITIAL_STATE__ = {...}
    │  ├─ Extract fields:
    │  │  ├─ title → name
    │  │  ├─ price → price
    │  │  ├─ rating → rating
    │  │  ├─ reviews → reviews
    │  │  └─ images[0] → imageUrl
    │  └─ Mark as: "json_state"
    │
    ├─ Extraction Method 2: HTML Regex (if JSON fails)
    │  ├─ Find: <h1>...</h1> → name
    │  ├─ Find: price patterns → price
    │  ├─ Find: rating patterns → rating
    │  ├─ Find: review count → reviews
    │  └─ Mark as: "html_regex"
    │
    ├─ Validation: Is Wireless Mouse?
    │  ├─ Check for: "wireless" OR "bluetooth"
    │  ├─ Check for: "mouse"
    │  ├─ Exclude: "keyboard", "keycap", etc.
    │  └─ Set: isWirelessMouse = true/false
    │
    └─ Return: Product Object
       ├─ name, price, rating, reviews...
       ├─ extractionMethod: "json_state" | "html_regex"
       └─ isWirelessMouse: true | false
```

---

## 📈 Scaling Architecture

### Current Setup (Small Scale)
```
Manual Trigger
    └─ 1 search
       └─ 20 products
          └─ Linear processing
          └─ Time: 2-3 minutes
```

### Scaled Setup (Medium Scale)
```
Scheduled Trigger (Hourly)
    ├─ Multiple searches (different keywords)
    ├─ 50-100 products per search
    ├─ Batch processing
    └─ Database instead of Sheets
```

### Enterprise Setup (Large Scale)
```
Scheduled Trigger (Multiple times daily)
    ├─ Proxy rotation
    ├─ 1000+ products
    ├─ Parallel workers
    ├─ Elasticsearch storage
    ├─ Redis caching
    └─ Monitoring/alerting
```

---

## 🎯 Production Checklist

```
Pre-Launch
  ├─ [ ] Workflow imports without errors
  ├─ [ ] All nodes connected properly
  ├─ [ ] Google Sheets authenticated
  ├─ [ ] Test execution successful
  └─ [ ] Data visible in Google Sheet

Deployment
  ├─ [ ] Schedule configured (if needed)
  ├─ [ ] Monitoring alerts setup
  ├─ [ ] Backup of workflow created
  ├─ [ ] Documentation reviewed
  └─ [ ] Team trained

Ongoing
  ├─ [ ] Check execution logs weekly
  ├─ [ ] Monitor data quality
  ├─ [ ] Track any HTTP errors
  ├─ [ ] Update code if website changes
  └─ [ ] Review and archive old data
```

---

**This completes the architectural documentation!**  
All diagrams are text-based for easy reference and modification.
