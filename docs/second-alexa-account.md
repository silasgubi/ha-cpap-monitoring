# Speaking on Echo devices in another home

Usually the person being looked after lives in their own home, with Echo devices registered to **their** Amazon account. Alexa Media Player can hold more than one Amazon account, so the same Home Assistant can speak in both homes.

## Before you start

- Alexa Media Player is installed and working for your own account.
- The other person's Echo is set up in the Alexa app on their account and online.
- You can get their Amazon login and a 2-step verification code with their permission, or sit with them while you do this.

## Steps

1. **Settings → Devices & Services → Alexa Media Player → Add entry** (a second "hub").
2. Enter their Amazon e-mail and password. When asked for a code, use an **authenticator-app** (TOTP) code. Alexa Media Player does not handle SMS codes well. If their account only has SMS 2-step verification, add an authenticator app to it first in the Amazon account settings.
3. Amazon may e-mail them or send a push asking to approve the new sign-in. They need to approve it.
4. Their Echo devices now appear under the second entry. Note the **`notify.<device>_announce`** entities.
5. Put those entities in place of `notify.USER1_ECHO_1_announce` / `notify.USER1_ECHO_2_announce` in `user1_morning_summary.yaml` and `user1_no_data_alert.yaml`.

## Test

**Developer Tools → Actions**:

```yaml
action: notify.send_message
target:
  entity_id: notify.YOUR_DEVICE_announce
data:
  message: "Teste do Home Assistant."
```

An announcement plays a short chime before the message, and it also shows on Echo Show screens.

## Problems

- **Login fails**: check the e-mail and password, and use a fresh code (they expire every 30 seconds).
- **No devices after login**: the Echo may be registered to a different Amazon account than the one you used. Check which account the Alexa app on their phone is signed into.
- **Wrong language or voice**: set on the device itself (Alexa app → Devices → [device] → Language), not in Home Assistant.
- **Nothing is heard**: Do Not Disturb on their Echo blocks announcements. Check it in the Alexa app.
