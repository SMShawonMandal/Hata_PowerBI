# Hata Footwear — Power BI Business Intelligence Dashboard

A comprehensive, multi-page Power BI dashboard built for **Hata**, a Bangladeshi footwear brand operating across **12 stores** in Dhaka and Chattogram. This project demonstrates end-to-end BI development — from raw data to star schema modeling to interactive report design.

---

## 📊 Dashboard Overview

| Metric | Value |
|--------|-------|
| **Total Revenue** | ৳1.40 Billion |
| **Gross Profit** | ৳447.23 Million |
| **Profit Margin** | 32.1% |
| **Total Orders** | 249K |
| **Total Customers** | 32K |
| **Return Rate** | 3.6% |
| **Employees** | 158 |
| **Stores** | 12 |
| **Data Period** | January 2023 – December 2024 |

---

## 📄 Report Pages

### Page 1: Executive Overview
High-level KPIs, revenue trend (with gross profit overlay), revenue by city, payment method distribution, and a ranked best sellers table.


### Page 2: Sales & Products Analytics
Category × Brand matrix with conditional formatting, revenue share treemap, top/bottom products by profit, units sold by size, revenue by color, and material distribution.


### Page 3: Customers & Operations
Customer KPIs (32K customers, 99.6% repeat rate), revenue by customer city, orders by gender, top 10 customers, store performance matrix with employee counts, and top employees by revenue.


### Page 4: Returns Analysis
6 KPI cards including net revenue and orders affected, monthly return trend with markers, return details by product category with return rate conditional formatting, and reason breakdowns via bar chart and donut.


### Page 5: Payment Analytics
Payment method percentage cards, multi-line trend chart showing Cash/Credit Card/Debit Card/Mobile Banking over time, store-level stacked bar breakdown, and payment summary table.


### Page 6: Store & Employee Performance
Store performance comparison matrix (Revenue, Orders, AOV, Employee Count, Revenue/Employee), city revenue trend (Dhaka vs Chattogram), top 10 employees by revenue, and employee details table with tenure and salary.


---

## 🏗️ Data Model Architecture

### Star Schema Design

The data model follows a **star schema** architecture centered on `fact_Sales`, with satellite fact tables for returns and payments.

```
                    dim_Date
                       │
                       │ (Active: SaleDate)
                       │ (Inactive: ReturnDate, PaymentDate)
                       │
dim_Customers ──── fact_Sales ──── dim_Stores
                   │   │   │            │
                   │   │   │            │ (Inactive)
                   │   │   │            │
                   │   │   │       dim_Employees
                   │   │   │
          fact_SaleItems  fact_Returns
               │          fact_Payments
               │
        dim_Products
               │
          dim_Sizes
```

### Tables

| Table | Type | Records | Description |
|-------|------|---------|-------------|
| `fact_Sales` | Fact | ~249K | Core transaction table |
| `fact_SaleItems` | Fact | ~508K | Line-item details per sale |
| `fact_Returns` | Fact | ~18K | Return transactions |
| `fact_Payments` | Fact | ~249K | Payment method per sale |
| `dim_Date` | Dimension | Calendar | Auto-generated date table |
| `dim_Customers` | Dimension | ~32K | Customer demographics |
| `dim_Products` | Dimension | ~50 | Product catalog |
| `dim_Stores` | Dimension | 12 | Store locations |
| `dim_Employees` | Dimension | 158 | Employee details |
| `dim_Sizes` | Dimension | ~10 | Size reference |

### Relationships

- **9 Active Relationships**: Standard dimension-to-fact connections
- **5 Inactive Relationships**: Used with `USERELATIONSHIP()` in DAX to avoid circular/ambiguous paths
  - `dim_Date → fact_Returns[ReturnDate]`
  - `dim_Date → fact_Payments[PaymentDate]`
  - `dim_Products → fact_Returns[Product_ID]`
  - `dim_Sizes → fact_Returns[Size_ID]`
  - `dim_Stores → dim_Employees[StoreID]`

---

## 📐 DAX Measures (40+)

### Revenue & Profitability
```dax
m_TotalRevenue = SUMX(fact_SaleItems, fact_SaleItems[Quantity] * RELATED(dim_Products[Price]))
m_TotalCost = SUMX(fact_SaleItems, fact_SaleItems[Quantity] * RELATED(dim_Products[Cost]))
m_GrossProfit = [m_TotalRevenue] - [m_TotalCost]
m_ProfitMarginPct = DIVIDE([m_GrossProfit], [m_TotalRevenue], 0)
m_NetRevenue = [m_TotalRevenue] - [m_TotalReturnAmount]
```

### Orders & Customers
```dax
m_TotalOrders = DISTINCTCOUNT(fact_Sales[Sale_ID])
m_TotalCustomers = DISTINCTCOUNT(fact_Sales[Customer_ID])
m_AvgOrderValue = DIVIDE([m_TotalRevenue], [m_TotalOrders], 0)
m_RepeatCustomerPct = DIVIDE([m_RepeatCustomers], [m_TotalCustomers], 0)
```

### Returns (using USERELATIONSHIP)
```dax
m_TotalReturnsQty = SUM(fact_Returns[QuantityReturned])
m_ReturnRatePct = DIVIDE([m_TotalReturnsQty], [m_TotalQtySold], 0)
m_ReturnsByReturnDate = CALCULATE(
    SUM(fact_Returns[ReturnAmount]),
    USERELATIONSHIP(dim_Date[Date], fact_Returns[ReturnDate])
)
```

### Employee & Store (using USERELATIONSHIP)
```dax
m_EmployeeCount = CALCULATE(
    DISTINCTCOUNT(dim_Employees[EmployeeID]),
    USERELATIONSHIP(dim_Stores[StoreID], dim_Employees[StoreID])
)
m_RevenuePerEmployee = DIVIDE([m_TotalRevenue], DISTINCTCOUNT(fact_Sales[Employee_ID]), 0)
```

### Payments
```dax
m_CashPct = DIVIDE([m_CashRevenue], [m_TotalRevenue], 0)
m_CreditCardPct = DIVIDE([m_CreditCardRevenue], [m_TotalRevenue], 0)
m_DebitCardPct = DIVIDE([m_DebitCardRevenue], [m_TotalRevenue], 0)
m_MobilePct = DIVIDE([m_MobileBankingRevenue], [m_TotalRevenue], 0)
```

### Time Intelligence
```dax
m_RevenuePrevMonth = CALCULATE([m_TotalRevenue], PREVIOUSMONTH(dim_Date[Date]))
m_RevenuePrevYear = CALCULATE([m_TotalRevenue], SAMEPERIODLASTYEAR(dim_Date[Date]))
m_RevenueMoMGrowthPct = DIVIDE([m_TotalRevenue] - [m_RevenuePrevMonth], [m_RevenuePrevMonth], 0)
```

### Rankings
```dax
m_ProductRankByRevenue = IF(HASONEVALUE(dim_Products[ProductName]),
    RANKX(ALL(dim_Products[ProductName]), [m_TotalRevenue], , DESC, Dense), BLANK())
```

---

## 💡 Key Insights

| Insight | Detail |
|---------|--------|
| **Customer Loyalty** | 99.6% repeat customer rate — extremely loyal customer base |
| **Healthy Returns** | 3.6% return rate across 18K returns — well within industry standards |
| **Top Performer** | Pahartali store leads in Revenue/Employee (৳16.64M) with just 8 employees |
| **Payment Mix** | Nearly equal split: Cash 25.0%, Debit Card 25.1%, Credit Card 24.8%, Mobile 25.0% |
| **Revenue Leader** | "Hush Puppies ANDERSON Slip-On" is the best seller at ৳99.9M revenue |
| **Category Split** | Men's Footwear dominates (৳891M, 64%) vs Women's (৳465M, 33%) vs Kids' (৳38M, 3%) |
| **Geographic Split** | Revenue nearly balanced between Dhaka and Chattogram stores |
| **Return Reasons** | Evenly distributed: Wrong Size, Unsatisfied, Changed Mind, Gift Return, Defective Product (~20% each) |

---

## 🛠️ Technical Highlights

- **Star Schema**: Designed with inactive relationships to handle circular paths between Date/Product/Size dimensions and multiple fact tables
- **USERELATIONSHIP()**: Used extensively for returns-by-return-date, payments-by-payment-date, and employee-count-per-store measures
- **Power Query (M)**: Date format parsing (dd/MM/yyyy with locale), delimiter-based column splitting for store Area labels, calculated columns for employee tenure and full names

- **40+ DAX Measures**: Organized in display folders covering Revenue, Profitability, Orders, Customers, Returns, Payments, Employees, Time Intelligence, and Rankings
- **Interactive Slicers**: Year, Quarter, City, Area, Category, Brand, Gender, Role, and Reason filters across all pages

---

## 📁 Project Structure

```
Hata/
├── README.md                        # This file
├── pbix/                            # Power BI report file
│   └── Hata.pbix
├── pdf/                             # Exported PDF report
│   └── Hata_Dashboard.pdf
└── csv/                             # Source CSV files
    ├── dim_Stores.csv
    ├── dim_Products.csv
    ├── dim_Sizes.csv
    ├── dim_Customers.csv
    ├── dim_Employees.csv
    ├── fact_Sales.csv
    ├── fact_SaleItems.csv
    ├── fact_Returns.csv
    └── fact_Payments.csv
```

---

## 🚀 How to Use

1. **Download** or clone this repository
2. Open `pbix/Hata.pbix` in **Power BI Desktop** (free)
3. If prompted, update the data source paths to point to the `csv/` folder
4. Click **Refresh** to load the data
5. Navigate between the 6 report pages using the tabs at the bottom

---

## 👤 Author

**Shawon Mandal**

---

## 📜 License

This project is for portfolio and educational purposes. The dataset is generated by me using faker library and free to use.
