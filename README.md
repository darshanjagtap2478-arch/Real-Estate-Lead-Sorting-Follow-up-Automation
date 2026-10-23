# Real Estate Lead Sorting and Follow-up Automation (n8n)

> Scores incoming real estate leads, alerts agents, lets leads book site visits, and flags Hot leads that nobody acknowledged. Three n8n workflows on Google Sheets, Gmail and Google Calendar. Template only, no credentials.

## Workflows
| File | What it does |
|---|---|
| `1_Lead_Ingestion_Template.json` | Webhook receives a lead, validates and scores it (Hot / Warm / Cold), saves it to Google Sheets, alerts the agent, and emails the lead (Hot leads get a site-visit booking link). |
| `2_Site_Visit_Booking_Template.json` | Serves a booking form from the emailed link, creates a Google Calendar event, logs the booking, and emails the lead and the agent. |
| `3_Missed_Alert_Detector_Template.json` | Every 5 minutes, marks Hot leads still `Pending` after 15 minutes as `Missed` (reason: Off-hours or No-ack). |

## Quick start
1. In n8n: **Import from File** for each workflow.
2. Create a Google Sheet with the tabs and columns below.
3. Replace the placeholders:
   - `YOUR_GOOGLE_SHEET_ID` (all Google Sheets nodes, all 3 workflows)
   - `YOUR_CALENDAR_ID` (workflow 2, `Create an event`)
   - In workflow 1, `Code in JavaScript`: `BASE_URL` (your public n8n URL, no trailing slash), `AGENT_EMAIL`, `AGENT_NAME`
   - In workflow 2, replace `Project A/B/C` with your projects in **both** `Code in JavaScript1` and the dropdown in `Render Booking Form`
4. Connect your own credentials: Google Sheets, Gmail, Google Calendar (OAuth2).
5. Activate workflow 1 and 2, test, then activate workflow 3.

## Google Sheet tabs
| Tab | Columns |
|---|---|
| `Master Leads` | Lead ID, Name, Email, Phone, Location, Budget Min, Budget Max, Property Type, Source, Notes, Score, Status, Assigned Agent, Agent Email, Valid, Validation Errors |
| `Scored Leads` | Lead ID, Name, Email, Phone, Score, Status, Response_Status, Timestamp, Agent Email, Fallback_Reason |
| `Audit Log` | Timestamp, Event, Lead ID, Detail, Status |
| `Bookings` | Booking ID, Lead ID, Lead Name, Lead Email, Property, Date, Time, Agent Email, Timestamp, Calendar Event ID |

## Send a lead
```bash
curl -X POST https://YOUR-N8N-DOMAIN/webhook/lead-ingestion \
  -H "Content-Type: application/json" \
  -d '{"name":"Test Lead","email":"lead@example.com","phone":"9876543210",
       "location_pref":"Thane","budget_min":"5000000","budget_max":"8000000",
       "property_type":"Apartment","source":"Website","notes":"Need to buy this week"}'
```
Property types: Any, Apartment, Villa, Plot, Commercial. Sources: Website, Landing Page, Facebook, Instagram, Referral, Google Ads, Other.

## Scoring
Budget (up to 20), location (20), phone (15), email (10), property type (15), source (5 to 20) and urgency words in notes (20). **Hot** = 80+, **Warm** = 50 to 79, **Cold** = below 50.

## How agents use it
New Hot and Warm leads appear in `Scored Leads` with `Response_Status = Pending`. After contacting a lead, the agent sets it to `Acknowledged` by hand. Workflow 3 only marks leads that are still `Pending`.

## Notes
- Times in workflows 2 and 3 are fixed to IST (+05:30). Business hours for workflow 3 are Mon to Sat, 9:00 to 19:00.
- The webhooks are public. Keep your n8n URL private and consider adding authentication before real use.
- There is no duplicate-lead check, and no double-booking check on the calendar.
- Test with fake data first. Lead data is personal data, so follow your local privacy rules.

## Security
Never commit API keys, sheet IDs, credential exports, real emails or your real n8n URL.
