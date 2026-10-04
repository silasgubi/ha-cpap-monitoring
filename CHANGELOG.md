# Changelog

## 1.0.0 (2026-10-04)

First public release. In daily use since June 2026, with a parent's CPAP and the caregiver's own.

- Per-night alerts for AHI, mask leak and low usage, triggered when myAir syncs a new night (push to the caregiver, plus Alexa for the second user)
- 08:30 spoken good-morning summary on the patient's Echo devices
- Daily Telegram summary for the patient's children, with an 11:00 retry when myAir syncs late
- "No use recorded" warning at 11:15 (Alexa, push and Telegram), once a day
- Monthly Telegram report on day 1 in the style of ResMed's PDF (average usage, AHI, leak, % nights ≥4h/≥6h), accumulated night by night
- Optional Gemini wording (`ai_task.generate_data`) for the Alexa summary and both Telegram messages, with fixed-text fallback. Each prompt is written for its audience: Alexa speaks to the patient, Telegram talks about her to her children.
- Second CPAP user: alerts, 09:00 summary and "no new data" reminder, with Do Not Disturb handling on their Echo
- Lovelace dashboard: overview, one view per user, 7-day trends, monthly accumulator, test buttons and recent activity
- Thresholds as helpers (AHI 5, leak 24 L/min, usage 4 h, score 70)
