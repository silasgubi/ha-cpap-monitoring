# Architecture

## Data flow

```
 CPAP (Maria)  ─┐                    ┌─ ResMed myAir integration ─ sensor.cpap_*      (user1)
 CPAP (João)   ─┴─► myAir cloud ─────┤                             sensor.cpap_*_2    (user2)
                                     │
                                     ▼
                     packages/cpap_monitoring.yaml
                     ├─ binary_sensor.cpap_userN_*_alert   (value vs threshold)
                     ├─ input_number thresholds + monthly accumulator
                     └─ input_boolean toggles, input_text guards
                                     │
          ┌────────────────┬─────────┴────────┬───────────────────┐
          ▼                ▼                  ▼                   ▼
   Push (caregiver)   Alexa (patient)   Telegram (family)     Dashboard
                            ▲                  ▲
                            └──── optional Gemini wording (ai_task) ────┘
```

## Where the data comes from

The myAir integration polls ResMed's cloud API. The machine uploads a night's session once, usually shortly after the mask comes off in the morning, so values change **once per day**. Nothing here is real-time.

`sensor.most_recent_sleep_date` changes when a new night arrives. Every per-night automation triggers on that change instead of a fixed time. This also means a night that stays out of range two days in a row still alerts twice: the alert binary sensor doesn't need to flip, because the date did.

## Who hears what

| Channel | Audience | Tone | Automations |
| --- | --- | --- | --- |
| Alexa announce on Maria's Echo | Maria | Speaks **to** her, warm, simple advice | `user1_morning_summary`, `user1_no_data_alert` |
| Telegram group | Maria's children | Talks **about** her, third person, advice for them | `telegram_family_summary`, `cpap_monthly_report`, `user1_no_data_alert` |
| Push + persistent notification | Caregiver | Short and technical | `user1_alerts`, `user1_no_data_alert`, `user2_*` |
| Alexa announce on João's Echo | João | Speaks to him | `user2_*` |

Maria's per-night alerts go only to the caregiver, not to her Alexa. She hears one friendly summary at 08:30 that already covers anything out of range, and the caregiver gets the numbers.

## Schedule

| Time | Automation | Condition |
| --- | --- | --- |
| New night synced, 07:00–22:00 | `user1_alerts`, `user2_alerts` | Toggle on + alert binary sensor on |
| New night synced | `cpap_month_accumulate` | Night not counted yet |
| 08:30 | `user1_morning_summary` | Toggle on + data from last night |
| 08:35, 11:00 | `telegram_family_summary` | Toggle on + fresh data + night not sent yet |
| 08:40 on day 1 | `cpap_monthly_report` | Always |
| 09:00 | `user2_morning_summary` | Toggle on + fresh data |
| 11:15 | `user1_no_data_alert` | Nothing newer than yesterday, once a day |
| 11:30 | `user2_no_data_alert` | Same, for João |

## Design decisions

**Monthly totals are accumulated, not queried.** Home Assistant's recorder usually keeps about 10 days of history, too short for a 30-day average. `cpap_month_accumulate` adds each night to `input_number.cpap_month_*` and `cpap_monthly_report` reads the averages on day 1, then resets them. `input_text.cpap_month_last_accumulated_date` prevents counting a night twice. These helpers must not have `initial:`, because Home Assistant would reset them to that value on every restart.

**Telegram has a late retry.** myAir sometimes syncs after 08:35 (Maria slept in, or the machine was switched off late), so the daily summary tries again at 11:00. `input_text.cpap_telegram_family_last_sent` makes sure only one of the two runs sends.

**"No data" means "not used", most of the time.** If nothing newer than yesterday arrives by 11:15, the usual cause is that the CPAP wasn't used, less often a Wi-Fi or sync problem. The warning says so gently, once a day (`input_text.cpap_user1_no_data_alerted_date`).

**AI writes the words, never the numbers.** Gemini receives the real values as a list of facts and is told not to add any number or clinical claim. The fixed template stays as a fallback.

**Do Not Disturb on João's Echo.** If his bedroom Echo stays on DND overnight, the `user2_*` automations switch DND off, speak, then switch it back on.

## Thresholds

| Helper | Default | Basis |
| --- | --- | --- |
| `input_number.cpap_ahi_threshold` | 5 events/h | ResMed myAir report guide |
| `input_number.cpap_leak_threshold` | 24 L/min | ResMed 95th-percentile reference |
| `input_number.cpap_usage_min_hours` | 4 h | ResMed minimum recommendation |
| `input_number.cpap_score_threshold` | 70 | myAir scoring (dashboard only) |

The monthly report always counts nights at ≥4 h and ≥6 h, the fixed reference points of ResMed's own PDF report, independent of the alert threshold.
