# 🧠 DSAI3202 – Lab 3 & 4: Data Preprocessing & Text Feature Engineering on Azure

## Overview
This project demonstrates an end-to-end **data preprocessing** and **text feature engineering** workflow using **Microsoft Azure Data Factory**, **Azure Databricks**, and **Microsoft Fabric**.  
The dataset used is from **Goodreads**, containing books, authors, and user reviews.

---

## Architecture Overview
Lakehouse architecture implemented on **ADLS Gen2** with three layers:

```
/raw/         → Raw JSON dumps  
/processed/   → Silver (cleaned Parquet)  
/gold/        → Gold (curated feature datasets)
```

---

## Lab 3: Data Preprocessing on Azure

### Steps Overview
1. **Azure Data Lake Setup**
   - Created a storage account and container `lakehouse`.
   - Uploaded raw Goodreads files using **AzCopy**.

2. **Data Ingestion (Azure Data Factory)**
   - Pipelines copied raw data from `/raw/` → `/processed/` in **Parquet** format.
   - Used “Preserve hierarchy” option for schema consistency.

3. **Data Cleaning (Databricks)**
   - Removed null/invalid IDs, trimmed text, and dropped duplicates.
   - Saved cleaned Parquet data to `/processed/reviews/`.

4. **Curated Gold Dataset**
   - Joined Books, Authors, and Reviews to form `curated_reviews`.
   - Saved as Delta under `/gold/curated_reviews/`.

5. **Fabric Data Preprocessing**
   - Applied transformations in **Power Query Editor**:
     - Fixed data types and removed nulls.
     - Added features: *average rating per book* and *review count per author*.

6. **Advanced Feature Engineering (Databricks)**
   - Created numeric features: `review_length_words`, `avg_rating_per_book`, `review_count_per_book`.
   - Saved under `/gold/features_v1/`.

---

## Lab 4: Text Feature Engineering on Azure

### Objective
Extend Lab 3 by engineering **text-based features** (TF-IDF, sentiment) to create ML-ready datasets.

### Steps
1. **Load Clean Data**
   ```python
   df = spark.read.format("delta").load(
       "abfss://lakehouse@goodreadsreviews60103737.dfs.core.windows.net/gold/features_v1/"
   )
   ```

2. **Text Cleaning**
   ```python
   from pyspark.sql.functions import lower, trim, regexp_replace
   df = df.withColumn("clean_text", lower(trim(df["review_text"])))
   df = df.withColumn("clean_text", regexp_replace("clean_text", "[^a-zA-Z0-9 ]", ""))
   ```

3. **Sentiment Analysis (VADER)**
   ```python
   from nltk.sentiment import SentimentIntensityAnalyzer
   sid = SentimentIntensityAnalyzer()
   sentiments = pdf["clean_text"].apply(sid.polarity_scores).apply(pd.Series)
   ```

4. **TF-IDF Vectorization**
   ```python
   from sklearn.feature_extraction.text import TfidfVectorizer
   tfidf = TfidfVectorizer(max_features=1000, stop_words='english', ngram_range=(1,2))
   ```

5. **Combine and Save**
   ```python
   train_spark_df.write.mode("overwrite").format("delta").save(
       "abfss://lakehouse@goodreadsreviews60103737.dfs.core.windows.net/gold/features_v2/train_features/"
   )
   ```

✅ Output confirmed: Delta table successfully written!

---

## Repository Structure

```
📁 AzureLab3-Data-Preprocessing/
├── notebooks/
│   ├── curated_reviews_cleaning.ipynb
│   ├── gold_features_preparation.ipynb
│   ├── text_feature_engineering.ipynb
├── screenshots/
│   ├── lab3/
│   └── lab4/
└── README.md
```

---

## Results & Conclusion
- **Lab 3:** Completed preprocessing, cleaning, and feature aggregation.  
- **Lab 4:** Engineered advanced text features using TF-IDF and sentiment analysis.  
- Final output stored as Delta tables for scalable analytics and ML integration.

**Student ID:** 60103737  
**Course:** DSAI3202 — Data Engineering and Preprocessing on Azure  
**Date:** November 2025
