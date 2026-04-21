# Task 2 — Phishing Detection & Awareness Report

### Future Interns Cybersecurity Internship | Aspiring SOC Analyst Portfolio



## Overview

This task involved the collection, analysis, and classification of real-world phishing email samples. The goal was to identify phishing indicators, perform infrastructure investigation, and produce an awareness document to help users recognise and avoid phishing attacks.

> **Scope:** Passive analysis only — no systems were attacked or exploited.  
> **Approach:** Security education + threat analysis.


## Tools Used

|Tool                                                                      |Purpose                                     |
|--------------------------------------------------------------------------|--------------------------------------------|
|[rf-peixoto/phishing_pot](https://github.com/rf-peixoto/phishing_pot)     |Real-world phishing `.eml` sample collection|
|[MXToolbox Email Header Analyzer](https://mxtoolbox.com/EmailHeaders.aspx)|Header parsing, SPF/DKIM/DMARC analysis     |
|[Whois.com](https://www.whois.com)                                        |Domain and IP infrastructure investigation  |
|RIPE / ARIN WHOIS                                                         |IP geolocation and ownership lookup         |


## Samples Analysed

### Sample 1 — Fake Microsoft Security Alert

|Field             |Value                                    |
|------------------|-----------------------------------------|
|**Subject**       |Microsoft account unusual signin activity|
|**Sender Domain** |thcultarfdes.co.uk (unregistered)        |
|**Originating IP**|89.144.44.2                              |
|**IP Owner**      |MSCode, Poland 🇵🇱                         |
|**SPF**           |❌ FAIL (spf=none)                        |
|**DKIM**          |❌ FAIL (not signed)                      |
|**DMARC**         |❌ No record found                        |
|**Classification**|🔴 PHISHING                               |

**Key Indicators:**

- Sender domain `thcultarfdes.co.uk` not registered — completely fabricated
- Secondary domain `access-accsecurity.com` also unregistered
- All authentication checks failed
- Email impersonates Microsoft using fear-based urgency language
- Originating IP traced to Poland, unrelated to Microsoft infrastructure


### Sample 2 — Fake Brazilian Bank Rewards Lure (Bradesco/LIVELO)

|Field             |Value                                                                               |
|------------------|------------------------------------------------------------------------------------|
|**Subject**       |CLIENTE PRIME - BRADESCO LIVELO: Seu cartão tem 92.990 pontos LIVELO expirando hoje!|
|**Sender Domain** |atendimento.com.br                                                                  |
|**Originating IP**|137.184.34.4                                                                        |
|**IP Owner**      |DigitalOcean, LLC 🇺🇸                                                                 |
|**SPF**           |❌ FAIL (spf=temperror — DNS timeout)                                                |
|**DKIM**          |❌ FAIL (not signed)                                                                 |
|**DMARC**         |❌ No record / name servers not found                                                |
|**Classification**|🔴 PHISHING                                                                          |

**Key Indicators:**

- Sent from a cheap DigitalOcean VPS — typical phishing infrastructure
- Domain `atendimento.com.br` has broken nameservers (QREFUSED)
- Portuguese-language lure targeting Brazilian banking customers
- Financial urgency tactic: “92,990 points expiring today!”
- No legitimate Bradesco/LIVELO sending infrastructure involved


### Sample 3 — Fake Binance Withdrawal Notification

|Field             |Value                                                      |
|------------------|-----------------------------------------------------------|
|**Subject**       |[Binance] Withdraw Successful - 2023-07-30 51:51:51(UTC)   |
|**Sender Display**|Binance <noreply-supportbinancewallet.irs@auswestbc.com.au>|
|**Originating IP**|69.169.224.12                                              |
|**IP Owner**      |Amazon Web Services (AWS SES) 🇺🇸                            |
|**SPF**           |⚠️ PASS (AWS SES abused) — alignment FAIL                   |
|**DKIM**          |❌ FAIL (misaligned — amazonses.com vs auswestbc.com.au)    |
|**DMARC**         |❌ No record found                                          |
|**Mailer**        |PHPMailer 6.1.5                                            |
|**Classification**|🔴 PHISHING                                                 |

**Key Indicators:**

- Most sophisticated sample — abused AWS SES to partially pass SPF
- Sender email `@auswestbc.com.au` impersonates Binance (real domain: `@binance.com`)
- Domain `auswestbc.com.au` registered to individual (Steve Cleaver, AU), status: `serverRenewProhibited` — suspended/flagged
- Sent using PHPMailer — common bulk phishing tool
- Timestamp anomaly in subject: `51:51:51` is not a valid time — indicates automated/fake generation
- Financial panic tactic: unexpected withdrawal notification


## Comparative Summary

|                  |Sample 1        |Sample 2        |Sample 3         |
|------------------|----------------|----------------|-----------------|
|**Lure Type**     |Account security|Financial reward|Crypto withdrawal|
|**Language**      |English         |Portuguese      |English          |
|**SPF**           |❌ Fail          |❌ Fail          |⚠️ Pass (abused)  |
|**DKIM**          |❌ Fail          |❌ Fail          |❌ Fail           |
|**DMARC**         |❌ None          |❌ None          |❌ None           |
|**Infrastructure**|Fake domain (PL)|DigitalOcean VPS|AWS SES abuse    |
|**Sophistication**|Low             |Medium          |High             |
|**Classification**|🔴 Phishing      |🔴 Phishing      |🔴 Phishing       |


## MITRE ATT&CK Mapping

|Technique                        |ID       |Samples      |
|---------------------------------|---------|-------------|
|Phishing: Spearphishing Link     |T1566.002|All 3        |
|Impersonation                    |T1656    |All 3        |
|Abuse of Cloud Services (AWS SES)|T1583.006|Sample 3     |
|Financial Lure Social Engineering|T1598    |Samples 2 & 3|


## IOC Table

|Type  |Value                 |Associated Sample|
|------|----------------------|-----------------|
|IP    |89.144.44.2           |Sample 1         |
|IP    |137.184.34.4          |Sample 2         |
|IP    |69.169.224.12         |Sample 3         |
|Domain|thcultarfdes.co.uk    |Sample 1         |
|Domain|access-accsecurity.com|Sample 1         |
|Domain|atendimento.com.br    |Sample 2         |
|Domain|auswestbc.com.au      |Sample 3         |


## Prevention Guidelines

**Do:**

- Always check the sender’s actual email address, not just the display name
- Verify unexpected security alerts by going directly to the official website
- Look for authentication warnings in your email client
- Report suspicious emails to your IT/security team

**Don’t:**

- Click links in unsolicited emails about account alerts or rewards
- Trust emails just because they look official — branding can be faked
- Act on urgent financial requests without independent verification
- Assume an email is safe because it passed spam filters
