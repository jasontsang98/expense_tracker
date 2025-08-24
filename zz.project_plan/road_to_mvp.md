# ✅ Expense Tracker MVP Checklist

This file tracks progress toward building the MVP as per the confirmed spec.

---

## Phase 0 — Setup
- [x] Decide dev environment (Ubuntu VM, WSL2, or native)
- [x] Install Git, Docker, docker-compose
- [ ] Init repo structure (`/app`, `/db`, `/dbt`, `/dash`, `/ops`)
- [ ] Add `.env.example` (Postgres creds, app secrets)
- [ ] Docker Compose baseline: Postgres, empty app, Metabase, dbt container

---

## Phase 1 — Database Foundations
- [ ] Create schema:
  - [ ] `bank`
  - [ ] `account`
  - [ ] `monthly_balance` (unique `(account_id, month)`)
  - [ ] `subscription`
  - [ ] `transaction`
  - [ ] (optional) `change_log` (audit)
- [ ] Seed 1–2 demo banks + accounts

---

## Phase 2 — Core User Flows
- [ ] **Banks & Accounts**
  - [ ] CRUD forms
  - [ ] Validation: unique bank name
- [ ] **Monthly Balances**
  - [ ] Entry form: account, month, EoM balance, notes
  - [ ] Enforce one per account/month
  - [ ] Show table with history
- [ ] **CSV Import**
  - [ ] Support columns: `bank_name, account_name, month, eom_balance`
  - [ ] Validate bank/account existence
  - [ ] Handle duplicates (skip or overwrite)
  - [ ] Generate import report (inserted, updated, skipped)
- [ ] **Subscriptions**
  - [ ] CRUD form: name, category, amount, frequency, next charge date, payment method, account
  - [ ] Table view
- [ ] **Transactions**
  - [ ] Manual add/edit/delete
  - [ ] Filter by month/account/category
  - [ ] Bulk delete

---

## Phase 3 — dbt Transformations
- [ ] Init dbt project (Postgres target)
- [ ] Create staging models:
  - [ ] `stg_accounts`
  - [ ] `stg_monthly_balances`
  - [ ] `stg_transactions`
  - [ ] `stg_subscriptions`
- [ ] Create core marts:
  - [ ] `fct_net_worth_monthly`
  - [ ] `fct_cashflow_monthly`
  - [ ] `fct_subscriptions_monthly`
  - [ ] `dim_calendar_month`
- [ ] Add dbt tests (uniqueness, not-null, enums, FKs)
- [ ] Manual trigger after writes to run `dbt run`

---

## Phase 4 — Dashboards
- [ ] Connect Metabase (or Superset) to Postgres
- [ ] Build initial dashboards:
  - [ ] Net worth trend (line)
  - [ ] Savings rate by month (bar)
  - [ ] Cashflow: income vs expense (bar/area)
  - [ ] Top 5 subscriptions vs total expenses
  - [ ] Accounts table: last 6 months + MoM change
  - [ ] Transactions filterable view
- [ ] Enable auth in Metabase

---

## Phase 5 — Ops
- [ ] Nightly Postgres backups via `pg_dump` to `/backups` (7–14 day retention)
- [ ] Restore instructions in README
- [ ] Basic login for app (single user)
- [ ] Request logging + input validation

---

## Phase 6 — Close Out
- [ ] End-to-end test:
  - [ ] Add bank/account → balances (form + CSV)
  - [ ] Add subscriptions → transactions
  - [ ] Run dbt → dashboard updated
- [ ] Document Quick Start in README
- [ ] Mark MVP complete
