# FUNHUB Jukebox — Deploy & Setup Guide

This is the per-location setup guide for the jukebox kiosk. Follow it in order for a
new location (DIX30, Cathcart, Trois-Rivières, Sherbrooke). It also doubles as the
troubleshooting reference for a kiosk that stops working.

App URL: **https://rico-funhub.github.io/jukebox**
Current version shows in the bottom corner of the connect screen (e.g. `v1.2.5`).

---

## 1. What you need before you start

- A **Spotify Premium account** for the location. Premium is mandatory — a free
  account cannot use the Spotify API at all (Spotify blocks it entirely as of 2026).
  This is not optional and there is no workaround.
- Access to the **Spotify Developer Dashboard** (developer.spotify.com/dashboard),
  logged in with that Premium account.
- The kiosk device (touchscreen) with a browser.
- The 4-digit operator **PIN** (default `1205` — change it per location in settings).

---

## 2. Create the Spotify app (Key A)

Every kiosk runs on its own Spotify "app", identified by a **Client ID**. You create
that app once per location.

1. Go to **developer.spotify.com/dashboard** and log in with the location's Premium
   account.
2. Click **Create app**.
3. Name it something clear, e.g. `FUNHUB Jukebox — DIX30`.
4. **Redirect URI** — enter this EXACTLY, character for character:

   ```
   https://rico-funhub.github.io/jukebox
   ```

   - No trailing slash.
   - `https`, not `http`.
   - If this does not match exactly, login fails with
     `redirect_uri: Not matching configuration`. This is the #1 setup error.
5. Under **APIs**, tick **Web API** and **Web Playback SDK**.
6. Save.
7. Open the app's **Settings** and copy the **Client ID** (a 32-character
   letters-and-numbers string).

**About the Client ID and Client Secret:**
- The **Client ID is not a secret.** It is a public identifier. It is safe to type
  into the kiosk, and it is visible in normal browser traffic. This is by design.
- The **Client Secret does NOT matter for this app.** The jukebox uses PKCE auth and
  never uses the secret. You can ignore it, rotate it, or regenerate it — the jukebox
  is unaffected.
- The only sensitive thing — the account email and password — is entered ONLY on
  Spotify's own login page, never in the jukebox.

---

## 3. (Optional but recommended) Create a second app (Key B)

Key B is a **second Premium Spotify app** used as a failover. Two apps = two separate
rate-limit buckets. If Key A gets rate-limited (see §7), staff can switch to Key B and
keep running.

- Key B must ALSO be a Premium account app. A non-Premium Key B will not work — it is
  blocked by Spotify exactly like a free account.
- It can be a second app on the same Premium account, or an app on a different Premium
  account. Either way it counts as a separate rate-limit bucket.
- Give it the **same Redirect URI** (`https://rico-funhub.github.io/jukebox`) and the
  same **Web API + Web Playback SDK** tick boxes.
- Copy its Client ID for the setup step below.

If you don't have a second Premium account yet, skip this — the jukebox works fine on
Key A alone. You can add Key B later in settings at any time.

---

## 4. First-time kiosk setup

1. On the kiosk, open **https://rico-funhub.github.io/jukebox**
   - If it's an existing kiosk that's misbehaving, open
     **https://rico-funhub.github.io/jukebox/?reset=1** instead (clears stuck state;
     keeps saved keys/settings).
2. You'll land on the **Set up this jukebox** screen.
3. Paste **Key A** Client ID into the Key A field (required).
4. If you made a Key B, paste it into the Key B field (optional).
5. Tap **Save & connect**.
6. The Spotify login page opens. Log in with the location's **Premium** account and
   approve.
7. You should return to the jukebox on the attract/standby screen.

If login shows `redirect_uri: Not matching configuration`, the Redirect URI in the
Spotify dashboard doesn't match — go back to §2 step 4 and fix it exactly.

---

## 5. Operator settings (behind PIN)

Reach settings by tapping the hidden operator gesture (5 taps) and entering the PIN.
Key things to configure per location:

- **PIN** — change from the default `1205`.
- **Credits per tap / per song** — how the card reader maps to song credits.
- **Free mode** — no card needed; guests pick for free (for events).
- **Playback mode** — In-app player (default; audio plays from the kiosk browser) or
  External device (send to a chosen Spotify Connect device on the location's computer).
- **App keys (A / B)** — view/change keys, switch the active key.
- **Coin input** — keyboard key or gamepad button for the coin mechanism.
- **On-screen keyboard** — on for touchscreens without a physical keyboard.

---

## 6. The "clear queue" reality

Spotify's Web API has **no clear-queue command** — it does not exist for any third-party
app. So:

- The **Reset playback (September)** button in settings stops the current song and
  starts one fresh track. With Autoplay disabled on the account (recommended), this
  leaves the queue effectively empty on most devices.
- For a guaranteed full clear, staff use the **real Spotify app** on the location's
  computer (same account) and hit Clear there.
- You cannot embed Spotify's web player in the kiosk to get its Clear button — Spotify
  blocks embedding. This is a Spotify limitation, not a jukebox bug.

**Recommendation:** disable Autoplay on the location's Spotify account so the queue
doesn't auto-refill.

---

## 7. Rate limits & "Out of Service" (important)

The jukebox runs in Spotify **Development Mode**, which has a strict, undocumented rate
limit measured per app (Client ID) over a rolling 30-second window. Heavy use — or heavy
testing — can trip it, and cooldowns can last minutes to several hours. This is a
Spotify constraint; no code or setting shortens it.

**What the jukebox does when rate-limited:**
- It shows an **Out of Service** screen and stays there. It does NOT auto-recover
  (deliberately — see below).
- It never tells a guest it's working unless it actually is.

**How staff bring it back:**
- On the Out of Service screen, tap **Staff unlock** and enter the PIN.
- The jukebox runs a **real search test**. If Spotify has lifted the cooldown, it
  reopens. If it's still limited, it stays offline and shows the error code — try again
  later.
- If **Key B** is set, a **Switch to Key B** button appears on that screen. Switching to
  the other key (separate rate-limit bucket) is the way to keep running during a Key A
  cooldown. Switching keys requires a quick Spotify reconnect.

**Avoiding it:** one kiosk on its own Premium account, under normal guest traffic, rarely
hits the limit. It's most likely during setup/testing (many rapid searches and reloads).
When testing, don't hammer search repeatedly on a cooled-down key — that can extend the
cooldown. Leave it alone until it clears.

**Extended Quota Mode** (higher limits) is **not available** to a jukebox — it requires a
registered business app with 250,000+ monthly active users. Don't waste time applying;
it will be rejected. Two Premium keys is the real mitigation.

---

## 8. Taking the jukebox offline on purpose

To close the jukebox (end of night, maintenance, keeping guests off it):

- Settings (PIN) → **Take offline**. Guests see a "Temporarily Unavailable" screen.
- It stays closed, survives reload, and no guest can tap out of it.
- Reopen: Staff unlock + PIN on that screen (or `?reset=1`).

---

## 9. Acceptance test (do this before calling a kiosk "live")

Run this on the real kiosk with the real account and card reader. This is the only way to
know it truly works — passing code checks is not the same as working hardware.

1. Open the app (or `?reset=1`), enter Key A, connect with the Premium account.
2. Tap a card → confirm credits appear.
3. Search a song → pick it → **confirm audio actually plays** over the speakers.
4. Let the song finish → confirm the next queued song plays.
5. Turn on Free mode → confirm no-card play → turn it off.
6. (If used) Switch to External device → pick the device → confirm audio moves there.
7. Settings → Take offline → confirm guests see the closed screen → reopen with PIN.
8. (If Key B set) Force a key switch → confirm reconnect works.

If all pass, the kiosk is genuinely deployable.

---

## 10. Quick troubleshooting

| Symptom | Cause / Fix |
|---|---|
| `redirect_uri: Not matching configuration` | Redirect URI in Spotify dashboard doesn't exactly match `https://rico-funhub.github.io/jukebox`. Fix it (no trailing slash, https). |
| Search fails with `E-API-403` | The account is not Premium, or the key's app has no Premium owner. Premium is required. |
| "Connect does nothing" / blank | Cached old version. Hard-load `?reset=1`. |
| Songs won't queue | Check playback mode isn't stuck on External with no device. Re-connect on Key A. |
| Stuck Out of Service for hours | Key A is in a Dev-Mode cooldown. Switch to Key B, or wait it out. |
| Need to change/fix a key | Connect screen → "Having issues connecting?" → Re-enter app keys. Or settings → App keys. |

---

## 11. Facts worth remembering

- Premium is mandatory (both keys).
- Redirect URI must match exactly.
- Client ID is public; Client Secret is irrelevant to this app.
- No clear-queue API exists; disable Autoplay and use the real Spotify app for a hard clear.
- Dev-Mode rate limits are the ceiling; two Premium keys + low call volume is the honest best.
- `?reset=1` is the universal "get me back to a clean state" escape (keeps keys/settings).
