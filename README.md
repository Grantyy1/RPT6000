# RPT6000

## Introduction
This COBOL program generates a Year-To-Date Sales Report that compares each customer's sales figures from the current year against the previous year. It calculates the change in dollar amount and the percentage change between the two periods. The report is organised using a control break structure, producing subtotals at the sales representative level and the branch level, as well as a grand total across all branches.

RPT6000 builds on RPT5000 by adding a sales representative name lookup. Instead of showing only a numeric salesrep code, the program now reads a separate SALESREP master file at startup, loads the names into an indexed table in working storage, and uses a SEARCH statement to look up and display the salesrep name on each detail line. If a customer references a salesrep number that does not exist in the SALESREP file, the program prints "UNKNOWN" in place of the name so the report still balances cleanly.

The report outputs the following information for each customer:

1. The Branch Number
2. The Sales Rep Number
3. The Sales Rep Name (looked up from the SALESREP file)
4. The Customer Number
5. The Customer Name
6. Sales Figure This Year (YTD)
7. Sales Figure Last Year (YTD)
8. Change in Amount
9. Change in Percentage

---

## 📑 Table of Contents
- [📌 What does it do?](#what-does-it-do)
- [🖥️ Output Code Example](#output-of-the-code-running)
- [🛠️ COBOL Concepts used](#cobol-concepts-covered-in-this-assignment)
- [👥 Authors](#authors)

---

## What does it do?
For each run, the program will:

1. Open the CUSTMAST and SALESREP input files and the RPT6000 output file.
2. Initialise the SALESREP table in working storage and load it from the SALESREP file using paragraph 200-LOAD-SALESREP-TABLE, which reads each record and stores the salesrep number and name into the next available table entry referenced by an index.
3. Read customer master records from the CUSTMAST input file one at a time.
4. For each customer record, use a SEARCH statement to look up the salesrep name in the loaded table. If no match is found, "UNKNOWN" is moved to the salesrep name field on the detail line.
5. Calculate the sales difference between this year and last year for each customer.
6. Calculate the percentage change. If last year's sales were zero, the change percent column shows "N/A" instead of dividing by zero, and if the calculation overflows the field it shows "OVRFLW" instead. This is handled using a REDEFINES clause that lets the same six bytes hold either an edited numeric value or a literal string.
7. Print a detail line for each customer showing branch, salesrep number, salesrep name, customer number, customer name, sales figures, and the calculated changes.
8. Print a salesrep subtotal line each time the sales representative number changes within a branch.
9. Print a branch subtotal line each time the branch number changes.
10. Print a grand total line at the end of the report summarising all branches.
11. Include report headings with the current date, time, and page number on each new page, with line counting handled in 350-WRITE-REPORT-LINE so page breaks trigger automatically.
12. Close all files and stop.

The program uses an EVALUATE TRUE structure in 300-PREPARE-SALES-LINES to drive the control break logic, deciding on each iteration whether to print a customer line, a salesrep total, a branch total, or all three depending on what changed since the previous record.

---

## Output of the code running
```
DATE:  04/07/2026                          YEAR-TO-DATE SALES REPORT                          PAGE:    1
TIME:  21:24                                                                                  RPT6000

                                                      SALES        SALES         CHANGE     CHANGE
BRANCH   SALESREP             CUSTOMER              THIS YTD      LAST YTD       AMOUNT     PERCENT
------ ------------- --------------------------   -----------   -----------    -----------  -------

  12   12 AJONES     11111 INFORMATION BUILDERS       1,234.56      1,111.11        123.45    +11.1
                     12345 CAREER TRAINING CTR       12,345.67     22,222.22      9,876.55-   -44.4
                                    SALESREP TOTAL  $13,580.23    $23,333.33     $9,753.10-   -41.8*
                                      BRANCH TOTAL  $13,580.23    $23,333.33     $9,753.10-   -41.8**

  22   10 UNKNOWN    22222 HOMELITE TEXTRON CO       34,545.00          0.00     34,545.00     N/A
                                    SALESREP TOTAL  $34,545.00         $0.00    $34,545.00     N/A *
       14 KBAKER     34567 NEAS MEMBER BENEFITS         111.11          0.00        111.11     N/A
                     55555 PILOT LIFE INS. CO.       10,000.00      1,000.00      9,000.00   +900.0
                                    SALESREP TOTAL  $10,111.11     $1,000.00     $9,111.11   +911.1*
                                      BRANCH TOTAL  $44,656.11     $1,000.00    $43,656.11   OVRFLW**

  34   10 UNKNOWN    00111 DAUPHIN DEPOSIT BANK      14,099.00     19,930.00      5,831.00-   -29.3
                     54321 AIRCRAFT OWNERS ASSC       5,426.12     40,420.00     34,993.88-   -86.6
                                    SALESREP TOTAL  $19,525.12    $60,350.00    $40,824.88-   -67.6*
       17 STRACKER   33333 NORFOLK CORP               6,396.35      4,462.88      1,933.47    +43.3
                                    SALESREP TOTAL   $6,396.35     $4,462.88     $1,933.47    +43.3*
                                      BRANCH TOTAL  $25,921.47    $64,812.88    $38,891.41-   -60.0**

  47   11 TSMITH     12121 GENERAL SERVICES CO.      11,444.00     11,059.56        384.44     +3.5
                     24680 INFO MANAGEMENT CO.       17,481.45     11,892.47      5,588.98    +47.0
                                    SALESREP TOTAL  $28,925.45    $22,952.03     $5,973.42    +26.0*
       21 FFRANKLIN  99999 DOLLAR SAVINGS BANK        5,059.00      4,621.95        437.05     +9.5
                     76543 NATL MUSIC CORP.           2,383.46      4,435.26      2,051.80-   -46.3
                                    SALESREP TOTAL   $7,442.46     $9,057.21     $1,614.75-   -17.8*
                                      BRANCH TOTAL  $36,367.91    $32,009.24     $4,358.67    +13.6**

                                       GRAND TOTAL $120,525.72   $121,155.45       $629.73-    -0.5***
```

---

## COBOL Concepts covered in this assignment

This program builds on the techniques from RPT5000 and adds several new concepts from chapters 6, 10, and 11.

**Carried over from RPT5000:**
- Program header level documentation
- File handling with fixed-length sequential records
- Defining working storage data items including switches, control fields, and accumulators
- Two-level control break processing (salesrep within branch)
- COMPUTE with ROUNDED and ON SIZE ERROR
- EVALUATE TRUE and nested IF logic for control break decisions
- Accumulating subtotals at the salesrep, branch, and grand total levels
- Page overflow handling with heading reprints
- Formatting numeric output with edited picture clauses

**New in RPT6000:**
- Defining a table in working storage with the OCCURS clause and INDEXED BY to declare an index
- Loading a table from a separate input file at program startup rather than hard-coding values
- Using a SEARCH statement with WHEN and AT END to look up an entry by key
- Using the SET statement to position an index before searching
- REDEFINES clause to let one storage area hold either an edited numeric field (PIC +++9.9) or a six-byte literal string (PIC X(6)), so the same column can display "+11.1", "N/A", or "OVRFLW" depending on the situation
- Floating dollar sign edited fields (PIC $$$,$$9.99-) on subtotal and total lines
- PACKED-DECIMAL declared at the group level so all subordinate items inherit the usage
- Handling missing reference data gracefully by printing "UNKNOWN" instead of failing

---

## Authors

👨‍💻 **Grant Peverett**

- **GitHub Profile**: [Grantyy1](https://github.com/Grantyy1)
- **Email**: grpeve01@wsc.edu
