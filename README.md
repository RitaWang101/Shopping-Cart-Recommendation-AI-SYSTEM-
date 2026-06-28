# Shopping Cart Recommendation AI System

## Overview

This project is an AI-powered grocery shopping and budget recommendation system. It simulates an intelligent shopping assistant that helps users browse grocery products, manage a shopping cart, track spending against a budget, and receive personalized product substitution recommendations when the cart goes over budget.

The system combines three recommendation approaches:

1. **Budget-Saving Recommendation System**
   Uses semantic similarity to identify cheaper alternatives to expensive cart items.

2. **Personalized Collaborative Filtering System**
   Uses user purchase history and deep learning embeddings to recommend products based on behavioral patterns.

3. **Hybrid AI Recommendation System**
   Combines collaborative filtering and semantic similarity using a weighted ranking strategy to generate recommendations that are both personalized and context-aware.

The project also includes two evaluation frameworks: traditional recommendation metrics and LLM-as-a-Judge evaluation.

---

## Key Features

* Interactive grocery shopping web application
* Product catalog loaded from grocery CSV data
* Budget tracking and over-budget detection
* Cart-aware product replacement recommendations
* Semantic similarity-based budget substitutions
* Personalized collaborative filtering recommendations
* Hybrid recommendation engine using 60% CF and 40% semantic similarity
* Session-based anonymous user tracking
* Purchase history storage for personalization
* SQLAlchemy database models for users, orders, products, and events
* Traditional recommendation evaluation metrics
* LLM-as-a-Judge evaluation using OpenAI API
* Evaluation outputs saved as CSV, JSON, Markdown, and log files

---

## System Architecture

### Backend

The backend is built with Flask and SQLAlchemy. It provides API endpoints for product browsing, budget recommendations, personalized recommendations, hybrid recommendations, and checkout.

Main backend components:

* `main.py`
  Flask application entry point. Handles product loading, recommendation APIs, session tracking, cart checkout, and route definitions.

* `models.py`
  SQLAlchemy database models for products, users, budgets, orders, order items, shopping carts, and user behavior events.

* `import_csv.py`
  Imports grocery product data from CSV into the database and parses price, rating, review count, and nutrition fields.

---

## Recommendation Systems

### 1. Budget-Saving Semantic Recommendation

The budget-saving system recommends cheaper product alternatives when the user’s cart exceeds the budget.

Main file:

* `semantic_budget.py`

Core logic:

* Loads grocery product data.
* Builds product text representations using title, category, description, and features.
* Uses sentence-transformer embeddings to calculate semantic similarity.
* Finds cheaper substitutes for expensive cart items.
* Prioritizes same-category or semantically similar products.
* Returns expected savings and explanation text.

This approach is useful for users who want to reduce cart total while keeping product choices similar.

---

### 2. Personalized Collaborative Filtering

The collaborative filtering system uses user purchase behavior to generate personalized recommendations.

Main files:

* `recommendation_engine.py`
* `train_cf_model.py`
* `cf_inference.py`

Core logic:

* Extracts user-product interaction data from purchase history and user events.
* Converts purchases, cart actions, and views into implicit feedback signals.
* Trains a TensorFlow/Keras collaborative filtering model.
* Uses user embeddings and product embeddings to learn user-product preference patterns.
* Supports cold-start behavior for new or unknown users.
* Saves trained model files and mappings under `ml_data/`.

Model design:

* User embedding layer
* Product embedding layer
* Dot-product interaction
* Sigmoid output for implicit preference score
* Negative sampling for training
* Evaluation using recommendation metrics such as Precision@K, Recall@K, MAP@K, and AUC

This approach is useful when the system has enough user purchase history to learn personalized preferences.

---

### 3. Hybrid AI Recommendation System

The hybrid AI system combines collaborative filtering and semantic similarity.

Main file:

* `blended_recommendations.py`

Core logic:

* Retrieves CF recommendations for the current user.
* Builds a semantic user profile from purchase history.
* Computes semantic similarity between the user profile and candidate products.
* Combines scores using weighted blending:

```python
blended_score = 0.6 * cf_score + 0.4 * semantic_score
```

The hybrid system is designed to balance:

* Personalization from collaborative filtering
* Product-context relevance from semantic similarity
* Budget-friendly substitutions
* Better ranking quality than using either method alone

---

## Web Application Workflow

1. User opens the grocery shopping web app.
2. The app loads grocery products from the product catalog.
3. User adds products to the cart.
4. User sets a shopping budget.
5. If cart total exceeds the budget, the app triggers recommendation systems.
6. The system returns cheaper alternatives.
7. User can apply a recommended replacement.
8. Checkout stores the order and purchase history.
9. Purchase history improves future personalized recommendations.

---

## API Endpoints

### Product Catalog

```http
GET /api/products
```

Returns paginated grocery products and category filters.

Optional query parameters:

```text
subcat
limit
skip
```

---

### Budget-Saving Recommendations

```http
POST /api/budget/recommendations
```

Request body:

```json
{
  "cart": [
    {
      "title": "Example Product",
      "subcat": "Snacks",
      "price": 12.99,
      "qty": 1
    }
  ],
  "budget": 50
}
```

Returns semantic similarity-based cheaper substitutes.

---

### Personalized CF Recommendations

```http
GET /api/cf/recommendations
POST /api/cf/recommendations
```

The GET endpoint returns general personalized recommendations.

The POST endpoint returns cart-aware cheaper alternatives based on collaborative filtering.

---

### Hybrid AI Recommendations

```http
GET /api/blended/recommendations
POST /api/blended/recommendations
```

The GET endpoint returns general hybrid AI recommendations.

The POST endpoint returns cart-aware cheaper alternatives using both personalization and semantic similarity.

---

### Checkout

```http
POST /api/checkout
```

Stores completed orders, order items, and user purchase events for future recommendation training.

---

## Evaluation Framework

This project includes two types of recommendation evaluation.

---

### 1. Traditional Metrics Evaluation

Main files:

* `traditional_evaluation_metrics.py`
* `evaluate_systems_traditional.py`
* `evaluate_captured_recommendations.py`
* `TRADITIONAL_METRICS_README.md`

Metrics include:

* Precision@K
* Recall@K
* NDCG@K
* Hit Rate@K
* Total potential savings
* Average savings per item
* Savings percentage
* Diversity score
* Category match
* Price appropriateness
* Catalog coverage
* Gini coefficient

Traditional metrics are useful because they are fast, repeatable, objective, and cost-free.

---

### 2. LLM-as-a-Judge Evaluation

Main files:

* `llm_judge_evaluation.py`
* `test_llm_evaluation.py`
* `demo_llm_evaluation.py`
* `LLM_EVALUATION_README.md`
* `FINAL_EVALUATION_SCORES.md`

The LLM evaluation framework compares recommendation systems using:

* Pairwise comparison
* Criteria-based scoring
* Scenario testing
* JSON-based output reports

Evaluation criteria include:

* Relevance
* Savings
* Diversity
* Explanation quality
* Feasibility
* User experience

Example test scenarios:

* Budget-conscious shopper
* Health-focused shopper
* New user / cold-start shopper

This evaluation approach is useful for assessing recommendation quality from a user experience perspective, not only from numerical metrics.

---

## Example Evaluation Results

A live traditional evaluation compared the three systems under a budget-conscious cart scenario.

| System          | Recommendations | Total Savings | Avg Savings / Item | Savings % | Diversity | Category Match |
| --------------- | --------------: | ------------: | -----------------: | --------: | --------: | -------------: |
| Personalized CF |               9 |        $50.60 |              $5.62 |     52.7% |      0.33 |           0.56 |
| Hybrid AI       |               8 |        $46.60 |              $5.83 |     48.6% |      0.38 |           0.62 |
| Budget-Saving   |               0 |         $0.00 |              $0.00 |      0.0% |      0.00 |           0.00 |

Key takeaway:

* Personalized CF generated the highest total savings.
* Hybrid AI achieved better category matching and stronger per-item savings.
* Budget-Saving needs further debugging or less aggressive filtering in some live-session scenarios.

---

## Project Structure

```text
Shopping-Cart-Recommendation-AI-SYSTEM-/
│
├── main.py
├── models.py
├── import_csv.py
├── recommendation_engine.py
├── train_cf_model.py
├── cf_inference.py
├── semantic_budget.py
├── blended_recommendations.py
│
├── traditional_evaluation_metrics.py
├── evaluate_systems_traditional.py
├── evaluate_captured_recommendations.py
├── evaluate_recommendations.py
├── run_proper_evaluation.py
│
├── llm_judge_evaluation.py
├── test_llm_evaluation.py
├── demo_llm_evaluation.py
│
├── build_history_and_evaluate.py
├── setup_test_users.py
│
├── captured_recommendations.json
├── evaluation_with_history_llm.json
├── evaluation_with_history_traditional.csv
├── live_session_traditional_results.csv
├── traditional_evaluation_results.csv
├── evaluation_scores_table.md
├── FINAL_EVALUATION_SCORES.md
│
├── LGBM_README.md
├── LLM_EVALUATION_README.md
├── TRADITIONAL_METRICS_README.md
├── requirements.txt
├── pyproject.toml
├── replit.md
│
├── attached_assets/
└── ml_data/
```

---

## Technologies Used

### Backend

* Python
* Flask
* Flask-SQLAlchemy
* SQLAlchemy
* PostgreSQL / SQLite-compatible configuration
* Pandas
* NumPy

### Machine Learning & AI

* TensorFlow / Keras
* Sentence Transformers
* PyTorch
* Scikit-learn
* LightGBM documentation and planned re-ranking extension
* OpenAI API for LLM-as-a-Judge evaluation

### Evaluation

* Traditional recommender-system metrics
* Precision@K
* Recall@K
* NDCG@K
* Hit Rate@K
* Cost savings metrics
* Diversity and category relevance metrics
* LLM-based qualitative scoring

### Frontend

* Flask templates
* HTML
* Tailwind CSS
* Interactive cart and recommendation UI

---

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/RitaWang101/Shopping-Cart-Recommendation-AI-SYSTEM-.git
cd Shopping-Cart-Recommendation-AI-SYSTEM-
```

### 2. Create a virtual environment

```bash
python -m venv venv
source venv/bin/activate
```

For Windows:

```bash
venv\Scripts\activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

---

## Environment Variables

Optional environment variables:

```bash
export FLASK_SECRET_KEY="your-secret-key"
export DATABASE_URL="sqlite:///grocery_app.db"
export OPENAI_API_KEY="your-openai-api-key"
```

`OPENAI_API_KEY` is only required for LLM-as-a-Judge evaluation.

---

## How to Run the Web App

```bash
python main.py
```

Then open:

```text
http://localhost:5000
```

---

## How to Train the Collaborative Filtering Model

Before training, the system needs user-product interaction data from completed purchases.

### 1. Run the app

```bash
python main.py
```

### 2. Complete several checkout sessions in the web app

This generates user purchase history and behavior events.

### 3. Extract and prepare recommendation data

```bash
python recommendation_engine.py
```

### 4. Train the CF model

```bash
python train_cf_model.py
```

The trained model and artifacts are saved under:

```text
ml_data/
```

---

## How to Run Traditional Evaluation

```bash
python evaluate_captured_recommendations.py
```

or:

```bash
python evaluate_systems_traditional.py
```

Outputs may include:

```text
live_session_traditional_results.csv
traditional_evaluation_results.csv
evaluation_with_history_traditional.csv
```

---

## How to Run LLM Evaluation

First set the OpenAI API key:

```bash
export OPENAI_API_KEY="your-openai-api-key"
```

Start the Flask app:

```bash
python main.py
```

Run a demo without API cost:

```bash
python demo_llm_evaluation.py
```

Run evaluation:

```bash
python test_llm_evaluation.py
```

or run a specific scenario:

```bash
python test_llm_evaluation.py budget_conscious
```

---

## Main Results and Insights

* The system successfully supports a full shopping workflow from browsing to checkout.
* Purchase history is stored and reused for personalization.
* The semantic budget system can recommend cheaper substitutes using product text similarity.
* The CF model learns user-product preference patterns from implicit feedback.
* The hybrid system combines personalization and semantic similarity for stronger recommendation ranking.
* Traditional evaluation shows that Personalized CF and Hybrid AI can generate meaningful savings.
* LLM-as-a-Judge evaluation helps assess explanation quality, feasibility, and user experience.
* Future improvements should focus on better price calibration, stronger category matching, and filtering out unrealistic substitutions.

---

## Future Improvements

* Improve budget-saving filtering so it consistently returns useful substitutes.
* Add stricter discount calibration to avoid unrealistic price gaps.
* Improve product category normalization.
* Add acceptance-rate tracking from user clicks.
* Add A/B testing between recommendation systems.
* Add model retraining automation.
* Add better cold-start handling for new users.
* Improve Hybrid AI weighting dynamically based on user behavior.
* Integrate LightGBM LambdaMART re-ranking once deployment dependencies are finalized.
* Add screenshots or demo GIFs to the README.

---

## Notes

Some model files and generated evaluation outputs are included for demonstration and reproducibility. In a production repository, generated cache files, session files, local model artifacts, and Python cache folders should be excluded using `.gitignore`.

---

## Author

Ruiqi Wang
Master of Science in Communication Data Science
University of Southern California

---

## Project Status

Completed functional prototype with working Flask application, recommendation APIs, collaborative filtering pipeline, hybrid recommendation logic, and evaluation framework.
