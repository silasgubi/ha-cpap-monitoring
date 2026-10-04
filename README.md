# ha-cpap-monitoring

Home Assistant package to keep an eye on a family member's CPAP therapy (ResMed machines that sync to myAir; built and tested with an AirSense 10): nightly alerts for the caregiver, a spoken good-morning summary for the person using the machine, and daily and monthly Telegram updates for the rest of the family.

**[Português abaixo](#português)**

> [!IMPORTANT]
> **Not a medical device.** This project only reads the same numbers the myAir app already shows and repeats them through Home Assistant. It does not diagnose anything and must not replace the guidance of the doctor or sleep technician who follows the patient. Thresholds are configurable and the defaults come from ResMed's public myAir report guide.

## Who it's for

Built for a common situation: an older parent uses a CPAP at home, and their adult children want to know whether the therapy is going well without calling every morning to ask. It also handles a second CPAP user in the same Home Assistant (for example, the caregiver who uses one too).

Throughout the files, the example people are:

| In the files | Role | Entity IDs use |
| --- | --- | --- |
| **Maria** | The person being looked after (first myAir device) | `user1` / no suffix (`sensor.cpap_*`) |
| **João** | Her son, the caregiver, who also uses a CPAP (second myAir device) | `user2` / `_2` suffix (`sensor.cpap_*_2`) |

Replace both names with your own (a search and replace is enough). If you only have one CPAP, skip every `user2_*` file.

## What it does

| When | What | Who receives it |
| --- | --- | --- |
| New night synced (07:00–22:00) | Alert if AHI, mask leak or usage is out of range | Maria: caregiver's phone (push) · João: his phone + his Alexa |
| 08:30 | Spoken "good morning" with last night's results | Maria, on her Echo devices |
| 08:35, retry 11:00 | Daily summary in the family Telegram group | Maria's children |
| 09:00 | Morning summary | João (Alexa + push) |
| 11:15 | "No use recorded" warning if myAir has nothing newer than yesterday | Maria (Alexa), caregiver (push), children (Telegram) |
| 11:30 | Same warning for the second user | João (Alexa + push) |
| Day 1, 08:40 | Monthly report in the style of ResMed's PDF (average usage, AHI, leak, % nights ≥4h/≥6h) | Maria's children (Telegram) |

**Each channel has its own audience.** Alexa speaks **to** Maria ("Bom dia, Maria!"). Telegram talks **about** her to her children, in the third person, and any advice is for them (call her, help adjust the mask straps). Push notifications are short and technical, for the caregiver.

**Optional AI wording.** The Alexa summary and both Telegram messages can be written fresh each day by Gemini (Home Assistant's `ai_task.generate_data`), using the real numbers as facts it can't change. If the AI call fails, each automation falls back to a fixed template, so a message is always delivered. Without AI, you delete that one step and the fixed template is always used.

**Language.** Everything Maria hears and the family reads is in Brazilian Portuguese, and so are a few dashboard labels. The AI prompts are in Portuguese too: rewrite them in your language and the generated messages follow.

## Requirements

| Requirement | Needed for |
| --- | --- |
| A recent Home Assistant release (the AI wording needs a version with the AI Task integration) | Everything |
| [ResMed myAir integration](https://github.com/prestomation/resmed_myair_sensors) (HACS) | Data source |
| [Alexa Media Player](https://github.com/alandtse/alexa_media_player) (HACS) | Spoken messages (optional) |
| [Mushroom Cards](https://github.com/piitaya/lovelace-mushroom) (HACS) | Dashboard |
| Home Assistant Companion app on a phone | Push alerts |
| Telegram bot (built-in `telegram_bot` integration) | Family messages (optional) |
| Google Generative AI integration with an AI Task entity | AI wording (optional) |

## Install

Full step-by-step guide: **[docs/installation-guide.md](docs/installation-guide.md)**. In short:

1. Copy `config/packages/cpap_monitoring.yaml` to your `packages/` folder.
2. Copy the automations you want from `config/automations/cpap/`.
3. Replace every placeholder in CAPITALS (`REPLACE_...`, `YOUR_...`, `USER1_ECHO_...`). The guide has the full list.
4. Restart, turn on the toggles and add `config/dashboards/cpap_monitor.yaml` as a dashboard.

## Repository layout

```
config/                                  # mirrors a Home Assistant config folder
├── packages/cpap_monitoring.yaml        # template sensors, alert binary sensors, helpers
├── automations/cpap/
│   ├── user1_alerts.yaml                # Maria: AHI / leak / low-usage alerts (push)
│   ├── user1_morning_summary.yaml       # Maria: 08:30 spoken summary (Alexa, AI optional)
│   ├── user1_no_data_alert.yaml         # Maria: 11:15 "no use recorded"
│   ├── telegram_family_summary.yaml     # Family: 08:35 daily Telegram (AI optional)
│   ├── cpap_month_accumulate.yaml       # Adds each new night to the monthly totals
│   ├── cpap_monthly_report.yaml         # Family: day-1 monthly Telegram report (AI optional)
│   ├── user2_alerts.yaml                # João: alerts (push + Alexa)
│   ├── user2_morning_summary.yaml       # João: 09:00 summary
│   └── user2_no_data_alert.yaml         # João: 11:30 "no new data"
└── dashboards/cpap_monitor.yaml         # Lovelace dashboard (overview + one view per user)

docs/
├── installation-guide.md                # Step by step, including placeholders
├── architecture.md                      # Data flow and design decisions
├── second-alexa-account.md              # Speaking on Echo devices in another home
└── troubleshooting.md                   # Common problems
```

## Default thresholds

| Metric | Alert when | Source |
| --- | --- | --- |
| AHI | > 5 events/h | ResMed myAir report guide |
| Mask leak | > 24 L/min | ResMed 95th-percentile reference |
| Usage | < 4 h/night | ResMed minimum recommendation |
| myAir score | < 70 (dashboard only) | myAir scoring guide |

Ask the patient's doctor before changing them.

## Ideas for later

- Weekly e-mail report
- Side-by-side comparison view of both users
- Richer per-breath data from the SD card via [HMS-CPAP](https://github.com/hms-homelab/hms-cpap) (needs a Wi-Fi SD card)

## Credits

- [prestomation/resmed_myair_sensors](https://github.com/prestomation/resmed_myair_sensors): the integration that makes this possible
- [HMS-CPAP](https://github.com/hms-homelab/hms-cpap): dashboard inspiration
- [Alexa Media Player](https://github.com/alandtse/alexa_media_player): spoken notifications

## License

[MIT](LICENSE)

---

## Português

Pacote para Home Assistant que acompanha o uso do CPAP de um familiar (aparelhos ResMed que sincronizam com o myAir, testado com AirSense 10). Ele manda alertas para quem cuida, um "bom dia" falado na Alexa para quem usa o aparelho e um resumo diário e mensal no Telegram para os filhos.

> [!IMPORTANT]
> **Não é um dispositivo médico.** O projeto só repete pelo Home Assistant os números que o app myAir já mostra. Não diagnostica nada e não substitui a orientação do médico ou do técnico do sono.

**Para quem é:** pensado para quem tem pai ou mãe usando CPAP e quer saber se a terapia está indo bem sem precisar ligar toda manhã para perguntar. Nos arquivos, **Maria** é a mãe (primeiro aparelho, `user1`) e **João** é o filho que cuida e também usa CPAP (segundo aparelho, `user2`). Troque pelos nomes da sua família.

**Cada canal fala com uma pessoa diferente:**
- A **Alexa** fala **com** a Maria.
- O **Telegram** fala **sobre** ela com os filhos, em terceira pessoa, e as recomendações são para eles.
- O **push** é para quem cuida.

As mensagens faladas e as do Telegram podem ser escritas pelo Gemini a cada dia, sempre com os números reais. Se a IA falhar, entra um texto fixo e a mensagem sai do mesmo jeito.

**Instalação:** siga o [guia de instalação](docs/installation-guide.md), que está em inglês. Os pontos principais são copiar o package e as automações e trocar todos os placeholders em MAIÚSCULAS (`REPLACE_...`, `YOUR_...`, `USER1_ECHO_...`). As mensagens e os prompts já estão em português do Brasil.

Dúvidas e sugestões são bem-vindas nas issues, em português ou inglês.
