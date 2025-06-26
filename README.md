# Indonesia Renewable Energy Transition Dashboard

---

## Project Overview

This project delivers an **interactive data dashboard** focused on **Indonesia's renewable energy (RE) transition**, with a specific emphasis on the government's **2025 targets**. Utilizing robust data science methodologies—including data acquisition, cleaning, feature engineering, and machine learning models—the dashboard aims to provide data-driven insights and projections into the country's progress toward a cleaner energy mix.

**Dashboard Title:** "Transisi Energi Terbarukan Indonesia: Seberapa Dekat Kita dengan Target 2025?"

---

## Key Questions Addressed

This dashboard seeks to answer critical questions regarding Indonesia's energy transition:

- How does the **rate of increase in renewable energy's share** in the national/regional energy mix compare to government targets (e.g., the **23% RE target by 2025** in Indonesia, or other global targets)?
- Are there specific types of renewable energy that show **significant acceleration or deceleration**?

---

## Core Metrics Analyzed

The dashboard visualizes and analyzes the following key metrics:

- **Total Clean Energy Generation (per Energy Type):** Volume of electricity generated (in TWh) by various renewable sources (Solar, Wind, Hydro, Biofuel) and non-renewable sources (Coal, Gas, Oil), as well as total electricity generation.
- **Renewable Energy Share in National/Regional Energy Mix (%):** The percentage contribution of renewable energy to the total electricity generation in Indonesia. This is a direct metric for measuring energy transition progress against government targets.
- **Annual Growth Rate (YoY Change):** The year-over-year percentage change for total RE generation or per RE type. This metric indicates the speed of RE development and identifies the fastest-growing RE types.
- **Carbon Intensity of Electricity:** Emissions per unit of electricity (gCO₂/kWh), serving as an indicator of transition success.
- **Electricity Consumption Per Capita:** Access and consumption of electricity per person.

---

## Project Structure

The project follows a well-organized folder structure to maintain code readability and manageability:

```
Project_Indonesia_RE_Dashboard/
├── dashboard/
│   └── app.py
├── data/
│   ├── indo_energy_filled.csv
│   └── indo_energy_prepared.csv
├── models/
│   ├── model_renewables_share_elec_rf.pkl
│   ├── model_renewables_yoy_rf.pkl
│   └── model_fossil_yoy_rf.pkl
├── assets/
├── requirements.txt
└── README.md
```

- **`dashboard/`**: Contains all Python code related to the Dash application, including the main `app.py` file that defines the layout and logic of the dashboard.
- **`data/`**: Stores prepared and cleaned dataset files (e.g., `indo_energy_filled.csv`, `indo_energy_prepared.csv`).
- **`models/`**: Dedicated to storing trained and saved Machine Learning models (`.pkl` files), allowing the dashboard to load these for predictions.
- **`assets/`**: Generally used for static files like custom CSS, images, or fonts.
- **`requirements.txt`**: Lists all required Python libraries (e.g., pandas, plotly, dash, scikit-learn) for easy dependency installation.
- **`README.md`**: This main project documentation file.

---

## Data Source & Preparation (Phase 2 & 3 Highlights)

The analysis is based on **Our World in Data (OWID) historical energy data for Indonesia (1985–2023)**.

### Data Acquisition & Initial Exploration

The `owid-energy-data.csv` file was downloaded, and data was filtered specifically for Indonesia. Initial exploration using Pandas (`.head()`, `.info()`, `.describe()`, `.isnull().sum()`) was performed to understand the dataset's structure, identify data types, and check for missing values. The relevant data for Indonesia spans from 1985 to 2023.

### Data Cleaning & Feature Engineering

- The dataset was filtered to years **1985–2023** where data was more consistent.
- Missing values in `biofuel_electricity` were filled with `0` based on domain knowledge, assuming no significant biofuel generation in early years where data was absent.
- Missing values in `carbon_intensity_elec` were imputed using a **Random Forest Regressor** model. This model achieved a Mean MAE of 2.39 and MSE of 11.40 on the training data, indicating good performance for imputation.
- New features were engineered, including:
  - `renewables_yoy_growth` and `fossil_yoy_growth` (annual growth rates).
  - `share_solar_in_renew`, `share_wind_in_renew`, `share_hydro_in_renew` (share of each renewable source within total RE generation).
  - Additional engineered features such as `renewables_ratio_gen`, `fossil_ratio_gen`, `log_per_capita_elec`, `carbon_x_fossil` were created to improve model performance for `renewables_yoy_growth` prediction.
- Outliers were detected but retained in the dataset, as they represent real-world phenomena from the OWID source.
- The processed dataset is saved as `indo_energy_filled.csv`.

---

## Modeling & Evaluation (Phase 4 & 5 Highlights)

Various machine learning models were developed and rigorously evaluated to generate predictions and derive key insights.

### 1\. Prediction of Renewables Share in Electricity

- **Objective:** To predict Indonesia's RE share in electricity generation by 2025 and assess proximity to the 23% target.
- **Models Used:** Linear Regression, Polynomial Regression (degree 2), and Random Forest Regressor.
- **Best Model (Single Predictor - year):** **Random Forest** showed the best performance with MAE: 0.58, MSE: 0.68, and R²: 0.93.
- **Prediction for 2025:** The model predicts `renewables_share_elec` to be approximately **18.68% by 2025**. This highlights a **4.32% gap** to the government's 23% target.

### 2\. Prediction of YoY Growth for Renewables & Fossil

- **Objective:** To project the annual growth rate of RE and fossil energy.
- **Models Used:** Linear, Polynomial, and Random Forest.
- **Best Model:** **Random Forest** consistently performed best for both targets.
  - `renewables_yoy_growth`: MAE: 6.08, MSE: 71.98.
  - `fossil_yoy_growth`: MAE: 1.67, MSE: 6.10.
- **Prediction for 2025:** `renewables_yoy_growth` is predicted to be around **11.29%**.
- **Generalization Challenges:** TimeSeriesSplit Cross-Validation revealed that while in-sample performance was good, generalization (R² scores) for `renewables_yoy_growth` remained low/negative, indicating inherent volatility and difficulty in accurately predicting this metric with limited data.

### 3\. Prediction of Specific RE Types (Solar, Wind, Hydro)

- **Objective:** To identify acceleration or deceleration in specific RE types.
- **Models Used:** Linear, Polynomial, and Random Forest.
- **Best Model:** **Random Forest** achieved very high R² scores for all three, effectively capturing their historical trends:
  - Solar Electricity: R² = 0.98.
  - Wind Electricity: R² = 0.99.
  - Hydro Electricity: R² = 0.99.

### 4\. Model Interpretability & Feature Importance (SHAP, PDP, Lasso/Ridge, EBM)

- **Objective:** To understand which factors most influence `renewables_yoy_growth` and to ensure model transparency.
- **Key Findings:**
  - **Consistently Important Features across Random Forest (SHAP), Lasso, Ridge, and EBM:**
    - `carbon_intensity_elec` (Negative impact)
    - `renewables_share_energy` (Positive impact)
    - `share_hydro_in_renew` (Positive impact)
    - `fossil_share_elec` (Negative impact)
    - `fossil_yoy_growth` (Negative impact)
  - These features are crucial for understanding the dynamics of RE growth and formulating actionable policies.
  - Partial Dependence Plots (PDP) confirmed non-linear relationships for these key features with `renewables_yoy_growth`.
  - Residual Plots for Random Forest showed some non-random patterns and skewness, suggesting the model still slightly underestimates the target but is generally stable.

### 5\. Model Reliability & Generalization

- **Confidence Intervals:** Predictions for `renewables_yoy_growth` include a risk range (e.g., 2025: 11.23% ± 0.82%), acknowledging inherent uncertainty.
- **Generalization to 2024-2025:** The models are considered "sufficiently generalized" to future years. While not perfectly accurate due to data volatility, they do not overfit and provide reasonable, stable projections based on historical trends (CAGR-based extrapolation of future features).

### 6\. Model Persistence

- Trained models are saved using `joblib` (e.g., `model_renewables_share_elec_rf.pkl`, `model_renewables_yoy_rf.pkl`, `model_fossil_yoy_rf.pkl`) for easy loading and use in the dashboard.

---

## Dashboard Design & Key Insights (Phase 6 & Final Narrative)

The dashboard is structured with multiple tabs for clear navigation and comprehensive insights:

- **Main Title:** Transisi Energi Terbarukan Indonesia: Seberapa Dekat Kita dengan Target 2025?
- **Subtitle:** Analisis mendalam dan proyeksi berbasis data dari pembangkitan energi terbarukan di Indonesia.

### Tabs & Core Narrative

#### 1\. Ringkasan & Target (Overview & Targets)

- **Goal:** Provide an overview of RE trends in Indonesia and direct comparison with government targets.
- **Key Insight:** Indonesia's RE share in electricity generation is predicted to reach \~18.68% by 2025, falling short of the 23% government target by 4.32%.
- **Call to Action:** Prioritize policies and investments to accelerate the clean energy mix.

#### 2\. Faktor Pendorong (Growth Drivers)

- **Goal:** Explain key factors influencing annual RE growth based on model interpretability.
- **Key Insight:** Model highlights `carbon_intensity_elec`, `fossil_yoy_growth`, and `fossil_share_elec` as negative drivers, while `renewables_share_energy` and `share_hydro_in_renew` are positive drivers.
- **Call to Action:** Focus on increasing RE share in total energy and reducing carbon intensity and fossil growth.

#### 3\. Simulasi & Skenario (What-If Analysis)

- **Goal:** Allow users to test the impact of changes in specific features on RE growth predictions.
- **Key Insight:** Simulations show that a 1% increase in `renewables_share_energy` can boost YoY Growth by 0.5–0.7%.
- **Call to Action:** Use these insights for policy simulations, new plant targets, and energy transition roadmaps.

#### 4\. Keandalan Prediksi (Model Reliability)

- **Goal:** Communicate the model's confidence levels and limitations.
- **Key Insight:** The model has an uncertainty of ±0.82% for 2025 annual RE growth predictions, reflecting high volatility in the `renewables_yoy_growth` variable. Residual plots indicate some non-linear patterns not fully captured.
- **Call to Action:** Use predictions as directional indicators, not absolute figures, always combining with policy considerations and external factors.

#### 5\. Metodologi & Sumber (Methodology & Sources)

- **Goal:** Provide transparency on data used, analytical processes, and model validation.
- **Content:** Summary of project phases, data sources (OWID), data preparation (cleaning, missing value handling, feature engineering), modeling & validation (Random Forest as main model, TimeSeriesSplit CV), and main insights.
- **Key Insight:** Despite `renewables_yoy_growth` prediction performance not being perfect (low R²), the consistency of important features provides a strong basis for policy recommendations. The 23% RE target by 2025 is likely unattainable at the current growth rate.

---

## Technologies Used

- **Python:** Core programming language.
- **Pandas:** Data manipulation and analysis.
- **Plotly & Dash:** For interactive dashboard development and visualizations.
- **Dash Bootstrap Components (dbc):** For responsive and modern dashboard layouts.
- **Scikit-learn:** Machine learning models (Linear Regression, Polynomial Features, Random Forest Regressor, StandardScaler, LassoCV, RidgeCV).
- **Joblib:** For saving and loading trained models.
- **Numpy:** Numerical operations.
- **Shap:** For model explainability (SHAP values).
- **InterpretML (Explainable Boosting Machine - EBM):** For interpretable generalized additive models.
- **Matplotlib & Seaborn:** For static data visualizations.

---

## How to Run the Dashboard

To run this dashboard locally, follow these steps:

### Prerequisites

- Python 3.8+ (recommended Python 3.9 or higher).
- Git (for cloning the repository).

### 1\. Clone the Repository

```bash
git clone https://github.com/your-username/Indonesia-RE-Transition-Dashboard.git # Replace with your actual repo URL
cd Indonesia-RE-Transition-Dashboard
```

### 2\. Set Up a Virtual Environment

It's highly recommended to use a virtual environment to manage project dependencies.

```bash
python3 -m venv .venv
```

### 3\. Activate the Virtual Environment

- **On Windows:**
  ```bash
  .venv\Scripts\activate
  ```
- **On macOS/Linux:**
  ```bash
  source .venv/bin/activate
  ```

Once activated, you should see `(.venv)` at the beginning of your terminal prompt.

### 4\. Install Dependencies

Install all the required Python libraries using the `requirements.txt` file:

```bash
pip install -r requirements.txt
```

### 5\. Run the Dashboard

Navigate into the `dashboard` folder and run the `app.py` file:

```bash
cd dashboard
python app.py
```

The dashboard will typically run on your local machine and provide a URL (e.g., `http://127.0.0.1:8050/`). Open this URL in your web browser to view the interactive dashboard.

---

**Note:** This `README.md` provides a comprehensive overview of the project. For more detailed code and analysis, please refer to the respective files within the repository.

---

**Author:** [JAMALLUDIN/https://github.com/Halozy-bit]
**Contact:** [Jamalludin1999z@gmail.com/https://www.linkedin.com/in/jamalludin-449009195/]
