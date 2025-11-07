# DSAI3202 - Lab 3: Data Preprocessing on Azure (Final Report)

## I. Introduction
This lab explains how to preprocess data using **Microsoft Azure**, **Databricks**, and **Microsoft Fabric**.  
The dataset comes from **Goodreads** and includes books, authors, and user reviews.  
The goal is to build a Lakehouse setup, load the data, clean and transform it, and prepare a final “Gold” dataset ready for analysis.

---

## II. Step 1: Setting up Azure Data Lake Storage (ADLS Gen2)
**Azure Data Lake Storage Gen2** is used as the main storage for all data layers.  
A container named **`lakehouse`** was created with the following folders:
- `/raw/`
- `/processed/` (Silver layer)
- `/gold/`

The original Goodreads data was uploaded to the **raw** folder using **AzCopy** from **Azure Cloud Shell**.

---

## III. Step 2: Data Ingestion and Transformation using Azure Data Factory
**Azure Data Factory (ADF)** pipelines were used to move raw JSON files into the Silver layer in **Parquet** format.  
The **Copy Activity** was set up to keep the folder structure and store files in:
- `/processed/books/`
- `/processed/authors/`
- `/processed/reviews/`

---

## IV. Step 3: Data Cleaning in Databricks
**Databricks** was used to open, inspect, and clean the Silver layer data.  
Main cleaning steps:
- Removed missing or invalid entries  
- Changed ratings to integer type  
- Removed empty or very short reviews  
- Trimmed text and removed duplicate rows  

The cleaned reviews were saved back to ADLS as Parquet files.

---

## V. Step 4: Creating the Gold Table
The cleaned **Books**, **Authors**, **Book_Authors**, and **Reviews_Clean** data were joined in Databricks to make one final Gold table called **`curated_reviews`**.  
Columns included:
- `review_id`, `book_id`, `author_id`, `title`, `name`, `user_id`, `rating`, `review_text`

This table was saved and registered as a **Delta table** for use with Spark SQL.

---

## VI. Step 5: Data Preprocessing in Microsoft Fabric
A **Microsoft Fabric workspace** was created and connected to the curated Delta table stored in ADLS Gen2.  

Transformations done in **Power Query** included:
- Fixing data types (IDs as text, rating as number)  
- Handling missing or null values  
- Cleaning and standardizing text  
- Adding features like:
  - Average rating per book  
  - Number of reviews per book  

The final dataset was then published to the Fabric Warehouse for checking.

---

## VII. Step 6: Advanced Cleaning and Feature Engineering in Databricks
Because not all transformations could be finished in Fabric, the last steps were completed in **Databricks**.  

Tasks included:
- Removing nulls and duplicates  
- Making text lowercase and trimmed  
- Removing reviews shorter than 10 characters  
- Fixing data types  
- Adding new columns:
  - `review_length_words`  
  - `avg_rating_per_book`  
  - `review_count_per_book`

The final enriched data was saved to:
```
/gold/features_v1/
```

### Example Spark Code
```python
from pyspark.sql.functions import col, trim, lower, length, split, size, avg, count, round as rnd

df = curated_reviews.filter(col('rating').isNotNull() & col('review_text').isNotNull())
df = df.dropDuplicates(['review_id', 'user_id', 'book_id'])
df = df.withColumn('review_text', lower(trim(col('review_text'))))
df = df.filter(length(col('review_text')) >= 10)
df = df.withColumn('review_length_words', size(split(col('review_text'), ' ')))

book_features = df.groupBy('book_id').agg(
    rnd(avg('rating'), 2).alias('avg_rating_per_book'),
    count('review_id').alias('review_count_per_book')
)

df_features = df.join(book_features, 'book_id', 'left')
df_features.write.format('delta').mode('overwrite').save(
    'abfss://lakehouse@goodreadsreviews60103737.dfs.core.windows.net/gold/features_v1/'
)
```

---

## VIII. GitHub Submission Structure
Your GitHub repository should look like this:

```
📁 AzureLab3-Data-Preprocessing/
├── notebooks/
│   ├── curated_reviews.ipynb          # Cleans Silver layer (Books, Authors, Reviews)
│   ├── gold_features_preparation.ipynb         # Final cleaning + feature creation for Gold layer
├── README.md                              # This report and explanations
├── screenshots/                     
```

---

## IX. Conclusion
This lab showed the complete data preprocessing process using **Azure Data Factory**, **Databricks**, and **Microsoft Fabric**.  
The final **`features_v1`** dataset was created in the Gold layer — cleaned, enriched, and ready for analytics.  

Overall, this project helped us understand how data pipelines work in the cloud, including schema management and feature engineering.

---

### 🏁 Notebook Summary
| Notebook | Purpose | Output |
|-----------|----------|--------|
| **curated_reviews.ipynb** | Cleans and checks Silver layer data | `/processed/reviews/` |
| **gold_features.ipynb** | Final cleaning and feature creation for Gold layer | `/gold/features_v1/` |

---

**Course:** DSAI3202 — Data Engineering and Preprocessing on Azure  
**Date:** *November 2025*
