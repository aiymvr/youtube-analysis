# Comparative Analysis of Engagement Dynamics in YouTube Shorts and Long-Form Videos

A data science research notebook investigating how audience engagement on YouTube differs between **Shorts** (≤60s) and **long-form videos**, and how that engagement evolves and decays over time.

## Research Question

> How does the time until a YouTube video reaches peak engagement differ between Shorts and long-form content, and how does engagement evolve over time following that peak?

## What This Notebook Does

The analysis is organized into five stages:

1. **Data Collection & Preprocessing** — Pulls video data from the YouTube Data API v3, cleans it, and engineers engagement features.
2. **Exploratory Data Analysis (EDA)** — Visualizes distributions, upload patterns, and the views/engagement relationship.
3. **Bayesian Regression** — Models how video characteristics (type, duration, likes, comments, upload timing, age) influence view counts.
4. **Time Series Analysis** — Examines how daily engagement rates evolve for each video type using stationarity tests and ARIMA models.
5. **Survival Analysis** — Treats "engagement decay below a threshold" as an event, using Kaplan-Meier curves, a log-rank test, and a Cox Proportional Hazards model to compare how long Shorts vs. long-form videos sustain engagement.

## Data Collection

- Video IDs are gathered by querying the YouTube Search API across 16 topic categories (`gaming`, `technology`, `education`, `music`, `podcast`, `sports`, `news`, `entertainment`, `science`, `movies`, `finance`, `tutorial`, `travel`, `fitness`, `food`, `coding`), fetching up to 100 results per query.
- Duplicate video IDs are removed, then full metadata (snippet, statistics, content details) is retrieved in batches of 50 via the Videos API.
- Each video is labeled `Short` (duration ≤ 60s) or `Long` based on its ISO 8601 duration, parsed with `isodate`.

## Data Cleaning & Feature Engineering

- Drops duplicate rows, missing values, and videos with zero views.
- Removes the top 1% of videos by view count as extreme outliers.
- Derives:
  - `upload_year`, `upload_month`, `upload_day`, `upload_hour` from the publish timestamp
  - `like_view_ratio`, `comment_view_ratio`, `engagement_rate` = (likes + comments) / views
  - `days_since_upload`
- Saves the cleaned dataset to `youtube_shorts_vs_long_cleaned.csv`.

## Modeling Approaches

| Stage | Method | Purpose |
|---|---|---|
| Regression | Bayesian linear regression (PyMC) | Estimate the effect of video type, duration, likes, comments, upload hour, and age on log-transformed views |
| Time Series | ADF stationarity tests, ACF/PACF plots, ARIMA | Model how average daily engagement rate evolves for Shorts vs. long-form videos |
| Survival | Kaplan-Meier estimator, log-rank test, Cox Proportional Hazards | Compare how quickly engagement decays below a threshold between the two video types, and quantify hazard ratios |

## Requirements

- Python 3.x (developed for Google Colab)
- A YouTube Data API v3 key, provided via Colab's `userdata.get('YOUTUBE_API_KEY')` (or set as an environment variable if adapting for local use)

### Python packages

```
pandas
numpy
isodate
google-api-python-client
matplotlib
pymc
arviz
statsmodels
lifelines
```

Install with:

```bash
pip install pandas numpy isodate google-api-python-client matplotlib pymc arviz statsmodels lifelines
```

## Running the Notebook

1. Open `Youtube.ipynb` in Google Colab (or Jupyter).
2. Add your YouTube Data API v3 key as a Colab secret named `YOUTUBE_API_KEY` (Colab → Secrets), or adapt the API key loading if running elsewhere.
3. Run the cells in order:
   - Data collection cells will call the YouTube API and may take a few minutes depending on quota and query count.
   - The cleaned dataset is cached to `youtube_shorts_vs_long_cleaned.csv` and reloaded later in the Time Series section, so you can skip re-collecting data on subsequent runs if that file exists.
4. Review the plots and printed statistical summaries (regression coefficients, ADF/ARIMA results, Kaplan-Meier curves, Cox hazard ratios) generated throughout.

## Key Outputs

- `youtube_shorts_vs_long_cleaned.csv` — cleaned, feature-engineered dataset of collected videos
- Distribution plots (video type counts, view distribution, uploads by hour)
- Views vs. engagement rate scatter plot (Shorts vs. long-form)
- Bayesian posterior summaries and trace/posterior plots
- ARIMA model summaries and AIC/BIC comparison for Shorts vs. long-form engagement time series
- Kaplan-Meier survival curves, median survival times, and log-rank test results
- Cox Proportional Hazards model summary and hazard ratio plot

## Notes & Caveats

- The YouTube Data API has daily quota limits; collecting up to ~1,600 videos across 16 queries can consume a significant portion of the default quota.
- View counts are heavily right-skewed; the top 1% are filtered out, and a log transform (`log1p`) is used for the regression target.
- The "event" in the survival analysis is defined as normalized engagement dropping below a fixed threshold (0.2) relative to each video type's maximum — this threshold is a modeling choice and can be adjusted to test sensitivity.
