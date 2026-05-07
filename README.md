# Vendor Reconciliation System — ERP × Excel

> A live-connected, dynamic reconciliation model built entirely in Microsoft Excel,  
> pulling real-time data from the database to track and audit high-volume vendor bills.

\---

## The Problem

Every month, reconciling vendor invoices against warehouse receipts was a fully manual process:

* Vendor invoices in ERP sometimes covered **multiple Purchase Orders in a single bill**
* The PO number appeared at the invoice header level — not at the individual line level
* This made it impossible to accurately match each invoiced quantity to its corresponding PO and warehouse receipt
* The result: risk of **double-counting**, undetected overbilling, and hours spent on manual cross-checking

\---

## The Solution

A dynamic Excel data model with a **live Power Query connection to Database** — replacing the manual process with a one-click refresh report.

\---

## How It Was Built

### Step 1 — Connect to ERP via Power Query (Live Connection)

Instead of exporting files from ERP manually, Power Query connects directly to the Database and pulls live data from three core models:

|Model|What It Contains|
|-|-|
|`account.move.line`|Every invoice line — quantity, price, product, and label|
|`stock.picking` / receipts|Warehouse receiving records per PO|
|`purchase.order`|Master PO list — vendor, date, total amount|

The connection is set to refresh on demand, so the report always reflects the latest ERP data.

\---

### Step 2 — Extract PO Number at the Line Level

The key challenge: ERP stores the PO reference in the invoice **header**, not on each line.

The fix was extracting the PO number from the **Label field** of each `account.move.line` using a Power Query text transformation. This made it possible to link every single invoiced line back to its exact purchase order — the foundation of accurate reconciliation.

\---

### Step 3 — Handle Returns Automatically

Vendor credit notes and return bills were identified automatically. Their quantities and amounts are multiplied by **−1**, so they offset the original invoices correctly in all totals and variances — no manual adjustment needed.

\---

### Step 4 — Build the Star Schema Data Model

Inside Excel's Data Model (Power Pivot), four tables are connected using **Purchase Order Number as the central key**:

```
                    ┌─────────────────┐
                    │  Purchase Orders│  ← Dimension (center)
                    │   (PO Number)   │
                    └────────┬────────┘
           ┌─────────────────┼─────────────────┐
           ▼                 ▼                 ▼
   ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐
   │ Vendor Bills │  │  Inventory   │  │ Vendor Statement │
   │    (BILL)    │  │  Receipts    │  │  (Supplier Data) │
   └──────────────┘  └──────────────┘  └──────────────────┘
```

This structure allows any report or pivot table to compare data across all three sources simultaneously, filtered by PO.

\---

### Step 5 — DAX Measures for Dynamic Analysis

Custom DAX measures were written to power the reconciliation logic:

|Measure|Purpose|
|-|-|
|`ERP Qty`|Total invoiced quantity recorded in ERP|
|`Vendor Qty`|Total quantity per the supplier's own statement|
|`Variance Qty`|ERP Qty − Vendor Qty (flags over/under billing)|
|`ERP Amount`|Total invoiced value in ERP|
|`Vendor Amount`|Total value per supplier statement|
|`Variance Amount`|Financial difference per PO|
|`Need\\\_to\\\_Bill`|Received quantity not yet invoiced|

\---

### Step 6 — Reconciliation Reports

Three report sheets are driven by the data model:

**Summary\_Report** — one row per PO, showing all variances side by side. The go-to view for the monthly close.

**DTL\_Report** — receipt-level breakdown, showing every warehouse reference linked to its PO.

**Variance Sheets (per PO)** — deep-dive matching: each system bill line is placed side-by-side with the corresponding vendor statement line, with a `Link\\\_ID` column to manually flag matched rows.

\---

## Results

Metrics:


* Purchase Orders tracked
* Total volume reconciled
* Total value (ERP)
* Total value (Vendor)
* Discrepancies identified
* Largest single variance
* Critical overbilling flagged
* Month-end reconciliation time from days → one refresh


\---

## Tools \& Skills

|Category|Details|
|-|-|
|ERP Integration|ERP · Power Query Live Connection |
|Data Engineering|M Language · ETL Pipeline · Data Cleansing · Text transformations|
|Data Modeling|Star Schema · Power Pivot · Table Relationships|
|DAX|Custom measures · Variance logic · Conditional aggregation|
|Financial Analysis|Vendor Reconciliation · AP Auditing · Variance Analysis|

\---

## File Structure

```
📁 Vendor\\\_Reconciliation.xlsx
├── 📊 Sum\\\_Report         — Monthly summary: all POs, all variances
├── 📊 DTL\\\_Report         — Receipt-level detail per PO
├── 📊 Variance Qty -P0-     — Deep-dive: ERP vs Vendor per line
├── 📊 Vendor Statement      — Vendor's own statement (imported)
├── 📊 Inventory             — Warehouse receipts from database
├── 📊 BILL                  — Vendor invoices from database
└── 📊 PO                    — Purchase order master list
```


\---

## Author

Accountant · Data Analyst  
Building bridges between ERP systems and financial reporting using Excel, Power Query, and DAX.

