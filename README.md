from pathlib import Path

readme = r'''# FinPay – Student Wallet, Payments & Cashback System

A campus wallet application built with **Python, FastAPI, SQLAlchemy and SQLite**. Students can add money to their wallet, pay registered campus merchants and earn cashback. Merchants can view payments received, while administrators can generate monthly reconciliation reports.

> **Project type:** Academic / demonstration project  
> **Architecture:** Layered Architecture  
> **Frontend:** HTML, CSS and JavaScript served by FastAPI

---

## Table of Contents

- [Key Features](#key-features)
- [Technology Stack](#technology-stack)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Roles and Permissions](#roles-and-permissions)
- [Business Rules](#business-rules)
- [Payment Workflow](#payment-workflow)
- [Database Overview](#database-overview)
- [Getting Started](#getting-started)
- [Using the Application](#using-the-application)
- [API Reference](#api-reference)
- [Running Tests](#running-tests)
- [Configuration](#configuration)
- [Error Responses](#error-responses)
- [Limitations and Future Improvements](#limitations-and-future-improvements)

---

## Key Features

| Module | Features |
|---|---|
| Authentication | User registration and login, JWT-based authentication, role-based access |
| Student wallet | Automatic wallet creation, wallet top-up, balance display and transaction history |
| Payments | Pay registered merchants, validate payment amount and available balance |
| Cashback | Calculate and credit cashback for eligible payments |
| Merchant | Create a merchant profile and view payments received |
| Admin | Generate monthly sales, cashback and merchant settlement reports |
| Reconciliation | Compare wallet balances with ledger entries to identify mismatches |
| Web interface | Separate dashboards for students, merchants and administrators |

---

## Technology Stack

| Component | Technology |
|---|---|
| Programming language | Python 3.10+ |
| Backend framework | FastAPI |
| ASGI server | Uvicorn |
| ORM | SQLAlchemy |
| Database | SQLite |
| Authentication | JWT (PyJWT) |
| Password security | PBKDF2-SHA256 with salt |
| Frontend | HTML, CSS and JavaScript |
| Testing | pytest and FastAPI TestClient |

---

## Architecture

FinPay follows **Layered Architecture**. Each layer has a specific responsibility, which keeps request handling, business rules and database access separate.

```text
             Browser / API Client
                     |
                     v
              Router / API Layer
                     |
                     v
               Service Layer
              (Business Rules)
                     |
                     v
             Repository Layer
               (Data Access)
                     |
                     v
                Model Layer
                     |
                     v
                SQLite DB
```

### Layer responsibilities

| Layer | Location | Responsibility |
|---|---|---|
| Router / API | `app/routers/` | Receives HTTP requests, applies role checks and calls services |
| Service | `app/services/` | Implements wallet, payment, cashback and reconciliation rules |
| Repository | `app/repositories/` | Performs database queries and data operations |
| Model | `app/models/` | Defines database tables and relationships |
| Core | `app/core/` | Provides database setup, security, errors and utilities |

### Why Layered Architecture?

- Keeps business logic separate from HTTP request handling.
- Makes payment and cashback rules easier to locate and maintain.
- Separates database operations from business decisions.
- Provides a clear structure for adding or changing features.

For payment processing, the service layer coordinates the related operations so they are committed together or rolled back if an operation fails.

---

## Project Structure

```text
FINPAY-main/
├── app/
│   ├── main.py
│   ├── schemas.py
│   ├── core/
│   │   ├── database.py
│   │   ├── errors.py
│   │   ├── security.py
│   │   └── utils.py
│   ├── routers/
│   │   ├── auth_router.py
│   │   ├── wallet_router.py
│   │   ├── merchant_router.py
│   │   ├── payment_router.py
│   │   ├── cashback_router.py
│   │   └── reconciliation_router.py
│   ├── services/
│   │   ├── auth_service.py
│   │   ├── wallet_service.py
│   │   ├── merchant_service.py
│   │   ├── payment_service.py
│   │   ├── cashback_service.py
│   │   └── reconciliation_service.py
│   ├── repositories/
│   │   ├── user_repository.py
│   │   ├── wallet_repository.py
│   │   ├── merchant_repository.py
│   │   ├── transaction_repository.py
│   │   └── cashback_repository.py
│   ├── models/
│   │   ├── user.py
│   │   ├── wallet.py
│   │   ├── merchant.py
│   │   ├── transaction.py
│   │   └── cashback.py
│   └── static/
│       └── index.html
├── tests/
│   └── test_flow.py
├── requirements.txt
├── .gitignore
└── README.md
```

---

## Roles and Permissions

| Action | Student | Merchant | Admin |
|---|:---:|:---:|:---:|
| Register and log in | Yes | Yes | Yes |
| Top up wallet | Yes | — | — |
| View wallet balance and history | Yes | — | — |
| Pay a merchant | Yes | — | — |
| View own payment and cashback history | Yes | — | — |
| Create merchant profile | — | Yes | — |
| View payments received | — | Yes | — |
| View merchant list and cashback rules | Yes | Yes | Yes |
| View monthly reconciliation report | — | — | Yes |

---

## Business Rules

### Wallet

- A wallet is created automatically when a student registers.
- A top-up amount must be greater than zero.
- The maximum top-up amount is **₹50,000 per transaction**.
- A payment is rejected when the wallet balance is insufficient.

### Cashback

The current cashback rules are:

| Rule | Value |
|---|---:|
| Cashback rate | 2% |
| Minimum eligible payment | ₹100 |
| Maximum cashback per payment | ₹50 |

Examples:

- A ₹500 eligible payment earns ₹10 cashback.
- A ₹50 payment earns no cashback.
- A ₹5,000 payment earns ₹50 cashback because of the cap.

The cashback settings are maintained in `app/services/cashback_service.py`.

### Account rules

- Email addresses must be unique.
- Passwords must contain at least 6 characters.
- A merchant account can create one merchant profile.

---

## Payment Workflow

Example: a student pays a merchant ₹500.

1. The payment router receives the request and checks that the user has the student role.
2. The payment service validates the amount and merchant.
3. The wallet service checks the student's available balance and records the debit.
4. The cashback service calculates eligible cashback and records the credit.
5. The transaction and related ledger records are committed together.
6. The payment is marked as successful.

The application is designed so that a failed payment operation is rolled back rather than leaving only part of the payment recorded.

---

## Database Overview

| Table | Purpose |
|---|---|
| `users` | Stores student, merchant and admin accounts |
| `wallets` | Stores wallet balances |
| `wallet_ledger` | Records top-ups, payments and cashback entries |
| `merchants` | Stores merchant business profiles |
| `transactions` | Stores payment transaction details and status |
| `cashback_records` | Stores cashback details associated with transactions |

The wallet ledger records balance changes. Reconciliation uses the ledger entries to check whether each stored wallet balance matches the sum of its recorded entries.

---

## Getting Started

Follow these steps to run FinPay locally.

### 1. Prerequisites

Install:

- Python **3.10 or newer**
- Git (if you are cloning the repository)

Check your Python installation:

```bash
python --version
```

### 2. Get the project

If you are cloning from GitHub:

```bash
git clone https://github.com/YOUR_USERNAME/FinPay-Student-Wallet-System.git
cd FinPay-Student-Wallet-System
```

If you downloaded the ZIP, extract it and open the project directory that contains `app`, `tests` and `requirements.txt`. In the supplied project ZIP, that directory is named `FINPAY-main`.

### 3. Create and activate a virtual environment

**Windows PowerShell:**

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

If PowerShell blocks activation, you can use the environment's Python directly:

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

**macOS / Linux:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 4. Install dependencies

With the virtual environment activated:

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

### 5. Start the application

Run this command from the project root—the directory containing the `app` folder:

```bash
python -m uvicorn app.main:app --reload
```

To use a different port, for example `8001`:

```bash
python -m uvicorn app.main:app --reload --port 8001
```

### 6. Open FinPay

When the server starts successfully, open:

| Page | URL |
|---|---|
| FinPay web application | http://127.0.0.1:8000/ |
| Interactive API documentation | http://127.0.0.1:8000/docs |
| Alternative API documentation | http://127.0.0.1:8000/redoc |
| Health check | http://127.0.0.1:8000/health |

The SQLite database is configured to use `finpay.db` by default and is created by the application when it initializes the database.

---

## Using the Application

A basic demonstration flow:

1. **Create a merchant account:** Register with the Merchant role and create a business profile, for example, `KLE Canteen`.
2. **Create a student account:** Register with the Student role.
3. **Add money:** Top up the student wallet.
4. **Make a payment:** Select the merchant and enter a payment amount.
5. **Check cashback:** View the payment and cashback in the student dashboard.
6. **Check merchant activity:** Log in as the merchant and view payments received.
7. **Check reconciliation:** Log in as an admin and open the monthly reconciliation report.

Use the application's logout option when switching between accounts.

> This is a demonstration workflow. Use test accounts and sample amounts only.

---

## API Reference

The interactive API documentation is available at:

- Swagger UI: `http://127.0.0.1:8000/docs`
- ReDoc: `http://127.0.0.1:8000/redoc`

After logging in, use the **Authorize** option in Swagger UI and provide the access token returned by the login endpoint.

| Method | Endpoint | Access | Description |
|---|---|---|---|
| POST | `/auth/register` | Public | Register a user |
| POST | `/auth/login` | Public | Log in and receive an access token |
| GET | `/auth/me` | Authenticated | Get the current user |
| GET | `/wallet/balance` | Student | View wallet balance |
| POST | `/wallet/topup` | Student | Add money to wallet |
| GET | `/wallet/history` | Student | View wallet ledger |
| POST | `/merchants` | Merchant | Create merchant profile |
| GET | `/merchants` | Authenticated | List merchants |
| GET | `/merchants/me` | Merchant | View own merchant profile |
| POST | `/payments` | Student | Pay a merchant |
| GET | `/payments/history` | Student | View own payments |
| GET | `/payments/received` | Merchant | View payments received |
| GET | `/cashback/rules` | Authenticated | View cashback rules |
| GET | `/cashback/history` | Student | View cashback history |
| GET | `/reconciliation/monthly?year=YYYY&month=MM` | Admin | Generate monthly report |
| GET | `/health` | Public | Check application health |

### Example request bodies

Register a student:

```json
{
  "name": "Chaitra",
  "email": "student@example.com",
  "password": "secret123",
  "role": "student"
}
```

Top up a wallet:

```json
{
  "amount": 1000
}
```

Pay a merchant:

```json
{
  "merchant_id": 1,
  "amount": 500
}
```

The exact response schemas are available in Swagger UI.

---

## Running Tests

From the project root, with the virtual environment activated, run:

```bash
python -m pytest -q
```

The project includes an end-to-end test file at `tests/test_flow.py`. It exercises the main application flow, including authentication, merchant profile creation, wallet top-up, payments, cashback and reconciliation.

---

## Configuration

FinPay supports the following environment variables:

| Variable | Default | Purpose |
|---|---|---|
| `FINPAY_DB` | `sqlite:///./finpay.db` | SQLAlchemy database connection URL |
| `FINPAY_SECRET` | Development key in application | Secret used to sign JWT tokens |

### Set the JWT secret in Windows PowerShell

Set a development secret before starting the server:

```powershell
$env:FINPAY_SECRET = "replace-this-with-a-long-random-secret"
python -m uvicorn app.main:app --reload
```

For macOS / Linux:

```bash
export FINPAY_SECRET="replace-this-with-a-long-random-secret"
python -m uvicorn app.main:app --reload
```

**Security note:** Never commit real secrets, passwords, access tokens or production environment files to GitHub. The built-in development secret is not suitable for deployment. Use a strong, private secret in any deployed environment.

---

## Error Responses

Application and validation errors are returned as HTTP responses.

| Status code | Meaning | Example |
|---|---|---|
| 400 | Business rule violation | Insufficient balance or duplicate email |
| 401 | Authentication required or invalid credentials | Invalid login or token |
| 403 | Permission denied | Student attempts an admin-only operation |
| 404 | Requested resource not found | Merchant or wallet not found |
| 422 | Request validation failed | Missing or invalid request field |

---

## Limitations and Future Improvements

This project is intended for learning and demonstration. It is not a production banking or payment platform.

Current limitations and possible improvements:

- **Admin registration:** Admin registration is available for demonstration. A production system should restrict admin account creation.
- **Money precision:** The current implementation uses floating-point amounts rounded to two decimal places. A production financial system should use `Decimal` or integer paise.
- **Settlement:** Merchant settlement is reported from transaction data; actual bank payouts, fees and merchant wallet operations are outside the current scope.
- **Database:** SQLite is used for local development. PostgreSQL could be considered for a larger deployment.
- **Additional features:** Refunds, downloadable statements, pagination, rate limiting, password reset and audit logging.

---

## Contributing

Suggestions and improvements are welcome.

1. Fork the repository.
2. Create a feature branch.
3. Make and test your changes.
4. Open a pull request describing the change.

---

## Disclaimer

FinPay is an educational project created to demonstrate wallet operations, role-based access, payment workflows, cashback calculation, layered software architecture and reconciliation. It does not connect to real banks or payment gateways and should not be used to store or transfer real money.
'''
path = Path("/mnt/data/README_FinPay_Improved.md")
path.write_text(readme, encoding="utf-8")
print(f"Created: {path}")
print(f"Size: {path.stat().st_size} bytes")
