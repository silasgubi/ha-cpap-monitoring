# Troubleshooting

## Sensors are "unavailable", or no alert ever fires

The per-night automations only run when `sensor.most_recent_sleep_date` changes.

1. Does the myAir app on the phone show last night? If not, the machine hasn't uploaded: check its Wi-Fi/cellular connection.
2. If the app has it, wait for the integration's next poll, or reload the ResMed myAir integration.
3. myAir logins expire now and then. Look for a re-authentication notice under **Settings → Devices & Services**.

## The morning summary or Telegram message didn't go out

Both skip when the data is older than yesterday, so nothing gets announced from a stale night.

- `sensor.most_recent_sleep_date` should be yesterday's date by 08:30. If myAir synced late, the Telegram summary retries at 11:00. The Alexa summary does not retry.
- Check that the toggle is on (`input_boolean.cpap_alerts_user1` or `input_boolean.cpap_telegram_family_notify`).
- Open the automation's **Traces** to see which condition stopped it.

## Telegram fails with an HTTP 500

The group is not registered as an **allowed chat ID** in the Telegram bot integration. Open **Settings → Devices & Services → Telegram bot**, add the group's chat ID (a negative number) and test again. The token alone is not enough.

## The message arrives, but it's the fixed template instead of the AI text

The `ai_task.generate_data` step failed and the fallback took over. **Settings → System → Logs** shows why: wrong AI Task entity ID, Gemini quota, or the integration not fully set up.

## The AI gives advice to the patient in the family group

The Telegram prompts must say who reads the group (her children) and that she is mentioned in the third person. If you rewrote the prompt and dropped that paragraph, put it back. See "Keep each prompt aimed at its audience" in the [installation guide](installation-guide.md).

## Alexa runs the automation but says nothing

1. Make sure the `notify.<device>_announce` entity exists and belongs to the right Echo (**Developer Tools → Actions**, `notify.send_message`, pick the entity, send a test).
2. Do Not Disturb on: the `user2_*` automations switch it off for a few seconds. For Maria's Echo, check DND in the Alexa app.
3. For an Echo in another Amazon account, see [second-alexa-account.md](second-alexa-account.md).

## The monthly report shows far fewer nights than it should

Check that the `input_number.cpap_month_*` helpers in `cpap_monitoring.yaml` have **no** `initial:` line. With one, every Home Assistant restart resets the month's totals. Also check that `cpap_month_accumulate.yaml` is installed and on.

## A threshold went back to its default after a restart

That's expected. The threshold helpers have `initial:` so a fresh install starts with sensible values. To change one permanently, edit its `initial:` in the package.

## Dashboard cards are blank or show an error

Mushroom Cards isn't installed, or an entity ID in the dashboard doesn't match yours. Test one in **Developer Tools → Template**: `{{ states('sensor.cpap_ahi_events_per_hour') }}`.
