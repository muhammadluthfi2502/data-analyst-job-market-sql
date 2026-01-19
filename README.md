# 💼 **Analyzing the Data Analyst Job Market with SQL**
<br>

**Tool** : PostgreSQL <br>  
**Visualization** : Looker Studio <br>  
**Dataset** : Kaggle – Data Analyst Job Listings (Glassdoor)  
<br>
<br>

**Table of Contents**
- [STAGE 0: Problem Statement](#-stage-0-problem-statement)
	- [Background Story](#background-story)
	- [Objective](#objective)
- [STAGE 1: Data Preparation](#-stage-1-data-preparation)
	- [Create Database and ERD](#create-database-and-erd)
- [STAGE 2: Data Analysis](#-stage-2-data-analysis)
	- [Salary Trends by Location](#1-salary-trends-by-location)
	- [Salary Trends by Seniority Level](#2-salary-trends-by-seniority-level)
	- [Top Companies by Hiring Volume](#3-top-companies-by-hiring-volume)
	- [Salary Distribution by State](#4-salary-distribution-by-state)
	- [Easy Apply vs Salary](#5-easy-apply-vs-salary)
	- [Visualization Helper Dataset](#6-visualization-helper-dataset)
- [STAGE 3: Summary](#-stage-3-summary)
<br>
<br>

---

## 📂 **STAGE 0: Problem Statement**

### **Background Story**
Pasar kerja untuk Data Analyst berkembang pesat, namun pencari kerja sering tidak memiliki gambaran yang jelas mengenai variasi gaji berdasarkan lokasi, senioritas, dan perusahaan.  
Informasi yang terfragmentasi membuat proses negosiasi gaji dan perencanaan karier menjadi kurang optimal.

Dalam proyek ini dilakukan analisis terhadap data lowongan pekerjaan Data Analyst untuk memahami pola kompensasi dan karakteristik pasar kerja secara kuantitatif menggunakan SQL.

---

### **Objective**
Mengumpulkan insight dari analisis data lowongan kerja Data Analyst untuk:
1. Mengidentifikasi negara bagian dengan gaji tertinggi  
2. Menganalisis perbedaan gaji berdasarkan level senioritas  
3. Menentukan perusahaan dengan volume hiring tertinggi  
4. Menguji hubungan antara fitur *Easy Apply* dan tingkat gaji  
5. Menyediakan dataset siap visualisasi untuk dashboard BI  

<br>
<br>

---

## 📂 **STAGE 1: Data Preparation**

Dataset yang digunakan berasal dari Kaggle (Glassdoor Data Analyst Jobs) dengan total **2,253 baris data**.  
Dataset mencakup informasi lowongan kerja seperti:
- Job title  
- Company name  
- Salary range  
- Location (city, state, country)  
- Seniority level  
- Industry, sector  
- Rating perusahaan  
- Fitur Easy Apply  

### **Create Database and ERD**
**Langkah-langkah yang dilakukan meliputi:**
1. Membuat database PostgreSQL bernama `job_market_intelligence`  
2. Mengimpor data CSV ke tabel staging (`staging_jobs`)  
3. Membuat skema dimensional (`dim_company`, `dim_location`, `dim_job_title`, `dim_seniority`)  
4. Membuat tabel fakta (`fact_job_postings`)  
5. Menentukan Primary Key dan Foreign Key  
6. Memastikan integritas relasi dan jumlah baris konsisten  

<details>
<summary>Click untuk melihat Queries</summary>

```sql
-- Create staging table
CREATE TABLE staging_jobs (
  job_title_raw TEXT,
  title_clean TEXT,
  seniority_level TEXT,
  salary_min INT,
  salary_max INT,
  company_name TEXT,
  industry TEXT,
  sector TEXT,
  company_type TEXT,
  founded_year INT,
  size TEXT,
  revenue TEXT,
  rating NUMERIC,
  location TEXT,
  city TEXT,
  state TEXT,
  country TEXT,
  easy_apply BOOLEAN
);

-- Dimension: company
CREATE TABLE dim_company (
  company_id SERIAL PRIMARY KEY,
  company_name TEXT UNIQUE,
  industry TEXT,
  sector TEXT,
  company_type TEXT,
  founded_year INT,
  size TEXT,
  revenue TEXT,
  headquarters TEXT,
  rating NUMERIC
);

-- Dimension: location
CREATE TABLE dim_location (
  location_id SERIAL PRIMARY KEY,
  city TEXT,
  state TEXT,
  country TEXT
);

-- Dimension: job title
CREATE TABLE dim_job_title (
  job_title_id SERIAL PRIMARY KEY,
  job_title_raw TEXT UNIQUE,
  title_clean TEXT
);

-- Dimension: seniority
CREATE TABLE dim_seniority (
  seniority_id SERIAL PRIMARY KEY,
  seniority_level TEXT UNIQUE
);

-- Fact table
CREATE TABLE fact_job_postings (
  job_id SERIAL PRIMARY KEY,
  company_id INT REFERENCES dim_company(company_id),
  location_id INT REFERENCES dim_location(location_id),
  job_title_id INT REFERENCES dim_job_title(job_title_id),
  seniority_id INT REFERENCES dim_seniority(seniority_id),
  salary_min INT,
  salary_max INT,
  easy_apply BOOLEAN
);
```

</details>

<br>
<br>

---

## 📂 **STAGE 2: Data Analysis**

### **1. Salary Trends by Location**
Analisis rata-rata gaji minimum, maksimum, dan midpoint berdasarkan negara bagian.

<details>
<summary>Click untuk melihat Queries</summary>

```sql
SELECT
  l.state,
  COUNT(*) AS job_count,
  ROUND(AVG(f.salary_min), 0) AS avg_salary_min,
  ROUND(AVG(f.salary_max), 0) AS avg_salary_max,
  ROUND(AVG((f.salary_min + f.salary_max) / 2), 0) AS avg_salary_midpoint
FROM fact_job_postings f
JOIN dim_location l ON f.location_id = l.location_id
WHERE f.salary_min IS NOT NULL
  AND f.salary_max IS NOT NULL
GROUP BY l.state
HAVING COUNT(*) >= 5
ORDER BY avg_salary_midpoint DESC;
```

</details>

---

### **2. Salary Trends by Seniority Level**
Perbandingan gaji berdasarkan level Junior, Mid, Senior, dan Lead.

<details>
<summary>Click untuk melihat Queries</summary>

```sql
SELECT
  s.seniority_level,
  COUNT(*) AS job_count,
  ROUND(AVG(f.salary_min), 0) AS avg_salary_min,
  ROUND(AVG(f.salary_max), 0) AS avg_salary_max,
  ROUND(AVG((f.salary_min + f.salary_max) / 2), 0) AS avg_salary_midpoint
FROM fact_job_postings f
JOIN dim_seniority s ON f.seniority_id = s.seniority_id
WHERE f.salary_min IS NOT NULL
  AND f.salary_max IS NOT NULL
GROUP BY s.seniority_level
ORDER BY avg_salary_midpoint DESC;
```

</details>

---

### **3. Top Companies by Hiring Volume**
Menentukan perusahaan dengan jumlah lowongan terbanyak.

<details>
<summary>Click untuk melihat Queries</summary>

```sql
SELECT
  c.company_name,
  COUNT(*) AS job_count
FROM fact_job_postings f
JOIN dim_company c ON f.company_id = c.company_id
GROUP BY c.company_name
ORDER BY job_count DESC
LIMIT 10;
```

</details>

---

### **4. Salary Distribution by State**
Distribusi midpoint gaji berdasarkan negara bagian.

<details>
<summary>Click untuk melihat Queries</summary>

```sql
SELECT
  l.state,
  ROUND(AVG((f.salary_min + f.salary_max) / 2), 0) AS avg_salary_midpoint
FROM fact_job_postings f
JOIN dim_location l ON f.location_id = l.location_id
WHERE f.salary_min IS NOT NULL
  AND f.salary_max IS NOT NULL
GROUP BY l.state
ORDER BY avg_salary_midpoint DESC;
```

</details>

---

### **5. Easy Apply vs Salary**
Menguji apakah fitur *Easy Apply* memengaruhi tingkat gaji.

<details>
<summary>Click untuk melihat Queries</summary>

```sql
SELECT
  easy_apply,
  COUNT(*) AS job_count,
  ROUND(AVG((salary_min + salary_max) / 2), 0) AS avg_salary_midpoint
FROM fact_job_postings
WHERE salary_min IS NOT NULL
  AND salary_max IS NOT NULL
GROUP BY easy_apply;
```

</details>

---

### **6. Visualization Helper Dataset**
Dataset ringkas untuk keperluan dashboard Looker Studio.

<details>
<summary>Click untuk melihat Queries</summary>

```sql
SELECT
  f.job_id,
  c.company_name,
  l.state,
  l.city,
  s.seniority_level,
  jt.title_clean,
  f.salary_min,
  f.salary_max,
  (f.salary_min + f.salary_max) / 2 AS salary_midpoint,
  f.easy_apply
FROM fact_job_postings f
JOIN dim_company c ON f.company_id = c.company_id
JOIN dim_location l ON f.location_id = l.location_id
JOIN dim_seniority s ON f.seniority_id = s.seniority_id
JOIN dim_job_title jt ON f.job_title_id = jt.job_title_id;
```

</details>

<br>
<br>

---

## 📂 **STAGE 3: Summary**

- Negara bagian seperti **California** dan **New York** menawarkan gaji rata-rata tertinggi untuk posisi Data Analyst.  
- **Senior** dan **Lead** roles memiliki midpoint gaji sekitar 15–20% lebih tinggi dibandingkan level Mid.  
- Sebagian besar lowongan berasal dari segelintir perusahaan besar yang mendominasi volume hiring.  
- Lowongan dengan fitur **Easy Apply** cenderung memiliki midpoint gaji sedikit lebih rendah dibandingkan lowongan reguler.  
- Dataset hasil query akhir dapat langsung digunakan untuk membangun dashboard di Looker Studio.
