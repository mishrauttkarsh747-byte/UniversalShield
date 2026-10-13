# Data

Everything in `data/samples/` is **synthetic**: it was written by hand for this project to
demonstrate the features and the ML pipeline. It contains no real messages, people, phone
numbers or transactions. It is released under the project's MIT licence.

> **Safety:** the "fraud" URLs use invented look-alike domain names. Some of them may still be
> registered by somebody on the real internet. **Do not open them.** UniversalShield itself
> never fetches URLs, it only analyses their text.

## Files

| File | Rows | Used for |
|---|---|---|
| `samples/messages_synthetic.csv` | 100 (50 fraud, 50 legitimate) | Training/evaluating the optional message model; demo inputs |
| `samples/urls_synthetic.csv` | 80 (40 fraud, 40 legitimate) | Training/evaluating the optional URL model |
| `samples/sample_permission_report.txt` | 1 app | Demo of the permission-report upload (fictional flashlight app) |
| `samples/sample_permission_report.json` | 1 app | Same feature, JSON format (fictional loan app) |
| `devdb.mv.db` (created at runtime) | n/a | The dev profile's H2 database. Git-ignored and not part of the deliverable |

## Schemas

### Labelled text (`*_synthetic.csv`)

```csv
id,label,text
m001,1,URGENT: Your bank account will be blocked today. Update KYC now at http://kyc-update-bank.top/login
```

| Column | Type | Meaning |
|---|---|---|
| `id` | string | Unique row id (`m…` messages, `u…` URLs) |
| `label` | `0` or `1` | `1` = fraud/phishing, `0` = legitimate |
| `text` | string | The message text or the full URL. Quote with `"` if it contains commas; double any `"` inside |

UTF-8, header row `id,label,text` required, RFC 4180 quoting. `DatasetLoader` stops with the
row number if a row does not have exactly 3 columns or its label is not numeric.

### Permission report (upload)

Either plain text with one permission per line (lines in `aapt dump badging` style such as
`uses-permission: name='android.permission.READ_SMS'` are understood), or JSON:

```json
{ "appName": "Example app", "permissions": ["android.permission.READ_SMS", "CAMERA"] }
```

Maximum 64 KB, UTF-8. The file is parsed in memory and never stored.

### Transactions

No transaction dataset is bundled. The transaction detector learns "normal" only from the
user's own stored history (median of their last payments, see docs/SCORING.md). If you add a
dataset for experiments, use: `id,label,amount,currency,payment_method,recipient_is_new,attempts_last_hour,requested_by_other_party,description`.

## How the synthetic data was made

- Fraud examples imitate widely published scam patterns: KYC/account-blocking SMS, OTP
  requests, fake delivery fees, prize and refund lures, job/investment offers, "digital arrest",
  look-alike and brand-in-subdomain URLs, IP-address hosts and URL shorteners.
- Legitimate examples are ordinary personal, delivery, banking-notification and work messages
  and well-known real sites (wikipedia.org, github.com, spring.io...).
- Because the same authors wrote the rules and the data, metrics on this data are optimistic.
  See docs/ML.md.

## Using a real dataset

Only use data you have the right to use (check its licence), remove personal data first, and
never commit it if the licence forbids redistribution. Convert it to the `id,label,text`
schema and pass it to `TrainingCli` with `--data`.
