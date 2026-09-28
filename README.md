# NIFTY ML Analysis 
LIVE LINK = user id = vishal
password = navin
https://stock-ml-platform.onrender.com

## Product overview

NIFTY ML Analysis is a browser-based market research tool for reviewing NIFTY index open-interest (OI) signals alongside similar observations from a historical dataset. A user enters five market values and receives an OI signal, a K-nearest-neighbour (KNN) comparison, and supporting analysis in one dashboard.

The product is decision support for research and informational use. It is not financial advice, does not execute trades, and does not guarantee a market direction or outcome.

## User problem

Reviewing current call/put open interest and comparing it with past market observations can require switching between data and calculations. This tool brings those inputs, a rule-based OI view, and the three closest historical observations together so a user can review the reasoning and uncertainty in one place.

## Product management case study

**Target user hypothesis:** A retail-market participant who already follows NIFTY options and wants a quick, understandable way to compare current OI inputs with a small set of historical examples. This is a product hypothesis, not a user-research finding; interviews and usability testing have not been conducted for this project.

**User need:** “Help me review the current OI picture alongside relevant historical context without hiding conflicting signals or risk.”

**Product goal:** Make one analysis workflow understandable and usable from input entry through evidence review. The current project delivers a working dashboard and API; it does not yet establish that this goal improved user outcomes.

**MVP decisions and trade-offs:**

- Keep the first version focused on five manually entered values and one analysis response, rather than expanding into trading, accounts, or portfolio management.
- Pair a current-input OI signal with a three-neighbour historical KNN comparison to provide two views, while explicitly surfacing disagreements.
- Show similar observations and risk context to make the output more inspectable; keep the HIGH/MEDIUM label tied to signal agreement, not a claimed probability.
- Use a small local CSV and a lightweight Python API to keep the prototype runnable. This makes data coverage and model reliability important limitations, not solved problems.
- Defer live market data, saved history, analytics instrumentation, and authentication enforcement; these are not implemented.

**Next product validation steps:** Interview target users about their current OI-analysis workflow; test whether they understand the inputs, signal conflict, and risk sections; then instrument the completion and time-to-analysis KPIs below. Evaluate directional performance separately with a chronological holdout before making any predictive-accuracy claims.

## Product flow

```text
User opens dashboard
        |
        v
Enters NIFTY price + call/put OI changes + total call/put OI
        |
        v
Dashboard validates required numeric inputs
        |
        v
POST /api/predict  -------------------->  Backend loads train.csv
        |                                         |
        |                                         v
        |                             OI-flow signal + 3-nearest KNN
        |                                         |
        |                                         v
        |                             Compare signals and derive analysis
        |                                         |
        <------------- JSON result ---------------+
        |
        v
Dashboard shows signals, historical matches, analysis outputs, and risk context
        |
        v
User reviews evidence and makes an independent research decision
```

## Features

- Browser dashboard with responsive layouts for desktop, tablet, and mobile.
- Manual entry and validation for current NIFTY price, call and put OI changes, and total call and put OI.
- Primary OI-flow signal and a KNN signal based on the three closest rows in `train.csv`.
- Final signal with a **HIGH** or **MEDIUM** agreement label. This label describes whether the two signals agree; it is not a calibrated probability of success.
- Expected price change and target price derived from the selected historical observations.
- Supporting views for similar candles, key drivers, put-call ratio (PCR), OI flow, risk, invalidation conditions, a model-generated strategy, and smart-money OI analysis.
- JSON prediction endpoint (`POST /api/predict`) and health-check endpoint (`GET /health` or `GET /api/health`).
- Command-line analysis and a deployable Python web service configuration.

## User benefits

- **Faster review:** brings the current inputs and related model outputs into a single workflow.
- **More context:** shows both the current-input OI signal and the historical KNN signal, including when they disagree.
- **More transparent analysis:** exposes similar past observations and the inputs that drive the comparison rather than presenting only a final signal.
- **Risk awareness:** surfaces historical outcomes, risk context, and conditions that would invalidate the setup.
- **Flexible access:** the dashboard adapts to common screen sizes, and the API can be used by other clients.

These are intended product benefits, not measured or guaranteed user outcomes.

## KPIs and product metrics

The project does not currently persist usage events, collect user feedback, or record realized trade outcomes. The metrics below are recommended for evaluation; no targets or performance improvements are claimed.

| KPI | Definition | Why it matters |
|---|---|---|
| Analysis completion rate | Successful analyses ÷ analysis form submissions | Measures whether users can complete the core workflow. |
| Time to analysis | Median time from starting input entry to a rendered result | Measures workflow speed. |
| Input validation error rate | Submissions with invalid or missing values ÷ all submissions | Identifies input friction and confusing fields. |
| Signal disagreement rate | Analyses where OI and KNN signals differ ÷ successful analyses | Indicates how often the dashboard needs to communicate conflicting evidence. |
| Repeat analysis rate | Users who complete another analysis in a defined period ÷ users who completed one | Indicates whether the workflow supports recurring research needs. |
| Directional accuracy | Held-out observations where the signal direction matches the subsequent NIFTY move ÷ eligible held-out observations | Evaluates predictive performance without using the same observations for training and evaluation. |
| Signal coverage | Eligible evaluation observations that receive a usable signal ÷ all eligible observations | Shows the share of cases for which the model can be evaluated. |

Before setting KPI targets, instrument privacy-conscious events and establish a baseline. Evaluate model metrics on a chronological holdout with documented data quality checks; the current app does not calculate or verify backtest accuracy.

## Current project metrics and scope

- **User inputs per analysis:** 5 required values.
- **Historical comparisons per prediction:** 3 nearest observations.
- **Historical data file:** `train.csv` currently contains 100 data rows. This is a small project dataset, not evidence of model reliability.
- **Model version reported by the API:** `1.0.0`.
- **Usage analytics, saved analysis history, and realized-outcome tracking:** not implemented.
- **Live market-data feed or automatic refresh:** not implemented; values are entered manually.

## Run the dashboard

Install the dependency and start the local server:

```bash
pip install -r requirements.txt
python app.py
```

Open `http://localhost:8000/` in a browser. Enter the five values and select **ANALYZE NIFTY**.

### Open it on a phone or tablet

Run `python app.py` on the computer hosting the project, keep the phone or tablet on the same Wi-Fi, and open the computer's local IPv4 address with port `8000`, for example:

```text
http://192.168.29.154:8000/
```

The interface adapts to phone, tablet, and desktop widths. The dashboard uses relative API requests, so the same LAN address works for the dashboard and prediction requests.

## Run the prediction flow from the command line

Interactive CLI:

```bash
python predict.py
```

Pass a JSON payload directly:

```bash
python predict.py --json '{"current_nifty":23352,"call_oi_change":2142000,"put_oi_change":587000,"total_call_oi":230200000,"total_put_oi":254200000}'
```

Start the JSON API directly:

```bash
python app.py
```

Then send a `POST` request to `/api/predict` with the same JSON payload. The root URL serves the dashboard, and `/health` and `/api/health` return server status.

## Data and interpretation limitations

- Predictions depend on the contents and quality of `train.csv`; the current dataset is limited in size.
- KNN compares four OI features and uses the three nearest historical rows. A small or unrepresentative dataset can produce unstable comparisons.
- The HIGH/MEDIUM label reflects signal agreement only. It must not be interpreted as a measured confidence percentage.
- Expected movement and strategy values are model-generated from nearby historical rows and may not reflect live market conditions, execution costs, or future results.
- The dashboard does not fetch live market data, place orders, or retain a performance history.
- The dashboard includes a sign-in screen, but the current backend does not enforce authentication. Do not use that screen as an access-control or security boundary.

Use the analysis for research and informational purposes only. It is not financial advice and does not guarantee market direction, profit, or future performance.
