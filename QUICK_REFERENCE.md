# Quick Reference Card - n8n Wireless Mouse Scraper

## 🚀 Quick Start (5 Minutes)

### Step 1: Import Workflow
```
1. Open n8n → Workflows → Import
2. Select scrapergpt.json
3. Click "Import"
```

### Step 2: Setup Google Sheets
```
1. Create Google Sheet
2. Add headers: Name, Price, Discount, Rating, Reviews, Image URL, Product URL, Store, Source, Date
3. Copy Sheet ID from URL
4. Share with n8n service account (or authenticate)
```

### Step 3: Configure Nodes
```
In "Save to Google Sheets" node:
- Set documentId → YOUR_SHEET_ID
- Set sheetName → "Sheet1" (or your sheet)
- Click "Connect" for Google authentication
```

### Step 4: Test
```
1. Click "Execute Workflow" (play icon)
2. Check Google Sheet for data
3. Verify 10+ rows were added
```

---

## 📋 Node Checklist

- [ ] Manual Trigger - No config needed
- [ ] Set Variables - Edit searchKeyword if needed
- [ ] Fetch Search Results - Headers set ✓
- [ ] Parse Search Results - Code loads ✓
- [ ] Split Out Items - Configured ✓
- [ ] Wait 2 Seconds - Can adjust delay
- [ ] Fetch Product Detail - Headers set ✓
- [ ] Extract Product Details - Code loads ✓
- [ ] Validate Product Data - IF conditions set ✓
- [ ] Save to Google Sheets - Google auth + Sheet ID

---

## 🔧 Most Important Settings

| Setting | Value | Why |
|---------|-------|-----|
| Wait Delay | 2-5 seconds | Avoid IP blocking |
| Product Limit | 20 (testing) | Start small |
| Validation Rule | Name + Price + Wireless | Quality control |
| User-Agent | Modern Chrome | Look like real browser |
| Accept-Language | en-US | Realistic headers |

---

## 🆘 Troubleshooting Quick Fixes

| Error | Solution |
|-------|----------|
| "Could not load workflow" | Check JSON syntax (already validated ✓) |
| "403 Forbidden" | Increase wait delay to 5-10 seconds |
| "No data in Google Sheet" | Check Google authentication credentials |
| "Products not extracted" | HTML structure may have changed - check DevTools |
| "Wrong products" | Validation filter too strict - adjust IF node |

---

## 📊 Expected Output

Google Sheet should have:
```
Name                          | Price | Discount | Rating | Reviews | Image URL | Product URL | Store | Source | Date
Wireless Mouse USB Receiver   | 1500  | 20%      | 4.5    | 150     | https://... | https://... | Store1 | daraz | 2025-12-21
Bluetooth Wireless Mouse      | 2000  | 15%      | 4.2    | 89      | https://... | https://... | Store2 | daraz | 2025-12-21
...
```

---

## 🎯 Customization Options

### Change Search Keyword
In **Set Variables** node:
```javascript
"searchKeyword": "=\"gaming mouse\"" // Change this
```

### Change Delay Between Requests
In **Wait 2 Seconds** node:
```
minutes: 5  // 5 seconds = 5000ms
```

### Increase Product Limit
In **Parse Search Results** code:
```javascript
.slice(0, 50) // Changed from 20 to 50
```

### Switch to Temu
In **Set Variables** node:
```javascript
"baseUrl": "=\"https://www.temu.com\""
"searchUrl": "={{ $json.baseUrl + '/search?q=' + $json.searchKeyword }}"
```

Then update parsing code for Temu structure (see ADVANCED_CODE_EXAMPLES.md)

---

## 🔐 Best Practices

✅ **DO:**
- Test with small batches first (10-20 products)
- Use 3-5 second delays
- Include proper headers
- Validate product data before saving
- Check Google Sheet regularly
- Monitor execution logs

❌ **DON'T:**
- Use 0 delay (will get blocked instantly)
- Scrape 1000+ products at once (first test run)
- Ignore HTTP errors
- Save products without validation
- Run without delays
- Ignore rate limit responses

---

## 📈 Performance Tips

| Change | Impact |
|--------|--------|
| Reduce delay to 1s | Faster, higher block risk |
| Increase to 5s | Slower, safer |
| Add proxy rotation | Much safer, cost increase |
| Use database instead of Google Sheets | Faster for 1000+ items |
| Schedule to off-peak hours | Lower block risk |

---

## 🔄 Workflow Flow Diagram

```
START
  ↓
Manual Trigger
  ↓
Set Variables (keyword, URL, etc)
  ↓
HTTP GET: Search page
  ↓
Parse HTML: Extract 20 product URLs
  ↓
LOOP: For each URL
  ├→ Wait 2 seconds
  ├→ HTTP GET: Product detail page
  ├→ Extract: Name, Price, Rating, etc
  ├→ Validate: Required fields + wireless keyword
  └→ Save to Google Sheet (if valid)
  ↓
END
```

---

## 📱 Mobile/Remote Access

If running on remote server:

1. **SSH into server**:
   ```bash
   ssh user@server.com
   cd /path/to/n8n
   ```

2. **Check execution logs**:
   ```bash
   curl http://localhost:5678/api/executions
   ```

3. **Monitor Google Sheet**:
   Open in browser → See real-time updates

---

## 💾 Backup & Recovery

### Backup Workflow
```
1. Right-click workflow → Export
2. Save as backup-scrapergpt-DATE.json
3. Store in safe location
```

### Restore from Backup
```
1. Delete broken workflow
2. Import → Select backup JSON
3. Reconfigure Google Sheets node
```

---

## 🎓 Learning Resources

- **n8n Docs**: https://docs.n8n.io/
- **HTTP Request Node**: https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.httprequest/
- **Code Node**: https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.code/
- **Google Sheets Integration**: https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-google-sheets/

---

## 📞 When to Get Help

Check these before asking for help:
- [ ] Run test execution manually
- [ ] Check execution logs (red errors)
- [ ] Verify Google Sheet credentials
- [ ] Test each node individually
- [ ] Check if website structure changed
- [ ] Review DevTools Network tab

---

**Last Updated**: December 21, 2025  
**Workflow**: scrapergpt.json  
**Status**: ✅ Ready to Use
