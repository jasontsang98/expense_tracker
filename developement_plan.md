# Personal Finance Tracker MVP - Development Plan

## Project Overview
A containerized web application for manual tracking of bank balances, expenses, and subscriptions with automated data transformation and dashboard visualization.

## Technology Stack
- **Database**: PostgreSQL 15
- **Backend API**: Python FastAPI
- **Frontend**: HTML/CSS/JavaScript (vanilla, no frameworks)
- **Data Transformation**: dbt Core
- **Dashboard**: Grafana
- **Containerization**: Docker + Docker Compose
- **Environment**: Ubuntu (local development) and linux server

## Database Schema Design

### Core Tables

#### `banks`
- `id` (Primary Key, Serial)
- `bank_name` (VARCHAR, NOT NULL)
- `account_type` (VARCHAR, NOT NULL) - checking, savings, credit
- `account_nickname` (VARCHAR) - user-friendly name
- `created_at` (TIMESTAMP)
- `updated_at` (TIMESTAMP)
- `is_active` (BOOLEAN, default TRUE)

#### `bank_balances`
- `id` (Primary Key, Serial)
- `bank_id` (Foreign Key to banks.id)
- `balance_amount` (DECIMAL(15,2), NOT NULL)
- `balance_date` (DATE, NOT NULL)
- `entry_date` (TIMESTAMP, default NOW())
- `notes` (TEXT)

#### `subscriptions`
- `id` (Primary Key, Serial)
- `subscription_name` (VARCHAR, NOT NULL)
- `amount` (DECIMAL(10,2), NOT NULL)
- `billing_date` (INTEGER) - day of month (1-31)
- `category` (VARCHAR)
- `status` (VARCHAR) - active, cancelled, paused
- `created_at` (TIMESTAMP)
- `updated_at` (TIMESTAMP)
- `next_due_date` (DATE) - calculated field

#### `transactions`
- `id` (Primary Key, Serial)
- `transaction_date` (DATE, NOT NULL)
- `amount` (DECIMAL(15,2), NOT NULL)
- `transaction_type` (VARCHAR, NOT NULL) - income, expense
- `category` (VARCHAR)
- `description` (TEXT)
- `bank_id` (Foreign Key to banks.id, NULLABLE)
- `created_at` (TIMESTAMP)
- `updated_at` (TIMESTAMP)

## API Endpoints Structure

### Bank Management
- `GET /api/banks` - List all banks/accounts
- `POST /api/banks` - Create new bank/account
- `PUT /api/banks/{id}` - Update bank/account
- `DELETE /api/banks/{id}` - Deactivate bank/account

### Bank Balance Management
- `GET /api/bank-balances` - List balances (with filters)
- `POST /api/bank-balances` - Add new balance entry
- `PUT /api/bank-balances/{id}` - Update balance entry
- `DELETE /api/bank-balances/{id}` - Delete balance entry
- `GET /api/bank-balances/latest` - Get latest balance per bank

### Subscription Management
- `GET /api/subscriptions` - List all subscriptions
- `POST /api/subscriptions` - Create new subscription
- `PUT /api/subscriptions/{id}` - Update subscription
- `DELETE /api/subscriptions/{id}` - Delete subscription
- `GET /api/subscriptions/due-soon` - Get subscriptions due within 7 days

### Transaction Management
- `GET /api/transactions` - List transactions (paginated, with filters)
- `POST /api/transactions` - Create new transaction
- `PUT /api/transactions/{id}` - Update transaction
- `DELETE /api/transactions/{id}` - Delete transaction
- `POST /api/transactions/bulk-delete` - Delete multiple transactions

### Utility Endpoints
- `POST /api/refresh-dashboard` - Trigger dbt run and dashboard refresh
- `GET /api/health` - Health check endpoint

## Frontend Pages Structure

### Page Layout
- Header with navigation menu
- Simple CSS grid/flexbox layout
- Minimal styling for MVP

### 1. Bank Balance Entry (`/balances`)
- Form to add new balance entry:
  - Dropdown for bank selection
  - Date picker (default to today)
  - Amount input (decimal)
  - Notes textarea (optional)
  - Submit button
- Table showing recent balance entries:
  - Bank name, date, amount, notes
  - Edit/Delete buttons per row
  - Sortable by date (newest first)

### 2. Subscription Management (`/subscriptions`)
- Form to add new subscription:
  - Subscription name input
  - Amount input (decimal)
  - Billing date dropdown (1-31)
  - Category input (text)
  - Status dropdown (active/cancelled/paused)
  - Submit button
- Table showing all subscriptions:
  - Name, amount, billing date, next due date, status
  - Edit/Delete buttons per row
  - Highlight subscriptions due within 3 days

### 3. Transaction Entry (`/transactions`)
- Form to add new transaction:
  - Date picker
  - Amount input (decimal)
  - Type radio buttons (Income/Expense)
  - Category input (text)
  - Description textarea
  - Bank dropdown (optional)
  - Submit button
- Table showing all transactions:
  - Date, type, amount, category, description
  - Edit/Delete buttons per row
  - Bulk select checkboxes
  - Bulk delete button
  - Pagination (20 records per page)
  - Filter by date range, type, category

### 4. Dashboard (`/dashboard`)
- Embedded Grafana iframe
- Button to refresh dashboard data
- Basic KPIs displayed:
  - Total net worth
  - Monthly income/expense summary
  - Upcoming subscription payments

## dbt Transformation Models

### Staging Models (`models/staging/`)

#### `stg_bank_balances.sql`
- Clean and standardize bank balance data
- Add calculated fields for month-over-month changes

#### `stg_transactions.sql`
- Clean transaction data
- Standardize categories
- Add month/year extracts

#### `stg_subscriptions.sql`
- Calculate next due dates
- Add monthly/annual cost projections

### Mart Models (`models/marts/`)

#### `mart_monthly_summary.sql`
- Monthly aggregations:
  - Total income by month
  - Total expenses by month
  - Net savings by month
  - Account balances at month-end

#### `mart_category_analysis.sql`
- Spending by category over time
- Top expense categories
- Category trends

#### `mart_net_worth.sql`
- Calculate total net worth over time
- Account-level contributions to net worth

#### `mart_subscription_calendar.sql`
- Upcoming subscription payments
- Monthly subscription costs
- Annual subscription projections

## Docker Configuration

### Services in `docker-compose.yml`

#### PostgreSQL Database
- Image: `postgres:15`
- Environment variables for database credentials
- Volume for data persistence
- Port mapping: 5432:5432

#### FastAPI Backend
- Custom Dockerfile
- Dependencies: FastAPI, SQLAlchemy, Psycopg2, dbt-core
- Environment variables for database connection
- Port mapping: 8000:8000
- Volume mounting for live development

#### Frontend Web Server
- Nginx or simple Python HTTP server
- Serve static HTML/CSS/JS files
- Port mapping: 3000:80
- Proxy API calls to backend

#### dbt Service
- Custom Dockerfile with dbt-core and dbt-postgres
- Volume mounting for dbt project files
- Triggered via API calls (not always running)

#### Grafana
- Image: `grafana/grafana:latest`
- Port mapping: 3001:3000
- Volume for dashboard configurations
- Environment variables for admin credentials

## Development Phases

### Phase 1: Infrastructure Setup (Days 1-2)
1. Create project directory structure
2. Set up Docker Compose configuration
3. Create PostgreSQL database with initial schema
4. Test database connectivity
5. Set up basic FastAPI application structure
6. Create health check endpoint

### Phase 2: Backend API Development (Days 3-5)
1. Implement database models using SQLAlchemy
2. Create CRUD operations for all entities
3. Implement API endpoints for bank management
4. Implement API endpoints for balance management
5. Implement API endpoints for subscription management
6. Implement API endpoints for transaction management
7. Add basic error handling and validation

### Phase 3: Frontend Development (Days 6-8)
1. Create basic HTML templates for all pages
2. Implement navigation between pages
3. Create forms for data entry
4. Implement data tables with edit/delete functionality
5. Add client-side validation
6. Implement API calls using vanilla JavaScript
7. Add basic CSS styling for usability

### Phase 4: Data Pipeline Setup (Days 9-10)
1. Set up dbt project structure
2. Create staging models for data cleaning
3. Create mart models for aggregations
4. Test dbt transformations locally
5. Integrate dbt execution with API endpoint
6. Set up Grafana data source connection
7. Create basic dashboard panels

### Phase 5: Integration & Testing (Days 11-12)
1. End-to-end testing of all workflows
2. Test data pipeline trigger functionality
3. Verify dashboard updates after data changes
4. Test Docker container startup and connectivity
5. Document setup and usage instructions
6. Create sample data for demonstration

## File Structure
```
personal-finance-tracker/
├── docker-compose.yml
├── README.md
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── main.py
│   ├── models/
│   ├── api/
│   ├── database.py
│   └── config.py
├── frontend/
│   ├── index.html
│   ├── balances.html
│   ├── subscriptions.html
│   ├── transactions.html
│   ├── dashboard.html
│   ├── css/
│   └── js/
├── dbt/
│   ├── Dockerfile
│   ├── dbt_project.yml
│   ├── profiles.yml
│   ├── models/
│   │   ├── staging/
│   │   └── marts/
│   └── macros/
├── grafana/
│   ├── dashboards/
│   └── provisioning/
└── sql/
    └── init.sql
```

## Success Criteria for MVP
1. All Docker services start successfully
2. User can add/edit/delete bank accounts and balances
3. User can manage subscription entries
4. User can add/edit/delete income and expense transactions
5. Dashboard displays basic financial summary
6. Data transformations run successfully when triggered
7. All data persists between container restarts

## Post-MVP Enhancement Opportunities
1. Email notifications for due subscriptions
2. Data import/export functionality
3. Advanced filtering and search capabilities
4. Mobile-responsive design improvements
5. User authentication and multi-user support
6. Automated bank integration via APIs
7. Budget planning and goal tracking features
8. Advanced analytics and forecasting