# 💰 Expense Tracker Application Specification

## 🎯 Purpose
A personal expense tracker focused on **monthly bank balances** (instead of parsing messy PDFs), with features for subscriptions, manual income/expense tracking, data transformations via **dbt**, and dashboards.  

---

## ☑️ Confirmed Product Spec (MVP v1)

### A. Core User Flows
1. **Create bank & account first**
   - “Banks & Accounts” page: create a Bank, then one or more Accounts under it (e.g., “CBA → NetBank Saver”).
   - Monthly Balance page shows a dropdown of **Account** (bank name displayed alongside).
   - **CSV import** allowed; importer must verify each row’s Bank/Account exists and the bank name matches exactly.

2. **Monthly balances (primary tracking)**
   - Entry form: Account, Month (YYYY-MM), End-of-Month Balance, optional Notes.
   - Only **one balance per Account per Month** (uniqueness enforced).
   - Allows edit history (audit log) so mistakes can be corrected.

3. **Subscriptions**
   - Tab to add/edit/delete subscriptions: Name, Category, Amount, Frequency (monthly/yearly), Next Charge Date, Payment Method, Account.
   - Save in DB.
   - Future: notifications via **Telegram bot or Discord webhook/DM** (“due tomorrow”).

4. **Income & expenses (manual)**
   - Tab with add form + table list (filter by month/account/category).
   - Quick actions: delete row, bulk delete by date range.
   - Optional tag/notes for context.

5. **Transformations & dashboard (monthly)**
   - On new/updated balance or transaction, run transformations (dbt) and refresh dashboards.
   - Views: Net worth (month-end), Month-over-Month deltas, Savings rate, Subscriptions vs total spend.

6. **Deployment**
   - Local Ubuntu via Docker Compose: **Postgres + App + dbt + Dashboard (Metabase/Superset)**.
   - Same stack later on Linux server.

7. **Scope**
   - Currency: **AUD only**.
   - UX: keep forms minimal and fast; polish later.

---

## 🧱 Data Model (Conceptual)

**Banks & accounts**
- `bank`: `id`, `name` (unique), `institution_code`, `created_at`
- `account`: `id`, `bank_id` (FK), `name`, `account_ref`, `is_active`, `created_at`

**Monthly balances**
- `monthly_balance`: `id`, `account_id` (FK), `month` (date set to first of month), `eom_balance` (numeric), `notes`, `created_at`, `updated_at`  
  - Unique: `(account_id, month)`
  - Optional `balance_source` (“manual”, “csv”)

**Subscriptions**
- `subscription`: `id`, `account_id` (nullable), `name`, `category`, `amount`, `frequency` (monthly|yearly), `next_charge_date`, `renewal_day`, `payment_method`, `is_active`, `created_at`, `updated_at`

**Transactions (manual income/expense)**
- `transaction`: `id`, `account_id` (FK), `date`, `type` (income|expense), `amount`, `category`, `notes`, `created_at`  
  - Ledger sign derived from `type`.

**Notifications (later)**
- `notification_channel`: `id`, `type` (discord|telegram), `destination`, `is_enabled`
- `notification_log`: `id`, `subscription_id`, `scheduled_for`, `sent_at`, `status`, `channel_type`, `message_snapshot`

**Audit (recommended)**
- `change_log`: `id`, `entity`, `entity_id`, `action` (create|update|delete), `changed_by`, `changed_at`, `diff` (JSON)

---

## 📥 CSV Import Rules (Monthly Balances)

- Required columns: `bank_name`, `account_name`, `month` (YYYY-MM), `eom_balance`.
- Validation:
  - **Bank/account existence**: reject rows where `bank_name` or `account_name` does not match.
  - **Month format** & duplicates: if `(account, month)` exists → either overwrite or skip (with report).
  - **Currency**: assume AUD; reject non-AUD rows.
- Import report: totals, inserted, updated, skipped (with reasons).

---

## 🧮 dbt Transformation Plan (Monthly)

**Staging models**
- `stg_accounts`: accounts + banks.
- `stg_monthly_balances`: valid monthly balances.
- `stg_transactions`: normalized transactions.
- `stg_subscriptions`: with next/last charge derived.

**Core marts**
- `fct_net_worth_monthly`: net worth over time, deltas, trends.
- `fct_cashflow_monthly`: income vs expense, savings rate.
- `fct_subscriptions_monthly`: recurring costs, next-charge countdowns.
- `dim_calendar_month`: month sequence to fill gaps.

**Tests**
- Uniqueness on `(account_id, month)`.
- Not-null for keys and amounts.
- Accepted values for enums.
- Relationships (FKs).

**Triggers**
- After a write (balance, subscription, transaction), trigger partial `dbt run`.

---

## 📊 Dashboard (Initial)

- **Overview**
  - Net worth over time (line chart).
  - Savings rate by month (bar).
  - Cashflow: income vs expense.
  - Top 5 subscription costs, total % of expenses.

- **Accounts**
  - Table: all accounts with last 6 months EoM balances, MoM change, trend.

- **Subscriptions**
  - Table: name, amount, frequency, next charge date, alerts.

- **Transactions**
  - Filter by month/account/category; quick add/delete.

---

## 🔔 Notifications: Discord vs Telegram (Future)

- **Telegram bot**: simple, reliable, direct personal DM.
- **Discord webhook**: quick alerts into a channel.
- **Discord bot**: more setup, allows direct DMs.

**Recommendation**:  
Start with **Telegram bot** for personal alerts. Add **Discord webhook** later for “finance-alerts” channel.

**Logic**:
- Daily check (09:00 AEST): find subscriptions due tomorrow.
- Send DM: subscription name, amount, account, due date.

---

## 🐳 Containerized Architecture

- **postgres**: persistent volume, nightly dump to `/backups`.
- **app**: web forms, CSV import, triggers dbt.
- **dbt**: separate container; invoked on demand.
- **metabase**: simple BI dashboard, auth enabled.
- **scheduler** (later): cron for daily alerts.

---

## 🔐 Operational Notes

- **Backups**: nightly `pg_dump` with 7–14 day retention.
- **Access**: app behind simple login.
- **Data quality**: dbt tests enforce validity.
- **Performance**: monthly data volume is light → dbt scales easily.

---

## 🛣️ Roadmap

**MVP**
- Banks/Accounts, Monthly balances (form + CSV), Subscriptions CRUD, Transactions CRUD, Metabase Overview dashboard, dbt monthly models, Postgres backups.

**v1.1**
- Telegram DM alerts, audit log, overwrite/skip controls in CSV import.

**v1.2**
- Inline edits, quick month copy-forward, richer insights.

**v1.3**
- Discord integration, category budgets, monthly budget vs actuals.

---

## 🧭 Feedback & Potential Improvements

1. **Single source of truth**  
   - EoM balances are low-effort. Consider back-filling transactions later for richer analysis.

2. **dbt from day one**  
   - Modularize models; use selectors for efficient runs.

3. **Notifications**  
   - Telegram first, Discord later for group alerts.

4. **Strict uniqueness**  
   - Enforce `(account, month)` uniqueness with clear overwrite rules.

5. **Simplicity**  
   - Keep forms small; don’t overcomplicate data entry.

6. **Future proofing**  
   - Use DECIMAL for amounts. Isolate “currency” in config table (set AUD) for easy extension.

---
