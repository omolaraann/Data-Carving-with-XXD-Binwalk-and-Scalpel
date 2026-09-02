## 1. Practical Overview

This laboratory demonstrates practical techniques for examining, identifying and recovering digital artefacts when normal file-system metadata may be missing, damaged or unreliable.

The exercise focuses on **data carving**, which involves identifying files from their underlying binary structures, such as file signatures, rather than relying exclusively on file-system metadata.

The laboratory covers:

* Hexadecimal analysis using `xxd`
* Identification of JPEG file signatures
* Hex dump creation and reconstruction
* Cryptographic hash verification
* String and metadata-like content analysis
* File-system and block-level analysis using The Sleuth Kit
* Embedded-file identification using Binwalk
* Partition analysis using `mmls`
* Deleted-file recovery using Scalpel
* Verification of recovered files using MD5 and SHA-256
* Documentation of forensic findings, screenshots, commands and limitations

All analysis was performed on the authorised laboratory evidence provided for the exercise. The original evidence files were preserved and were not intentionally modified.

---

# 2. Objectives

The objectives of this practical were to:

1. Explain why data carving is useful when file-system metadata is unavailable or unreliable.
2. Examine binary file content using `xxd`.
3. Identify JPEG Start of Image (SOI) and End of Image (EOI) signatures.
4. Create and reverse a hexadecimal dump.
5. Verify reconstructed content using cryptographic hashes.
6. Examine visible strings and metadata-like information within a JPEG file.
7. Use The Sleuth Kit to examine forensic image structures at inode, block and sector levels.
8. Identify embedded content using Binwalk.
9. Analyse the partition structure of a forensic USB image.
10. Recover deleted files using Scalpel.
11. Hash recovered evidence and document the results.
12. Maintain a repeatable and forensically defensible workflow.

---

# 3. Evidence and Working Directory

## 3.1 Authorised Evidence

The laboratory supplied the following evidence:

| Evidence            | Description                       | Purpose                            |
| ------------------- | --------------------------------- | ---------------------------------- |
| `Ch01InChap01.dd`   | Forensic DD image                 | Sleuth Kit / data-unit analysis    |
| `J_ub_law.jpg`      | JPEG sample                       | Hexadecimal and signature analysis |
| `120M.7z`           | Compressed USB forensic image     | Scalpel carving exercise           |
| `File_carving.docx` | Instructor demonstration material | Binwalk practice, if available     |

---

## 3.2 Working Directory

A dedicated working directory was created to separate laboratory evidence from analysis output.

Example structure:

```text
Lab3_Data_Carving/
├── evidence/
├── working/
├── hex/
├── binwalk/
├── scalpel/
├── recovered/
├── hashes/
├── screenshots/
└── report/
```

The original evidence was retained in the `evidence/` directory and analysis outputs were written to separate working/output directories.

---

# 4. Evidence Preservation and Initial Hashing

Before analysis, cryptographic hashes were calculated for the supplied evidence files.

### Commands

```bash
md5sum J_ub_law.jpg
sha256sum J_ub_law.jpg
```

For the DD image:

```bash
md5sum Ch01InChap01.dd
sha256sum Ch01InChap01.dd
```

For the extracted USB image:

```bash
md5sum <USB_IMAGE>
sha256sum <USB_IMAGE>
```

### Initial Hash Results

| Evidence          | MD5                 | SHA-256             |
| ----------------- | ------------------- | ------------------- |
| `J_ub_law.jpg`    | **[INSERT RESULT]** | **[INSERT RESULT]** |
| `Ch01InChap01.dd` | **[INSERT RESULT]** | **[INSERT RESULT]** |
| `<USB_IMAGE>`     | **[INSERT RESULT]** | **[INSERT RESULT]** |

These hashes provide reference values that can be used to detect unintended changes to the evidence during the investigation.

### Screenshot

![Initial Evidence Hashes](screenshots/01_initial_hashes.png)

---

# 5. Part A — JPEG File Signature and Hex Analysis

## 5.1 Examining the JPEG with xxd

The JPEG sample was examined using `xxd` to view its binary contents in hexadecimal format.

```bash
xxd J_ub_law.jpg | head
```

The beginning of the file was examined for the JPEG file signature.

JPEG files normally begin with the **Start of Image (SOI)** marker:

```text
FF D8
```

The SOI marker indicates the beginning of a JPEG file.

### Observed Header

```text
[INSERT YOUR xxd OUTPUT / RELEVANT BYTES]
```

### Screenshot

![JPEG Header Analysis](screenshots/02_jpeg_header_xxd.png)

---

## 5.2 Identifying the JPEG Footer

The end of the JPEG was examined to identify the **End of Image (EOI)** marker.

The JPEG EOI signature is:

```text
FF D9
```

Command used:

```bash
xxd J_ub_law.jpg | tail
```

### Observed Footer

```text
[INSERT YOUR OUTPUT]
```

The presence of `FF D9` at the end of the JPEG confirms the expected JPEG termination marker.

### Screenshot

![JPEG Footer Analysis](screenshots/03_jpeg_footer_xxd.png)

---

# 6. Creating a Plain Hexadecimal Dump

A plain hexadecimal representation of the JPEG was created using:

```bash
xxd -p J_ub_law.jpg > J_ub_law.hex
```

The resulting file contained the hexadecimal representation of the original JPEG data.

The generated dump was inspected to confirm that the binary content had been represented as hexadecimal data.

### Screenshot

![Plain Hex Dump](screenshots/04_plain_hex_dump.png)

---

# 7. Reconstructing the JPEG from the Hex Dump

The hexadecimal dump was converted back into binary JPEG data using the reverse-hexdump functionality of `xxd`.

```bash
xxd -r -p J_ub_law.hex J_ub_law_reconstructed.jpg
```

The reconstructed file was then examined and compared against the original file.

### File Comparison

```bash
cmp J_ub_law.jpg J_ub_law_reconstructed.jpg
```

If no output was returned, this indicated that the files were identical byte-for-byte.

### Screenshot

![JPEG Reconstruction](screenshots/05_jpeg_reconstruction.png)

---

# 8. Hash Verification of Reconstructed JPEG

The original and reconstructed JPEG files were hashed.

```bash
md5sum J_ub_law.jpg J_ub_law_reconstructed.jpg
```

```bash
sha256sum J_ub_law.jpg J_ub_law_reconstructed.jpg
```

### Results

| File               | MD5          | SHA-256      |
| ------------------ | ------------ | ------------ |
| Original JPEG      | **[INSERT]** | **[INSERT]** |
| Reconstructed JPEG | **[INSERT]** | **[INSERT]** |

### Finding

The reconstructed JPEG produced **[matching / non-matching]** MD5 and SHA-256 values when compared with the original.

**Interpretation:**
[If matching: The matching hashes demonstrate that the reconstructed file contains the same binary content as the original.]

[If non-matching: Explain the observed difference based on the actual result.]

### Screenshot

![Hash Comparison](screenshots/06_hash_comparison.png)

---

# 9. Part B — Strings and Metadata-Like Content Analysis

The JPEG was examined for readable strings that could potentially provide information about the source or creation of the file.

The following command was used:

```bash
strings J_ub_law.jpg
```

The output was reviewed for information relating to:

* Camera manufacturer
* Camera/software information
* File creation or modification information
* Names
* Dates
* Other readable metadata-like strings

### Search Examples

```bash
strings J_ub_law.jpg | grep -i camera
```

```bash
strings J_ub_law.jpg | grep -Ei "make|model|software|date|name"
```

### Findings

| Item searched          | Result                      |
| ---------------------- | --------------------------- |
| Camera manufacturer    | **[FOUND / NOT FOUND]**     |
| Camera model           | **[FOUND / NOT FOUND]**     |
| Software               | **[FOUND / NOT FOUND]**     |
| Name                   | **[FOUND / NOT FOUND]**     |
| Date                   | **[FOUND / NOT FOUND]**     |
| Other relevant strings | **[INSERT ACTUAL FINDING]** |

Only information actually observed in the command output was recorded.

### Screenshot

![Strings Analysis](screenshots/07_strings_analysis.png)

---

# 10. Part C — The Sleuth Kit Analysis

The supplied forensic image was examined using The Sleuth Kit (TSK).

The tools used included:

* `img_stat`
* `mmls`
* `fsstat`
* `fls`
* `icat`
* `blkcat`

---

## 10.1 img_stat

The image metadata was examined using:

```bash
img_stat Ch01InChap01.dd
```

### Findings

```text
[INSERT RELEVANT OUTPUT]
```

The output was reviewed to identify the image type, size and other available image-level information.

### Screenshot

![img\_stat](screenshots/08_img_stat.png)

---

# 11. Partition Analysis with mmls

The partition structure was examined using:

```bash
mmls Ch01InChap01.dd
```

### Observed Partition Structure

```text
[INSERT YOUR ACTUAL mmls OUTPUT]
```

The partition information was used to determine the appropriate offset for subsequent file-system analysis.

**Important:** The offset used in the analysis was based on the actual `mmls` output rather than assuming the example value supplied in the laboratory instructions.

### Identified Offset

**Partition start sector:** `[INSERT ACTUAL VALUE]`

### Screenshot

![mmls Partition Analysis](screenshots/09_mmls.png)

---

# 12. File-System Analysis with fsstat

The file-system information was examined using:

```bash
fsstat -o <OFFSET> Ch01InChap01.dd
```

### Findings

```text
[INSERT RELEVANT OUTPUT]
```

The output was reviewed to identify file-system characteristics and structures relevant to further investigation.

### Screenshot

![fsstat Analysis](screenshots/10_fsstat.png)

---

# 13. File Listing with fls

Files and directories within the forensic image were examined using:

```bash
fls -o <OFFSET> Ch01InChap01.dd
```

Deleted entries were specifically investigated where appropriate.

Example:

```bash
fls -d -o <OFFSET> Ch01InChap01.dd
```

### Findings

```text
[INSERT ACTUAL OUTPUT]
```

### Identified Inode / Metadata Address

**Inode / metadata address:** `[INSERT ACTUAL VALUE]`

### Screenshot

![fls Analysis](screenshots/11_fls.png)

---

# 14. Extracting File Content with icat

Where an appropriate metadata/inode address was identified, `icat` was used to extract the corresponding file content.

```bash
icat -o <OFFSET> Ch01InChap01.dd <INODE> > extracted_file
```

The extracted file was then examined and hashed.

### Result

**Extracted file:** `[INSERT NAME]`
**Inode:** `[INSERT VALUE]`
**File type:** `[INSERT TYPE]`

### Screenshot

![icat Extraction](screenshots/12_icat.png)

---

# 15. Block-Level Examination with blkcat

Where required, block-level data was examined using:

```bash
blkcat -o <OFFSET> Ch01InChap01.dd <BLOCK_NUMBER>
```

### Observed Block

**Block number:** `[INSERT ACTUAL VALUE]`

### Finding

```text
[INSERT RELEVANT OBSERVATION]
```

The block-level examination demonstrated how raw storage units can be inspected independently of normal file browsing.

### Screenshot

![blkcat Analysis](screenshots/13_blkcat.png)

---

# 16. Part D — Embedded File Discovery with Binwalk

Binwalk was used to identify known file signatures and potentially embedded content within the supplied evidence.

The initial scan was performed using:

```bash
binwalk <TARGET_FILE>
```

### Results

```text
[INSERT ACTUAL BINWALK OUTPUT]
```

The output was examined for recognised embedded file structures, compression formats or other identifiable signatures.

### Screenshot

![Binwalk Scan](screenshots/14_binwalk_scan.png)

---

# 17. Binwalk Extraction

Where embedded content was identified and extraction was appropriate, Binwalk extraction was performed using:

```bash
binwalk -e <TARGET_FILE>
```

The resulting extraction directory was reviewed.

### Extracted Content

```text
[INSERT ACTUAL FINDINGS]
```

If the instructor-provided `File_carving.docx` was unavailable, this was documented as a laboratory limitation rather than assuming its contents.

### Screenshot

![Binwalk Extraction](screenshots/15_binwalk_extraction.png)

---

# 18. Part E — USB Forensic Image Preparation

The supplied `120M.7z` archive was extracted to obtain the USB forensic image.

The archive was extracted using the appropriate extraction utility.

Example:

```bash
7z x 120M.7z
```

### Extracted Image

**Image filename:** `[INSERT ACTUAL IMAGE NAME]`

**Image size:** `[INSERT]`

### Screenshot

![USB Image Extraction](screenshots/16_usb_extraction.png)

---

# 19. Hashing the USB Image

The extracted USB image was hashed before analysis.

```bash
md5sum <USB_IMAGE>
```

```bash
sha256sum <USB_IMAGE>
```

### Results

| Image              | MD5          | SHA-256      |
| ------------------ | ------------ | ------------ |
| USB forensic image | **[INSERT]** | **[INSERT]** |

### Screenshot

![USB Image Hash](screenshots/17_usb_hash.png)

---

# 20. USB Partition Analysis

The USB image was examined using:

```bash
mmls <USB_IMAGE>
```

### Actual Partition Structure

```text
[INSERT YOUR ACTUAL OUTPUT]
```

The partition start sector was identified from the output.

**Confirmed partition offset:** `[INSERT VALUE]`

The value was determined from the evidence rather than copied directly from the laboratory example.

### Screenshot

![USB mmls](screenshots/18_usb_mmls.png)

---

# 21. Part F — Scalpel File Carving

Scalpel was configured to enable the required file types for carving, particularly JPEG/JPG files.

The configuration file was reviewed:

```bash
sudo nano /etc/scalpel/scalpel.conf
```

The required JPEG signatures were enabled according to the laboratory instructions.

### Configuration

```text
[INSERT RELEVANT ENABLED SIGNATURES]
```

### Screenshot

![Scalpel Configuration](screenshots/19_scalpel_configuration.png)

---

# 22. Running Scalpel

A dedicated output directory was created for recovered evidence.

```bash
mkdir -p scalpel_output
```

Scalpel was then run against the forensic USB image.

```bash
sudo scalpel <USB_IMAGE> -o scalpel_output
```

### Result

```text
[INSERT ACTUAL SCALPEL OUTPUT]
```

The carving process was allowed to operate against the forensic image while preserving the original evidence.

### Screenshot

![Scalpel Execution](screenshots/20_scalpel_execution.png)

---

# 23. Recovered Files

The Scalpel output directory was examined to identify recovered artefacts.

```bash
find scalpel_output -type f
```

The recovered files were reviewed and categorised according to their identified file types.

### Recovery Summary

| # | Recovered File | File Type  |       Size | MD5        | SHA-256    |
| - | -------------- | ---------- | ---------: | ---------- | ---------- |
| 1 | `[INSERT]`     | JPEG       | `[INSERT]` | `[INSERT]` | `[INSERT]` |
| 2 | `[INSERT]`     | JPEG       | `[INSERT]` | `[INSERT]` | `[INSERT]` |
| 3 | `[INSERT]`     | `[INSERT]` | `[INSERT]` | `[INSERT]` | `[INSERT]` |

At least two recovered JPEG files were examined where available.

---

# 24. Recovered JPEG — Evidence 1

**Filename:** `[INSERT]`

**File type:**

```bash
file <RECOVERED_FILE>
```

**Size:**

```bash
ls -lh <RECOVERED_FILE>
```

**MD5:**

```bash
md5sum <RECOVERED_FILE>
```

**SHA-256:**

```bash
sha256sum <RECOVERED_FILE>
```

### Findings

[Describe what was actually observed when the recovered JPEG was examined.]

### Screenshot

![Recovered JPEG 1](screenshots/21_recovered_jpeg_01.png)

---

# 25. Recovered JPEG — Evidence 2

**Filename:** `[INSERT]`

**File type:**

```bash
file <RECOVERED_FILE>
```

**Size:**

```bash
ls -lh <RECOVERED_FILE>
```

**MD5:**

```bash
md5sum <RECOVERED_FILE>
```

**SHA-256:**

```bash
sha256sum <RECOVERED_FILE>
```

### Findings

[Describe what was actually observed.]

### Screenshot

![Recovered JPEG 2](screenshots/22_recovered_jpeg_02.png)

---

# 26. Scalpel audit.txt

Scalpel generates an audit file documenting aspects of the carving operation.

The audit file was reviewed:

```bash
cat scalpel_output/audit.txt
```

or, depending on the generated directory structure:

```bash
find scalpel_output -name audit.txt -exec cat {} \;
```

### Audit Findings

```text
[INSERT RELEVANT AUDIT OUTPUT]
```

The audit information was retained as part of the laboratory evidence and documentation.

### Screenshot

![Scalpel Audit](screenshots/23_scalpel_audit.png)

---

# 27. Evidence Hash Verification

Cryptographic hashes were calculated for recovered files to provide unique identifiers for the recovered artefacts.

Example:

```bash
md5sum scalpel_output/*/*
```

```bash
sha256sum scalpel_output/*/*
```

### Hash Table

| Artefact             | MD5        | SHA-256    |
| -------------------- | ---------- | ---------- |
| `[Recovered file 1]` | `[INSERT]` | `[INSERT]` |
| `[Recovered file 2]` | `[INSERT]` | `[INSERT]` |
| `[Recovered file 3]` | `[INSERT]` | `[INSERT]` |

Hash values provide a means of documenting the recovered artefacts and detecting subsequent changes.

---

# 28. Key Findings

The practical demonstrated that files can be identified and recovered based on their underlying binary structures even when conventional file-system information may not be sufficient.

### Key findings included:

* The JPEG sample was examined at hexadecimal level using `xxd`.
* The JPEG SOI signature `FF D8` and EOI signature `FF D9` were identified.
* A hexadecimal dump was successfully converted back into binary JPEG data.
* The reconstructed JPEG was compared against the original using cryptographic hashes.
* Strings and metadata-like information were examined without assuming information that was not present.
* The Sleuth Kit was used to examine forensic image structures and identify relevant data units.
* `mmls` was used to determine the actual partition offset from the forensic image.
* Binwalk was used to identify recognised file signatures and embedded structures.
* Scalpel was used to carve recoverable files from the USB forensic image.
* Recovered artefacts were individually examined and hashed.
* Scalpel's `audit.txt` was reviewed as part of the carving documentation.

---

# 29. Forensic Integrity and Handling

The following forensic handling principles were applied throughout the practical:

1. Original evidence files were preserved.
2. Analysis was performed on the authorised laboratory evidence.
3. Evidence was hashed before analysis.
4. Analysis outputs were stored separately from the original evidence.
5. Actual offsets, inode values and block numbers were obtained from command output.
6. No placeholder values were used in the final findings.
7. Recovered files were hashed to provide verifiable identifiers.
8. Errors and limitations were documented rather than concealed.
9. Commands used during analysis were recorded to support repeatability.

---

# 30. Limitations

The following limitations were identified during the exercise:

* [INSERT ANY COMMAND ERRORS OR TOOL LIMITATIONS]
* [INSERT IF File_carving.docx WAS NOT AVAILABLE]
* [INSERT ANY FILES THAT COULD NOT BE RECOVERED]
* [INSERT ANY CORRUPTED RECOVERED FILES]
* [INSERT ANY OTHER OBSERVED LIMITATION]

Where an expected artefact or metadata value was not present, it was recorded as **"not found"** rather than inferred.

---

# 31. Conclusion

This laboratory provided practical experience in data carving and low-level forensic examination of digital storage evidence.

The use of `xxd` demonstrated how binary file structures and signatures can be examined directly. JPEG SOI and EOI markers provided an example of how known file signatures can assist with identifying file boundaries and reconstructing binary content.

The Sleuth Kit provided a method for examining forensic images at the image, partition, file-system, metadata and block levels. Binwalk demonstrated the identification of embedded or recognised file structures, while Scalpel demonstrated signature-based recovery of files from a forensic USB image.

The exercise reinforced the importance of preserving original evidence, calculating cryptographic hashes, using actual evidence-derived values, maintaining an auditable workflow and clearly documenting both successful findings and limitations.

---

# 32. Tools Used

| Tool        | Purpose                                   |
| ----------- | ----------------------------------------- |
| `xxd`       | Hexadecimal inspection and reconstruction |
| `strings`   | Extraction of readable strings            |
| `img_stat`  | Forensic image information                |
| `mmls`      | Partition analysis                        |
| `fsstat`    | File-system analysis                      |
| `fls`       | File and deleted-file listing             |
| `icat`      | File extraction                           |
| `blkcat`    | Block-level examination                   |
| `binwalk`   | Embedded file/signature identification    |
| `Scalpel`   | File carving and recovery                 |
| `md5sum`    | MD5 hashing                               |
| `sha256sum` | SHA-256 hashing                           |
| `7z`        | Extraction of the supplied archive        |

---

# 33. Repository Structure

```text
Lab3-Data-Carving-XXD-Binwalk-Scalpel/
│
├── README.md
│
├── report/
│   └── Lab3_Forensic_Report.pdf
│
├── screenshots/
│   ├── 01_initial_hashes.png
│   ├── 02_jpeg_header_xxd.png
│   ├── 03_jpeg_footer_xxd.png
│   ├── 04_plain_hex_dump.png
│   ├── 05_jpeg_reconstruction.png
│   ├── 06_hash_comparison.png
│   ├── 07_strings_analysis.png
│   ├── 08_img_stat.png
│   ├── 09_mmls.png
│   ├── 10_fsstat.png
│   ├── 11_fls.png
│   ├── 12_icat.png
│   ├── 13_blkcat.png
│   ├── 14_binwalk_scan.png
│   ├── 15_binwalk_extraction.png
│   ├── 16_usb_extraction.png
│   ├── 17_usb_hash.png
│   ├── 18_usb_mmls.png
│   ├── 19_scalpel_configuration.png
│   ├── 20_scalpel_execution.png
│   ├── 21_recovered_jpeg_01.png
│   ├── 22_recovered_jpeg_02.png
│   └── 23_scalpel_audit.png
│
├── hashes/
│   └── hashes.txt
│
└── evidence/
    └── [Evidence files / references as permitted]
```

> **Evidence note:** Large forensic images should not be uploaded to GitHub unless specifically required by the course. The README, report, screenshots, hash records and relevant small artefacts should be sufficient unless the instructor explicitly requires the original evidence image.

---

## 34. Academic and Forensic Integrity Statement

This practical was completed using the authorised laboratory evidence supplied for SBT-DF202. The analysis was conducted for educational and forensic-training purposes. Findings recorded in this repository are based on observed command output and recovered artefacts from the supplied evidence.

No conclusions were made from information that was not directly observed during the analysis.
