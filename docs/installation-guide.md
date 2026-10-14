# Installation Guide

This guide assumes you can edit your Home Assistant configuration files (via the File editor or Studio Code Server add-on, Samba, or SSH) and have HACS installed.

Reminder of the example names used in every file: **Maria** is the person being looked after (first myAir device, `user1`), **João** is the caregiver who also uses a CPAP (second myAir device, `user2`).

## 1. Install the integrations

Via HACS:

- **ResMed myAir**: `prestomation/resmed_myair_sensors`. Add it in **Settings → Devices & Services → Add Integration → ResMed myAir** and log in with the CPAP user's myAir account. Add a second entry for a second CPAP.
- **Mushroom Cards**: `piitaya/lovelace-mushroom` (dashboard).
- **Alexa Media Player**: `alandtse/alexa_media_player`. Optional, only for spoken messages. If the Echo devices are in another person's home and Amazon account, see [second-alexa-account.md](second-alexa-account.md).

The **Home Assistant Companion app** on the caregiver's phone provides the push notifications.

## 2. Check the myAir entity IDs

The automations expect the IDs the integration creates by default:

| | First device (Maria) | Second device (João) |
| --- | --- | --- |
| AHI | `sensor.cpap_ahi_events_per_hour` | `sensor.cpap_ahi_events_per_hour_2` |
| Mask leak | `sensor.cpap_mask_leak` | `sensor.cpap_mask_leak_2` |
| Usage minutes | `sensor.cpap_usage_minutes` | `sensor.cpap_usage_minutes_2` |
| myAir score | `sensor.cpap_total_myair_score` | `sensor.cpap_total_myair_score_2` |
| Last night synced | `sensor.most_recent_sleep_date` | `sensor.most_recent_sleep_date_2` |

Check yours at **Settings → Devices & Services → ResMed myAir → [device] → Entities**. If they differ, search and replace them in every file you copy. The full list is in [../config/README.md](../config/README.md).

## 3. Enable packages and automation folders

In `configuration.yaml`:

```yaml
homeassistant:
  packages: !include_dir_named packages/

# Keep the UI-created automations AND load the ones in automations/
automation ui: !include automations.yaml
automation manual: !include_dir_merge_list automations/
```

If your `configuration.yaml` already has `automation: !include automations.yaml`, replace that line with the two above.

## 4. Copy the files

```
config/packages/cpap_monitoring.yaml       →  <your HA config>/packages/
config/automations/cpap/*.yaml             →  <your HA config>/automations/cpap/
```

Copy only what you need:

| You want | Files |
| --- | --- |
| Alerts for Maria (required) | `user1_alerts.yaml` |
| Spoken morning summary for Maria | `user1_morning_summary.yaml` |
| "No use recorded" warning | `user1_no_data_alert.yaml` (also posts to Telegram) |
| Daily Telegram for the family | `telegram_family_summary.yaml` |
| Monthly Telegram report | `cpap_month_accumulate.yaml` **and** `cpap_monthly_report.yaml` |
| A second CPAP user | `user2_alerts.yaml`, `user2_morning_summary.yaml`, `user2_no_data_alert.yaml` |

## 5. Replace the placeholders

Every value you must change is in CAPITALS. Search the copied files for `REPLACE`, `YOUR_` and `USER1_` / `USER2_`.

| Placeholder | Replace with | Files |
| --- | --- | --- |
| `notify.mobile_app_YOUR_PHONE` | The caregiver's phone, from the Companion app (`notify.mobile_app_<phone name>`) | `user1_alerts`, `user1_no_data_alert`, all `user2_*` |
| `notify.USER1_ECHO_1_announce`, `notify.USER1_ECHO_2_announce` | Maria's Echo devices (Alexa Media Player creates `notify.<device>_announce`). Using only one? Delete the second line. | `user1_morning_summary`, `user1_no_data_alert` |
| `notify.USER2_ECHO_announce` | João's Echo | all `user2_*` |
| `input_boolean.YOUR_DO_NOT_DISTURB` | Your own "Do Not Disturb" toggle (a helper, not the Echo's DND switch). João's Echo only speaks while it is **off**. Push and persistent notifications go out either way. **If this entity doesn't exist, the condition never passes and the Echo never speaks.** No such toggle? Either create an `input_boolean` and leave it off, or delete the `if` block and keep only its `notify.send_message` step. | all `user2_*` |
| `REPLACE_WITH_FAMILY_CHAT_ID` | The Telegram group ID, a negative number (section 8) | `telegram_family_summary`, `cpap_monthly_report`, `user1_no_data_alert` |
| `ai_task.YOUR_AI_TASK_ENTITY` | Your AI Task entity (section 9). Not using AI? Delete the whole `ai_task.generate_data` step. The message then always uses the fixed text. | `user1_morning_summary`, `telegram_family_summary`, `cpap_monthly_report` |
| `Maria`, `João` | The real first names. They appear in titles, spoken text and AI prompts. | all |

The dashboard needs no placeholders. Rename the "Maria" and "João" views if you like.

## 6. Restart and turn things on

1. **Developer Tools → YAML → Check configuration**, then **Settings → System → Restart**.
2. Turn on the toggles. They start **off** so nothing fires before you are ready:
   - `input_boolean.cpap_alerts_user1`: Maria's alerts, morning summary and "no use" warning
   - `input_boolean.cpap_alerts_user2`: the same for João
   - `input_boolean.cpap_telegram_family_notify`: Telegram daily summary and the Telegram part of the "no use" warning

The monthly report and its accumulator have no toggle. Leave those two files out if you don't want them.

## 7. Add the dashboard

**Settings → Dashboards → Add Dashboard → New dashboard from scratch**. Open it, then **⋮ → Edit dashboard → ⋮ → Raw configuration editor**, paste the whole of `config/dashboards/cpap_monitor.yaml` and save.

The "Testes" buttons on each user's view fire an automation once, ignoring time windows and toggles. Use them to check each channel.

## 8. Telegram family notifications (optional)

Sends the daily summary, the monthly report and the "no use" warning to a Telegram group of relatives who don't use Home Assistant.

1. **Create a bot**: in Telegram, talk to [@BotFather](https://t.me/BotFather), send `/newbot` and keep the API token.
2. **Create the group** with the relatives and add the bot to it.
3. **Get the chat ID**: send any message in the group, then open `https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates` in a browser and copy the `"chat":{"id": ...}` value. Group IDs are negative.
4. **Add the integration via the UI**: **Settings → Devices & Services → Add Integration → Telegram bot**, choose **Polling**, paste the token. Recent Home Assistant versions reject `telegram_bot:` in YAML.
5. **Register the group as an allowed chat ID** in that integration entry. Without this, every `telegram_bot.send_message` fails with an HTTP 500 and no clear message.
6. Put the chat ID in place of `REPLACE_WITH_FAMILY_CHAT_ID` in the three files (section 5).
7. Turn on `input_boolean.cpap_telegram_family_notify` and press "Teste Telegram" on the dashboard.

The daily summary tries at 08:35 and again at 11:00, in case myAir synced late. It sends once per night (`input_text.cpap_telegram_family_last_sent` remembers which night went out).

## 9. AI-personalized morning summary (and Telegram messages) (optional)

Replaces the fixed wording of `user1_morning_summary.yaml` (Alexa), `telegram_family_summary.yaml` and `cpap_monthly_report.yaml` with a message Gemini writes fresh each time. The real numbers (usage, AHI, leak, score, alert flags) are passed as facts the model may not change. If the call fails at runtime (quota, network, API error), each automation falls back to its fixed text, so delivery is never skipped.

1. **Get a Gemini API key** at [aistudio.google.com/apikey](https://aistudio.google.com/apikey). The free tier is enough for three short messages a day.
2. **Settings → Devices & Services → Add Integration → Google Generative AI**, paste the key.
3. In that integration entry, **add an AI Task** subentry. This creates an entity such as `ai_task.google_ai_task`.
4. Put that entity ID in place of `ai_task.YOUR_AI_TASK_ENTITY` in the three files.
5. Test with the "Resumo Matinal" and "Teste Telegram" dashboard buttons. If a message comes out as the fixed template, check **Settings → System → Logs** for the `ai_task.generate_data` error (wrong entity ID, quota, or the integration isn't fully set up).

### Keep each prompt aimed at its audience

Each prompt says who will hear or read the message, and that matters more than it seems:

- **Alexa** (`user1_morning_summary.yaml`) speaks **to** Maria: "Bom dia, Maria!", gentle advice, "talk to one of your children if in doubt".
- **Telegram** (`telegram_family_summary.yaml`, `cpap_monthly_report.yaml`) is read by her **children**. She is mentioned in the third person, and any advice is for them (call her, help adjust the mask).

If you remove the "who reads this" paragraph, the model tends to give advice straight to the patient in a group she isn't in. To change the language, rewrite the prompt in your language. The fixed fallback texts are in Portuguese too.

## 10. Adjust the thresholds

**Settings → Devices & Services → Helpers**: `CPAP AHI Alert Threshold` (5), `CPAP Mask Leak Alert Threshold` (24 L/min), `CPAP Minimum Usage Hours` (4 h), `CPAP myAir Score Threshold` (70).

These helpers have an `initial:` value in the package, so **a change made in the UI lasts only until the next restart**. To change one permanently, edit its `initial:` in `cpap_monitoring.yaml`. Talk to the patient's doctor before changing them.

The monthly accumulator helpers (`input_number.cpap_month_*`) deliberately have **no** `initial:`. With one, every restart would reset the month's totals.

## 11. Verify

- **Settings → Entities**, search "CPAP": you should see `binary_sensor.cpap_user1_ahi_alert` and the other derived sensors.
- **Developer Tools → Template**: `{{ states('sensor.most_recent_sleep_date') }}` should show last night's date once myAir has synced.
- The first real alert arrives when the next night syncs, between 07:00 and 22:00.
