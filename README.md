# Namoza Developer Assignment

Submission for the Namoza Developer Assignment — Developer, Position 1 (Client Web + Martech).
Client scenario: OrthoNow, a 9-clinic orthopaedic chain across Bengaluru, Hyderabad, and Chennai.

**Live demo:** https://santusht98-sketch.github.io/namoza-assignment/task2/index.html
**Loom walkthrough:** [add your Loom link here]

---

## Project Structure
namoza-assignment/
├── task1/
│   └── GTM-Schema.md            # Event schema, dataLayer JSON, funnel + Ads conversion reasoning
├── task2/
│   └── index.html               # Consultation landing page (single self-contained file)
├── task3/
│   └── Integration-Design.md    # HubSpot / WhatsApp / Ads integration architecture (written answer)
├── screenshots/
│   ├── mobile/                  # Mobile view screenshots
│   │   ├── 01-hero.jpeg
│   │   ├── 02-form.jpeg
│   │   └── 03-faq.jpeg
│   └── desktop/                 # Desktop view + Lighthouse screenshots
│       ├── 01-hero.png
│       ├── 02-form.png
│       ├── 03-faq.png
│       ├── 04-pagespeed-desktop.png
│       └── 05-pagespeed-mobile.png
└── README.md


---

## Tasks

### Task 1 — [GTM Event Schema](./task1/GTM-Schema.md)
Full event tracking schema for OrthoNow's site: event names, GTM trigger types, key parameters, and the GA4 report/audience each event feeds. Includes the 3-step booking form's dataLayer JSON, the GA4 → GTM event mapping, the primary Google Ads conversion and why it was chosen over the alternatives, and how funnel drop-off is surfaced in GA4 Funnel Exploration.

### Task 2 — [Consultation Landing Page](./task2/index.html)
Rebuild of OrthoNow's "Book a Consultation" landing page for the campaign *"Get an expert orthopaedic opinion — Book your consultation at OrthoNow."* Single HTML file, vanilla JS, no frameworks or external assets. The form submit fires a `consultation_form_submitted` dataLayer push (visible live in the browser console) and swaps to a thank-you state without a page reload.

**PageSpeed Mobile score:** see [`screenshots/desktop/05-pagespeed-mobile.png`](./screenshots/desktop/05-pagespeed-mobile.png)

## Screenshots

### Mobile
![Hero](screenshots/mobile/01-hero.jpeg)
![Booking Form](screenshots/mobile/02-form.jpeg)
![FAQ](screenshots/mobile/03-faq.jpeg)

### Desktop
![Hero](screenshots/desktop/01-hero.png)
![Booking Form](screenshots/desktop/02-form.png)
![FAQ](screenshots/desktop/03-faq.png)

### Lighthouse Scores
![Desktop PageSpeed](screenshots/desktop/04-pagespeed-desktop.png)
![Mobile PageSpeed](screenshots/desktop/05-pagespeed-mobile.png)

---

### Task 3 — [Integration Design](./task3/Integration-Design.md)
Written architecture for connecting the landing page to HubSpot CRM and the Karix WhatsApp Business API: why a direct backend API call was chosen over the HubSpot Forms API / Zapier / Make, the phone-number deduplication trap (HubSpot dedups on email; this form only collects phone), the fallback for the biggest failure point, and how the 2-minute WhatsApp SLA is monitored.

---

## Running Task 2 locally

`task2/index.html` is fully self-contained — no build step, no server required.

```bash
open task2/index.html        # macOS
start task2/index.html       # Windows
```

Open the browser console before submitting the form to see the `consultation_form_submitted` dataLayer push.

---

## Technologies Used

- HTML5, CSS3, vanilla JavaScript (no frameworks)
- Google Tag Manager / GA4 / Google Ads (schema design, Task 1)
- HubSpot CRM + Karix WhatsApp Business API (integration design, Task 3)
- Git, GitHub, GitHub Pages

---

## Author

Santusht
[GitHub profile link] · [santusht98@gmail.com]