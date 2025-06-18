# SkySense
Readme file and codes
# 1. Povilas

-------------------------------------------------------------------------
Prerequisites
-------------------------------------------------------------------------
Python Libraries
	The following libraries are required:
		mysql-connector-python (version 9.3.0): For MySQL database connections.
		matplotlib (version 3.10.3): For generating visualizations (bar and pie charts).
	Built-in Python Libraries (no installation needed):
	json: For JSON parsing/serialization.
	collections: For data structures like defaultdict.
	os: For file system operations.
	datetime: For handling timestamps.

If need be install external libraries using pip:
	pip install mysql-connector-python==9.3.0 matplotlib==3.10.3

MySQL Database
	A local MySQL server must be running.
	Ensure you have a database named tweet_data (or adjust the database name in the scripts).
	Update the database connection parameters in each script (see below).

-------------------------------------------------------------------------
Setup Instructions
-------------------------------------------------------------------------

Update Database Connection Parameters:
	All scripts (MySQL_server_populator.py, Conversation_mining.py, TimeDistributionOfConversations.py, ReplyCountFrequencyPieChart.py, piechartforKLMvsTotal.py) connect to a MySQL database.
Modify the connection parameters in each script to match your MySQL server:

conn = mysql.connector.connect(
    host="localhost",
    user="root",
    password="your_password_here",  # Replace with your MySQL password
    database="tweet_data"          # Replace with your database name if different
)

Since a local server is used, host and user typically remain as localhost and root, respectively. Update password and database as needed.

-------------------------------------------------------------------------
Prepare JSON Files for Data Loading:
-------------------------------------------------------------------------

Ensure you have a folder containing curated JSON files with tweet data (from a prior data cleaning step).

Update the folder_path variable in MySQL_server_populator.py (line ~47) to point to your JSON files:
	folder_path = r'your_folder_path_here'  # e.g., r'C:\Users\YourName\Documents\curatedfiles'
	To get the correct path, right-click the folder containing your JSON files and select "Copy as Path" (Windows) or equivalent.

Example:
folder_path = r'C:\Users\povil\Desktop\Uni stuff\DBL 1\Data\curatedfiles\curatedfiles'

-------------------------------------------------------------------------
Output Directory for Visualizations:
-------------------------------------------------------------------------
The visualization scripts (TimeDistributionOfConversations.py, ReplyCountFrequencyPieChart.py, piechartforKLMvsTotal.py) save images to a specified folder.

Update the plt.savefig and os.path.exists paths in these scripts to a valid directory on your system:

plt.savefig('your_folder_path/filename.png', transparent=True, bbox_inches='tight')
if os.path.exists('your_folder_path/filename.png'):

Example for ReplyCountFrequencyPieChart.py:

plt.savefig('C:/Users/YourName/Desktop/output/reply_count_frequency_pie_chart.png', transparent=True, bbox_inches='tight')
if os.path.exists('C:/Users/YourName/Desktop/output/reply_count_frequency_pie_chart.png'):

Ensure the folder exists and you have write permissions.

-------------------------------------------------------------------------
Running the Scripts
Follow these steps in order to process and analyze the tweet data:
-------------------------------------------------------------------------

Populate the Database:
	Run MySQL_server_populator.py to load JSON tweet data into the MySQL database.

This script creates a tweets table and inserts data from JSON files. Ensure the folder_path is correctly set.

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Mine Conversations:
	Run Conversation_mining.py to analyze tweet conversations and populate the dim_conversations table.

No additional changes are needed if the database connection parameters are correct.

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Generate Visualizations:
Run the following scripts to create visualizations based on the mined data:
	TimeDistributionOfConversations.py: Creates a bar chart of conversation counts by month.
	ReplyCountFrequencyPieChart.py: Creates a pie chart of reply count frequencies.
	piechartforKLMvsTotal.py: Creates a pie chart comparing @KLM conversations to total conversations.
Ensure the output paths for plt.savefig are updated as described above.
Output images will be saved to the specified directories.
\

# 2. Abu
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

### Step 6: Relevance and Bot Detection (`step_6_relevance.py`)
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
- Final script consolidates multiple bot detection criteria under a unified `pbot` label.

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



# 3. David

# 4. Dragos

# 5. Rares

# 6. Alp



