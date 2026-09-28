# the_alchemical_code_garden

A sci-fi/alchemical landing page for *The Alchemical Medium* — Philosophy of Coding in the Era of the Apex Mind by Joseph Wanyora Gikara.

Visitors can **read** the HTML ebook, **download** PDF/EPUB, and **support** the work via a custom M-Pesa Daraja STK push flow backed by Firebase Functions.

## Files

| File | Purpose |
|------|---------|
| `index.html` | Landing page — hero, action buttons, M-Pesa modal, synopsis modal |
| `index.png` | Hero background artwork |
| `The Alchemical Medium.html` | Read-in-browser version of the book |
| `the-alchemical-medium-3.pdf` | Downloadable PDF |
| `the-alchemical-medium.epub` | Downloadable EPUB |
| `firebase.json` | Firebase Hosting + Functions config |
| `functions/` | Firebase Functions (TypeScript) backend |

## Frontend

`index.html` is a self-contained page. To wire up M-Pesa, fill in your **Firebase config** in the `firebaseConfig` object at the top of the module script, then deploy the functions (see below). The callable function is named `initiateStkPush` in region `us-central1`.

The payment flow:

1. Click an amount button (KSh 100 / 250 / 500 / 1000) → opens the M-Pesa modal.
2. Enter an M-Pesa number, click **Send**.
3. `initiateStkPush` runs the STK push; the pending transaction is logged to Firestore `transactions`.
4. Safaricom calls the callback endpoint → final status recorded in `transactions`.

## Setup

### 1. Frontend config

Paste your **Firebase project web config** (Project settings → Your apps) into the
`firebaseConfig` object near the top of `index.html`. The callable function is
`initiateStkPush` in the `us-central1` region (change the region in the module
script if you deploy elsewhere).

### 2. Install the Firebase CLI

```bash
npm install -g firebase-tools
firebase login
firebase use --add   # link to your project
```

### 3. Configure Daraja (Safaricom)

Create an app at [developer.safaricom.co.ke](https://developer.safaricom.co.ke/), then copy the values into `functions/.env`:

```bash
cp functions/.env.example functions/.env
# edit functions/.env with your real keys
```

Required values:

- `DARAJA_ENV` — `sandbox` or `production`
- `DARAJA_CONSUMER_KEY` / `DARAJA_CONSUMER_SECRET` — OAuth credentials
- `DARAJA_SHORTCODE` — paybill/till number (sandbox test: `174379`)
- `DARAJA_PASSKEY` — from the Daraja dashboard
- `DARAJA_CALLBACK_URL` — the deployed HTTPS callback, e.g.
  `https://us-central1/<PROJECT_ID>.cloudfunctions.net/mpesaCallback`
  (or the hosting rewrite `https://<your-domain>/mpesa/callback`)

When you run `firebase deploy --only functions`, firebase-tools automatically makes
the values defined in `functions/.env` available to your deployed functions
(they are loaded in both the emulator and production). **Do not commit
`functions/.env`** — it is already covered by `.gitignore`.

> Names must not use the reserved prefixes `X_GOOGLE_`, `FIREBASE_`, `EXT_`, `KIT_`
> (this is why the Firebase web config lives in `index.html`, not in `.env`).

### 4. Install dependencies

```bash
cd functions && npm install && cd ..
```

### 5. Deploy

```bash
firebase deploy --only functions,hosting   # production
```

### 6. Run locally for testing

```bash
firebase emulators:start
```

## Firestore

Transactions (both `pending` and final) are written to the `transactions` collection:

```
phone: string
amount: number
accountReference: "the-alchemical-medium"
checkoutRequestId: string
status: "pending" | "success" | "failed"
resultCode / resultDesc: (final)
mpesaReceipt / partyA: (final, when present)
initiatedAt / reportedAt: serverTimestamp
```
