

# NEXUS Enterprise Accounting System

## Overview (EN)

This document provides a comprehensive guide to the accounting system implemented in our ERP solution. The system is built on modern accounting principles with a modular architecture that separates core functionality from specialized accounting processes.

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Core Accounting Module](#core-accounting-module)
3. [Specialized Accounting Modules](#specialized-accounting-modules)
   - [General Accounting](#general-accounting)
   - [Supplier Accounting](#supplier-accounting)
   - [Customer Accounting](#customer-accounting)
4. [Financial Reporting](#financial-reporting)
5. [End-to-End Accounting Process](#end-to-end-accounting-process)
   - [System Setup](#system-setup)
   - [Daily Operations](#daily-operations)
   - [Period-End Procedures](#period-end-procedures)
   - [Reporting Processes](#reporting-processes)
6. [Technical Implementation](#technical-implementation)

## Architecture Overview

The accounting system is designed with a modular architecture:

```
┌─────────────────────────────────────────────────┐
│                                                 │
│              Financial Reporting                │
│                                                 │
└───────────────────┬─────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────┐
│                                                 │
│                Core Accounting                  │
│           (Chart of Accounts, Periods)          │
│                                                 │
└───────┬─────────────────┬────────────┬──────────┘
        │                 │            │
        ▼                 ▼            ▼
┌───────────────┐ ┌───────────────┐ ┌────────────────┐
│               │ │               │ │                │
│    General    │ │   Supplier    │ │    Customer    │
│  Accounting   │ │  Accounting   │ │   Accounting   │
│               │ │               │ │                │
└───────────────┘ └───────────────┘ └────────────────┘
```

- **Core Accounting**: Defines the fundamental structures (Chart of Accounts, fiscal periods) used by all other modules.
- **Specialized Modules**: Handle specific types of accounting transactions (general entries, supplier invoices, customer invoices).
- **Financial Reporting**: Consolidates data from all modules to generate financial statements and reports.

## Core Accounting Module

The Core Accounting module (`core_accounting`) is the foundation of the entire accounting system. It does not record transactions itself but provides the essential structures and references that all other accounting modules rely on.

### Key Components

#### Account Types

Account Types define the fundamental classification of accounts in the system:

- **Asset Accounts**: Resources owned by the company (debit normal)
- **Liability Accounts**: Obligations owed by the company (credit normal)
- **Equity Accounts**: Residual interest in the assets after deducting liabilities (credit normal)
- **Revenue Accounts**: Income earned by the company (credit normal)
- **Expense Accounts**: Costs incurred by the company (debit normal)

Each account type has properties such as:
- Whether it normally increases with credits or debits (`is_credit_normal`)
- Whether it appears on the balance sheet or income statement (`is_balance_sheet`)
- Sequence for reporting order

#### Account Categories

Account Categories provide a more detailed classification within each account type:

- Examples include "Current Assets," "Fixed Assets," "Short-term Liabilities," etc.
- Categories can have parent-child relationships to create a hierarchical structure
- Each category is linked to an account type

#### Chart of Accounts

The Chart of Accounts defines all accounts used in the system:

- **Basic Information**: Account number, name, description
- **Structure**: Type, category, parent account, level in hierarchy
- **Behavior**: Credit/debit normal, control account status, postability
- **Reporting**: Financial statement section, reporting group
- **Controls**: Reconciliation requirements, manual entry permissions

Key features include:
- Hierarchical account structure with parent-child relationships
- Control accounts that can't be directly posted to
- Financial statement section classification
- Reporting group categories for specialized reports

#### Fiscal Years and Accounting Periods

The system defines:

- **Fiscal Years**: Financial reporting years for the organization
- **Accounting Periods**: Subdivisions of fiscal years (months, quarters)
- Status tracking (open/closed) for periods and years

#### Account Balances

The system maintains:

- Account balances at the end of each accounting period
- Debit, credit, and closing balances for each account
- Balance calculations based on account type (is_credit_normal)

## Specialized Accounting Modules

### General Accounting

The General Accounting module (`general_accounting`) records miscellaneous financial transactions that don't fit into specialized modules and serves as the central repository for all financial transactions.

#### Key Components

- **General Ledger Entry**: Records transactions with details like transaction type, dates, amounts, currency, etc.
- **General Ledger Line**: Individual line items with account, debit/credit amount, and analysis dimensions
- **Audit Logging**: Comprehensive tracking of changes to GL entries and lines

#### Transaction Types
- Manual journal entries
- Adjusting entries
- Accruals and deferrals
- Internal transfers
- Period-end closing entries
- Correction entries

#### Workflow States
- Draft: Initial state, can be modified
- Posted: Finalized and affects account balances
- Cancelled: Voided transaction

### Supplier Accounting

The Supplier Accounting module (`accounting_supplier`) handles all financial transactions related to suppliers and expenses.

#### Key Components

- **Supplier Invoice Header**: Records invoice details like dates, invoice number, amounts, currency, etc.
- **Supplier Invoice Line**: Individual line items with account, description, debit amount
- **Audit Logging**: Tracks changes to supplier invoices

#### Features
- Record and manage incoming supplier invoices
- Support for different invoice types (standard, credit notes, debit notes)
- VAT/tax handling with automatic calculations
- Multi-currency support
- Workflow status tracking (draft, posted, cancelled)
- Payment tracking and management

### Customer Accounting

The Customer Accounting module (`accounting_customer`) handles all financial transactions related to customers and revenue.

#### Key Components
- Customer Invoice management
- Revenue tracking and categorization
- Payment receipt processing
- Customer account management

#### Features
- Record and manage outgoing customer invoices
- Support for different invoice types (standard, credit notes, proforma)
- VAT/tax handling
- Multi-currency support
- Revenue categorization and analysis
- Payment receipt tracking
- Customer account balance monitoring

## Financial Reporting

The Financial Reporting module (`financial_reporting`) generates financial statements and reports based on data from all accounting modules.

### Key Components

- **Report Templates**: Define the structure and format of financial reports
- **Generated Reports**: Records of reports that have been generated
- **Report Types**:
  - Balance Sheet
  - Income Statement
  - Cash Flow Statement
  - Statement of Changes in Equity
  - Tax Returns
  - Custom Reports

### Reports Generation Process

1. **Data Collection**: Aggregates data from all accounting modules
2. **Data Processing**: Applies accounting rules and calculations
3. **Report Formatting**: Structures data according to report templates
4. **Output Generation**: Creates reports in various formats (PDF, Excel, XML, etc.)

## End-to-End Accounting Process

### System Setup

#### 1. Chart of Accounts Setup

1. **Define Account Types**:
   - Create necessary account types (Asset, Liability, Equity, Revenue, Expense)
   - Set credit/debit normal behavior for each type
   - Establish reporting sequence

2. **Create Account Categories**:
   - Define categories within each account type
   - Establish category hierarchies if needed
   - Assign appropriate codes to categories

3. **Build Chart of Accounts**:
   - Create account entries with unique account numbers
   - Assign each account to the appropriate account type and category
   - Configure account properties (postable status, reconciliation requirements)
   - Set up account hierarchies for reporting
   - Define financial statement sections and reporting groups

#### 2. Fiscal Period Configuration

1. **Define Fiscal Years**:
   - Create fiscal years with start and end dates
   - Align with the organization's reporting calendar

2. **Configure Accounting Periods**:
   - Set up monthly or quarterly periods within fiscal years
   - Ensure periods cover the entire fiscal year without gaps or overlaps

#### 3. System Controls Setup

1. **Configure User Access**:
   - Set up appropriate permissions for accounting staff
   - Define approval workflows for transactions

2. **Establish Validation Rules**:
   - Configure rules for transaction validation
   - Set up controls for preventing posting to closed periods

### Daily Operations

#### 1. Supplier Invoice Processing

1. **Invoice Receipt and Entry**:
   - Record supplier invoices with all relevant details
   - Attach supporting documentation
   - Assign proper account codes to invoice lines

2. **Invoice Approval and Posting**:
   - Submit for approval based on established workflows
   - Review and approve invoice coding
   - Post approved invoices to the general ledger

3. **Payment Processing**:
   - Track payment due dates
   - Process payments
   - Record payment transactions in the system

#### 2. Customer Invoice Processing

1. **Invoice Creation**:
   - Generate customer invoices with appropriate line items
   - Assign correct revenue accounts
   - Calculate taxes and totals

2. **Invoice Issuance**:
   - Review and approve invoices
   - Issue to customers
   - Post to the general ledger

3. **Payment Receipt Processing**:
   - Record customer payments
   - Apply payments to appropriate invoices
   - Update customer account balances

#### 3. General Accounting Transactions

1. **Journal Entry Creation**:
   - Create journal entries for miscellaneous transactions
   - Ensure balanced entries (debits = credits)
   - Provide adequate descriptions and documentation

2. **Review and Approval**:
   - Review journal entries for accuracy
   - Obtain necessary approvals
   - Ensure proper account coding

3. **Posting and Documentation**:
   - Post approved entries to the general ledger
   - Maintain supporting documentation

### Period-End Procedures

#### 1. Account Reconciliation

1. **Bank Reconciliation**:
   - Reconcile bank statements with system records
   - Investigate and resolve discrepancies
   - Record adjusting entries if needed

2. **Subledger Reconciliation**:
   - Reconcile accounts receivable aging to general ledger
   - Reconcile accounts payable aging to general ledger
   - Investigate and resolve discrepancies

3. **General Account Review**:
   - Review account balances for reasonableness
   - Investigate unusual balances or transactions
   - Ensure proper accruals and deferrals

#### 2. Adjusting Entries

1. **Accruals and Deferrals**:
   - Record accruals for expenses incurred but not yet recorded
   - Record deferrals for expenses paid in advance
   - Process revenue recognition adjustments

2. **Depreciation and Amortization**:
   - Calculate and record depreciation on fixed assets
   - Record amortization of intangible assets

3. **Other Adjustments**:
   - Record inventory adjustments
   - Process foreign currency revaluations
   - Record any other period-end adjustments

#### 3. Period Closing

1. **Pre-Closing Review**:
   - Verify all transactions are posted for the period
   - Ensure all reconciliations are complete
   - Review preliminary financial statements

2. **Period Close Process**:
   - Process closing entries if required
   - Mark accounting period as closed
   - Lock transactions in closed periods

### Reporting Processes

#### 1. Financial Statement Generation

1. **Balance Sheet Preparation**:
   - Generate balance sheet as of period end
   - Compare with previous periods
   - Review for accuracy and completeness

2. **Income Statement Generation**:
   - Generate income statement for the period
   - Compare with budget and previous periods
   - Review for accuracy and completeness

3. **Cash Flow Statement**:
   - Generate cash flow statement
   - Analyze cash flow by operating, investing, and financing activities

#### 2. Management and Analysis Reports

1. **Budget vs. Actual Reports**:
   - Generate budget comparison reports
   - Analyze variances
   - Provide explanations for significant variances

2. **Financial Ratio Analysis**:
   - Calculate key financial ratios
   - Trend analysis over time
   - Benchmark against industry standards

3. **Custom Reports**:
   - Generate specialized reports as needed
   - Department or project-specific reports
   - Cost center analysis

#### 3. External Reporting

1. **Tax Reporting**:
   - Generate reports for tax filing purposes
   - Export data in required formats
   - Maintain compliance with tax requirements

2. **Regulatory Reporting**:
   - Prepare reports for regulatory bodies
   - Ensure compliance with reporting standards
   - Submit within required timeframes

3. **Shareholder/Stakeholder Reporting**:
   - Generate annual financial reports
   - Create investor presentations
   - Provide required disclosures

## Technical Implementation

The accounting system is implemented using:

- **Backend**: Django with Django REST Framework
- **Frontend**: Next.js with shadcn/UI components
- **Database**: Structured to maintain referential integrity and audit trails
- **API**: RESTful API for integration with other system modules
- **Security**: Role-based access control with detailed permissions

All accounting modules follow these principles:

1. **Double-Entry Accounting**: Every transaction maintains balance between debits and credits
2. **Audit Trails**: Comprehensive logging of all changes to financial data
3. **Data Integrity**: Validation rules ensure accounting consistency
4. **Workflow States**: Transactions follow defined workflows from draft to posted
5. **Separation of Concerns**: Core structures separate from transaction processing

