# Restaurant Ordering and Management System with Operational Analytics

A full-stack database application for restaurant ordering, checkout, refunds, and operational analytics. The project is designed for DSC5003 Track 1 and doubles as a data-analysis portfolio project.

## Portfolio Highlights

- **End-to-end transaction lifecycle:** menu data, table reservation, order creation, checkout, payment and refund history.
- **Relational data modeling:** 10 business tables in MySQL with foreign keys, constraints, indexes and 3 analytical views.
- **Metric governance:** gross sales, net sales, refund rate, average order value, menu engineering, hourly demand, repeat-customer rate and repeat-order rate.
- **Product analysis:** separates high-volume and high-value products into Stars, Plowhorses, Puzzles and Dogs while keeping the no-cost limitation explicit.
- **Customer analysis:** measures both the share of customers who return and the share of orders generated after a first purchase.
- **Database evidence:** transaction tests, concurrent table protection, 1K/10K/50K scale experiments, index benchmarks and `EXPLAIN` plans.
- **Real-behavior validation:** applies the same metrics to 1,067,371 UCI Online Retail II transaction lines, 53,628 orders, 5,304 products and 5,942 identified customers.

## Evidence at a Glance

| Evidence | Result |
| --- | --- |
| Synthetic system | 1,000 orders; HK$327,252 net sales; 4.02% refund rate |
| Demand concentration | 19:00 peak in the simulated system; 21.7% of orders |
| Repeat behavior | 73.6% repeat-customer rate; 74.2% repeat-order rate |
| UCI validation | £19.29m net sales; 8.04% refund rate |
| UCI repeat behavior | 75.4% repeat-customer rate; 86.8% repeat-order rate |

The UCI dataset is an e-commerce dataset used only to validate that the analytical methods work on real customer behavior. Its results must not be presented as actual restaurant performance.

## Portfolio Status

The web application, database schema, analytical endpoints, tests, performance evidence and written analysis are complete. The Tableau workbook is intentionally treated as a future enhancement and is not claimed as part of this repository version.

## Project Positioning

The application is not only a POS prototype. It follows one transaction from menu master data to order creation, checkout, payment, and refund records, then converts those records into product metrics and operating insights.

Core capabilities:

- Relational data modeling in MySQL with foreign keys, check constraints, unique constraints, and indexes.
- Transaction-aware order creation, item updates, checkout, and refunds.
- Lightweight web interface for restaurant staff.
- SQL-backed revenue, menu performance, refund, and hourly-demand analytics.
- Deterministic synthetic data generation for correctness and performance tests.

## Architecture

```text
Browser UI
   |
   v
FastAPI application
   |-- order and menu services
   |-- analytical queries
   v
MySQL 9
   |-- normalized operational tables
   |-- order facts and master dimensions
```

The operational schema follows 3NF. Analytical endpoints aggregate the same normalized data using meaningful SQL queries. Historical order items retain the unit price at purchase time so later menu price changes do not corrupt revenue history.

## Technology Stack

- Python 3.12
- FastAPI and Pydantic
- SQLAlchemy 2
- MySQL 9
- HTML, CSS, and vanilla JavaScript
- Pytest and SQLite for isolated transaction-flow tests

## Run Locally

Start the project database:

```powershell
docker compose up -d
```

Create and activate the Python environment:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements-dev.txt
```

Initialize and seed the database:

```powershell
python -m scripts.initialize_db
python -m scripts.seed_data --orders 1000 --reset
python -m scripts.export_schema
```

Start the application:

```powershell
python -m uvicorn app.main:app --reload --port 8010
```

Open:

- Application: `http://127.0.0.1:8010`
- API documentation: `http://127.0.0.1:8010/docs`

The database container uses host port `3308` to avoid conflicting with other local MySQL instances.

## Run Tests

```powershell
.\.venv\Scripts\python.exe -m pytest -q
```

The current tests cover:

- A complete create, add-item, checkout, and partial-refund workflow.
- Table state protection against concurrent active orders.
- Rejection of checkout for an empty order.

Evaluation and report evidence:

```powershell
python -m scripts.check_concurrency
python -m scripts.run_evaluation
python -m scripts.export_explain_plans
python -m scripts.write_evaluation_summary
python -m scripts.generate_report_charts
```

Real behavior validation:

```powershell
pip install -r requirements-analysis.txt
python -m scripts.download_real_dataset
python -m scripts.prepare_real_dataset
python -m scripts.validate_system_analytics
python -m scripts.validate_real_analytics
```

## Main Data Flow

1. A menu category and menu item define the current sale price.
2. Opening an order reserves a dining table and creates an `open` status record.
3. Adding items snapshots the menu item price in `order_items`.
4. Checkout calculates the total, creates a payment, closes the order, and releases the table.
5. A refund creates a refund record and changes the payment/order status without deleting history.
6. Analytics aggregate completed orders, menu lines, payments, and refunds into business metrics.

## Metric Definitions

- Revenue: total `orders.total_amount` for paid and refunded orders. Refund amount is reported separately.
- Orders: paid, partially refunded, or fully refunded orders.
- Average order value: revenue divided by completed orders.
- Menu sales: summed `order_items.line_total`, using historical purchase prices.
- Hourly demand: order count and sales grouped by local `placed_at` hour.

The complete metric dictionary is in `docs/metrics.md`.

## Repository Structure

```text
app/
  models.py             relational schema
  schemas.py            API input and output contracts
  services/             transactional order operations
  routers/              menu, order, and analytics APIs
frontend/               full-stack web interface
scripts/                database initialization and synthetic data
tests/                  transaction and API tests
docs/                   design, metrics, plan, and report outline
sql/schema.sql          generated MySQL DDL
```

Analytical outputs for the BI exercise are generated under `data/processed/bi`. Raw and processed datasets are excluded from Git and can be recreated using the scripts above.

## One-Month Delivery Plan

- Week 1: requirements, ER model, normalized schema, API contracts, order workflow.
- Week 2: complete catalog management, checkout/refund workflows, error handling, and tests.
- Week 3: analytical queries, dashboard, synthetic scale tests, index benchmarks, and `EXPLAIN` results.
- Week 4: report, contribution statement, screenshots, repository cleanup, and 5–8 minute demo.

The detailed work breakdown is in `docs/project-plan.md`.

## References and Originality

The project is an original implementation. External resources are used only as business or educational references and must be cited in the final report:

- Chen, P. P.-S. (1976), The Entity-Relationship Model.
- MySQL `classicmodels` sample database, used only as order/order-item design inspiration.
- Floreant POS and uniCenta oPOS, used only to study restaurant workflow expectations.
