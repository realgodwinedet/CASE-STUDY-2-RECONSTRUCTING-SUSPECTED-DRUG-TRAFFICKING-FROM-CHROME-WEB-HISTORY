# SBT-DF204 — Computer Forensics Case Studies

## Case Study 2: Reconstructing Suspected Drug Trafficking From Chrome Web History

* **Student Name:** Godwin Edet Ikpi
* **Registration Number:** 2025/FWSD/11267
* **Course:** SBT-DF204 — Computer Forensics Case Studies
* **Case Title:** Reconstructing Suspected Drug Trafficking From Chrome Web History
* **Evidence Examined:** Chrome History SQLite database (`History`)

---

### 1. Executive Conclusion

Based on the supplied assessment brief, this investigation was conducted to determine whether the Chrome History database contained a coherent sequence of browser activity associated with suspected drug advertising, communications with a buyer, payment activity, cryptocurrency verification, and downloaded proof-of-payment records.

The analytical conclusions are strictly anchored in the observed SQLite records, including visit IDs, URL IDs, timestamps, transition values, `from_visit` relationships, and download logs. In accordance with forensic integrity standards, any fields requiring verification were populated directly from the examined database copy, ensuring that no speculative assumptions were treated as observed facts.

---

### 2. Evidence Acquisition and Integrity

The acquisition and preservation of the digital evidence followed strict chain-of-custody protocols. The original evidence file (`History`) was obtained from the designated repository (`[https://raw.githubusercontent.com/frankwxu/digital-forensics-lab/main/Digital_Currency/LabFiles/History](https://raw.githubusercontent.com/frankwxu/digital-forensics-lab/main/Digital_Currency/LabFiles/History)`) with a file size of 196,608 bytes and SHA-256 hash `b991b0fa69662c44caf599e468c8088576ebd9cebbf46d02c89cf51bcde3fd19`.

The original file was preserved under read-only permissions (`/home/kali/webhistory/History`), and a working copy was established at `/home/kali/Forensic_Evidence/Working_Copies/History_working_copy.sqlite` on October 6, 2026, at 12:45:00 UTC. All SQL queries and extractions were executed safely against this working copy using the SQLite3 command-line interface.

---

### 3. Database Method and Schema Examination

The investigation utilized structured SQL queries to interrogate the primary tables specified in the browser architecture: `urls`, `visits`, `downloads`, and `downloads_url_chains`.

* **Schema Verification:** Interrogation via `sqlite_master` and `PRAGMA table_info` confirmed the presence of key tables. The `urls` table stores unique URL identifiers, raw strings, and page titles. The `visits` table tracks navigation timing (`visit_time`), transition bits (`transition`), visit duration, and parent relationships (`from_visit`). The `downloads` table records file acquisition paths and byte counts, which are mapped through `downloads_url_chains` for complete source tracing.
* **Timestamp Normalization:** Chrome WebKit timestamps—recorded as microseconds elapsed since January 1, 1601 UTC—were normalized using SQLite datetime transformations. All primary investigative timelines are reported in UTC, maintaining strict separation from local timezone interpretations (such as UTC+1).

---

### 4. Transition and Navigation Analysis

Navigation behaviors were evaluated by decoding the bitwise transition flags (`transition & 0xFF`) alongside `from_visit` pointers, URLs, and page titles:

* **Link Clicks (`transition & 0xFF = 0`):** Identified standard user navigation from page to page, observed across Kraken funding/ledger pages and Mempool explorer links accessed via direct clicks (Visit IDs 66, 70, 71).
* **Omnibox Activity (`transition & 0xFF = 5`):** Captured direct user-initiated search queries and typed URL entries, such as searches for blockchain explorers (Visit IDs 40, 51).
* **Form Submissions (`transition & 0xFF = 7`):** Documented active form-handling actions, specifically corresponding to Gmail webmail composer submissions and email thread updates (`mail.google.com`).
* **Qualifiers and Parent Linkage (`from_visit`):** Sequential navigation chains were successfully reconstructed, linking search engine results directly to blockchain explorers and secure webmail threads.

---

### 5. Investigation Findings

* **E01 (Acquisition & Integrity):** The evidence file `History` (196,608 bytes; SHA-256: `b991b0fa69662c44caf599e468c8088576ebd9cebbf46d02c89cf51bcde3fd19`) was securely preserved under read-only permissions at `/home/kali/Forensic_Evidence/Working_Copies/History_working_copy.sqlite`.
* **E02 (Schema Architecture):** Database integrity and schema structures (`urls`, `visits`, `downloads`, `downloads_url_chains`) were fully verified via metadata inspection.
* **E03 (Chronological Scope):** Recorded web activity spans April 19, 2022, running from initial email drafting at 14:04:35 UTC through file acquisitions and cryptocurrency verifications up to 15:02:56 UTC.
* **E04 (Navigation Dynamics):** Activity demonstrated a blend of standard link traversals, direct omnibox queries, and active form submissions linked via parent `from_visit` pointers.
* **E05 (Communication Signatures):** Active interaction with webmail interfaces (`mail.google.com`) was verified, featuring draft subjects and page titles referencing "cheaper than Rx supplements" (Visit IDs 44–46, 72–74).
* **E06 (Financial & Payment Correlation):** Communications were substantiated by Gmail composer sessions explicitly referencing transaction ID `517b21569144339a96137ad8978408ea52b2fc144c98d3b0b168afdc5`, combined with lookups on `mempool.space`, `kraken.com`, and the downloaded payment proof (`proof_of_payment.png`, 33,844 bytes).
* **E07 (Timeline Coherence):** The reconstructed chronological timeline aligns file downloads, blockchain queries, exchange ledger checks, and email threads into a cohesive behavioral sequence.
* **E08 (Attribution & Limitations):** There is medium-to-high confidence that the browser profile engaged in cryptocurrency tracking, exchange verification, and pharmaceutical-related email correspondence. Limitations include potential shared profile environments, lack of direct device operator proof, and the absence of full message body contents.

---

### 6. Reconstructed Timeline

| Stage | UTC Timestamp | Visit / URL ID | Artifact | Observed Fact | Interpretation and Limit |
| --- | --- | --- | --- | --- | --- |
| **Communication** | 2022-04-19 14:04:35 | Visit ID 44 (URL ID 35) | `urls`, `visits` | Gmail composer session opened with title "cheaper than Rx supplements". | User drafted or viewed email correspondence regarding pharmaceutical products. |
| **File Download** | 2022-04-19 14:56:58 | Download ID 1 | `downloads`, `downloads_url_chains` | File `proof_of_payment.png` downloaded from Imgur (33,844 bytes) to `C:\Users\FSCS_User\Desktop\`. | Receipt or payment proof acquired locally. Requires endpoint analysis. |
| **Crypto Lookup** | 2022-04-19 15:01:33 | Visit ID 70 (URL ID 54) | `urls`, `visits` | Mempool explorer queried for TXID `517b21569144339a96137ad8978408ea52b2fc144c98d3b0b168afdc5`. | User verified public ledger status of a specific transaction hash. |
| **Exchange Ledger** | 2022-04-19 15:02:05 | Visit ID 71 (URL ID 55) | `urls`, `visits` | Kraken History Ledger page accessed ($41,563.70 USD$). | User reviewed financial exchange transaction history. |
| **Communication** | 2022-04-19 15:02:32 | Visit ID 72 (URL ID 56) | `urls`, `visits` | Gmail webmail composer loaded referencing supplement correspondence. | Active webmail session involving product communication. |
| **Correlation** | 2022-04-19 15:02:56 | Visit ID 74 (URL ID 37) | `urls`, `visits` | Gmail thread updated referencing transaction ID `txid: 517b21569144...`. | Directly ties financial transaction hash to supplement communications. |

---

### 7. Evidence Correlation Matrix

* **Suspected Advertisement / Posting:** Mapped to `mail.google.com` (Rx supplements) across Visit IDs 44–46, indicating access to email drafts referencing pharmaceutical products. *Limitation:* Indicates drafting/viewing text; server logs required for transmission proof.
* **Buyer Communication:** Mapped to `mail.google.com` composer interfaces (Visit IDs 48, 72), suggesting active correspondence with external parties.
* **Payment Reference:** Linked to `kraken.com` and Imgur via Download ID 1 and Visit ID 66, demonstrating local retention of the visual payment receipt `proof_of_payment.png` (33,844 bytes).
* **Cryptocurrency Verification:** Linked to `mempool.space` and `blockchain.com` via Visit ID 70 and 53, demonstrating public block explorer queries for specific transaction hashes, though not direct proof of wallet ownership.
* **Proof-of-Payment Download:** Linked to `[i.imgur.com/dTgrkP7.png](https://i.imgur.com/dTgrkP7.png)` (Download ID 1), confirming local storage of the transaction proof artifact on the Windows desktop path.

---

### 8. Attribution Assessment

The Chrome History database establishes that specific browser-profile activities occurred; however, it does not independently prove human identity or physical device operation. The following constraints govern attribution:

* The browser profile or underlying device may have been utilized by multiple individuals.
* Recorded URLs and titles do not guarantee full page content visibility or the completion of every displayed action.
* Redirects, cached data, automated scripts, and background browser processes can influence recorded history entries.
* The absence of history logs does not confirm an event did not happen, as history entries are subject to deletion or truncation.
* Financial, cryptocurrency, or communication references indicate page visits rather than verified real-world transactions.
* Definitive attribution requires corroborating endpoint logs, account authentication records, network artifacts, or file-system evidence.

---

### 9. Evidence Log

| Evidence ID | Source Record | Finding | Why it matters | SQL / Screenshot Reference |
| --- | --- | --- | --- | --- |
| **E01** | `sqlite_master` / File | Database verified and locked (`chmod 400`) | Ensures chain of custody and evidence integrity | `sha256sum History` |
| **E02** | `downloads` table | `proof_of_payment.png` downloaded at 14:56:58 UTC | Establishes local retention of transaction proof | `SELECT * FROM downloads;` |
| **E03** | `visits` / `urls` tables | Timestamps converted to UTC across all visits | Establishes accurate chronological order of events | `SELECT datetime(...) FROM visits;` |
| **E04** | `visits.transition` | Transition bits and qualifiers evaluated | Differentiates link clicks from omnibox searches | `SELECT transition FROM visits;` |
| **E05** | `urls` (Titles & URLs) | Kraken funding/ledger and Mempool explorer visits | Documents cryptocurrency and exchange interactions | `SELECT * FROM urls WHERE url LIKE '%kraken%';` |
| **E06** | `urls` (Gmail sessions) | Gmail sessions linking supplement ads and TXIDs | Ties financial transaction hashes to communications | `SELECT * FROM urls WHERE title LIKE '%supplements%';` |
| **E07** | Reconstructed Timeline | Chronological sequence mapped from 14:04 to 15:02 UTC | Provides a comprehensive behavioral narrative | Joined query across `visits` and `urls` |
| **E08** | Attribution Assessment | Profile activity recorded without device operator proof | Prevents over-attribution and accounts for shared access | N/A (Analytical synthesis) |

---

### 10. SQL Evidence Appendix

* **A. Table listing:** `SELECT name FROM sqlite_master WHERE type = 'table' ORDER BY name;`
* **B. URL schema:** `PRAGMA table_info(urls);`
* **C. Visits schema:** `PRAGMA table_info(visits);`
* **D. Downloads schema:** `PRAGMA table_info(downloads);`
* **E. Visit timeline:** Joined SQL query mapping `visit_id`, `url_id`, URLs, titles, and UTC visit times ordered chronologically.
* **F. Downloads:** `SELECT * FROM downloads ORDER BY start_time;`

---

### 11. Screenshot Appendix

* **Appendix A1:** Evidence file properties, filename, size, and hash verification.
* **Appendix A2:** SQLite schema and table listings.
* **Appendix A3:** Main SQL timeline output.
* **Appendix A4:** Relevant navigation and transition records (`transition & 255`).
* **Appendix A5:** Suspected posting and advertisement evidence (supplements and Gmail).
* **Appendix A6:** Communication and payment evidence (Kraken, Mempool, Blockchain).
* **Appendix A7:** Download evidence and URL chain (`downloads_url_chains`).
* **Appendix A8:** Final timeline and attribution correlation sequence.
