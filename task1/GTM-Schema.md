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

| Event Name | Trigger | Parameters | Purpose |
|------------|---------|------------|---------|
| booking_step_1_complete | User completes Step 1 of booking | step_number, page_name | Track first step of booking funnel |
| booking_step_2_complete | User completes Step 2 of booking | step_number, page_name | Measure booking progress |
| booking_step_3_complete | User completes Step 3 of booking | step_number, page_name | Track completed booking |
| consultation_form_submitted | User submits consultation form | name, phone | Record lead generation |
| click_call_button | User clicks Call button | button_text | Measure call intent |
| click_whatsapp | User clicks WhatsApp button | button_text | Measure WhatsApp engagement |
| patient_guide_download | User downloads patient guide | file_name | Track guide downloads |
| clinic_page_view | User visits clinic page | page_name | Measure clinic page views |
| article_read_50 | User scrolls 50% of article | article_title | Track content engagement |
| article_read_90 | User scrolls 90% of article | article_title | Measure highly engaged readers |

---

## dataLayer Examples
The following example demonstrates how a successful consultation form submission sends an event to the GTM dataLayer. GTM listens for this event and forwards it to GA4 and Google Ads for analytics and conversion tracking.

### Example: Consultation Form Submission

<!-- javascript
dataLayer.push({
  event: "consultation_form_submitted"
}); -->
### Example: Booking Step 1

javascript
dataLayer.push({
  event: "booking_step_1_complete",
  step_number: 1,
  page_name: "booking"
});


### Example: Booking Step 2

javascript
dataLayer.push({
  event: "booking_step_2_complete",
  step_number: 2,
  page_name: "booking"
});


### Example: Booking Step 3

javascript
dataLayer.push({
  event: "booking_step_3_complete",
  step_number: 3,
  page_name: "booking"
});


### Example: Consultation Form Submission

javascript
dataLayer.push({
  event: "consultation_form_submitted",
  name: "John Doe",
  phone: "+91XXXXXXXXXX"
});


---

## Google Analytics Mapping

Explain how GTM events are sent to Google Analytics 4 (GA4).
Each GTM event is mapped to a corresponding GA4 event using Google Tag Manager. Event parameters such as page name, button text, and booking step are passed to GA4 to support user journey analysis, funnel reporting, and engagement measurement.

---

## Google Ads Conversion

Explain which event is marked as a Google Ads conversion and why.
The consultation_form_submitted event is configured as the primary Google Ads conversion because it represents a completed lead submission. This conversion is used to measure campaign effectiveness and optimize advertising performance.

---

## Validation Checklist

- [ ] Event names follow naming convention
- [ ] dataLayer events tested
- [ ] GTM Preview mode verified
- [ ] GA4 DebugView verified
- [ ] Google Ads conversion configured