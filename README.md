# Data Mart Smartphones (Vietnam)

This project cleans smartphone data, normalizes fields for visualization, and builds a star-schema data mart (dimensions + fact) for downstream BI tools like Power BI.

## Repository contents
- main.ipynb — end-to-end ETL + feature engineering + data mart export (with documentation inside).
- smartphones_cleaned_v6.csv — source data.
- smartphones_vietnamese.csv — source with Vietnamese column names and missing values imputed.
- smartphones_vietnamese_viz.csv — normalized dataset for visualization (clean text, numeric types, PPI, bins, fast-charging split, price normalized to VND).
- output_dim_fact/
  - dim_thuong_hieu.csv, dim_cpu.csv, dim_man_hinh.csv, dim_bo_nho.csv, dim_camera.csv, dim_os.csv
  - fact_smartphone.csv

## What the notebook does
1) Rename columns EN → VI (adds resolution height to compute PPI).
2) Impute missing values by simple rules (mode/median/mean by CPU brand, defaults for categorical).
3) Normalize for visualization:
   - Clean categorical text lightly (brand, model, CPU brand, OS).
   - Cast numeric fields safely.
   - Fast-charging: split into two fields
     - Có_sạc_nhanh_(0_1): 0/1/NA
     - Công_suất_sạc_nhanh_(W): watt, 0 when unsupported
   - Memory card support: derive Có_hỗ_trợ_thẻ_nhớ_(0_1).
   - Compute PPI from resolution and screen size.
   - Price normalization to VND (robust token-based parsing: triệu/tr/k/usd/$, mixed separators; scale-up small malformed values; rollback common USD misparses). Adds Giá (triệu VND) and price bins.
   - Ensure “Giá (VNĐ)” is saved as nullable integer (Int64) for clean numeric handling with missing values.
4) Build Data Mart (Star Schema) with stable surrogate keys (sorted before assigning IDs):
   - dim_thuong_hieu, dim_cpu, dim_man_hinh (with ppi), dim_bo_nho (with Có_hỗ_trợ_thẻ_nhớ_(0_1)), dim_camera, dim_os
   - fact_smartphone: foreign keys + metrics (Giá (VNĐ), Đánh giá) + segment attributes and fast-charging fields

## How to run
Prerequisites:
- Python 3.9+ and pandas installed.

Steps:
1) Open main.ipynb in Jupyter or VS Code.
2) Run cells sequentially from the survey step through data mart export.
   - To rebuild everything after logic changes: re-run the normalization cell then the export cell.
3) Outputs will appear in `smartphones_vietnamese_viz.csv` and `output_dim_fact/`.

Optional quick setup (VS Code Terminal):
- Install pandas
  - pip install pandas

## Notes
- CSVs are written with UTF-8 (BOM) for Vietnamese compatibility in Excel/BI.
- Nullable integer Int64 is used so missing prices are preserved as <NA> instead of 0.
- Surrogate key stability is improved by sorting; for absolute stability across changing data, consider attribute hashing.

## Power BI (suggested)
- Pages: Market overview, Brand trends, Performance vs. price, Feature insights.
- Slicers: Brand, OS, Price bin, RAM/ROM, Screen group, Fast charging.
- Measures: Average/Median price, Share by brand, Price elasticity proxies.

## Structure
- Input → Rename/Impute → Normalize (PPI, price to VND, bins, fast-charging split) → Dimensions/Fact → CSV exports.

