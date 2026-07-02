## Written Answer (300–400 words)

The landing page form validates input client-side, then makes a direct API call to a backend endpoint — not the HubSpot Forms API, not a native embed, and not Zapier/Make. The Forms API only accepts a submission; it can't run custom lookup logic before deciding whether to write, so it can't support the phone-based dedup this design needs. Zapier and Make add an extra network hop, external cost, and reduced control over both the dedup logic and the 2-minute WhatsApp SLA. The backend runs three steps in order: (1) search HubSpot Contacts by phone number — HubSpot's default deduplication is on email, and this form never collects one — updating the existing contact if a match is found, otherwise creating a new one with Source = "Google Ads - Consultation Landing Page" and Lead Status = "New Enquiry"; (2) call Karix's WhatsApp Business API to send the confirmation message; (3) return success, which the frontend uses to show the thank-you state and which triggers the Google Ads conversion via the GTM dataLayer push that already fired on submit.

The biggest failure point is the phone-dedup step itself, not just API downtime. Two different patients sharing one phone number — a spouse or parent's number, common in Indian household bookings — collide into a single HubSpot contact, and naively updating on a match silently overwrites the first patient's name with the second's. The fallback: never blind-overwrite the name field on a match. When the incoming name differs from the contact on file, the backend logs the new submission as an activity note on the existing contact instead and flags it for manual review, so front-desk staff catch the mismatch before the wrong patient context reaches the doctor. Separately, if the HubSpot API call itself fails or times out, the lead is written to a retry queue and reattempted automatically, so no lead is lost to CRM downtime.

The WhatsApp SLA can break from Karix API downtime, WhatsApp template/session approval issues, or the HubSpot lookup running slow under load and delaying the step that follows it. I'd monitor this by timestamping the form submission and the WhatsApp delivery webhook, computing the delta on every send, and alerting ops automatically (Slack/PagerDuty) whenever it exceeds 90 seconds — a buffer before the actual 2-minute SLA is breached, giving the team time to react before a patient actually notices a delay.

# Integration Design

## Objective

This document describes how the Namoza/OrthoNow landing page integrates with Google Tag Manager (GTM), Google Analytics 4 (GA4), Google Ads, and HubSpot CRM. The design ensures accurate lead tracking, analytics reporting, and efficient lead management.
---

# System Architecture

User
  ↓
Landing Page
  ↓
Google Tag Manager (GTM)
  ↓
Google Analytics 4 (GA4)  +  Google Ads Conversion Tracking
  ↓
Backend API  →  HubSpot CRM (lead create/update)
  ↓
Karix WhatsApp Business API (patient confirmation)

The landing page sends user interactions to Google Tag Manager, which distributes the consultation_form_submitted event to GA4 for reporting and Google Ads for conversion tracking. Separately, the form's submit handler calls the backend directly — this path does not go through GTM, since it's carrying the patient's actual name and phone number, which should never sit in the dataLayer. The backend owns the HubSpot write and the WhatsApp send.

---

# User Journey
1. User visits the landing page.
2. User reads information about OrthoNow.
3. User fills in the consultation form (name, phone).
4. JavaScript validates the input client-side.
5. A dataLayer.push() fires consultation_form_submitted (no PII) for GTM/GA4/Ads.
6. In parallel, a POST /api/leads call sends the actual name and phone to the backend.
7. The backend searches HubSpot by phone number, then creates or updates the contact.
8. The backend calls the Karix WhatsApp API to send a confirmation message.
9. The backend returns a success response to the landing page.
10. The user sees the thank-you state, with no page reload.

---

# GTM Trigger Configuration

This section covers only the trigger and tags relevant to the CRM/Ads integration described in this document. The remaining events from the full GTM Event Schema (booking_step_1-3, click_call_button, click_whatsapp, patient_guide_download, clinic_page_view, article_read_50/90) are site-wide tracking events and are scoped in Task 1, not part of this specific consultation → CRM → WhatsApp → Ads flow.

| Trigger Name | Trigger Type | Firing Condition | Fires On |
|---|---|---|---|
| Consultation Form Submission | Custom Event | Event name equals `consultation_form_submitted` | Once per page |

"Once per page" is deliberate, not a default left unchanged — it stops a double-fire (e.g. a user double-clicking submit, or a retry after a slow network response) from recording two Google Ads conversions for one lead.

## Data Layer Variables

These are the GTM variables created to read values out of the dataLayer push so the tags below can use them:

| Variable Name | Type | Data Layer Key |
|---|---|---|
| DLV - Clinic | Data Layer Variable | clinic |
| DLV - Lead Source | Data Layer Variable | lead_source |
| DLV - Page Path | Data Layer Variable | page_path |

Note: `name` and `phone` are intentionally not read into GTM variables here — they never enter the dataLayer in the first place (see Google Analytics 4 Event Configuration above). GTM only sees what it needs to report a conversion, not who the patient is.

## Tags Fired by This Trigger

| Tag Name | Tag Type | Key Fields |
|---|---|---|
| GA4 - Consultation Submitted | Google Analytics: GA4 Event | Event Name: `generate_lead` · Parameters: clinic = {{DLV - Clinic}}, lead_source = {{DLV - Lead Source}}, page_path = {{DLV - Page Path}} |
| Ads - Consultation Conversion | Google Ads Conversion Tracking | Conversion ID/Label from Google Ads · Conversion Value: not set (lead conversions are counted, not valued, since no transaction has occurred yet) |

Both tags fire in parallel off the same trigger — there's no dependency between them, so no tag sequencing is needed here. (Sequencing would only matter if, say, the Ads tag needed a value that the GA4 tag calculated first, which isn't the case for a simple lead conversion.)

---

## Google Analytics 4 Event Configuration

| Event Name | Parameters |
|---|---|
| consultation_form_submitted | clinic, lead_source, page_path |

Name and phone are never sent to GA4 — GA4 only needs to know a conversion happened and where it came from. The patient's actual identity lives in HubSpot only, reached via the direct backend API call, not through GTM/GA4.

---

# CRM Integration Flow

After successful form submission:

1. User submits the consultation form.
2. The frontend validates the input.
3. A POST request is sent to the backend API at /api/leads.
4. The backend searches HubSpot Contacts by phone number, then creates or updates the contact.
5. The backend calls the Karix WhatsApp API to send the confirmation.
6. A success response is returned to the landing page.
7. The user sees a confirmation message, no page reload.

This workflow ensures every qualified lead is securely stored and available for follow-up, and that the WhatsApp confirmation and CRM write happen as one coordinated backend flow rather than three separate systems each assuming another already ran.

## Technology Choice

The integration uses a direct API call from a custom backend to HubSpot's Contacts API — not the HubSpot Forms API, native embed, Zapier, or Make. The Forms API only accepts a submission; it can't run the phone-number lookup this design needs before deciding whether to create or update a contact. Zapier and Make add an extra network hop and reduce control over both the dedup logic and the 2-minute WhatsApp SLA. A direct API call from a backend we control gives full control over lookup-before-write logic, retry behavior, and timing.
---

# Error Handling

- Validate required fields on both client and server.
- Display user-friendly validation messages inline, not via alert() in production.
- Prevent duplicate form submissions (disable the submit button on click, re-enable on error).
- Handle HubSpot/Karix API errors gracefully — never let a downstream failure block the user from seeing a thank-you state, since the retry queue handles recovery.
- Log integration errors (HubSpot write failures, Karix send failures) for debugging and monitoring, tagged with a request ID so a single lead's full journey can be traced across systems.
---

# Biggest Failure Point

The most critical failure point is the phone-number dedup step, not just raw API downtime — see the Written Answer above for the full reasoning and the same-phone-different-name fallback.

For pure availability failures: if HubSpot is temporarily unreachable, the backend writes the lead to a retry queue and reattempts automatically. Integration errors are logged and monitored so no lead is permanently lost to downtime.


# HubSpot Contact Deduplication

HubSpot normally identifies duplicate contacts using email addresses.

Because this landing page collects only phone numbers, the backend first searches HubSpot for an existing contact using the phone number.

If a matching phone number exists, the existing contact is updated.

Otherwise, a new contact is created.

This prevents duplicate patient records and maintains accurate CRM data.

### Same phone, different name

Shared phone numbers are common in Indian household bookings — a spouse or parent's number used to book for someone else. If a submission matches an existing contact but the name differs, the backend does not blindly overwrite the name field. Instead it logs the new name and details as an activity/note on the existing contact and flags it for manual review, so front-desk staff catch the mismatch before the wrong patient context reaches the doctor.

# WhatsApp SLA Monitoring

After a successful CRM update, the backend immediately calls the Karix WhatsApp API.

Every submission is timestamped at form-receipt and again at WhatsApp-delivery-webhook; the delta is computed on every send. If the message is not delivered within roughly 90 seconds — a buffer ahead of the actual 2-minute SLA — the system logs the delay, retries the send, and alerts the operations team (Slack/PagerDuty) so a human can intervene before a patient actually notices a missed confirmation.

# Security Considerations

- Validate all inputs on both client and server.
- Use HTTPS for secure data transmission.
- Protect the /api/leads endpoint and all downstream calls (HubSpot, Karix) with authentication and rate limiting.
- Sanitize user input to prevent malicious attacks.
- Never expose HubSpot or Karix API keys in frontend code — all CRM/WhatsApp calls happen server-side only.
---

# Benefits of the Integration

- Accurate lead tracking, with PII correctly separated from analytics/ad platforms
- Reliable analytics reporting via GA4 and Google Ads
- Improved marketing attribution through consistent lead_source and clinic parameters
- Better conversion measurement without PII compliance risk
- Faster, more reliable lead management through direct CRM integration with a real dedup strategy
---

# Conclusion

This integration design provides a scalable and reliable architecture for tracking consultation requests. By connecting Google Tag Manager, Google Analytics 4, Google Ads, and HubSpot CRM through a backend that owns the sensitive parts of the flow, OrthoNow can accurately measure conversions, optimize marketing campaigns, and manage patient leads without either losing leads to downtime or exposing PII to ad platforms.
## API Request Example

### POST /api/leads

Request Body

json
{
  "name": "Santusht",
  "phone": "9876543210",
  "clinic": "Indiranagar",
  "lead_source": "google_ads_consultation_landing_page"
}


Response

json
{
  "success": true,
  "message": "Lead created successfully."
}
User
  │
  ▼
Landing Page
  │
  ▼
JavaScript Validation
  │
  ▼
POST /api/leads
  │
  ▼
Backend API
  │
  ├── Search HubSpot by phone → create or update contact
  ├── Call Karix WhatsApp API → send confirmation
  ├── (dataLayer.push already fired GA4 + Ads on submit, in parallel)
  └── Return success