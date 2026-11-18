# YFinance-ETL

An automated ETL pipeline that pulls stock data from YFinance after every Friday market close using a GitHub Action, stores it in a Supabase database, and feeds a Power BI dashboard that compares an equal-weight stock portfolio against the S&P 500.

## Overview

- **Extract:** pulls OHLC and company data for multiple tickers with `yf.download()`
- **Transform:** flattens YFinance's multi-index pandas output into a database-ready format
- **Load:** appends new rows to Supabase with pandas `to_sql()`
- **Schedule:** a GitHub Action runs the pipeline after market close on Fridays
- **Analyze:** Power BI imports the Supabase tables and view for the dashboard

## Pipeline

The pipeline uses an Extract, Transform, Load (ETL) pattern, which suits the OHLC and company data going into the Supabase database.

1. **Extract:** The YFinance library makes it easy to gather data on multiple tickers at once with `yf.download()`. Before downloading, the pipeline queries the database for the most recent date and only appends data after that date. If the database is empty, it loads roughly the past 4 years of data, along with company data beyond just the ticker (name, sector, industry).
2. **Transform:** `yf.download()` returns a pandas DataFrame with a multi-index structure, so it is flattened before loading.
3. **Load:** The flattened data is written to the database using pandas `to_sql()`.
4. **Logging:** A logger records and reports successes and failures at each stage (extracting, transforming, loading). This output appears in the GitHub Action run logs.

## Database

The Supabase database has three tables/views:

| Table | Type | Columns |
|---|---|---|
| `portfolio_ohlc_data` | Table | `id` (key), `date`, `ticker`, `open`, `high`, `low`, `close`, `volume` |
| `company_data` | Table | `id` (key), `ticker`, `name`, `sector`, `industry` |
| `v_daily_return` | View | `date`, `ticker`, `close`, `prev_close`, `daily_return` |

`v_daily_return` is a view created within Supabase that acts as a calculated table, showing each ticker's daily return compared with the previous day's close.

## Power BI Dashboard

The dashboard analyzes an **equal-weight portfolio of 8 stocks** (AMZN, CAT, JPM, MSFT, PG, PLD, UNH, XOM) against the **SPY (S&P 500 ETF Trust)** benchmark. It covers 1/3/2022 through 10/22/2026.

### Page 1: Portfolio Performance and Comparison

***IMAGE 1***

The overview page shows the portfolio and benchmark side by side:

- Portfolio total return of **92.34%** vs. benchmark return of **70.38%**, a difference of **+21.95 percentage points**
- Portfolio and benchmark cumulative returns plotted over time
- Initial and most recent return dates, plus the number of days covered
- Lists of the portfolio holdings and the benchmark

### Page 2: Risk and Return Breakdown

***IMAGE 2***

This page looks at what drove the portfolio's return and how much risk it took:

- **Total return by ticker** compared against the benchmark return line
- **Annualized volatility vs. annualized return** scatter plot for each ticker, with SPY as the reference point
- **Volatility metrics:** portfolio volatility (16.11%), average holding volatility (28.41%), and benchmark volatility (17.26%)
- **Diversification benefit:** 43.30%
- **Concentration of gains:** the top 2 tickers account for 67.47% of the total gain, and the average return without them is 40.05%
- A ticker slicer that updates the selected total return (defaults to all holdings)

### Page 3: Holdings Over Time Frames

***IMAGE 3***

This page compares each holding to the benchmark across multiple time frames:

- YTD, 1 month, 3 month, 1 year, 2 year, and 3 year returns per ticker
- Conditional coloring: **blue** = above the benchmark baseline, **red** = below the benchmark baseline
- Portfolio and benchmark rows for direct comparison
- A year selector (2022 to 2026) that drives the **rolling return** and **total return** line charts for the selected year

## Tech Stack

- **Python:** pandas, yfinance
- **GitHub Actions:** scheduled automation
- **Supabase:** PostgreSQL database
- **Power BI:** data modeling, DAX measures, and visualization
