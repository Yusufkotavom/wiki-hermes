# Wiki Log

> Chronological record of all wiki actions. Append-only.
> Format: `## [YYYY-MM-DD] action | subject`
> Actions: ingest, update, query, lint, create, archive, delete

## [2026-07-03] create | Wiki initialized
- Domain: personal knowledge hub + cross-project references + content production + social media + task/plan tracking
- Structure created with SCHEMA.md, index.md, log.md
- Initial project references imported: devk-studio, sanity-clean
- Remote origin set to Yusufkotavom/wiki-hermes on branch master
- .env files ignored by git

## [2026-07-03] create | Personal money tracker
- Created Hermes skill: ~/.hermes/skills/finance/money-tracker
- Created finance schema and files:
  - ~/wiki/finance/SCHEMA.md
  - ~/wiki/finance/categories.json
  - ~/wiki/finance/wallets.json
  - ~/wiki/finance/debts.json
  - ~/wiki/finance/transactions.json
  - ~/wiki/finance/receipts/2026-07
  - ~/wiki/finance/reports
- Supported flows: add/income/expense/transfer, debt/loan tracking, proof/receipts, month report
