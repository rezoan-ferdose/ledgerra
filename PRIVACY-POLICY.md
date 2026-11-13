# Privacy Policy — Ledgerra

**Effective date: 13 November 2026**
**Applies to:** Ledgerra for Android (`com.ledgerra.app`), including the Google Play build.

Ledgerra is a personal finance tracker. It has no account system, no backend of its own, and
no analytics. This policy explains exactly what the app touches and where that data goes.

---

## 1. The short version

* **Your financial data stays on your device.** Ledgerra has no servers, no accounts and no
  telemetry.
* **The Google Play build does not request SMS access at all.** Google Play only grants
  `READ_SMS` / `RECEIVE_SMS` to default SMS-handler apps, so the Play build removes both
  permissions and captures transactions from bank notifications, PDF statements, the share
  sheet and manual entry instead.
* **Nothing is collected by the developer.** The app's author never receives your
  transactions, your categories, your accounts or your statistics.
* The only data that can ever leave the device is data **you explicitly ask to be sent** to
  a destination **you choose** (section 6).

---

## 2. What "no data collection" means here

Ledgerra reads financial information from your device in order to show it back to you. That
information is processed **entirely on your device** and is never transmitted to the
developer or to Google. Under Google's Data Safety definitions, information that is
processed only on the device is not "collected".

The app contains no analytics SDK, no crash-reporting SDK, no advertising SDK, no
attribution SDK and no fingerprinting.

---

## 3. Data stored on your device

| Data | Where it lives | Why |
|---|---|---|
| Transactions (amount, merchant, date, category, account) | App-private SQLite database (`Room`) | The timeline, analytics and budgets you see |
| Categories, subcategories and merchant rules | App-private database | Your organisation of that data |
| Accounts, balances and budget limits | App-private database | Balance and budget tracking |
| App preferences (currency, theme, reminders, base currency) | `DataStore` preferences file | Your settings |
| AI model file (optional, ~1.6 GB, Qwen 2.5, Apache 2.0) | App-private storage | On-device AI chat, only if you ask for it |
| App-lock state (on/off, timeout, last successful authentication time) | `DataStore` preferences file | Knowing when to lock the app |
| Optional cloud-backup credentials (Drive refresh token, WebDAV password) | `EncryptedSharedPreferences`, AES-256 with a Keystore-held master key | So you are not asked for your password on every backup |

This data is inside the Android app sandbox. It is not readable by other apps. The app sets
`android:allowBackup="false"`, so Android's automatic cloud backup does not copy it.

The app lock never handles a biometric secret itself: the check is performed by Android's
own biometric prompt, and the app only records that it succeeded and when.

---

## 4. Permissions and special access, and exactly why each is used

The Google Play build of Ledgerra requests the following. Nothing else.

| Permission / access | Why Ledgerra needs it | Is it required? |
|---|---|---|
| **Notification access** (`NotificationListenerService`, granted in Settings → Notifications → Device & app notifications) | Reads the *confirmation text* of bank, card and UPI notifications so a payment can be turned into a transaction without you typing anything. | Required for automatic capture. The app still works without it — you can import PDF or CSV statements, or enter transactions manually. |
| **Notifications** (`POST_NOTIFICATIONS`) | Shows your transaction alerts and reminders, and confirms background work. | Optional; the app works without it. |
| **Exact alarms** (`SCHEDULE_EXACT_ALARM`) | **Not requested in the Play build.** Daily reminders and webhook sync use standard inexact alarms (`setAndAllowWhileIdle`) and WorkManager. | n/a |
| **Boot completed** (`RECEIVE_BOOT_COMPLETED`) | Re-schedules those reminders after the device restarts. | Optional. |
| **Biometrics** (`USE_BIOMETRIC`) | Unlocks the app behind fingerprint/face unlock when you enable app lock. | Optional; only if you turn on app lock. |
| **Internet** (`INTERNET`) | Only for the optional features listed in section 6: downloading the AI model, currency exchange rates, and the backup/webhook destinations you configure. | Optional; all core features work fully offline. |
| **Files and photos** | None. Ledgerra reads files only through Android's document picker, which hands over a single file you chose. No storage permission is declared. | n/a |
| **SMS** | **Not requested in the Play build.** The F-Droid and GitHub-release builds of Ledgerra do offer optional read-only SMS access for automatic capture, because Google Play restricts that permission to default SMS-handler apps. | n/a |
| **Advertising ID** (`AD_ID`) | Explicitly **removed** from the manifest (`tools:node="remove"`). | Never requested. |

### Notification access, in detail

This is the most sensitive capability the app has, so it is worth being precise.

* Ledgerra sees only notifications posted by other apps, and only the text of those
  notifications.
* It looks for text that matches a known bank or UPI payment format (an amount, a
  merchant, a transaction keyword).
* Matching text is reduced to a transaction row: amount, merchant, date, category.
* **The full notification body is not stored.** The notification title and body are
  discarded after parsing; only the extracted transaction is written to the database.
* You can revoke notification access at any time in Android Settings. The app stops
  capturing immediately.

---

## 5. The on-device AI assistant (optional)

* The assistant uses the Qwen 2.5 model running **locally** through LiteRT-LM.
* The model file is downloaded once from Hugging Face on your explicit request. The
  download is a file transfer; it does not carry any of your data.
* Inference runs on your device. Your transactions, categories and chat are never sent to
  a server for processing.
* Chat messages are held in the local database so you can scroll back through them. They
  are deleted with the rest of your data.

---

## 6. The only ways data can leave your device

Ledgerra has no backend, so it cannot send your data anywhere by itself. Data leaves only
when you turn on a feature **and** supply the destination. Every one of these is off by
default.

### 6.1 Cloud backup (optional)

You can back up your encrypted database to a destination you control:

* **Google Drive** — via the Drive `appDataFolder` scope. The backup is stored in your
  Drive in a hidden application-data folder that other apps and other users cannot see or
  list. Ledgerra uses `drive.appdata` only; it cannot read or write your personal Drive
  files.
* **WebDAV** — a server you specify.

The backup archive is encrypted **before** it is written — end-to-end, with AES-256-GCM and
a key derived from the passphrase you set in the cloud-backup settings. The passphrase
never leaves the device, and it is needed to restore a backup; there is no way to recover
a backup without it. Google Drive sign-in uses Google Sign-In, so Ledgerra never sees
your Google password.

### 6.2 Webhooks (optional)

You can configure Ledgerra to POST transaction summaries to an HTTP endpoint you specify.
That endpoint receives the data you chose to send it. Ledgerra cannot verify the security
of an endpoint you configure, and that endpoint is outside the app's and the developer's
control. Webhooks are disabled until you configure one.

### 6.3 Currency exchange rates (optional)

If you enable multi-currency mode, the app requests exchange rates from a public rates
API. The request carries no transaction data — only the currency pair being asked about.

### 6.4 Google Play services (Play build only)

The Play build includes Google's In-App Update and In-App Review libraries, which are used
for app-update prompts and optional review prompts. These are Google components operating
under Google's own policies; Ledgerra does not use them to send your financial data
anywhere.

---

## 7. Data retention and deletion

* **Uninstalling the app removes all of its data**, including the database, preferences,
  the AI model and any locally cached backups.
* **You can delete individual transactions** and whole categories/accounts at any time in
  the app.
* Deleted transactions are soft-deleted in the database so they can be restored briefly;
  they are excluded from every balance, total and report while deleted.
* **Export before uninstalling** if you want to keep a copy. Ledgerra can export
  transactions to CSV.
* Ledgerra cannot delete data that has already been sent to a third-party destination you
  configured (Drive, WebDAV, a webhook endpoint). Delete it there too.

---

## 8. Children's privacy

Ledgerra is not directed at children under 13 and is not intended for use by children.
Because the app collects no data and has no accounts, it does not knowingly collect
information from anyone, including children.

---

## 9. Security

* All app data is inside the Android app sandbox.
* The app requests `allowBackup="false"`, so Android does not copy the database to cloud
  backup.
* The advertising ID permission is explicitly removed from the manifest.
* Optional app lock is enforced by Android's own biometric prompt — the biometric
  credential never enters the app, and the app only records that the check succeeded and
  when.
* Optional cloud backup credentials are stored in `EncryptedSharedPreferences`
  (AES-256, with the master key held in the Android Keystore).
* Optional cloud backups are encrypted before upload.
* Ledgerra is free and open source under the AGPL-3.0 licence. You can read the code,
  build it yourself and check every claim on this page.

---

## 10. Changes to this policy

Material changes will be published in the app's release notes and in the repository
history for this file, and the "effective date" at the top will be updated.

---

## 11. Contact

Questions about this policy, or a privacy request, can be raised through the support
contact listed on the Google Play listing for Ledgerra, or via the issue tracker at:
https://github.com/rezoan-ferdose/ledgerra/issues

---

*Ledgerra — private, offline-first expense tracking.*
