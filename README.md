# israelisign — PKI Trust Anchors · עוגני אמון

Public, off-platform publication of the israelisign certificate hierarchy and
its SHA-256 fingerprints. Anyone holding a document signed by israelisign can
use this repository to cross-check the signing chain **without relying on the
israelisign platform itself**.

פרסום ציבורי של היררכיית התעודות של israelisign וטביעות האצבע (SHA-256) שלה.
הפרסום נעשה מחוץ לפלטפורמה, כדי שכל מי שמחזיק מסמך חתום יוכל לבדוק את שרשרת
החתימה **בלי להסתמך על הפלטפורמה עצמה**.

> **Scope · היקף.** This repository contains public X.509 certificates only.
> It holds no private keys, no customer data and no signed documents. The
> private keys never leave the platform's signing server.
>
> המאגר מכיל תעודות X.509 ציבוריות בלבד. אין בו מפתחות פרטיים, נתוני לקוחות
> או מסמכים חתומים. המפתחות הפרטיים אינם יוצאים משרת החתימה של הפלטפורמה.

## Chain · שרשרת

```
IsraeliSign Root CA 2026-10           self-signed · pathlen:1 · 2026-10-13 → 2051-10-07
├── IsraeliSign Platform CA 2026-10   issued by Root · pathlen:0 · 2026-10-13 → 2035-10-13
└── IsraeliSign Signer CA 2026-10     issued by Root · pathlen:0 · 2026-10-13 → 2035-10-13
                                      (issues 24 h one-shot signer certificates)
```

| Certificate | Subject | Serial | Valid (UTC) | Key · Signature |
|---|---|---|---|---|
| Root CA | `CN=IsraeliSign Root CA 2026-10, O=Taltul Technologies, C=IL` | `5051F1461A1E52ACBE31159A82BBFB85BF264E96` | 2026-10-13 17:46:28 → 2051-10-07 17:46:28 | RSA 4096 · sha256WithRSAEncryption |
| Platform CA | `CN=IsraeliSign Platform CA 2026-10, O=Taltul Technologies, C=IL` | `1000` | 2026-10-13 17:55:26 → 2035-10-13 17:55:26 | RSA 4096 · sha256WithRSAEncryption |
| Signer CA | `CN=IsraeliSign Signer CA 2026-10, O=Taltul Technologies, C=IL` | `1001` | 2026-10-13 17:57:25 → 2035-10-13 17:57:25 | RSA 4096 · sha256WithRSAEncryption |

All three are CA certificates with `Basic Constraints: critical, CA:TRUE` and
`Key Usage: critical, Certificate Sign, CRL Sign`.

## Files · קבצים

| File | Content |
|---|---|
| `root-ca.pem` / `root-ca.der` | Root CA certificate (PEM / DER) |
| `platform-ca.pem` / `platform-ca.der` | Platform CA certificate |
| `signer-ca.pem` / `signer-ca.der` | Signer CA certificate |
| `FINGERPRINTS.txt` | SHA-256 fingerprint of each certificate (SHA-256 of the DER encoding) |
| `timestamps/FINGERPRINTS.txt.*.tsr` | RFC 3161 time-stamp responses over `FINGERPRINTS.txt` (see below) |

## How to verify · כיצד לאמת

```console
# 1. Fingerprint of a certificate. Compare it with FINGERPRINTS.txt.
$ openssl x509 -in signer-ca.pem -noout -fingerprint -sha256

# The fingerprint is the SHA-256 of the DER file, so this gives the same value:
$ shasum -a 256 signer-ca.der

# 2. The two intermediate CAs chain to the Root.
$ openssl verify -x509_strict -CAfile root-ca.pem platform-ca.pem signer-ca.pem
```

Compare the fingerprints with **two other sources**:

1. the public verification page, https://platform.israelisign.com/verify;
2. the certificate chain embedded in the signed PDF itself.

All three must match. If they do not, one source has been tampered with.
Treat the signature as unverifiable until the discrepancy is explained.

The issuing certificates are also served from the addresses written into the
certificates themselves (RFC 5280 AIA/CDP: `http://crl.israelisign.com/root-ca.der`,
`http://crl.israelisign.com/root-ca.crl`). This repository is the
**independent copy**: it survives even if the platform does not.

יש להשוות את הטביעות לשני מקורות נוספים: דף האימות הציבורי, ושרשרת התעודות
המוטמעת בקובץ ה-PDF החתום עצמו. שלושת המקורות חייבים להתאים. אי-התאמה פירושה
שאחד המקורות שונה, ויש להתייחס לחתימה כאל בלתי ניתנת לאימות עד שההבדל יוסבר.

## Timestamps · חותמות זמן

`FINGERPRINTS.txt`, as published at tag `publication/2026-10-13` (SHA-256
`bc891f4c122cd440e47eee752cf2a98794843740aac8037efcc8e8aeb6f52d71`), was
time-stamped (RFC 3161) by two independent time-stamping authorities. Each
response proves that these exact bytes existed **no later than** its time.

| File | Authority | Time (UTC) |
|---|---|---|
| `timestamps/FINGERPRINTS.txt.digicert.tsr` | DigiCert (`http://timestamp.digicert.com`), chains to DigiCert Trusted Root G4 | 2026-10-14 07:23:33 |
| `timestamps/FINGERPRINTS.txt.freetsa.tsr` | freetsa.org (`https://freetsa.org/tsr`) | 2026-10-14 07:23:32 |

```console
$ git show publication/2026-10-13:FINGERPRINTS.txt > FINGERPRINTS.txt
$ openssl ts -reply -in timestamps/FINGERPRINTS.txt.digicert.tsr -text   # time, policy, serial

# Use OpenSSL 3 (tested: 3.0.13 on Ubuntu, 3.6 on macOS). The macOS built-in `openssl` is LibreSSL 3.3:
# it cannot build the DigiCert chain ("unable to get local issuer certificate"). On macOS, use Homebrew's openssl@3.
# DigiCert: against the system trust store (macOS: /etc/ssl/cert.pem; Debian/Ubuntu: /etc/ssl/certs/ca-certificates.crt)
$ openssl ts -verify -data FINGERPRINTS.txt -in timestamps/FINGERPRINTS.txt.digicert.tsr -CAfile /etc/ssl/cert.pem

# freetsa.org: its root and TSA certificates are published at https://freetsa.org/files/
$ curl -sO https://freetsa.org/files/cacert.pem && curl -sO https://freetsa.org/files/tsa.crt
$ openssl ts -verify -data FINGERPRINTS.txt -in timestamps/FINGERPRINTS.txt.freetsa.tsr -CAfile cacert.pem -untrusted tsa.crt
```

Both must print `Verification: OK`. Neither authority is a qualified (eIDAS)
trust service. Together they are independent third-party evidence of the
publication time, alongside the GitHub time of tag `publication/2026-10-13`.

`FINGERPRINTS.txt` כפי שפורסם בתג `publication/2026-10-13` קיבל חותמת זמן
(RFC 3161) משתי רשויות חותמות זמן בלתי תלויות. כל חותמת מוכיחה שהבתים המדויקים
האלה היו קיימים **לא יאוחר** מהמועד שבה. אף אחת מהרשויות אינה שירות אמון מוסמך
(eIDAS). יחד הן ראיה בלתי תלויה של צד שלישי למועד הפרסום, לצד מועד התג ב-GitHub.

## Previous chain · השרשרת הקודמת

The chain in force before this publication (Root `B8:C0:1E:50…`, Platform CA 1
`D5:EE:B6:FE…`, Signer CA 1 `C1:86:00:4C…`) is retired. Its certificates and
fingerprints stay published, unchanged, at tag
[`publication/2026-10-12`](https://github.com/management458/israelisign-pki-trust/tree/publication/2026-10-12),
for checking documents signed with it. Once the platform's signing server
switches to the chain above, it signs only with that chain. The exact time of
the switch is recorded as a `cert.rotated` event in the platform audit trail.

השרשרת שהייתה בתוקף לפני פרסום זה (שורש `B8:C0:1E:50…`, Platform CA 1
`D5:EE:B6:FE…`, Signer CA 1 `C1:86:00:4C…`) הוצאה משימוש. התעודות וטביעות
האצבע שלה נשארות מפורסמות, ללא שינוי, בתג `publication/2026-10-12`, לבדיקת
מסמכים שנחתמו בה. מרגע ששרת החתימה של הפלטפורמה עובר לשרשרת שלמעלה, הוא חותם
רק בה. המועד המדויק של המעבר נרשם כאירוע `cert.rotated` ביומן הביקורת של הפלטפורמה.

## Integrity of this repository · שלמות המאגר

- The `main` branch and all tags are protected against deletion and against
  history rewriting (force-push). A published commit can be superseded by a
  new commit. It cannot be altered or removed.
- Commits are created through GitHub and signed by GitHub ("Verified").
- Every publication is marked by a protected tag, `publication/YYYY-MM-DD`.

הענף `main` וכל התגיות מוגנים מפני מחיקה ומפני שכתוב היסטוריה. commit שפורסם
יכול להתעדכן ב-commit חדש, אבל אי אפשר לשנות או למחוק אותו.

## Provenance · מקור

This hierarchy was first published on 2026-09-07 at
`github.com/yossigabay/israelisign-pki-trust` (commit
`9838e080fec0f4ebca271cdeb52454c17dd79955`), a repository held under the
personal account of a former contractor. On 2026-10-12 the publication moved
to this repository, which is controlled by the platform owner. No certificate
changed. The seven published files are **byte-identical** to the 2026-09-07
publication, as their git blob identifiers show:

| File | Git blob id (identical in both publications) |
|---|---|
| `FINGERPRINTS.txt` | `897d3278495a5abe7194775246b2c48f0448fb1a` |
| `root-ca.pem` | `4b877a0decd9bd0fbfd37ce03c8dc44896cb50cc` |
| `root-ca.der` | `53c9fbc1c90d37e01cf2c97166828e320abd2fc1` |
| `platform-ca.pem` | `dc2a57a40e1f43aebb4ebbbaefc2c9b48f8edc43` |
| `platform-ca.der` | `57f246b93e27ae9c037646b80533b173929e0938` |
| `signer-ca.pem` | `d60d7bbc652c1edaea2bb0956cdd8d248d944efe` |
| `signer-ca.der` | `956770714c88c5825e0f39932f543f60d448b4ed` |

Anyone can reproduce this check with `git hash-object <file>`.

ההיררכיה פורסמה לראשונה ב-07.09.2026 במאגר שבחשבון האישי של קבלן לשעבר.
ב-12.10.2026 הפרסום עבר למאגר זה, שבשליטת בעלי הפלטפורמה. אף תעודה לא השתנתה:
שבעת הקבצים זהים בייט-בייט לפרסום המקורי, כפי שמראים מזהי ה-blob שבטבלה.

## Rotation log · יומן החלפת תעודות

| Date | Event | Old fingerprint (SHA-256) | New fingerprint (SHA-256) |
|---|---|---|---|
| 2026-07-28 | Initial issuance · הקמה ראשונית | — | Root `B8:C0:1E:50…`, Platform CA 1 `D5:EE:B6:FE…` |
| 2026-07-29 | Signer CA 1 issuance · הוספת רשות חותמים | — | Signer CA 1 `C1:86:00:4C…` |
| 2026-10-12 | Publication moved to this repository, no certificate change · העברת הפרסום, ללא החלפת תעודה | — | — |
| 2026-10-13 | Replacement of the whole chain · החלפת השרשרת כולה | Root `B8:C0:1E:50…`, Platform CA 1 `D5:EE:B6:FE…`, Signer CA 1 `C1:86:00:4C…` | Root `2F:F2:4E:6F…`, Platform CA `5C:DA:AC:93…`, Signer CA `26:72:BB:16…` |

At every rotation, the new certificate and the updated `FINGERPRINTS.txt` are
committed here **before** the new key signs anything, and a `cert.rotated`
event is recorded in the platform audit trail.
