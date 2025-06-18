

#  Abu
# Tweet Data Cleaning Pipeline

This repository documents a 6-step data cleaning process applied to Twitter data for the purpose of airline-related sentiment analysis and bot detection. Each step focuses on a specific data quality dimension to ensure the dataset is reliable, consistent, and relevant.

## Cleaning Steps Overview

### Step 1: Completeness (`step_1_completeness.py`)
- Removes rows with missing critical fields such as `id`, `text`, or `user_id`.
- Fills missing values in optional engagement metrics (`retweet_count`, `favorite_count`) with default values.
- Flags rows with missing conversation references.
- Generates a report confirming removal or imputation of missing values.

### Step 2: Accuracy (`step_2_accuracy.py`)
- Excludes tweets with invalid `created_at` timestamps (e.g., future dates or dates before March 21, 2006).
- Ensures `user_id` and engagement metrics are valid non-negative integers.
- Verifies `lang` contains supported language codes.
- Corrects or removes values that do not reflect realistic Twitter data.

### Step 3: Consistency (`step_3_consistency.py`)
- Standardizes the `created_at` field to the format `YYYY-MM-DD HH:MM:SS`.
- Converts `lang` values to lowercase for uniformity.
- Caps `retweet_count` if it exceeds 10 times `favorite_count` to enforce logical consistency.
- Outputs a consistency report for verification.

### Step 4: Uniqueness (`step_4_uniqueness.py`)
- Removes exact duplicate tweets based on `id`, keeping the first occurrence.
- Avoids removing similar or near-duplicate tweets (e.g., valid retweets).
- Generates a uniqueness report to confirm deduplication.

### Step 5: Validity (`step_5_validity.py`)
- Ensures fields match expected data types and formats:
  - Validates `created_at` format.
  - Checks that `user_id`, `retweet_count`, `favorite_count`, `reply_count`, and `quote_count` are non-negative integers.
  - Verifies that `lang` follows expected two- or three-letter language codes or language-region formats.
- Invalid values are set to `NULL`, and affected rows are flagged under `quality_flag = 'invalid_format'`.
- A validity report is generated to track issues.

### Step 6: Relevance and Bot Detection (`step_6_relevance.py`) and (`step_6_2_relevance.py`)
- **Bot Detection (flagged as `pbot`)** using the following criteria:
  - High retweet ratio (retweet count significantly exceeds other engagement metrics).
  - Repetitive text (same content posted over 50 times).
  - High posting frequency (e.g., >35 tweets/day).
  - Low follower-to-following ratio (fewer than 10 followers, over 100 followings).
  - Very short or generic content (e.g., fewer than 10 characters or containing common bot phrases).
- **Relevance Filtering**:
  - Flags tweets as `non_relevant` if they do not mention specific airline handles or are not from/to known airline account IDs.
- Removes `pbot` flags from known airline accounts to prevent false positives.

## Quality Flags Summary

| Flag                      | Description                          | Count     |
|---------------------------|--------------------------------------|-----------|
| `pbot`                    | Detected potential bots              | 2,392,494 |
| `missing_conversation_data` | Incomplete conversation threads     |   434,582 |
| `non_relevant`            | Not related to target airlines       |   260,687 |

## Notes
- All scripts are modular and can be executed independently.
- Flags are added rather than removing data, preserving records for review or downstream filtering.
- Final script consolidates multiple bot detection criteria under a unified `pbot` label.4

## Required Libraries for the cleaning part 
### Database Connection
import pymysql

### Data Handling
import pandas as pd

### Date & Time
from datetime import datetime

### Regular Expressions
import re


# Multilingual Sentiment Analysis Pipeline

This repository contains a multilingual sentiment analysis pipeline capable of processing input text in **64 languages**. The system efficiently handles diverse languages by segregating them based on their compatibility with existing pre-trained models or translation requirements.

## Workflow Overview

1. **Input:**  
   A dataset containing text in 64 different languages.

2. **Language Segregation:**  
   The input is categorized into three groups:
   - **Mainstream Languages (9 total):**  
     Directly processed using a multilingual model.
   - **English:**  
     Directly processed using an English-specific model.
   - **Other Languages:**  
     Translated to English before analysis.

---

## Models Used

### 1. `clapAI/roberta-large-multilingual-sentiment`  
- **Languages Supported:**
  - Spanish (`es`)
  - French (`fr`)
  - German (`de`)
  - Italian (`it`)
  - Dutch (`nl`)
  - Portuguese (`pt`)
  - Arabic (`ar`)
  - Japanese (`ja`)
  - Chinese (`zh-cn`)
- **Framework:** Hugging Face Transformers  
- **Purpose:** Multilingual sentiment analysis.

### 2. `cardiffnlp/twitter-roberta-base-sentiment-latest`  
- **Language:** English  
- **Model Type:** RoBERTa  
- **Specialization:**  
  - Social media content patterns  
  - Informal expressions and abbreviations

### 3. Google Translate API (via `deep-translator` library)  
- **Purpose:**  
  - Translate unsupported languages to English  
  - Enables broader coverage for sentiment analysis

---

##  Output Format

Each input text returns two key outputs:

- **Sentiment:**  
  - One of: `Positive`, `Neutral`, or `Negative`
  
- **Score:**  
  - A confidence value indicating the prediction reliability

---

## Dependencies

- [Transformers](https://huggingface.co/transformers/)
- [Deep Translator](https://pypi.org/project/deep-translator/)
- [Torch](https://pytorch.org/)
- [Hugging Face Datasets](https://huggingface.co/docs/datasets/)

# JSON to MySQL Ingestion Script (To add the conversation file after Sentiment scores are implemented and conversation mined

This Python script reads a JSON file containing tweet conversation data and inserts the records into a MySQL table named `dim_conversations`. It performs validation checks, handles duplicates gracefully, and logs progress throughout the process.

---

## File Structure

- `full_conversations_with_final_airline_tag.json` – Input JSON file.
- `import_json_to_sql.py` – Python script to run the ingestion.

---

## Table Requirements

The target MySQL table must exist before running the script:

**Table Name:** `dim_conversations`

**Expected Columns:**
- `tweet_id`
- `reply_count`
- `category`
- `conversation_json`
- `airline_tag`

---

## Prerequisites

- Python 3.x
- MySQL server running locally
- The following Python packages:
  - `pymysql`
  - `os`
  - `json` import

# Business Intelligence: Measuring Twitter Response Impact for Airlines

This project analyzes Twitter-based customer service interactions of major airlines—KLM, Lufthansa, and British Airways—to assess how engagement strategies affect public sentiment and customer satisfaction.

## 1. Context and Motivation

Modern airlines use Twitter extensively for customer service and brand communication. However, the effectiveness of these responses is not always clear. This project investigates key questions such as:

- Does replying faster improve customer satisfaction?
- Does simply replying at all help reduce frustration?
- Can the business value of a Twitter support team be quantified?

The goal is to analyze and demonstrate the impact of Twitter engagement strategies on public sentiment using data-driven methods.

## 2. Key Objectives

The analysis focuses on the following objectives:

- Quantify reply engagement for major airlines (KLM, Lufthansa, British Airways)
- Measure the effect of response time on customer sentiment
- Compare sentiment before and after the airlines’ replies
- Benchmark KLM’s performance against its competitors
- Account for external factors, such as the COVID-19 pandemic or labor strikes

## 3. Core Metrics

**Engagement Metrics:**

- **Reply Rate** = (Replies made by airline) / (Total incoming tweets) × 100
- **Response Speed** = Average time (in minutes) from customer tweet to the airline’s first reply

**Sentiment Metrics:**

- **Before/After Sentiment** – Sentiment of original customer tweets vs. sentiment after receiving a reply
- **Sentiment Lift** – Change in sentiment classification from original to response
- **Correlation Analysis** – Examines whether faster responses are associated with higher sentiment lift

## 4. Analytical Approach

The project uses the following techniques and methodologies:

- Sentiment classification (positive, neutral, negative) using labeled tweets
- Matching of customer tweets with corresponding airline replies
- Time-based grouping to simulate full conversation threads
- Correlation plots and statistical methods to assess relationships between engagement and sentiment

## 5. Graph Scripts

The following Python scripts are used to generate analytical visualizations for comparison across KLM, Lufthansa, and British Airways:

- **`reply_rate.py`**  
  Visualizes the reply rate (percentage of customer tweets that received a response) for each airline.

- **`graph_2_busi.py`**  
  Analyzes and plots average response time trends over time.

- **`Graph 3.py`**  
  Measures and visualizes sentiment lift across time bins, comparing pre- and post-reply sentiment distributions.

- **`Cor_business_idea.py`**  
  Performs Pearson and Spearman correlation analysis to identify statistical relationships between response speed and sentiment lift.

## 6. Output and Insights

Each script provides comparative insight into how airlines perform with respect to Twitter responsiveness and its measurable impact on customer sentiment. The analysis emphasizes KLM's strategy against its competitors to uncover patterns that could support actionable business decisions.

## 7. Dependencies

This project relies on the following Python libraries:

### Data Handling
import pandas as pd
import numpy as np
import json

### Plotting & Visualization
import matplotlib.pyplot as plt
import seaborn as sns

### Database Connection
from sqlalchemy import create_engine, text

### Date & Time
from datetime import datetime
import time

### Statistical Analysis
from scipy.stats import pearsonr, spearmanr




