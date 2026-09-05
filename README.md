# 🔎 Data Carving with XXD, Binwalk & Scalpel

> **SBT-DF202 — Computer and Digital Forensics | Practical Lab 3**

A hands-on digital forensics laboratory demonstrating **hexadecimal analysis, file-signature identification, forensic image examination, deleted-file recovery, file carving, embedded-content analysis, and cryptographic verification** using Kali Linux and open-source forensic tools.

---

## 📌 Overview

Data carving is a digital forensic technique used to identify and recover files from raw data based primarily on their **binary structure and file signatures**, rather than relying entirely on file-system metadata.

This practical explored a complete evidence-analysis workflow, beginning with preservation and hashing of laboratory evidence and progressing through low-level hexadecimal inspection, forensic file-system analysis, deleted-file recovery and signature-based carving.

The investigation used:

* `xxd` for hexadecimal inspection and reconstruction
* `strings` for readable-content analysis
* The Sleuth Kit for forensic image and file-system analysis
* `Binwalk` for signature and embedded-content identification
* `Scalpel` for signature-based file carving
* `7z` for forensic archive extraction
* `md5sum` and `sha256sum` for cryptographic verification

The work was conducted against **authorised laboratory evidence supplied for the exercise**.

---

## 🎯 Objectives

The practical objectives were to:

* Understand the principles of data carving.
* Examine binary data using hexadecimal representations.
* Identify JPEG file signatures and structural markers.
* Create and reverse a hexadecimal dump.
* Verify reconstructed evidence using cryptographic hashes.
* Extract and examine readable strings from binary files.
* Analyse a forensic image using The Sleuth Kit.
* Examine file-system structures and deleted directory entries.
* Recover deleted files using `icat`.
* Investigate raw storage blocks using `blkcat`.
* Identify recognised signatures and structures using Binwalk.
* Extract recoverable files using Scalpel.
* Validate recovered artefacts using MD5 and SHA-256.
* Document forensic findings, limitations and tool behaviour.

---

# 🧪 Evidence Examined

The laboratory provided the following primary evidence:

| Evidence            | Description               | Purpose                                   |
| ------------------- | ------------------------- | ----------------------------------------- |
| `J_ub_law.jpg`      | JPEG image                | Hexadecimal/signature analysis            |
| `Ch01InChap01.dd`   | Forensic disk image       | File-system and deleted-file analysis     |
| `120M.7z`           | Compressed forensic image | USB carving exercise                      |
| `File_carving.docx` | Instructor material       | Referenced by the laboratory instructions |

### Evidence Handling

The original evidence was retained separately from working and recovered artefacts.

The analysis directory was organised as follows:

```text
Lab3_Data_Carving/
│
├── original_evidence/
│   ├── J_ub_law.jpg
│   ├── Ch01InChap01.dd
│   └── 120M.7z
│
├── working/
│   └── J_ub_law_working.jpg
│
├── recovered/
│   ├── Billing_Letter.doc
│   ├── confirmation.txt
│   ├── letter1.txt
│   └── Regrets.doc
│
├── hashes/
│   ├── original_md5.txt
│   ├── original_sha256.txt
│   ├── recovered_md5.txt
│   └── recovered_sha256.txt
│
├── scalpel_output/
│   ├── audit.txt
│   ├── doc-2-0/
│   └── doc-3-0/
│
├── tsk_analysis/
│   ├── Billing_Letter_strings.txt
│   ├── Regrets_strings.txt
│   ├── binwalk_dd.txt
│   ├── fls_deleted_files.txt
│   ├── fls_recursive.txt
│   ├── fsstat.txt
│   ├── img_stat.txt
│   ├── mmls.txt
│   ├── recovered_file_types.txt
│   ├── recovered_files.txt
│   ├── scalpel_run.txt
│   └── scalpel_summary.txt
│
└── xxd_analysis/
    ├── J_ub_law.hex
    ├── J_ub_law_reconstructed.jpg
    ├── binwalk_jpeg.txt
    ├── exiftool_output.txt
    ├── jpeg_footer.txt
    ├── jpeg_header.txt
    ├── jpeg_metadata_strings.txt
    └── jpeg_strings.txt
```

---

# 🔐 1. Evidence Integrity & Hashing

Cryptographic hashes were calculated before and during the analysis to provide reference values for evidence integrity.

### Original Evidence

| Evidence          | MD5                                | SHA-256                                                            |
| ----------------- | ---------------------------------- | ------------------------------------------------------------------ |
| `J_ub_law.jpg`    | `83a360ac7f7e0ca318e5bfe39f95f137` | `238ff34393c50e52c0e8b14fcff8ec7dc29e23914dbc435f8ef998d172a91468` |
| `Ch01InChap01.dd` | `a117773bcf1fc88ec0ab8e0a349fbbcb` | `3ce8053e4f3d9c8ab98b3aadb2480685efb8e4980d34297b83bd5a09b1a7b122` |
| `120M.7z`         | `dfe7b5424e54cd1bf50d5df47aceeb3c` | `2d2a3d93c9ec65bcad9f89c9894429ea72caa8add07726a3b96ec9a6ab6a58ce` |

These values provide a reproducible reference for detecting unintended modification of the supplied evidence.

---

# 🧩 2. JPEG Hexadecimal Analysis with XXD

## 2.1 JPEG Signature Identification

The JPEG sample was examined using:

```bash
xxd J_ub_law.jpg | head
```

The JPEG began with the standard **Start of Image (SOI)** marker:

```text
FF D8
```

The SOI marker identifies the beginning of the JPEG binary structure.

The end of the file was subsequently examined using:

```bash
xxd J_ub_law.jpg | tail
```

The standard **End of Image (EOI)** marker was identified:

```text
FF D9
```

This confirmed the expected JPEG start and termination markers.

---

## 2.2 Creating a Plain Hex Dump

A plain hexadecimal representation was generated with:

```bash
xxd -p J_ub_law.jpg > J_ub_law.hex
```

The resulting hexadecimal file was retained under:

```text
xxd_analysis/J_ub_law.hex
```

---

## 2.3 Reconstructing the JPEG

The hexadecimal representation was converted back into binary form:

```bash
xxd -r -p J_ub_law.hex J_ub_law_reconstructed.jpg
```

The reconstructed file was then compared byte-for-byte with the original:

```bash
cmp J_ub_law.jpg J_ub_law_reconstructed.jpg
```

No difference was reported by `cmp`.

### Result

The reconstructed JPEG was therefore **byte-for-byte identical to the original**.

---

# 🧮 3. Cryptographic Verification of Reconstruction

The original and reconstructed JPEG were hashed using:

```bash
md5sum J_ub_law.jpg J_ub_law_reconstructed.jpg
```

and:

```bash
sha256sum J_ub_law.jpg J_ub_law_reconstructed.jpg
```

Both files produced matching cryptographic hashes.

| File               | MD5                                | SHA-256                                                            |
| ------------------ | ---------------------------------- | ------------------------------------------------------------------ |
| Original JPEG      | `83a360ac7f7e0ca318e5bfe39f95f137` | `238ff34393c50e52c0e8b14fcff8ec7dc29e23914dbc435f8ef998d172a91468` |
| Reconstructed JPEG | `83a360ac7f7e0ca318e5bfe39f95f137` | `238ff34393c50e52c0e8b14fcff8ec7dc29e23914dbc435f8ef998d172a91468` |

### Finding

The matching MD5 and SHA-256 values demonstrate that the hexadecimal dump was successfully reversed without altering the binary content of the JPEG.

---

# 🔍 4. JPEG Strings & Metadata Analysis

Readable content was examined using:

```bash
strings J_ub_law.jpg
```

Additional searches were performed to identify potentially useful metadata-like strings.

The JPEG was also examined using file-identification and metadata-analysis tools.

### Observations

The file was identified as a JPEG containing EXIF information associated with:

* **Manufacturer:** Nikon Corporation
* **Camera family/model information:** Nikon D4
* **Embedded EXIF/TIFF structures**
* Adobe copyright-related data at offset `0x68DC`

The file was identified as a JPEG EXIF image by the `file` utility.

These observations demonstrate how binary artefacts can contain useful contextual information even without opening the image in a graphical viewer.

---

# 💽 5. The Sleuth Kit Analysis

The forensic image `Ch01InChap01.dd` was analysed using multiple components of **The Sleuth Kit (TSK)**.

Tools used included:

```text
img_stat
mmls
fsstat
fls
icat
blkcat
```

---

## 5.1 Image Analysis — img_stat

The image was examined using:

```bash
img_stat Ch01InChap01.dd
```

The image was identified as a raw forensic image with a size of:

```text
1,474,560 bytes
```

The sector size was:

```text
512 bytes
```

---

## 5.2 Partition Analysis — mmls

The following command was executed:

```bash
mmls Ch01InChap01.dd
```

The image did not return a conventional partition-table listing.

This was significant because the image contained a **standalone FAT12 file system beginning at sector 0**, rather than a conventional partition beginning at a non-zero offset.

Therefore, subsequent file-system analysis was performed without applying an arbitrary partition offset.

---

## 5.3 FAT12 File-System Analysis — fsstat

The file system was examined using:

```bash
fsstat Ch01InChap01.dd
```

The image was identified as:

```text
FAT12
```

Important characteristics included:

| Attribute         | Observed value         |
| ----------------- | ---------------------- |
| File system       | FAT12                  |
| Sector size       | 512 bytes              |
| Boot sector       | Sector 0               |
| FAT 0             | Sectors 1–9            |
| FAT 1             | Sectors 10–18          |
| Root directory    | Sectors 19–32          |
| Data/cluster area | Beginning at sector 33 |

This provided the structural context required for examining deleted directory entries and recovering their content.

---

# 🗂️ 6. Deleted File Identification with FLS

The file-system directory structure was examined using:

```bash
fls -r -p Ch01InChap01.dd
```

Deleted entries were identified within the FAT12 image.

### Deleted artefacts identified

```text
Billing Letter.doc
confirmation.txt
letter1.txt
Regrets.doc
```

The image also contained allocated files including:

```text
Client Info.mdb
Income.xls
```

The deleted entries provided metadata references that could be used with `icat` to recover the underlying file content.

---

# 📤 7. Deleted File Recovery with ICAT

The deleted files were recovered from the forensic image using `icat`.

Recovered artefacts included:

| File                 | Result                 |
| -------------------- | ---------------------- |
| `Billing_Letter.doc` | Successfully recovered |
| `confirmation.txt`   | Successfully recovered |
| `letter1.txt`        | Successfully recovered |
| `Regrets.doc`        | Successfully recovered |

The recovered files were stored separately under:

```text
recovered/
```

### Recovered file types

```text
Billing_Letter.doc  → Microsoft Word document
confirmation.txt    → ASCII text
letter1.txt         → ASCII text
Regrets.doc         → Microsoft Word document
```

The recovered Word documents were identified as valid Word files and were subjected to further examination.

---

# 🧱 8. Block-Level Examination with BLKCAT

Raw storage blocks were examined using:

```bash
blkcat
```

This provided a lower-level view of the data stored within the forensic image.

Block-level examination demonstrated that forensic analysis can progress beneath normal file-system presentation and inspect the underlying storage units directly.

---

# 🕵️ 9. Embedded Content Analysis with Binwalk

Binwalk was used to identify known signatures and embedded structures.

### JPEG Analysis

Binwalk identified the JPEG structure, including:

```text
JPEG image data
EXIF metadata
TIFF little-endian structure
Adobe copyright-related data
```

The JPEG analysis demonstrated how signature-based tools can quickly identify recognised structures within binary evidence.

### DD Image

Binwalk was also executed against the forensic DD image.

No additional recognised signatures were reported in the resulting scan.

This does **not** mean that the image contained no recoverable data. The subsequent TSK analysis demonstrated that deleted files were present and recoverable from the FAT12 file system.

---

# 🧰 10. USB Forensic Image Preparation

The supplied:

```text
120M.7z
```

archive was extracted using:

```bash
7z x 120M.7z
```

The extracted image was subsequently used for the Scalpel carving exercise.

The original compressed evidence remained preserved in:

```text
original_evidence/
```

---

# 🪚 11. Scalpel File Carving

Scalpel was configured with the required signatures and executed against the USB forensic image.

The carving operation used:

```bash
sudo scalpel \
-c /etc/scalpel/scalpel.conf \
-o ~/Lab3_Data_Carving/scalpel_output \
~/Lab3_Data_Carving/original_evidence/Ch01InChap01.dd
```

The output was retained under:

```text
scalpel_output/
```

---

## 11.1 Scalpel Results

Scalpel produced:

```text
10 DOC candidates
0 JPEG candidates
```

Several of the carved DOC candidates were examined to determine whether they represented valid recoverable documents or false positives.

### Valid Word-document candidates

The following candidates contained recognisable Word-document content:

```text
doc-2-0/00000002.doc
doc-2-0/00000004.doc

doc-3-0/00000007.doc
doc-3-0/00000009.doc
```

The results indicated that:

* `00000002.doc` and `00000007.doc` represented duplicate Billing Letter content.
* `00000004.doc` and `00000009.doc` represented duplicate Regrets content.
* Other candidates were incomplete, invalid or over-carved.

---

# ⚠️ 12. Scalpel Limitations & False Positives

One of the most important findings from the exercise was that **file carving is not automatically equivalent to successful file recovery**.

Scalpel identified multiple DOC candidates, but not every candidate represented a valid standalone document.

Observed issues included:

* Duplicate recovered documents.
* Incomplete files.
* Invalid DOC candidates.
* Over-carved data.
* File-boundary uncertainty.

This demonstrates an important forensic principle:

> **A carving tool's output must be validated rather than automatically treated as confirmed evidence.**

The results were therefore compared against the files recovered using The Sleuth Kit.

---

# 🔐 13. Recovered Artefact Hashes

Recovered artefacts were independently hashed using MD5 and SHA-256.

| Artefact             | MD5                                | SHA-256                                                            |
| -------------------- | ---------------------------------- | ------------------------------------------------------------------ |
| `Billing_Letter.doc` | `9fe241d0dde27e83442010b3eee5ad32` | `7eefeb4187bb551f366da33bcab5b7224f905bb98f37797c28f183f72aec2e99` |
| `confirmation.txt`   | `18e391549e4a8bc990b264f590fb33bb` | `51b981e09b72739fac67c2d0124a2603cae8c35e075f5e2f66fbad03d53afcc0` |
| `letter1.txt`        | `49f68104660a091a4c015e86f3151237` | `341065f2c3c2168e246aa9a346439dfa4e6e4c55de4b1b8ecc78e4b83206545f` |
| `Regrets.doc`        | `ebcfbf22bdf81a60f6a16709d30c1dad` | `109048540a0a1a452f9346522a325219b471eae41b85a6298890e94ae750b965` |

These values provide reproducible identifiers for the recovered artefacts.

---

# 📊 14. Investigation Summary

| Technique                  | Tool        | Result                                     |
| -------------------------- | ----------- | ------------------------------------------ |
| Hexadecimal inspection     | `xxd`       | JPEG binary structure examined             |
| JPEG signature analysis    | `xxd`       | `FF D8` and `FF D9` identified             |
| Hex reconstruction         | `xxd -r`    | JPEG reconstructed successfully            |
| Binary comparison          | `cmp`       | Original and reconstructed files matched   |
| Cryptographic verification | MD5/SHA-256 | Matching hashes confirmed reconstruction   |
| String analysis            | `strings`   | Readable binary content examined           |
| Image analysis             | `img_stat`  | Raw forensic image characterised           |
| Partition analysis         | `mmls`      | No conventional partition table identified |
| File-system analysis       | `fsstat`    | FAT12 structure identified                 |
| File listing               | `fls`       | Deleted files identified                   |
| File recovery              | `icat`      | Four deleted artefacts recovered           |
| Block analysis             | `blkcat`    | Raw storage-level examination performed    |
| Signature analysis         | `Binwalk`   | Recognised JPEG structures identified      |
| File carving               | `Scalpel`   | DOC candidates recovered and validated     |
| Integrity verification     | MD5/SHA-256 | Recovered artefacts hashed                 |

---

# 🧠 15. Key Findings

### Finding 1 — JPEG structure can be identified from binary data

The JPEG SOI and EOI markers were directly identified within the hexadecimal representation:

```text
FF D8 → Start of Image
FF D9 → End of Image
```

### Finding 2 — Hexadecimal reconstruction preserved the evidence

The JPEG was converted to hexadecimal and reconstructed without modification.

The original and reconstructed files produced identical MD5 and SHA-256 hashes.

### Finding 3 — Deleted files remained recoverable

Although directory entries indicated deletion, the underlying FAT12 data allowed four files to be recovered:

```text
Billing_Letter.doc
confirmation.txt
letter1.txt
Regrets.doc
```

### Finding 4 — File-system metadata can assist recovery

The Sleuth Kit's `fls` output provided deleted-file metadata references that enabled targeted recovery using `icat`.

### Finding 5 — File carving requires validation

Scalpel produced ten DOC candidates, but only a subset represented meaningful Word-document content.

Duplicate, incomplete and over-carved results demonstrate why carved artefacts must be validated before being treated as confirmed evidence.

### Finding 6 — Different forensic tools provide complementary evidence

TSK and Scalpel did not simply produce identical outputs. Instead, their results demonstrated different approaches:

```text
The Sleuth Kit
     ↓
File-system-aware recovery
     ↓
Deleted directory entries
     ↓
Targeted extraction with icat

Scalpel
     ↓
Signature-based carving
     ↓
Raw-data candidates
     ↓
Validation required
```

---

# ⚖️ 16. Forensic Integrity

The following principles were applied throughout the exercise:

* Original evidence was preserved.
* Working copies and recovered artefacts were stored separately.
* Cryptographic hashes were calculated.
* Actual evidence-derived offsets and structures were used.
* Recovered artefacts were independently hashed.
* Tool output was retained for reproducibility.
* False positives and tool limitations were documented.
* Findings were based on observed evidence rather than assumptions.

---

# ⚠️ 17. Limitations

Several limitations were observed during the investigation.

### `mmls`

The standalone FAT12 image did not present a conventional partition table. Consequently, `mmls` did not provide a usable non-zero partition offset.

This was not treated as an error in the evidence. The file-system analysis demonstrated that the FAT12 file system began directly at sector 0.

### Binwalk

Binwalk did not identify additional recognised structures in the DD image.

This was not interpreted as proof that the image contained no recoverable artefacts, because TSK subsequently identified and recovered deleted files.

### Scalpel

Scalpel produced multiple DOC candidates, including duplicates and invalid/incomplete results. This demonstrated the limitations of signature-based carving when file boundaries or fragments are ambiguous.

---

# 🛠️ Tools

| Tool           | Purpose                               |
| -------------- | ------------------------------------- |
| **Kali Linux** | Forensic analysis environment         |
| `xxd`          | Hexadecimal inspection/reconstruction |
| `strings`      | Readable-string extraction            |
| `file`         | File-type identification              |
| `img_stat`     | Forensic image information            |
| `mmls`         | Partition analysis                    |
| `fsstat`       | File-system analysis                  |
| `fls`          | File and deleted-entry listing        |
| `icat`         | File recovery                         |
| `blkcat`       | Block-level examination               |
| `binwalk`      | Signature/embedded-content analysis   |
| `Scalpel`      | Signature-based file carving          |
| `7z`           | Archive extraction                    |
| `md5sum`       | MD5 integrity hashing                 |
| `sha256sum`    | SHA-256 integrity hashing             |

---

# 📁 Repository Structure

```text
Lab3_Data_Carving/
│
├── original_evidence/
├── working/
├── recovered/
├── hashes/
├── scalpel_output/
├── tsk_analysis/
└── xxd_analysis/
```

The repository intentionally separates:

* **Original evidence**
* **Working copies**
* **Recovered artefacts**
* **Hash records**
* **Scalpel output**
* **Sleuth Kit analysis**
* **XXD/hexadecimal analysis**

This separation makes the investigation easier to reproduce and review.

---

# 🎓 Academic Context

**Course:** SBT-DF202 — Computer and Digital Forensics
**Practical:** Lab 3 — Data Carving
**Environment:** Kali Linux
**Focus:** Digital forensics, file carving, deleted-file recovery and binary analysis

This repository documents an authorised educational laboratory exercise and is intended for **digital-forensics training and academic demonstration**.

---

## 👩🏽‍💻 Author

**Kafayat Omolara Animashawun, CISSP**

Cybersecurity | Threat Intelligence | Incident Response | Digital Forensics

---

## ⭐ Skills Demonstrated

```text
Digital Forensics
      │
      ├── Evidence Preservation
      ├── Cryptographic Hashing
      ├── Hexadecimal Analysis
      ├── File Signature Analysis
      ├── FAT12 File-System Analysis
      ├── Deleted File Recovery
      ├── File Carving
      ├── Embedded Data Analysis
      ├── Artefact Validation
      └── Forensic Documentation
```

---

> **Disclaimer:** This project contains laboratory evidence and artefacts supplied for an authorised academic exercise. It is not based on real-world investigative evidence.
