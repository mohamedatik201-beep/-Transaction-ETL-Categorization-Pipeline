# 💳 Transaction ETL & Categorization Pipeline

An end-to-end ETL pipeline that cleans, normalizes, and categorizes raw bank transaction data using rule-based text processing and a machine learning classifier (Naive Bayes + TF-IDF). Cleaned data is stored in a SQLite database for analysis.

---

## 📌 Project Overview

Raw bank transactions often come with messy, inconsistent descriptions (e.g. `EFTPOS PURCHASE WOOLWORTHS CBD TID12345`). This project:

1. **Extracts** raw transaction data from a CSV file
2. **Transforms** descriptions using a rule-based cleaning engine (regex + keyword matching)
3. **Classifies** transactions into spending categories using a ML pipeline
4. **Loads** the cleaned data into a SQLite database

---

## 🗂️ Dataset

**File:** `transaction_FM.csv`

| Column | Description |
|---|---|
| `transaction_id` | Unique transaction identifier |
| `persona_id` | Customer persona (e.g. P01) |
| `date` | Transaction date (DD/MM/YYYY) |
| `description` | Raw merchant description from the bank |
| `amount` | Transaction amount (AUD) |
| `category` | Label: `Essential`, `Non-Essential`, `Excluded` |
| `is_labelled` | Whether the transaction has a ground-truth label |

---

## ⚙️ Pipeline Steps

### 1. 🧹 Data Cleaning (`clean_engine`)

A rule-based function that:
- Strips common bank prefixes (`EFTPOS PURCHASE`, `VISA PURCHASE`, `POS`, etc.)
- Removes noise like TID numbers, date codes, and location suffixes
- Maps merchant names to standardized categories:

| Raw Description | Cleaned Label |
|---|---|
| `UBER *EATS` | `UBER EATS` |
| `WOOLWORTHS CBD` | `SUPERMARKET` |
| `DAILY GRIND COFFEE` | `COFFEE SHOP` |
| `NETFLIX` | `NETFLIX` |
| `BUPA` | `HEALTH INSURANCE (BUPA)` |

### 2. 🤖 ML Classification (TF-IDF + Naive Bayes)

For transactions not caught by the rule engine, a scikit-learn pipeline classifies them:

```
Pipeline([
    ('tfidf', TfidfVectorizer()),
    ('clf',   MultinomialNB())
])
```

- Train/test split: 80/20
- Evaluated with `classification_report` and `ConfusionMatrixDisplay`

### 3. 🗃️ Load to SQLite

Cleaned and classified transactions are stored in a local SQLite database (`ma_finance.db`) via SQLAlchemy:

```python
engine = create_engine('sqlite:///ma_finance.db')
df.to_sql('transactions_clean', con=engine, if_exists='replace', index=False)
```

---

## 🚀 Getting Started

### Prerequisites

```bash
pip install pandas scikit-learn sqlalchemy
```

### Run the notebook

```bash
jupyter notebook notebook.ipynb
```

Make sure `transaction_FM.csv` is in the same directory as the notebook.

---

## 📁 Project Structure

```
.
├── notebook.ipynb          # Main ETL + ML notebook
├── transaction_FM.csv      # Raw transaction data
├── ma_finance.db           # Output SQLite database (generated)
└── README.md
```

---

## 🛠️ Tech Stack

- **Python 3**
- **Pandas** — data manipulation
- **Scikit-learn** — TF-IDF vectorizer + Naive Bayes classifier
- **SQLAlchemy** — database ORM
- **SQLite** — lightweight local database
- **Regex (`re`)** — rule-based text cleaning

---

## 📊 Categories

The pipeline identifies the following spending categories:

`SUPERMARKET` · `RESTAURANT` · `COFFEE SHOP` · `UBER EATS` · `UBER TRIP` · `NETFLIX` · `SPOTIFY` · `APPLE SERVICES` · `HEALTH & PHARMACY` · `HEALTH INSURANCE (BUPA)` · `TELECOM (OPTUS)` · `TRANSPORT (METROCARD)` · `BANK TRANSFER FROM SAVER` · `BANK TRANSFER TO SAVER` · `7-ELEVEN` · `BOLT`

---

## 📝 Notes

- The dataset contains transactions from Australian merchants (Woolworths, Coles, Optus, etc.)
- The rule-based engine takes priority over the ML classifier for known merchants
- `amount` values were coerced to `float64` to handle formatting inconsistencies in the source data
- generated AI data
