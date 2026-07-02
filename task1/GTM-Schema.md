# GTM Event Schema

## Objective

Briefly describe the purpose of this document.
The purpose of this document is to define the Google Tag Manager (GTM) event tracking strategy for the landing page. These events help measure user interactions, optimize the booking journey, improve marketing performance, and enable accurate reporting in Google Analytics 4 (GA4) and Google Ads.

---

## Event Naming Convention

Explain the naming standards followed for GTM events.
All event names follow the GA4 recommended naming convention using lowercase letters and underscores. Event names are descriptive, consistent, and represent a single user action, making them easy to manage across GTM, GA4, and advertising platforms.

---

## Events

| Event Name | Trigger Type | Key Parameters | GA4 Report / Audience |
|---|---|---|---|
| booking_step_1_complete | Custom Event | step_number, step_name, clinic_location, specialty | Funnel Exploration: Booking Funnel (Step 1) |
| booking_step_2_complete | Custom Event | step_number, step_name, clinic_location, has_preferred_date | Funnel Exploration: Booking Funnel (Step 2) |
| booking_step_3_complete | Custom Event | step_number, booking_status, clinic_location, appointment_id | Funnel Exploration: Booking Funnel (Step 3) → Audience: "Booking Started, Not Confirmed" (started, no Step 3) for remarketing |
| consultation_form_submitted | Custom Event | clinic, lead_source, page_path | Conversions report → imported as Google Ads conversion (primary) |
| click_call_button | Click – All Elements (Click Text/ID contains "call") | button_text, page_name, timestamp | Engagement > Events report → Audience: "High Intent – Call Clickers" |
| click_whatsapp | Click – All Elements (Click Classes contains "whatsapp-widget") | button_text, page_name, timestamp | Engagement > Events report |
| patient_guide_download | Form Submission (Form ID = patient-guide-form) | file_name, page_name, timestamp | Engagement > Events report → Audience: "Guide Downloaders" nurture list |
| clinic_page_view | Page View (Page Path matches clinic page pattern) | clinic_location, page_title, page_url | Engagement > Pages and screens report, broken down by clinic_location |
| article_read_50 | Scroll Depth (Vertical Scroll = 50%) | article_title, scroll_percent, page_url | Engagement report |
| article_read_90 | Scroll Depth (Vertical Scroll = 90%) | article_title, scroll_percent, page_url | Audience: "Highly Engaged Blog Readers" for remarketing |

---

## dataLayer Examples
The following example demonstrates how a successful consultation form submission sends an event to the GTM dataLayer. GTM listens for this event and forwards it to GA4 and Google Ads for analytics and conversion tracking.

### Example: Booking Step 1

javascript
dataLayer.push({
  event: "booking_step_1_complete",
  step_number: 1,
  step_name: "location_specialty_selected",
  clinic_location: "Indiranagar",
  specialty: "Orthopaedics"
});


### Example: Booking Step 2 (no PII — patient details are entered here but never leave the form's local state until final submission)

javascript
dataLayer.push({
  event: "booking_step_2_complete",
  step_number: 2,
  step_name: "contact_details_entered",
  clinic_location: "Indiranagar",
  has_preferred_date: true
});


### Example: Booking Step 3

javascript
dataLayer.push({
  event: "booking_step_3_complete",
  step_number: 3,
  booking_status: "Confirmed",
  clinic_location: "Indiranagar",
  appointment_id: "BK1023"
});


### Example: Consultation Form Submission

dataLayer.push({
  event: "consultation_form_submitted",
  clinic: "Indiranagar",
  lead_source: "google_ads_consultation_landing_page",
  page_path: "/book-a-consultation"
});


---

## Google Analytics Mapping

Each GTM event is sent to GA4 via a GA4 Event tag. Where GA4 has a recommended event name for the action (which unlocks built-in reports), the tag maps our internal GTM event name to GA4's recommended name. Where there isn't a good fit, the custom name is kept as-is.

| GTM Event Name | GA4 Event Name Sent | Marked as GA4 Key Event? | Notes |
|---|---|---|---|
| consultation_form_submitted | generate_lead | Yes (primary) | GA4's recommended event for lead generation; unlocks default lead-gen reporting |
| booking_step_1_complete | booking_step_1_complete | No | No GA4 recommended equivalent for a form-step; kept custom for funnel exploration |
| booking_step_2_complete | booking_step_2_complete | No | Same as above |
| booking_step_3_complete | booking_step_3_complete | Yes (micro-conversion) | Marked as a Key Event to build the "started but didn't submit" remarketing audience |
| click_call_button | click_call_button | No | Custom — no close GA4 recommended equivalent |
| click_whatsapp | click_whatsapp | No | Custom |
| patient_guide_download | file_download | No | Maps to GA4's recommended file_download event; fired manually since it's gated behind a form rather than auto-tracked |
| clinic_page_view | clinic_page_view | No | Kept custom (separate from GA4's automatic page_view) so clinic_location can be used as a report dimension |
| article_read_50 | scroll_50 | No | GA4 auto-scroll only fires at 90%; 50% checkpoint added manually |
| article_read_90 | scroll_90 | No | Overrides/supplements GA4's default 90% auto-scroll event with article-specific parameters |

---

## Google Ads Conversion

The `consultation_form_submitted` event is configured as the primary Google Ads conversion because it represents a completed, qualified lead submission — the clearest and most reliable signal of campaign performance.

It's the strongest candidate over `click_call_button` or `booking_step_3_complete` because it's the only event that is both unambiguous (a completed lead, not a soft signal like a call click that could be a wrong number or general enquiry) and directly attributable to a single ad campaign's landing page — unlike `clinic_page_view` or the `article_read_*` events, which mix branded and non-paid traffic and can't be cleanly tied back to one campaign.

---
## Funnel Analysis

The booking funnel is measured using the following sequence in Google Analytics 4 Funnel Exploration:

booking_step_1_complete
↓

booking_step_2_complete
↓

booking_step_3_complete
↓

consultation_form_submitted

This funnel helps identify where users abandon the booking journey so the marketing team can optimize the booking experience.

## Validation Checklist

- [ ] Event names follow naming convention
- [ ] dataLayer events tested
- [ ] GTM Preview mode verified
- [ ] GA4 DebugView verified
- [ ] Google Ads conversion configured