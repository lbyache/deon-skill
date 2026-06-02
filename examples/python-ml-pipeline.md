# Example: The ML Pipeline That Passes Every Check (Python)

## The scenario

A data scientist opens a PR for a new churn-prediction model. The pipeline is well-written: typed, tested, linted with `ruff`, scanned with `bandit`. No secrets. No SQL injection. No obvious vulnerabilities.

```python
# churn_pipeline.py
import pandas as pd
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.model_selection import train_test_split
import mlflow
import logging

logger = logging.getLogger(__name__)


def load_training_data(db_conn) -> pd.DataFrame:
    query = """
        SELECT
            u.id,
            u.email,
            u.full_name,
            u.date_of_birth,
            u.phone_number,
            u.zip_code,
            COUNT(o.id)         AS order_count,
            SUM(o.total)        AS lifetime_value,
            MAX(o.created_at)   AS last_order_date,
            u.created_at        AS signup_date
        FROM users u
        LEFT JOIN orders o ON o.user_id = u.id
        GROUP BY u.id
    """
    df = pd.read_sql(query, db_conn)
    logger.info("Loaded training data: %s", df.to_dict(orient="records"))
    return df


def train(df: pd.DataFrame) -> GradientBoostingClassifier:
    features = [
        "email", "full_name", "date_of_birth",
        "phone_number", "zip_code",
        "order_count", "lifetime_value", "last_order_date"
    ]
    X = df[features]
    y = (df["last_order_date"] < pd.Timestamp.now() - pd.Timedelta(days=90)).astype(int)

    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

    model = GradientBoostingClassifier()
    model.fit(X_train, y_train)

    mlflow.log_param("features", features)
    mlflow.log_metric("accuracy", model.score(X_test, y_test))

    return model


def predict(model, user: dict) -> float:
    logger.info("Predicting churn for user: %s", user)
    return model.predict_proba([list(user.values())])[0][1]
```

`bandit`: **0 issues**. `ruff`: **0 issues**. Tests: **passing**.

---

## deon review

### Ethical Review

| # | Issue | Rule | Severity |
|---|-------|------|----------|
| 1 | PII fields used as model features: `email`, `full_name`, `date_of_birth`, `phone_number` | **C1** | High |
| 2 | Full user records logged at training and inference time | **D2** | Critical |
| 3 | No legal basis documented for processing `date_of_birth`, `phone_number` in an ML context | **C2** | High |
| 4 | PII fields logged to MLflow as tracked parameters | **D2** | Critical |

### Why this matters

The model predicts churn. For that, you need behavioral signals: purchase frequency, recency, order value. You do **not** need `full_name`, `email`, `date_of_birth`, or `phone_number` — and including them creates GDPR/CCPA exposure with zero predictive benefit.

Logging `df.to_dict(orient="records")` at training time writes full PII to your log aggregator, where it will likely sit indefinitely, accessible to anyone with log access, outside the data governance controls on your database.

### Checklist

- [ ] C1: Features reduced to what the model actually needs
- [ ] D2: No PII in training logs or MLflow params
- [ ] C2: Legal basis for each PII field documented if retained

### Remediation

#### 1. Data minimization — drop PII from features

The model needs behavioral signals. Strip PII at the query level, not in pandas.

```python
def load_training_data(db_conn) -> pd.DataFrame:
    # C1: Select only what the model needs. No PII.
    query = """
        SELECT
            u.id,
            u.zip_code,                          -- geographic signal, not identity
            COUNT(o.id)         AS order_count,
            SUM(o.total)        AS lifetime_value,
            MAX(o.created_at)   AS last_order_date,
            u.created_at        AS signup_date
        FROM users u
        LEFT JOIN orders o ON o.user_id = u.id
        GROUP BY u.id
    """
    return pd.read_sql(query, db_conn)
```

If `zip_code` is also unnecessary, drop it. Every field kept needs a justification.

#### 2. Safe logging — aggregate stats, never records

```python
# BAD: Dumps full PII records to logs
logger.info("Loaded training data: %s", df.to_dict(orient="records"))

# GOOD: Log shape and distribution, not content
logger.info(
    "Loaded training data: rows=%d, churn_rate=%.2f, date_range=%s to %s",
    len(df),
    df["is_churned"].mean() if "is_churned" in df.columns else float("nan"),
    df["last_order_date"].min(),
    df["last_order_date"].max(),
)
```

#### 3. Safe inference logging

```python
# BAD: Logs the full user dict (may contain PII)
def predict(model, user: dict) -> float:
    logger.info("Predicting churn for user: %s", user)
    return model.predict_proba([list(user.values())])[0][1]

# GOOD: Log only the ID and the prediction
def predict(model, user_id: int, features: dict) -> float:
    score = model.predict_proba([list(features.values())])[0][1]
    logger.info("Churn prediction: user_id=%s score=%.4f", user_id, score)
    return score
```

#### 4. Document legal basis if any PII is intentionally retained

```python
# If you genuinely need zip_code, document why
FEATURE_REGISTRY = {
    "order_count":      {"pii": False, "rationale": "primary churn signal"},
    "lifetime_value":   {"pii": False, "rationale": "engagement proxy"},
    "last_order_date":  {"pii": False, "rationale": "recency signal"},
    "zip_code":         {"pii": True,  "rationale": "regional demand pattern",
                         "legal_basis": "legitimate interest — Art. 6(1)(f) GDPR",
                         "review_date": "2026-12-01"},
}
```

---

### The clean version

```python
# churn_pipeline.py
import pandas as pd
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.model_selection import train_test_split
import mlflow
import logging

logger = logging.getLogger(__name__)

# C1: Only the features the model actually needs
FEATURES = ["order_count", "lifetime_value", "days_since_last_order", "account_age_days"]


def load_training_data(db_conn) -> pd.DataFrame:
    query = """
        SELECT
            COUNT(o.id)                                             AS order_count,
            COALESCE(SUM(o.total), 0)                              AS lifetime_value,
            EXTRACT(DAY FROM NOW() - MAX(o.created_at))            AS days_since_last_order,
            EXTRACT(DAY FROM NOW() - u.created_at)                 AS account_age_days
        FROM users u
        LEFT JOIN orders o ON o.user_id = u.id
        GROUP BY u.id
    """
    df = pd.read_sql(query, db_conn)
    # D2: Log shape, not records
    logger.info("Loaded training data: rows=%d columns=%s", len(df), list(df.columns))
    return df


def train(df: pd.DataFrame) -> GradientBoostingClassifier:
    X = df[FEATURES]
    y = (df["days_since_last_order"] > 90).astype(int)

    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

    model = GradientBoostingClassifier()
    model.fit(X_train, y_train)

    # D2: Log feature names, not values
    mlflow.log_param("features", FEATURES)
    mlflow.log_metric("accuracy", model.score(X_test, y_test))

    return model


def predict(model, user_id: int, features: dict) -> float:
    score = model.predict_proba([list(features.values())])[0][1]
    logger.info("Churn prediction: user_id=%s score=%.4f", user_id, score)
    return score
```

### Rules applied

| Rule | What it caught |
|------|---------------|
| C1: Data minimization | PII fields used as features with no predictive justification |
| D2: PII logging | Full user records logged at training and inference time |
| C2: Explicit consent | No documented legal basis for sensitive field processing |

### ACM/IEEE reference

> *"Software engineers shall ensure that their products... protect the privacy of those affected."*
> — ACM/IEEE Software Engineering Code of Ethics, Principle 1.06
