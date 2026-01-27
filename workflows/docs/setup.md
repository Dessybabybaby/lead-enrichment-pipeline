# Setup Guide - Lead Enrichment Pipeline

For Google Sheet:

Total Setup Time: 15 minutes

Step 1: Copy Google Sheet (5 min)

Open template: [https://docs.google.com/spreadsheets/d/1GZq6VN9YdE3qZohacxJiHZAie8o42CwRE6_7yT5DAEU/edit?usp=sharing]
File → Make a copy
Rename: "My Leads Database"
Share → Copy link
Extract Sheet ID from URL: spreadsheets/d/SHEET_ID/edit

Step 2: Import n8n Workflow (3 min)

Download: workflows/lead-enrichment-workflow-simple.json
n8n → Workflows → Import from File
Select downloaded file

Step 3: Connect Google Account (5 min)

Gmail Node:
Click "Send Lead Notification" node
Credentials dropdown → Create New → OAuth2
Click "Connect my account"
Authorize Gmail access
Save

Google Sheets Node:
Click "Add Lead to Sheet" node
Credentials dropdown → Create New → OAuth2
Click "Connect my account"
Authorize Google Sheets access
Document dropdown → Select "My Leads Database"
Save

Step 4: Update Configuration (2 min)

Update Sheet ID:
"Send Lead Notification" node
Find YOUR_SHEET_ID in HTML message
Replace with your actual Sheet ID (from Step 1)

Update Notification Email:
"Send Lead Notification" node
"To" field → Change to your email or assigned rep's email

Step 5: Test Workflow (5 min)

Activate workflow (toggle switch)
Copy webhook URL from Webhook node

Send test:
bash
curl -X POST https://YOUR-URL/webhook/lead-enrichment \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@microsoft.com",
    "company": "Microsoft",
    "role": "Director",
    "source": "Referral"
  }'
Check Google Sheet → New row appears
Check Gmail → Notification email received

Step 6: Connect to Forms (Optional)

Typeform
Form → Integrations → Webhooks
Paste n8n webhook URL
Google Forms
Install "Email Notifications for Google Forms" add-on
Configure webhook URL
Website Form
html
<form id="leadForm">
  <input name="email" required>
  <input name="company" required>
  <input name="role" required>
  <button type="submit">Submit</button>
</form>

<script>
document.getElementById('leadForm').addEventListener('submit', async (e) => {
  e.preventDefault();
  const formData = new FormData(e.target);
  
  await fetch('https://YOUR-URL/webhook/lead-enrichment', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      email: formData.get('email'),
      company: formData.get('company'),
      role: formData.get('role'),
      source: 'website'
    })
  });
  
  alert('Lead submitted!');
});
</script>
```

---

## Troubleshooting

**Worksheet not found:**
- Verify Sheet ID is correct
- Check OAuth2 credential has access to the sheet

**Email not sending:**
- Check Gmail OAuth2 connected
- Verify recipient email address
- Check spam folder

**Low scores for all leads:**
- Review test data (use recognizable companies like "Microsoft")
- Check scoring logic matches your criteria

---

## Customization

**Add your own industries:**
Edit `Extract & Validate Lead` node line 45+

**Change rep assignment rules:**
Edit `Calculate Fit Score` node line 100+

**Modify email template:**
Edit "Send Lead Notification" node HTML message
```
For airtable

# Setup Guide - Lead Enrichment Pipeline

## Step 1: Get API Keys (15 min)

### Hunter.io
1. Sign up: https://hunter.io
2. Dashboard → API → Copy API Key
3. Free tier: 25 verifications/month

### Clearbit
1. Sign up: https://clearbit.com
2. Dashboard → API Keys → Copy Secret Key
3. Free trial available

**Alternative:** BuiltWith API
- https://builtwith.com/api
- Free tier with limited queries

---

## Step 2: Create Airtable Base (10 min)

1. Copy template: [https://www.airtable.com/]
2. Or create manually:
   - Base: "Sales Leads Pipeline"
   - Table: "Leads"
   - Fields: Name, Email, Company, Role, Email Valid, Industry, Company Size, Revenue, Tech Stack, Fit Score, Assigned Rep, Status, Source, Enriched At

3. Get credentials:
   - Profile → Developer hub → Create Personal Access Token
   - Scopes: `data.records:read`, `data.records:write`
   - Copy token + Base ID

---

## Step 3: Import Workflow (5 min)

1. Download: `workflows/lead-enrichment-workflow.json`
2. n8n → Import from File
3. Set credentials:
   - Hunter.io API key (in HTTP Request node URL)
   - Clearbit API key (in Header Auth credential)
   - Airtable token + base ID
   - Slack OAuth2

---

## Step 4: Test (10 min)

1. Copy webhook URL from Webhook node
2. Send test payload:
```bash
curl -X POST https://YOUR-URL/webhook/lead-enrichment \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@stripe.com",
    "company": "Stripe",
    "role": "Engineer",
    "source": "test"
  }'
```

3. Check Airtable for new record
4. Check Slack for notification

---

## Step 5: Connect to Your Form (5 min)

### Google Forms
1. Form → Settings → Presentation
2. Add webhook integration (requires Zapier/Make bridge)

### Typeform
1. Form → Integrations → Webhooks
2. Paste n8n webhook URL
3. Test submission

### Website Form
1. Add to form submit handler:
```javascript
fetch('https://YOUR-URL/webhook/lead-enrichment', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    email: formData.email,
    company: formData.company,
    role: formData.role,
    source: 'website'
  })
});
```

---

## Troubleshooting

**Hunter.io fails:**
- Check API key in URL
- Verify monthly quota not exceeded
- Test with: `curl "https://api.hunter.io/v2/email-verifier?email=test@gmail.com&api_key=YOUR_KEY"`

**Clearbit fails:**
- Check Authorization header format: `Bearer YOUR_KEY`
- Verify domain extractable from email
- Alternative: Use BuiltWith API

**Airtable fails:**
- Verify base shared with integration
- Check field names match exactly (case-sensitive)
- Test connection with single field first

**Scoring always 0:**
- Check Hunter.io + Clearbit returning data
- Review scoring logic in Calculate Fit Score node
- Add `console.log()` statements for debugging
```
