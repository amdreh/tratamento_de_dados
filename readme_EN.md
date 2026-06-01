# Data Cleansing and Standardization in Google Sheets

## Description

This project consisted of cleaning, organizing, standardizing, and consolidating a contact database distributed across two tabs of a Google Sheets spreadsheet:

- `BASE_BRUTA` (Raw Database)
- `CONTATOS NOVOS` (New Contacts)

The goal was to transform unorganized, inconsistent, and partially filled data into a structured, clean database ready for queries, filtering, and integration with analytical tools.

---

## Initial Diagnosis

An initial analysis of the spreadsheet structure was conducted.

### Analysis Result

Both tabs contained relevant information and were retained in the consolidation process.

In the `CONTATOS NOVOS` tab, column `F` (missing a header) contained two records composed strictly of operational notes with no structural value, which were removed.

---

# Technologies Used

- **Google Sheets**
- **Google Apps Script (JavaScript)**
- **Regex (Regular Expressions)**
- **Native Google Sheets functions**

---

# Processing the `CONTATOS NOVOS` Tab

## 1. Correcting accented emails

The `telefone/email` column contained:

- Mixed phone numbers and emails
- Data entry errors featuring accented characters in emails

For normalization, the following function was created in Apps Script:

function REMOVER_ACENTOS(texto) {
  return texto.normalize("NFD").replace(/[\u0300-\u036f]/g, "");
}

### How it works

- `.normalize("NFD")` decomposes accented characters
- `.replace(...)` removes diacritics
- `return` sends the result back to the spreadsheet

---

## 2. Extracting phone numbers

=REGEXEXTRACT(C5;"(\(?\d{2}\)?\s?9?\d{4,5}-?\d{4})")

The expression identifies phone numbers in different formats, including:

- (21)99999-9999
- 21999999999
- 21 99999-9999

---

## 3. Extracting emails

=REGEXEXTRACT(C2;"[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}")

---

## 4. Error handling

Unmatched fields generated #N/A errors.

To avoid visual clutter:

=IFERROR(REGEXEXTRACT(C2;"[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}");"")

=IFERROR(REGEXEXTRACT(C5;"(\(?\d{2}\)?\s?9?\d{4,5}-?\d{4})");"")

---

## Result

The original `telefone/email` column was split into:

- Telefone (Phone)
- Email

After validation, the original column was removed.

---

# Processing the `BASE_BRUTA` Tab

## Identified Issue

The `dados` (data) column concentrated multiple types of information:

- Company name
- Contact name
- Phone number
- Date
- Email
- Notes

Selective extraction was required.

---

## 1. Removing phone numbers and dates

=IFERROR(REGEXREPLACE(REGEXREPLACE(A4; "\(?\d{2}\)?\s?9?\d{4}-?\d{4}|\d{10,11}"; ""); "\d{2,4}[/-]\d{2}[/-]\d{2,4}"; ""); A4)

---

## 2. Extracting phone numbers

=IFERROR(REGEXEXTRACT(A4; "(\(?\d{2}\)?\s?9?\d{4}-?\d{4}|\d{10,11})"); "")

---

## 3. Extracting dates

=IFERROR(REGEXEXTRACT(A4; "(\d{2,4}[/-]\d{2}[/-]\d{2,4})"); "")

---

## 4. Extracting emails

=IFERROR(REGEXEXTRACT(A4; "[^[:space:]]+@[^[:space:]/]+"); "")

---

# Data Validation

## Date validation

=IF(D4=""; "Empty"; IF(AND(REGEXMATCH(D4; "[/-]"); LEN(D4)>=8; LEN(D4)<=10); "OK"; "Review Date"))

---

## Phone number validation

=IF(C4=""; "Empty"; IF(AND(LEN(REGEXREPLACE(C4; "\D"; ""))>=10; LEN(REGEXREPLACE(C4; "\D"; ""))<=11); "OK"; "Review Phone"))

After verification, the formulas were replaced with static values.

---

# Text Cleaning

## Removing spaces, emails, and residual separators

=TRIM(REGEXREPLACE(REGEXREPLACE(B4; "\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b|\s*/\s*TI"; ""); "\s*[-/]\s*$"; ""))

---

## Standardizing separators

=TRIM(REGEXREPLACE(C4; "\s*/\s*/\s*|\s*/\s*"; " - "))

---

## Complementary extraction

=IFERROR(TRIM(REGEXEXTRACT(D4; "-\s*(.+)$")); "")

---

# Manual Processing

Following automation, a manual review was performed to:

- Move misclassified operational notes
- Fix record positioning
- Relocate extracted information into the correct columns

---

# Processing the `Contato` Column

The following were removed:

- Operational expressions
- Prefixes like `Falar com` (Speak with)

Names found in the `extras` column were manually relocated.

---

# Corrections in the `Extras` Column

The following data types were extracted and relocated:

- Names -> `Contato` column
- Emails -> `Email` column
- Phone numbers -> `Telefone` column
- Flags like `sem email` (no email)

The `REMOVER_ACENTOS` function was also applied here.

---

# Phone Number Standardization

## Removing non-numeric characters

=REGEXREPLACE(TO_TEXT(D2); "\D"; "")

---

## Final formatting

=IF(LEN(D3)=10; 
  REGEXREPLACE(TO_TEXT(D3); "(\d{2})(\d{4})(\d{4})"; "($1) $2-$3"); 
  REGEXREPLACE(TO_TEXT(D3); "(\d{2})(\d{5})(\d{4})"; "($1) $2-$3")
)

Standardized formats:

- (21) 9999-9999
- (21) 99999-9999

---

# Removing Duplicates

Process:

Data -> Data cleanup -> Remove duplicates

## Result

### BASE_BRUTA
- 23 rows removed
- 182 rows retained

### CONTATOS NOVOS
No duplicates found

---

# Date Standardization

The following steps were executed:

- Removal of invalid text strings
- Replacement of hyphens with forward slashes
- Application of native date formatting

---

# Final Consolidation

The structural layout of both tabs was aligned:

- Identical column names
- Identical ordering
- Identical formatting standards

After alignment:

- Records from `CONTATOS NOVOS` were appended to `BASE_BRUTA`
- A final duplicate check was executed
- No additional duplicates were found

---

# Final Outcome

The consolidated database is now:

- Structured
- Standardized
- Deduplicated
- Validated
- Ready for filters and analysis

---

## Skills Applied

- Data Cleaning
- Regex
- Exception Handling
- Google Apps Script
- Text Normalization
- Data Standardization
- Deduplication
- Automated Validation
- Rules-Driven Manual Review
