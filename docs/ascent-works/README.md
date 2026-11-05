# Ascent Works – Financial Reporting

**Status:** Assessment – reviewing existing reports and data-lake architecture
**Our team:** Chris Moore, Joe Wayman, Kevin Hansen
**Client contacts:** Crystal Estrada (Ascent Works), Leslie Cox Bryan (The Accounting Company)
**Internal contact for approach/access:** Jeremiah
**Kickoff meeting:** Oct 28, 2026 (recorded and summarized in Fyxer)

## Background
Ascent Works is frustrated with its monthly financial reporting, specifically that reports cannot be rerun on demand.

## What we learned (Oct 28 meeting)
- **Source data:** Applied Epic, replicated into a Fivetran data lake. North Labs loaded the data; StratTech created the account and access.
- **Current reports:** Generated automatically by North Labs around the 6th business day each month. Consolidated and divisional statements, year-to-date and budget-to-actual views, mostly P&L, plus a balance sheet. Output lands on a shared drive.
- **Main pain point:** Crystal cannot rerun reports on demand. She must contact North Labs or redo the work by hand.
- **Priority:** Automate the static financial reports first, then add dynamic AI reporting and dashboards. There is currently no accessible AI agent for querying Applied Epic data.
- **Forecasting:** The monthly renewal forecast is manual. Crystal runs several production reports, spreads revenue across 10-12 months, and copies results into the cash-flow and reforecasted-budget files, which causes errors. Partners want summaries of new vs. recurring business, shrinkage, and lift. The client-renewal forecast feeds the cash-flow forecast and the monthly reforecasted budget.
- **Proposed approach:** Assess whether existing North Labs reporting can be reused instead of rebuilt. Fivetran data-lake access may allow secured, live AI querying of Applied Epic data. A small proof of concept on the existing Fivetran setup could validate report automation.

## Next steps
- [ ] Chris, Joe, and Kevin review existing reports and the data-lake architecture
- [ ] Chris reviews the existing report automation for reuse
- [ ] Chris discusses the approach and access requirements with Jeremiah
- [ ] Leslie provides sample financial reports
- [ ] Chris shares a proposal for automating the static financial report
- [ ] Joe and Kevin estimate the forecasting-tool effort after the initial assessment

Due dates were not specified.

## Still open
- Delivery platform (Power BI or another consumption layer); is self-service needed for finance users?
- Refresh cadence and month-end/audit performance expectations
- Data access and security requirements
- Data quality and cleansing requirements
- Semantic model vs. direct data-lake reporting
- Governance, ownership, and long-term support
- Scalability and maintenance

## Notes
- Joe's feedback on the plan: a good start.
- Sources: Fyxer meeting summary and Kevin's follow-up email (Oct 28, 2026).
