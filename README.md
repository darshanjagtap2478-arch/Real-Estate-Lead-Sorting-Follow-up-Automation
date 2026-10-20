Real Estate Lead Automation (n8n)

An end-to-end lead pipeline for a real estate business, built with n8n, Google Sheets, Gmail and Google Calendar. It captures leads from a web form, validates and scores them, alerts the agent, lets the lead book a site visit, and flags Hot leads that were never followed up.

How it works

The system is made of three n8n workflows.

1. Lead Ingestion

Webhook: POST /webhook/lead-ingestion

The enquiry form submits the lead to the webhook.
The lead's details are cleaned (HTML characters stripped) and validated (name, email, 10-digit phone).
A score out of 100 is calculated and the lead is tagged Hot, Warm or Cold.
The lead is saved to the Master Leads and Scored Leads sheets.
Follow-up depends on the tag:
Hot (80+): instant email alert to the agent, then the lead gets a site-visit booking link.
Warm (50-79): email alert to the agent, then an acknowledgment email to the lead.
Cold (below 50): thank-you email to the lead and an entry in the Audit Log.
The form gets a clear response: success, rejected details (400) or save failed (500).

Scoring rules

Factor	Points
Budget min and max given (one only)	+20 (+10)
Preferred location given	+20
Phone has 10+ digits	+15
Valid email	+10
Specific property type chosen	+15
Source: Website / Landing Page	+20
Source: Referral / Google Ads	+15
Source: Facebook / Instagram	+10
Source: Other	+5
Urgency words in notes (e.g. "urgent", "this week", "ready")	+20

The total is capped at 100.

<img width="1551" height="595" alt="image" src="https://github.com/user-attachments/assets/0eca5f90-7114-49b4-8341-6ef7273b161d" />


2. Site Visit Booking

Webhook: GET/POST /webhook/book-visit?lead_id=...

The lead opens the booking link from their email and sees a form to pick a date, time and project.
The link is checked for a valid lead ID format, and the lead is looked up in Master Leads.
The chosen date, time and project are validated (the time cannot be in the past).
A one-hour Google Calendar event is created in IST, with the lead and the agent as attendees.
The booking is saved to the Bookings sheet, and confirmation emails go to both the lead and the agent.
The lead sees a confirmation page. Invalid links and failures show friendly error pages.

<img width="1502" height="380" alt="image" src="https://github.com/user-attachments/assets/0985f971-0b59-4f27-b62a-58624603a361" />


3. Missed Alert Detector

Runs every 5 minutes.

Reads the Scored Leads sheet.
Finds Hot leads whose Response_Status is still Pending 15+ minutes after the alert.
Marks them Missed and records the reason:
Off-hours: the alert fired on a Sunday or outside 9:00-19:00 IST.
No-ack: the alert fired during business hours but was not acted on.
Repository structure
.
├── 1. Lead Ingestion (Fixed).json          # n8n workflow 1
├── 2. Site Visit Booking (Fixed).json      # n8n workflow 2
├── 3. Missed Alert Detector (Sweep) (Fixed).json   # n8n workflow 3
├── lead_form.html                          # public enquiry form
├── booking_form_preview.html               # preview: date/time/project picker
├── booking_success_preview.html            # preview: booking confirmed
├── booking_error_preview.html              # preview: booking failed
└── booking_invalid_link_preview.html       # preview: invalid booking link

The booking_*_preview.html files are static previews of the pages that workflow 2 serves, so you can see the design without running n8n.

Setup
Import the three workflow JSON files into n8n.
Connect credentials for Google Sheets, Gmail and Google Calendar.
Create a Google Sheet with these tabs, then point the Google Sheets nodes at it:
Master Leads: Lead ID, Name, Email, Phone, Location, Budget Min, Budget Max, Property Type, Source, Notes, Score, Status, Assigned Agent, Agent Email, Valid, Validation Errors
Scored Leads: Lead ID, Name, Email, Phone, Status, Score, Response_Status, Timestamp, Agent Email, Fallback_Reason
Bookings: Booking ID, Lead ID, Lead Name, Lead Email, Property, Date, Time, Agent Email, Calendar Event ID, Timestamp
Audit Log: Timestamp, Event, Lead ID, Detail, Status
Edit the settings at the top of the Code in JavaScript node in workflow 1:
BASE_URL: the public URL of your n8n instance (no trailing slash)
AGENT_EMAIL and AGENT_NAME: who receives the alerts
Update the calendar in the Create an event node of workflow 2 to the calendar you want events created in.
Set the webhook URL in lead_form.html (the WEBHOOK_URL variable) to your n8n address, ending in /webhook/lead-ingestion.
Activate all three workflows. For local testing, expose n8n with a tunnel such as ngrok.
Tech stack

n8n, JavaScript (Code nodes), Google Sheets, Gmail, Google Calendar, HTML/CSS/JS.

<img width="1251" height="362" alt="image" src="https://github.com/user-attachments/assets/1a04bb9c-60f2-44eb-9dfe-2b068e8c5ddd" />


Author

Darshan Jagtap
