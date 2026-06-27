# Data Cleaning & Business Rules Implementation Log

## 1. Project Scope & Baseline
- **Target Dataset:** `raw_orders.xlsx`
- **Output Cleaned File:** `data/cleaned_orders.xlsx`
- **Initial Row Count:** 932 raw entries
- **Final Cleaned Volume:** 912 operational records (after duplicate removal)

---

## 2. Records Removed (Task 9 Requirement)
- **Exact Row Duplicates:** **20 records** were identified as exact redundant system duplicates and permanently removed from the dataset to prevent double-counting revenue.
- **Conflicting Order IDs:** **24 rows** shared duplicate `order_id` values but contained differing transactional data. Following data integrity guidelines, these were *not* deleted; instead, they were preserved and flagged using an internal spreadsheet tracking formula: `=IF(COUNTIF(A:A, A2)>1, "Flagged: Conflicting Order ID", "Unique")`.

---

## 3. Business Rules Applied & Flagged Records (Task 5 & Task 9 Combined)

This section details the explicit handling logic applied to individual column exceptions:

| Rule Domain | Condition Found | Applied Action / Operational Resolution (Task 5) | Flagging & Row Impacts (Task 9) |
| :--- | :--- | :--- | :--- |
| **Missing Region** | 26 Blank cells | Imputed empty cells with structural string fallback `"Unknown"`. | Flagged in quality report. |
| **Missing Ship Mode** | 22 Blank cells | Imputed empty cells with structural string fallback `"Unknown"`. | Flagged in quality report. |
| **Missing Discount** | 18 Blank cells | Treated as `0` after verifying that corresponding sales matched full base-rate pricing. | Processed as valid. |
| **Negative Discount** | 16 Rows (< 0) | Retained value for audit tracking; marked as an anomaly. | Flagged as `Invalid: Negative Discount` in `data_quality_flag`. |
| **High Discount** | 15 Rows (> 50%) | Isolated extreme promotional entries (e.g., 70%, 85%) exceeding business parameters. | Flagged as `Invalid: Exceeds Allowed Discount Threshold` in `data_quality_flag`. |
| **Order Status** | "Cancelled" | Excluded from the main dashboard metrics so it doesn't inflate final completed sales summaries. | Flagged as `Excluded: Non-Completed Transaction` in `data_quality_flag`. |
| **Payment Status** | "Failed" | Excluded from the main dashboard metrics so it doesn't inflate final completed sales summaries. | Flagged as `Excluded: Non-Completed Transaction` in `data_quality_flag`. |
| **Fulfillment Timeline**| Ship Date < Order Date | Caught 22 backward timelines violating physical shipping rules. | Flagged as `Invalid Shipping Record` in `data_quality_flag`. |

---

## 4. Analytical Assumptions Made (Task 9 Requirement)
- **Zero Promotional Adjustments:** Blank rows within the discount field indicate that no coupon or contract adjustment was applied, meaning the customer paid full unit price.
- **Formula Precedence:** Financial values (`calculated_sales`, `calculated_profit`) calculated programmatically from baseline quantities and unit rates take priority over pre-exported summary strings in the raw file.

---

## 5. Limitations of the Cleaning Process (Task 9 Requirement)
- **Root-Cause Obscurity:** The script can isolate and flag reverse shipping timelines or extreme discounts, but it cannot determine if they were caused by manual entry errors or a system sync bug.
- **Irrecoverable Data:** Flagging missing regions and shipping fields as `"Unknown"` keeps the data usable for equations, but it cannot recover the true missing logistical details without access to the original shipping manifest.
