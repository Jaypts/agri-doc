# 🌱 AgriDoc AI

**A phone-based plant doctor for smallholder farmers — built in Uganda, for Uganda.**

Photograph a sick leaf. Get a diagnosis, a treatment plan, and the biology behind it, in seconds, on the phone already in a farmer's pocket — no extension officer visit required, no waiting for symptoms to spread.

Built solo for the **Prometheus July AI Challenge**.

---

## The problem

In Uganda, agricultural extension officers — the trained staff who normally help farmers identify crop problems — are stretched far too thin to reach most smallholder farms regularly. By the time a farmer can get expert eyes on a sick plant, a treatable problem has often already spread through the field. Meanwhile, most farmers — even in remote areas — now carry a basic smartphone. That camera is the fastest expert-access point already sitting in their pocket. AgriDoc AI exists to put a plant pathologist's judgment, and the reasoning behind it, into that camera.

## What it does

Point the camera at a cassava, maize, banana, or coffee leaf. AgriDoc AI:

1. Runs an **on-device pixel analysis** of the leaf's color signature before any AI call is made
2. Sends the photo, that color data, live weather conditions for the farmer's district, and a **curated reference set** of the major documented diseases for that crop to a multimodal AI model
3. Returns a diagnosis with a **calibrated confidence score**, a plain-language explanation of the biology, a differential diagnosis (what else was considered and ruled out, and why), and a treatment plan shaped by *today's actual weather* on that farm
4. **Shows its work visually** — numbered pins land directly on the farmer's own photo, pointing at exactly what the AI saw
5. Turns that diagnosis into a **personal lesson**, filed into a growing curriculum built entirely from the farmer's own real cases

Nothing above is a black-box guess. Every layer is designed to be inspected, not just trusted.

---

## Why this fits the brief

| Judging criterion | How AgriDoc AI delivers |
|---|---|
| **Educational Impact** | Every diagnosis becomes a personalized lesson, not just an answer — the Learn Library grows out of the farmer's *own* real cases, on top of a full 5-lesson curriculum per crop. It teaches the reasoning (differential diagnosis, "why this happens," weather-linked cause) instead of handing over a verdict to memorize. |
| **Creative Use of AI/ML** | A multi-signal pipeline, not a single API call: on-device HSV color pre-filtering, a confidence-triggered two-model cascade (fast model → stronger model only when uncertain), retrieval-style grounding against a curated disease reference set, visual symptom localization on the photo itself, and a self-reported, farmer-verified confidence calibration loop. AI is core to almost every screen, not a bolt-on chatbot. |
| **Technical Execution** | Single-file, zero-cost-infrastructure deployment (Puter.js + free-tier Firebase + Open-Meteo) that still supports live multi-user data (Outbreak Map, calibration dashboard), offline-friendly on-device pre-processing, graceful multi-layer fallbacks, and a real auth/data layer — not a static prototype. |
| **The Pitch & Demo** | Every feature is designed to be *seen* working live: pins landing on a real photo, an AI catching its own wrong crop guess, a calibration bar filling in from real feedback. Nothing here needs to be taken on faith during a 2-minute video. |

---

## Key features

- 📷 **Live camera scanner** with on-device HSV pixel-color pre-analysis before any AI call
- 🧠 **Two-model confidence cascade** — a fast model handles most cases; a stronger model automatically reviews anything below the confidence threshold, and disagreements are shown honestly rather than silently resolved
- 📖 **Curated reference grounding** — diagnoses are cross-checked against a small reference set of documented crop diseases summarized from published Uganda-focused agricultural research, and the model says plainly when a case doesn't match anything in it
- 📍 **Visual Symptom Mapping** — numbered pins overlaid directly on the farmer's own photo, showing exactly what the AI saw and where
- 🔍 **Differential diagnosis** — the app shows what else it considered and ruled out, not just the final answer
- 📊 **Confidence calibration dashboard** — a live, honest measurement (built from real farmer ✅/❌ feedback across all users) of whether a stated confidence score is actually trustworthy
- 📚 **Personalized micro-lessons** — every real diagnosis generates a bespoke lesson tied to that exact case, growing a curriculum unique to each farmer
- 🌦️ **Live weather integration** (Open-Meteo) — temperature, humidity, soil moisture, ET₀, and UV feed directly into diagnosis reasoning and treatment timing
- 🗺️ **Outbreak Map** — live, opt-in aggregated disease reports across users, plotted on a validated Uganda district outline (honestly labeled as sample data where real reports are still sparse)
- 🌍 **Multilingual voice output** — English, Luganda, Swahili
- 🪴 **Soil Check** via AI vision, plus a guided hand-feel texture test
- 📓 **Field Journal** with follow-up tracking and farmer-editable notes
- 📤 **Export for Expert Review** — generates a CSV + photo bundle for agronomist validation
- 🔐 **Consent modal** before first diagnosis, and transparent data handling throughout

---

## How it actually works

AgriDoc AI does **not** run a single black-box guess. Each diagnosis goes through several real layers:

1. An independent pixel-level color analysis runs in the browser — no AI, just HSV color-space thresholding — measuring what % of the leaf is green, yellowing, or necrotic.
2. That measurement, plus a curated reference set of the major documented diseases for the selected crop, is fed into the prompt for a multimodal AI model (Gemini 3.5 Flash) as cross-checks against its own visual read.
3. If that first read comes back with low confidence, a second, stronger model (Gemini 3.1 Pro) automatically reviews the same photo — most diagnoses stay fast on the first model alone, but genuinely uncertain cases get a real second opinion instead of just a warning label.
4. The model is instructed to say plainly when a case doesn't match anything in the curated reference set, rather than forcing a fit — and when the two models disagree, that disagreement is shown to the farmer honestly rather than silently resolved.
5. Every diagnosis a farmer marks ✅ or ❌ correct feeds a live calibration measurement, so the app's own confidence scores can be checked against reality instead of just asserted.

This is an honest design choice: it doesn't replace a properly trained, validated classifier, but it's a real multi-signal, partially-grounded check rather than trusting one raw model guess with no cross-reference — and every layer is visible to the farmer, not hidden.

---

## Tech stack

- **AI**: Gemini 3.5 Flash (primary) and Gemini 3.1 Pro (low-confidence fallback), accessed via a hybrid architecture — direct Google API calls when a key is configured, with automatic fallback through [Puter.js](https://puter.com) (free-tier) if the direct call fails, so the app stays fully functional on zero-cost infrastructure
- **Backend/data**: Firebase (anonymous auth + Firestore) for diagnosis logging, live Outbreak Map data, and calibration data — all on the free tier
- **Weather**: Open-Meteo, live per-district conditions feeding directly into diagnosis reasoning
- **Frontend**: Single-file HTML/CSS/JS — no build step, no framework dependency, deployable anywhere a browser runs, which matters for low-infrastructure contexts

---

## Honest limitations

This is a hackathon prototype, not a validated medical device for plants:

- It has **not** been benchmarked against a labeled dataset of confirmed diagnoses — the "Export for Expert Review" feature exists specifically to start that validation process with real agronomists
- The Outbreak Map and calibration dashboard are only as meaningful as the (currently small) number of real users feeding them data — both are labeled honestly rather than dressed up with fabricated numbers
- The curated reference set covers the major, well-documented diseases per crop — it is a grounding aid, not an exhaustive database, and the app is built to say so rather than pretend otherwise

I'd rather ship something that's honest about what it doesn't know yet than something that looks more finished than it is.

---

## Getting started

AgriDoc AI is a single self-contained HTML file — no build process required.

1. Clone this repo
2. Open `AgriDoc_AI.html` directly in a browser, or serve it locally (`python3 -m http.server`, then visit `localhost:8000`)
3. (Optional) Add a Gemini API key in **Farm Profile → Developer settings** for faster direct-API responses — without one, the app runs automatically through the free Puter.js fallback

No `npm install`, no API keys required to try it, no cost to run.

---

## What's next

- Formal accuracy validation against agronomist-labeled photo sets, using the Export for Expert Review pipeline already built
- Expanding the curated reference set as more crops and districts are added
- SMS/USSD fallback for farmers without reliable data access
- Deeper district-level outbreak alerting as real usage grows

---

## Built by

I'm Jay, a student in Uganda building AI tools for the smallholder farmers I grew up around

GitHub: [@Jaypts](https://github.com/Jaypts)
