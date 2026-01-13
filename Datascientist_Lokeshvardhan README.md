
This enables consistent joins across sales, purchases, and inventory records.

---

## System Architecture & Methodology

### 1. Demand Engineering

- Sales aggregated at the **daily SKU level**
- Missing dates filled with zeros to preserve time continuity
- Product-level demand statistics computed:
  - Average daily demand
  - Demand standard deviation
  - Annual demand volume

These statistics drive forecasting, EOQ, and safety stock calculations.

---

### 2. ABC Revenue Classification

Products are ranked by total revenue contribution and classified using cumulative thresholds:

| Class | Description |
|------|------------|
| **A** | Top ~80% of revenue |
| **B** | Next ~15% |
| **C** | Bottom ~5% |

This enables differentiated inventory strategies across high- and low-impact SKUs.

---

### 3. Economic Order Quantity (EOQ)

Optimal order quantities are calculated using the classical EOQ formulation:

\[EOQ = \sqrt{\frac{2DS}{HC}}\]

Where:
- **D** = Annual demand  
- **S** = Ordering cost (assumed: \$50 per order)  
- **H** = Carrying rate (assumed: 22%)  
- **C** = Unit cost (derived from sales data)  

EOQ balances ordering frequency against holding costs to minimize total inventory cost.

---

### 4. Lead Time Analysis

Actual supplier lead times are derived from purchase order history:

- Lead Time = Receiving Date − PO Date  
- Negative or invalid values are clipped  
- Missing values default to **7 days** (explicit assumption)

Average lead time per product feeds into reorder calculations.

---

### 5. Reorder Point (ROP) & Safety Stock

The reorder trigger is calculated using:

\[
ROP = (\text{Avg Daily Demand} \times \text{Lead Time}) + Z \times \sigma \times \sqrt{\text{Lead Time}}
\]

**Assumptions:**
- Service Level: **95%**
- Z-score derived from the normal distribution

This ensures inventory buffers dynamically adjust to demand volatility.

---

### 6. Inventory Turnover & Capital Efficiency

Inventory efficiency is evaluated using:

- Average inventory value (beginning + ending inventory)
- Cost of goods sold from purchases
- Inventory turnover ratio per SKU

Low-turnover items are flagged as candidates for rationalization or liquidation.

---

### 7. Demand Forecasting (Top Products)

Short-term forecasts are generated for the **top 50 revenue-driving products**:

- **Model:** Holt-Winters Exponential Smoothing  
- **Granularity:** Daily  
- **Seasonality:** Weekly (7-day cycle)  

**Fallback Logic:**
- Products with limited history use average daily demand

**Forecast Horizon:**
- 30-day cumulative demand

---

### 8. Supplier Performance & Reliability

Supplier efficiency is assessed using:

- Average lead time
- Lead time variability
- Order frequency
- Total spend

A reliability score penalizes vendors with inconsistent delivery performance, highlighting operational risk.

---

## Outputs & Insights

The system produces:

### Product-Level Optimization Table
- ABC class  
- Annual demand  
- EOQ  
- Reorder point  
- Inventory turnover  
- 30-day demand forecast  

### Additional Insights
- Top-performing SKUs by revenue and efficiency  
- Supplier reliability rankings  
- Automated alerts for:
  - Low-turnover Class A items
  - Excessive safety stock
  - High-risk suppliers
  - SKU over-proliferation

---

## Visual Analytics

Included visualizations:

- ABC classification distribution
- Inventory turnover by ABC class
- EOQ vs Reorder Point comparison
- Revenue concentration across top SKUs
- Supplier lead-time variability
- Historical vs forecasted demand (example SKU)

---
## Assumptions & Limitations

- Ordering cost and carrying rate are fixed constants  
- No explicit minimum order quantity (MOQ) constraints  
- Lead time defaults applied where historical data is missing  
- Forecast accuracy depends on historical data sufficiency  

All assumptions are clearly defined and easily configurable.

---

## Future Enhancements

- Real-time supplier lead time ingestion  
- MOQ-aware EOQ optimization  
- Machine learning forecast benchmarks (XGBoost, Prophet)  
- Automated purchase order generation  
- Dashboard deployment for stakeholders  

---

## Summary

This system converts fragmented retail data into a **cohesive, automated inventory decision engine**. By combining demand forecasting, statistical safety stock, ABC prioritization, and supplier risk analysis, it enables:

- Lower working capital  
- Higher service levels  
- Reduced operational firefighting  
- Data-backed procurement decisions  

The result is a scalable framework ready for real-world retail supply chains.

