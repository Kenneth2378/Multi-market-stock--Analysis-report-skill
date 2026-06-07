---
name: a-share-stock-report
description: Generate stock analysis reports as PDFs for A-shares, B-shares, Hong Kong stocks, and US stocks, especially when the user asks to include recent three-month price action, valuation, fundamentals, news/sentiment, and inferred capital movements.
metadata:
  short-description: Multi-market stock PDF reports
---

# Multi-Market Stock Report

Use this skill when the user asks for stock analysis reports, especially PDF reports for A-shares, B-shares, Hong Kong stocks, and US stocks. It is designed for reports that combine fundamentals, valuation, technical analysis, recent three-month chart behavior, news/sentiment, and inferred `主力/资金动向`.

## Required stance

- Treat all market data as time-sensitive. Verify the latest available trading date, price, valuation, announcements, and financial results before writing the report.
- Do not present `主力动向` or `机构资金动向` as confirmed account-level behavior. Phrase it as inference from observable public signals: price, volume, turnover, amplitude, gaps, high-volume up/down days, short interest, institutional ownership, support/resistance, and market context.
- Include a clear investment disclaimer in every report: `本文仅供研究参考，不构成投资建议，也不构成任何买入、卖出或持有建议。`
- Use concrete dates and market time zones. If the current day is not a trading day or latest data is from the prior trading day, state that exact date.
- State the market, currency, and data source口径 for each stock. Examples: A股人民币, B股美元/港元, 港股港元, 美股美元.
- Before sharing or publishing generated artifacts, check that they do not include personal local paths, account names, API keys, cookies, screenshots, private files, or other sensitive information.
- When using public sources such as Eastmoney, StockAnalysis, Yahoo Finance, company announcements, exchange filings, or financial media, cite them as data sources and summarize facts in your own words. Do not copy full articles, long copyrighted passages, or proprietary reports into the output.

## Workflow

1. Identify securities
   - Resolve each company name to its stock code, market, exchange suffix, and trading currency.
   - Examples:
     - A股深市: `002698.SZ`, Eastmoney `0.002698`
     - A股沪市: `600845.SH`, Eastmoney `1.600845`
     - B股沪市: `900926.SH`, Eastmoney `1.900926`, usually USD-denominated
     - B股深市: `200xxx.SZ`, Eastmoney `0.200xxx`, usually HKD-denominated
     - 港股: `00241.HK` or `0241.HK`, commonly Eastmoney `116.00241`, Yahoo/StockAnalysis `0241.HK`
     - 美股: `TSLA`, `NVDA`, Yahoo/StockAnalysis ticker symbols
   - If one market data source is unstable for a market, cross-check with another public source and state the source limitation in the report.

2. Gather current and historical market data
   - Pull daily K-line data for at least the most recent three calendar months.
   - Include fields: date, open, close, high, low, volume, amount, amplitude, percent change, turnover.
   - Pull current quote and valuation fields where available: latest close/price, percent change, amount, turnover/volume, total market cap, float market cap, PE, PB, industry.
   - For A股 and B股, a usable Eastmoney daily K-line pattern:

```text
https://push2his.eastmoney.com/api/qt/stock/kline/get?secid=<SECID>&fields1=f1,f2,f3,f4,f5,f6&fields2=f51,f52,f53,f54,f55,f56,f57,f58,f59,f60,f61&klt=101&fqt=1&beg=<YYYYMMDD>&end=<YYYYMMDD>
```

   - A usable Eastmoney batch quote pattern:

```text
https://push2.eastmoney.com/api/qt/ulist.np/get?secids=<SECID1>,<SECID2>&fields=f12,f14,f2,f3,f4,f5,f6,f8,f9,f20,f21,f23,f100
```

   - For 港股 and 美股, use reliable public alternatives when Eastmoney is incomplete or unstable:
     - StockAnalysis quote/history/statistics pages
     - Yahoo Finance historical data
     - Nasdaq, NYSE, HKEX, company investor-relations pages
     - MarketWatch, Investing, AAStocks, Futu, or other public quote pages for cross-checking
   - Keep market-specific caveats in the report. For example, B股 liquidity may be low, 港股 turnover may differ by source, and 美股 latest data uses US market dates.

3. Gather fundamentals and news
   - Verify latest annual report and latest quarterly report from company announcements, exchange filings, CNINFO, HKEXnews, SEC filings, company investor-relations pages, Eastmoney/F10, Sina Finance, StockAnalysis, or another public source.
   - Capture revenue, YoY growth, net profit attributable to parent, gross margin/net margin if available, operating cash flow, total assets/equity/liabilities or leverage, and major governance/shareholder notes.
   - Check recent company announcements and major news. Prefer dated, sourceable facts over vague market rumors.
   - For copyrighted news and research reports, use short summaries and source names/links only. Avoid reproducing large excerpts or full tables unless the source license clearly permits it.

4. Compute technical summary
   - For the three-month window, compute: start close, latest close, range return, period high/low with dates, 20-day MA, 60-day MA when available, average turnover or volume, average transaction amount when available, top high-volume days, and a simple 14-day RSI if helpful.
   - Interpret structure:
     - High-volume rise followed by failure to hold: possible short-term distribution or speculative retreat.
     - High-volume breakout with continued elevated turnover: strong capital contest and possible institutional/active-money participation.
     - High-volume down days after a high platform: high-position disagreement and possible profit-taking.
     - Shrinking volume near support: possible stabilization, but require follow-up confirmation.
   - For US stocks, include institution/large-cap context where useful: index weight, institutional ownership, short interest trend, options/event volatility, and major earnings/news catalysts.

5. Report structure
   - Follow this Chinese outline unless the user asks otherwise:
     - `一、核心结论`
     - `二、基本面分析（6项）`
       - 业务模式
       - 赚钱能力
       - 成长性
       - 偿债与资产质量
       - 现金流质量
       - 公司治理与股东背景
     - `三、估值面分析（5项）`
       - PE/PB and relative industry position
       - historical valuation position
       - multi-metric valuation view
       - growth/price match
       - dividend suitability
     - `四、技术面与近三个月主力/资金动向`
       - trend, volume/price, indicators, chart pattern, main-capital inference
     - `五、消息与市场情绪（4项）`
       - recent announcements/news
       - institution view if available
       - crowd/social sentiment
       - sudden risk warnings
     - `六、风险提示与跟踪要点`
     - `主要数据来源`

6. Chart and PDF output
   - Generate one recent-three-month price/volume chart per stock. A simple line chart for closing price plus volume bars is sufficient; label the latest data date.
   - Show the trading currency on the chart/report: RMB, USD, or HKD.
   - Prefer a stable local pipeline:
     - Build a static HTML report with Chinese fonts such as Microsoft YaHei/SimHei/SimSun.
     - Embed the local chart image.
     - Export to PDF with local Chrome or Edge headless printing.
   - If the workspace is read-only, request filesystem approval before creating report files.
   - Save one PDF per stock with clear Chinese filenames, for example `中际旭创_股票分析报告.pdf`, `阿里健康_股票分析报告.pdf`, or `特斯拉_股票分析报告.pdf`.

## Quality checks

Before final response:

- Confirm all requested PDFs exist and have nonzero file sizes.
- Check the report states the latest market data date.
- Check each report has a chart, core conclusion, fundamentals, valuation, technical/main-capital or funds-flow section, news/sentiment section, risks, and data sources.
- Check each report states market, ticker, currency, latest trading date, and source limitations if any.
- Check the report includes the Chinese disclaimer: `本文仅供研究参考，不构成投资建议，也不构成任何买入、卖出或持有建议。`
- Check the report does not expose personal paths, account names, credentials, screenshots, or private files.
- Check data sources are cited by name or link, and that source content is summarized rather than copied at length.
- Keep the final response short and link the generated PDF files.
