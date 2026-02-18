# Troubleshooting & Testing Scripts

## 🧪 Test Each Component

### Test 1: Verify HTTP Headers Work
Use this standalone HTTP request to test headers:

```bash
curl -X GET "https://www.daraz.pk/search?q=wireless+mouse" \
  -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36" \
  -H "Accept: text/html,application/xhtml+xml,application/xml;q=0.9" \
  -H "Accept-Language: en-US,en;q=0.9" \
  -H "Accept-Encoding: gzip, deflate, br" \
  -H "Connection: keep-alive" \
  -H "Upgrade-Insecure-Requests: 1" \
  -H "Referer: https://www.daraz.pk/" \
  -H "DNT: 1" \
  -v
```

**Expected**: HTTP 200, HTML response (not 403/429)

---

### Test 2: Check If HTML Parsing Regex Works

Create file `test_parsing.js`:

```javascript
// Sample HTML snippet from Daraz
const html = `
  <a href="/p/product123" data-qa-locator="product-item">Product 1</a>
  <a href="/p/product456" data-qa-locator="product-item">Product 2</a>
`;

// Test regex extraction
const linkRegex = /href=['"](\/[^'"]*?)['"]\s+data-qa-locator=['"]product-item['"]|href=['"](\/p\/[^'"]*)['"]/gi;
const productUrls = [];
let match;

while ((match = linkRegex.exec(html)) !== null) {
  const url = match[1] || match[2];
  if (url && !productUrls.includes(url)) {
    productUrls.push(url);
  }
}

console.log('Extracted URLs:', productUrls);
// Expected output: ['/p/product123', '/p/product456']
```

**Run**: `node test_parsing.js`

---

### Test 3: Validate JSON Extraction

Create file `test_json.js`:

```javascript
// Sample embedded JSON from Daraz
const html = `
  <script>
  window.__INITIAL_STATE__ = {
    "listModule": {
      "data": {
        "items": [
          { "productUrl": "/p/item1" },
          { "productUrl": "/p/item2" }
        ]
      }
    }
  };
  </script>
`;

// Test JSON extraction
const jsonMatch = html.match(/window\.__INITIAL_STATE__\s*=\s*({.*?});/s);

if (jsonMatch) {
  try {
    const data = JSON.parse(jsonMatch[1]);
    console.log('Extracted data:', data.listModule.data.items);
  } catch (e) {
    console.error('JSON parse error:', e.message);
  }
}
```

---

### Test 4: Check Google Sheets Connection

In n8n **Code Node**:

```javascript
// Try a simple write to verify connection
const testData = {
  Name: "TEST PRODUCT",
  Price: "999",
  Rating: "5",
  Reviews: "100",
  Source: "test",
  Date: new Date().toISOString().split('T')[0]
};

console.log('Test data prepared:', testData);
// Next node (Google Sheets) should receive this
return testData;
```

**Expected**: No errors, data appears in Google Sheet

---

## 🔍 Debugging Checklist

### Is the website blocking me?

Check status code in execution log:
- **200** = Success ✓
- **403** = Forbidden (blocked)
- **429** = Too Many Requests (rate limited)
- **500+** = Server error

**Fix for 403/429**: Increase delay to 5-10 seconds

### Is data being extracted?

Check **Parse Search Results** output:
```javascript
console.log('Total products found:', productUrls.length);
```

**Expected**: `totalFound: 15-25` (or your limit)

If 0, the regex didn't match → Website structure changed

### Is product validation working?

Check **Extract Product Details** output:
```javascript
console.log('Is wireless mouse:', productData.isWirelessMouse);
```

**Expected**: `true` for wireless mice

If `false`, adjust validation keywords

### Is Google Sheets saving data?

Check **Save to Google Sheets** node response:
- Should return `{ "updatedRows": X }`
- If empty, check credentials/permissions

---

## 🚨 Common Error Messages & Fixes

### Error: "Could not authenticate with Google"
```
✗ Issue: Google credentials expired or revoked
✓ Fix: 
  1. Go to "Save to Google Sheets" node
  2. Click credentials dropdown → "Disconnect"
  3. Click "Connect" → Authenticate again
```

### Error: "429 Too Many Requests"
```
✗ Issue: Website throttled your IP for too many requests
✓ Fix:
  1. Increase wait delay: 2s → 5s minimum
  2. Add random delay: Math.random() * 3000
  3. Wait 1-2 hours before retrying
  4. Consider VPN if still blocked
```

### Error: "Cannot read property 'body' of undefined"
```
✗ Issue: HTTP request failed (no response)
✓ Fix:
  1. Check URL is valid
  2. Verify headers are correct
  3. Check internet connection
  4. Website might have changed structure
```

### Error: "No products extracted"
```
✗ Issue: Regex didn't match HTML structure
✓ Fix:
  1. Download actual HTML from browser
  2. Compare with regex pattern
  3. Update regex in Parse node
  4. Or switch to JSON extraction if available
```

### Error: "Invalid cell reference"
```
✗ Issue: Google Sheets column names don't match
✓ Fix:
  1. Verify column headers in sheet:
     Name | Price | Discount | Rating | Reviews | Image URL | Product URL | Store | Source | Date
  2. Check node mapping: Name → Name, Price → Price, etc
```

---

## 🔧 Advanced Debugging

### Enable Verbose Logging

Add to each **Code Node**:
```javascript
console.log('Input data:', JSON.stringify($input.all(), null, 2));
console.log('Processing...');
// ... your code ...
console.log('Output:', JSON.stringify(output, null, 2));
return output;
```

Check logs in execution details.

### Test Node Individually

1. Right-click node → "Execute This Node Only"
2. Use test data from previous successful run
3. Check output in Results panel

### Inspect Network Requests

In browser, open DevTools → Network tab:
1. Navigate to Daraz search page
2. Look for actual API endpoints
3. Check response format (JSON vs HTML)
4. Copy actual response and test with it

---

## 🧬 Website Structure Change Detection

If scraper stops working, the website likely changed:

### Step 1: Download Fresh HTML
```javascript
// In Fetch Search Results node, add:
const fs = require('fs');
fs.writeFileSync('page.html', $input.first().body);
console.log('HTML saved to page.html');
```

### Step 2: Compare with Old Pattern
```bash
# Compare old regex with new HTML
grep -i "product" page.html | head -20
```

### Step 3: Update Regex
Find new pattern and update in **Parse Search Results** node

### Step 4: Test Updated Pattern
```javascript
// Test new regex with actual HTML
const newRegex = /href=['"]([^'"]*new-pattern[^'"]*)['"]/gi;
const matches = html.match(newRegex);
console.log('Matches found:', matches.length);
```

---

## 📊 Performance Monitoring

### Track Execution Metrics

```javascript
// In final code node before Google Sheets
const startTime = $input.first().json.startTime || Date.now();
const endTime = Date.now();
const duration = (endTime - startTime) / 1000;

return {
  ...($input.first().json),
  executionTime: duration,
  executionSpeed: '1 product per ' + (duration / 1).toFixed(2) + 's'
};
```

### Expected Performance
- 1 product: 3-5 seconds (with delays)
- 20 products: 60-120 seconds
- 100 products: 5-10 minutes

If slower, check:
- Website latency
- Network connection
- Delay timing

---

## 🛡️ IP Ban Recovery

If you get blocked:

### Immediate:
```
1. Stop all scraping (pause workflow)
2. Clear browser cookies
3. Wait 2-4 hours
4. Try again with longer delays (10s+)
```

### Medium-term:
```
1. Use residential proxy service
2. Rotate proxy for each request
3. Add random User-Agent
4. Spread requests over longer time
```

### Long-term:
```
1. Use official Daraz/Temu API (if available)
2. Buy product data from data provider
3. Partner with site for data access
```

---

## 💡 Pro Tips

### Tip 1: Test with Single Product First
```javascript
// In Parse Search Results, limit to 1:
.slice(0, 1) // Instead of slice(0, 20)
```

### Tip 2: Add Detailed Logging
```javascript
console.log('Step 1: Fetched HTML, size:', html.length);
console.log('Step 2: Found', urls.length, 'products');
console.log('Step 3: Validation passed for', valid.length);
```

### Tip 3: Use Test Data
Instead of real scraping, create static test data:
```javascript
return [{
  name: 'Test Wireless Mouse',
  price: '1500',
  rating: 4.5,
  reviews: 100
}];
```

### Tip 4: Version Control Your Code
Before making changes:
```
1. Copy old code to separate text file
2. Label with date: "Version 2025-12-21"
3. Make changes
4. If broken, revert to old version
```

---

## 📈 Success Checklist

- [ ] Workflow imports without errors ✓
- [ ] Manual trigger executes
- [ ] HTTP request returns 200
- [ ] Parse node finds 10+ products
- [ ] Each product detail page loads
- [ ] Product data extracted correctly
- [ ] Validation passes for wireless mice
- [ ] Google Sheets receives data
- [ ] All 10 columns populated
- [ ] No duplicate products
- [ ] Execution completes in reasonable time (< 3 mins for 20 items)

---

## 📞 Still Stuck?

1. Check execution logs (red errors)
2. Review QUICK_REFERENCE.md
3. Compare with ADVANCED_CODE_EXAMPLES.md
4. Test individual nodes with sample data
5. Check if website HTML structure changed
6. Try alternative extraction method (JSON vs Regex)

---

**Last Updated**: December 21, 2025  
**Tested On**: Daraz.pk, n8n v1.0+
