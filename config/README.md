# config/

Mirrors a Home Assistant configuration folder: copy each file to the same relative path in yours. Setup steps and the placeholder list are in [../docs/installation-guide.md](../docs/installation-guide.md).

## Entities from the ResMed myAir integration

The integration names entities by the order the devices were added. The first device has no suffix, the second gets `_2`.

| | First device (`user1`) | Second device (`user2`) |
| --- | --- | --- |
| AHI | `sensor.cpap_ahi_events_per_hour` | `sensor.cpap_ahi_events_per_hour_2` |
| Mask leak | `sensor.cpap_mask_leak` | `sensor.cpap_mask_leak_2` |
| Usage minutes | `sensor.cpap_usage_minutes` | `sensor.cpap_usage_minutes_2` |
| Usage time | `sensor.cpap_usage_time` | `sensor.cpap_usage_time_2` |
| myAir score | `sensor.cpap_total_myair_score` | `sensor.cpap_total_myair_score_2` |
| Mask on/off count | `sensor.cpap_mask_on_off` | `sensor.cpap_mask_on_off_2` |
| Sleep data last collected | `sensor.cpap_sleep_data_last_collected` | `sensor.cpap_sleep_data_last_collected_2` |
| Most recent sleep date | `sensor.most_recent_sleep_date` | `sensor.most_recent_sleep_date_2` |

## Entities created by the package

| Entity | Purpose |
| --- | --- |
| `sensor.cpap_user1_usage_hours`, `sensor.cpap_user2_usage_hours` | Usage in hours |
| `binary_sensor.cpap_userN_ahi_alert` / `_leak_alert` / `_usage_alert` / `_score_low` | Value above (or below) its threshold |
| `input_number.cpap_ahi_threshold`, `cpap_leak_threshold`, `cpap_usage_min_hours`, `cpap_score_threshold` | Thresholds |
| `input_boolean.cpap_alerts_user1`, `cpap_alerts_user2` | Per-user on/off for alerts, summaries and "no data" warnings |
| `input_boolean.cpap_telegram_family_notify` | On/off for the family Telegram messages |
| `input_number.cpap_month_*`, `input_text.cpap_month_*` | Monthly report accumulator |
| `input_text.cpap_telegram_family_last_sent`, `cpap_userN_no_data_alerted_date` | Guards against sending twice |

## Automations

| File | Automation entity | When |
| --- | --- | --- |
| `automations/cpap/user1_alerts.yaml` | `automation.cpap_user1_ahi_alert`, `_mask_leak_alert`, `_low_usage_alert` | New night synced |
| `automations/cpap/user1_morning_summary.yaml` | `automation.cpap_morning_summary` | 08:30 |
| `automations/cpap/telegram_family_summary.yaml` | `automation.cpap_telegram_family_summary` | 08:35, retry 11:00 |
| `automations/cpap/user1_no_data_alert.yaml` | `automation.cpap_user1_no_data_alert` | 11:15 |
| `automations/cpap/cpap_month_accumulate.yaml` | `automation.cpap_monthly_accumulate_user1` | New night synced |
| `automations/cpap/cpap_monthly_report.yaml` | `automation.cpap_monthly_report_user1` | Day 1, 08:40 |
| `automations/cpap/user2_alerts.yaml` | `automation.cpap_user2_ahi_alert`, `_mask_leak_alert`, `_low_usage_alert` | New night synced |
| `automations/cpap/user2_morning_summary.yaml` | `automation.cpap_user2_morning_summary` | 09:00 |
| `automations/cpap/user2_no_data_alert.yaml` | `automation.cpap_user2_no_data_alert` | 11:30 |

Automation entity IDs come from each `alias`, so keep the aliases if you use the dashboard's test buttons.
