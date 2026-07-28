## Inspiration

I grew up around smallholder farmers in Uganda, and I kept seeing the same problem play out: a farmer notices something wrong with a leaf, but the nearest agricultural extension officer might be days away or covering more farms than they can realistically reach. By the time anyone with real expertise sees the plant, a treatable problem has often already spread through the field. At the same time, nearly every one of those farmers already carries a smartphone. That camera is the fastest access point to expert judgment that already exists — it just needed something smart enough sitting behind it. That gap is what I built AgriDoc AI to close.

## What it does

Point a phone camera at a cassava, maize, banana, or coffee leaf, and AgriDoc AI returns a diagnosis, a treatment plan shaped by that day's actual weather on the farm, and a plain-language explanation of the biology behind what's happening — all in seconds.

But I didn't want it to just hand over a verdict. So every diagnosis:
- Shows numbered pins landing directly on the farmer's own photo, pointing at exactly what the AI saw
- Lists what else was considered and ruled out, not just the final answer
- Comes with a confidence score that's actually checked against reality — a live dashboard tracks whether "90% confident" diagnoses are actually right 90% of the time, using real farmer-confirmed outcomes
- Turns into a personal micro-lesson, filed into a curriculum that's built entirely from that farmer's own real cases over time

Around the core diagnosis engine, there's a live Outbreak Map (opt-in, aggregated across users), a Soil Check tool, a Field Journal, a full 5-lesson curriculum per crop, multilingual voice output in English, Luganda, and Swahili, and an Export for Expert Review feature that packages photos and diagnoses into a bundle an agronomist can actually validate.

## How I built it

AgriDoc AI is a single self-contained HTML/CSS/JS file — no build step, no framework, deployable anywhere a browser runs, which mattered a lot given the low-infrastructure context I was designing for.

The diagnosis pipeline is layered, not a single API call:

1. An on-device pixel analysis (HSV color-space thresholding, no AI) runs first, measuring what percentage of the leaf is green, yellowing, or necrotic — instantly, before any network request.
2. That measurement, plus a curated reference set of the major documented diseases for the selected crop (summarized from published Uganda-focused agricultural research), gets fed into the prompt for a multimodal model — Gemini 3.5 Flash — as grounding context the model has to cross-check its visual read against.
3. If that first pass comes back under a confidence threshold, a stronger model (Gemini 3.1 Pro) automatically reviews the same photo. Most diagnoses stay fast on the first model; genuinely uncertain cases get a real second opinion instead of a shrug.
4. For the AI calls themselves, I built a hybrid architecture: direct calls to Google's Gemini API when a key is configured, with automatic fallback through Puter.js's free proxy if that fails — so the app stays fully functional on zero-cost infrastructure either way.

For data, I used Firebase's free tier (anonymous auth + Firestore) to power the live Outbreak Map and the confidence calibration dashboard, and Open-Meteo for real per-district weather that feeds directly into diagnosis reasoning and treatment timing.

## Challenges I ran into

Getting reliable AI access on zero-cost infrastructure was harder than I expected. Puter.js's free proxy has a storage ceiling that a photo-heavy app runs into quickly, which is what pushed me toward the hybrid direct-API-plus-fallback architecture instead of relying on one path.

The harder challenge was resisting the pull toward features that look impressive but overclaim. Early on I considered faking cross-user data on the Outbreak Map to make it look more alive — I killed that and labeled it honestly as sample data instead, because a farmer making a real decision based on a fabricated outbreak signal is a genuinely bad outcome, not just a bad look. That same instinct shaped the confidence calibration dashboard: instead of just displaying a confidence percentage and hoping it's trustworthy, I built the plumbing to actually check it against farmer-reported outcomes over time, and to say plainly when there isn't enough data yet rather than showing a fabricated number.

Getting the AI to point at real coordinates on a farmer's own photo (for the symptom pins) also took real tuning — spatial grounding from a vision model isn't perfectly reliable, so I built a validation check that silently falls back to plain text if the returned coordinates don't line up with what was actually asked for, rather than ever showing a pin I couldn't stand behind.

## What I learned

I started out planning a simultaneous multi-model ensemble for every diagnosis, and abandoned it for a confidence-triggered cascade instead — it's cheaper, faster for the common case, and easier to explain honestly to a farmer than "three AIs voted." That trade-off — cascade over ensemble — ended up teaching me more about designing AI systems for real constraints than the ensemble version ever would have.

I also came away convinced that a calibrated confidence score is worth more than a high one. A model that says 60% and is right 60% of the time is more useful, and more trustworthy, than one that says 95% and is right 70% of the time. Building the infrastructure to actually measure that, instead of just asserting it, was the single most technically interesting part of this project.

## Accomplishments that I'm proud of

Building a full diagnosis pipeline — on-device pre-processing, retrieval-grounded reasoning, a confidence-triggered model cascade, live multi-user data, and an honest self-calibration loop — as a single-file, zero-cost-infrastructure app, solo, in a short build window. And doing it without cutting the corner that mattered most to me: every honesty-vs-impressiveness decision in this project went to honesty, even when the impressive version would have been an easier demo.

## What's next for AgriDoc AI

The most important next step is formal validation — running the diagnosis engine against a labeled dataset with real agronomists, using the Export for Expert Review pipeline I already built for exactly that purpose. Beyond that: expanding the curated reference set as more crops and districts come online, an SMS/USSD fallback for farmers without reliable data access, and deeper district-level outbreak alerting as real usage — and real data — grows.
