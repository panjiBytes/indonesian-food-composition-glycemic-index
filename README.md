# Indonesian Food Composition and Glycemic Index Hybrid Dataset

[![Dataset Status](https://img.shields.io/badge/Dataset-Validated-success?style=flat-square)](#)
[![Data Format](https://img.shields.io/badge/Format-CSV%20(Semicolon%20Delimited)-blue?style=flat-square)](#)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg?style=flat-square)](https://opensource.org/licenses/MIT)

## Project Overview
This repository hosts a curated **Indonesian Food Composition and Glycemic Index Hybrid Dataset**. The database is designed to resolve a critical data gap in health informatics: standard national food composition tables record macro/micronutrients but lack metabolic response metrics, while global glycemic databases lack regional and traditional Indonesian cuisines.

By applying **advanced data enrichment and rule-based clustering**, this dataset integrates over 1,000 local food entries with precise Glycemic Index (GI) and Glycemic Load (GL) values, optimized for digital health monitoring platforms and automated dietary recommendation engines.

---

## Core Data Sources & Provenance
The data architecture is established by harmonizing and cross-referencing three authoritative pillars:
1. **Tabel Komposisi Pangan Indonesia (TKPI) 2018** – Published by the Ministry of Health of the Republic of Indonesia (Kemenkes RI). Serves as the baseline for macro and micronutrient metrics per 100g.
2. **International Tables of Glycemic Index and Glycemic Load Values (2008 / 2021)** – Authored by Atkinson, F. S., Foster-Powell, K., & Brand-Miller, J. C. (*American Diabetes Association*). Serves as the clinical gold standard for glycemic response.
3. **Daftar Bahan Makanan Penukar (2003)** – Authored by Waspadji, S., et al. (*Badan Penerbit FKUI*). Provides regional diabetic exchange validation.

---

## Data Pipeline & Curation Methodology
To handle missing clinical data and ensure a fully populated database, a structured **Feature Engineering and Domain-Specific Imputation Pipeline** was executed via automated Python scripts. The imputation strategies are classified into four distinct logical layers:

### 1. Group-Based Micronutrient Imputation
Missing micronutrient data fields (such as specific vitamins and minerals) were resolved using a **Group-Based Median Imputation** mechanism. Food items were programmatically classified into biological clusters (*e.g., Seafood, Poultry, Tubers*). Missing values within a specific column were then populated using the calculated median of that exact food group to maintain biochemical distribution consistency.

### 2. Exact Mapping Layer (`exact`)
Direct transposition of GI and GL metrics for local food items that have a seamless, scientifically verified equivalent in the international clinical tables (e.g., *White Rice, Boiled Potatoes, Raw Fruits*).

### 3. Partial/Cluster-Based Mapping Layer (`partial`)
A rule-based heuristic mapping for regional Indonesian culinary preparations. Items not directly indexed in global literature were mapped using cluster-based similarities of their core ingredients (e.g., *Wortel kukus* / steamed carrots are synchronized utilizing the baseline clinical data of *Wortel segar* / raw carrots).

### 4. Zero-Value Assignment for Non-Carbohydrate Matrices (`not_found` / Non-Carb)
Based on human metabolic physiology, pure proteins and lipids do not trigger postprandial blood glucose spikes. Animal-derived high-protein or pure fat products (e.g., *Abon, Dendeng, Ikan Asin, Telur Asin, Udang Kering*) were programmatically curated with a GI value of `0` and explicitly flagged as `Tidak signifikan (non-karbohidrat)`. This safeguards automated Glycemic Load calculation models against mathematical skewing.

### 5. Conservative Baseline Imputation for Ambiguous Matrices
For complex composite dishes, mixed condiments, or unique cultural recipes that could not be mapped to any specific biological cluster, a **Conservative Baseline Imputation** was applied, anchoring the missing GI data at a constant value of **55**. 
* **Scientific Rationale:** The value **55** represents the exact international *cut-off point* for Low Glycemic Index foods according to Atkinson et al. Setting this boundary provides a safe margin of error; it prevents the algorithm from underestimating glucose impact (which is clinically hazardous) while preventing excessive overestimation. Furthermore, the protein and fat content typically found in mixed Indonesian dishes naturally slow down gastric emptying, naturally moderating the glycemic response towards this baseline.

---

## Database Schema & Attributes
The dataset is structured as a semi-colon delimited `;` CSV file (`indonesian_foods_glycemic_index.csv`). The core layout includes the following attributes:

| Column Header | Data Type | Target Domain | Description |
| :--- | :---: | :---: | :--- |
| `name` | String | Identity | Standardized Indonesian food name or processed menu item. |
| `id` | Integer | Identity | Unique identifier / Primary Key. |
| `calories` | Float | Macronutrient | Total energy content (kcal) per 100g. |
| `proteins` to `fat` | Float | Macronutrient | Core macronutrient weight (g) per 100g. |
| `serat` to `vitamin_c` | Float | Micronutrient | Extended microelement profiles (Imputed via Group-Medians). |
| `bdd` | Integer | Yield | *Bagian yang Dapat Dimakan* (Edible Portion percentage). |
| `indeks_glikemik` | Integer | Metabolic Metric | Assigned Glycemic Index value (Scale 0 - 100; Imputed via Pipeline). |
| `referensi_ig` | String | Provenance | Academic/clinical source citation or audit trail tracker. |
| `status_pencocokan_ig` | String | Data Audit | Provenance tracer flag (`exact`, `partial→[base]`, or `not_found`). |
| `klasifikasi_ig` | String | Clinical Domain | Glycemic tier (*Rendah, Sedang, Tinggi, Tidak signifikan*). |

---

## 🚀 Algorithmic Integration
The database schema is fully optimized for relational databases (PostgreSQL, MySQL, SQLite) and object-relational mappers (Django ORM, Laravel Eloquent). 

### Real-Time Dynamic Glycemic Load Formula
Backend engines can compute the localized **Glycemic Load (GL)** of any custom user-defined consumption portion dynamically using the following standardized mathematical model:

$$GL = \frac{\text{Indeks Glikemik} \times \left( \text{Karbohidrat per 100g} \times \frac{\text{Berat Porsi (g)}}{100} \right)}{100}$$

---

## License
This hybrid dataset is open-source and distributed under the **MIT License**. It is freely available for academic research, health informatics development, and clinical software prototyping.
