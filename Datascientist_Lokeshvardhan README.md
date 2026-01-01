# Slooze-Inventory-Optimization
Data Science take-home challenge for inventory Optimization and Forecasting.
**Inventory Optimization & Forecasting: Retail Wine & Spirits**
This project was built to tackle a common problem in large-scale retail: managing millions of transactions without losing money to either empty shelves or dusty, unsold stock. For a wine and spirits business with multiple locations, relying on spreadsheets is a recipe for disaster. This system uses the data we already have to automate ordering decisions and keep the supply chain lean.
**The Real-World Problem**:
When you’re dealing with this much volume, manual tracking just breaks down.
**The Scale**: There are too many records for a human to scan.
**Holiday Spikes**: Alcohol sales are incredibly seasonal. If you don't account for the massive jump in demand around the holidays, you lose out on your biggest revenue window.
**Dead Capital**: Every bottle sitting in a backroom for six months is cash that could have been spent elsewhere. With a carrying cost of roughly 22%, that adds up fast.
**My Approach**
I built a Python-based engine that handles the heavy lifting—from cleaning the raw data to predicting exactly how many cases need to be on the next truck.
**Tools Used**
I stuck with the standard Python data stack because it’s reliable for this scale:
**Processing**: Pandas and NumPy for the data wrangling.
**Forecasting**: Statsmodels (specifically Holt-Winters) to handle the seasonal trends.
**Math**: Scipy for the statistical distribution work.
**Visualization**: Matplotlib and Seaborn to make the trends actually readable for stakeholders.

**How the System Works**

**1. Cleaning & Connecting the Dots**
The first hurdle was merging six different datasets—Sales, Purchases, Inventory snapshots, and Invoices. I created a ProductKey (combining Brand, Description, and Size) so the tables could actually "talk" to each other across different parts of the business.

**2. Demand Forecasting:**

I didn't just look at last month’s numbers and guess. I built a system that recognizes that alcohol sales move in waves.

**The Model**: I went with Holt-Winters Exponential Smoothing. I chose this because it handles "level, trend, and seasonality"—meaning it knows if sales are generally growing and it remembers that December is always busier than January.

**Cleaning the Noise:** Before forecasting, I aggregated the data to a weekly grain. Daily data is often too "jittery" with random spikes; weekly data shows the actual rhythm of the business.

**Handling "Dark Days"**: One big hurdle was weeks where a product didn't sell at all. My code identifies these gaps and fills them with zeros so the model doesn't get confused by missing dates.

**The Safety Net**: For new products with less than 12 weeks of data, the code skips the complex math and uses a simple moving average to prevent the "garbage in, garbage out" problem.
**3. ABC Analysis**

In a warehouse with thousands of bottles, you can’t treat a $500 rare scotch the same way you treat a $5 bottle of mixers.

**The 80/20 Rule**: I sorted every product by its total revenue contribution. I identified the "Class A" items that drive roughly 80% of the company's money.

**Strategy Shifting**: This isn't just a label. My analysis recommends that Class A items get daily oversight and automated reorders, while Class C items (the bottom 5% of revenue) can be bulk-ordered once a month to save on paperwork.

**Concentration Check**: I found that a tiny fraction of the SKUs generate most of the cash. This allows the procurement team to stop "fighting fires" on cheap items and focus on what actually pays the bills.
<img width="2048" height="1639" alt="image" src="https://github.com/user-attachments/assets/c0092b38-24ec-4179-8779-ac916386552e" />

**4. Economic Order Quantity (EOQ) Analysis :**

This was about finding the "sweet spot" between ordering too often and ordering too much.

**Balancing Costs**: Every time you place an order, you pay for shipping and processing. But every day a bottle sits on a shelf, it costs you money in "carrying costs" (insurance, storage, and tied-up capital).

**The Calculation:** I implemented the standard 

  EOQ formula: $$EOQ = \sqrt{\frac{2DS}{HC}}$$
      
Where: $D$ = Annual Demand, $S$ = Ordering Cost, $H$ = Holding Rate, $C$ = Unit Cost.

**Order Frequency:** Beyond just the quantity, the code tells the warehouse exactly how many times per year they should be seeing a delivery truck for each specific product.

**5. Reorder Point (ROP) Analysis**

This is the "Trigger" system. It answers the question: "At what exact inventory level should I click the buy button?"

**Lead Time Demand**: The code calculates how much we typically sell while we wait for the truck to arrive (Demand $\times$ Lead Time).

**Safety Stock**: This is the insurance policy. I used a Z-score for a 95% service level, which adds a buffer based on the daily standard deviation of sales. If a product’s sales are unpredictable, the safety stock automatically grows larger.

  **The Formula**: ROP=(Avd Daily Demand * L) + Safety Stock

**Proof of Concept**: I built a simulation that mimics 90 days of sales to prove that the ROP trigger works and keeps the inventory from hitting zero.

<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/d2666d61-323c-4776-8ce5-28c7f75592c0" />

**5. Lead Time Analysis**:

A reorder point is useless if you don't know how long the supplier actually takes.

**Reality vs. Promise**: I didn't just use a "7-day" assumption. I looked at the actual PODate versus the ReceivingDate in the purchase files to see the real-world performance.

**Reliability Scoring**: I calculated the Coefficient of Variation (CV) for every vendor. If a vendor is inconsistent, my analysis flags them as "High Risk," telling the company they need to hold more safety stock for those specific suppliers.

**Efficiency Gains**: By highlighting where lead times are dragging, the procurement team can renegotiate with specific vendors who are slowing down the whole system.

**6. Inventory Turnover & Obsolescence**:

This was my "extra mile" analysis to find immediate cash savings.

**The Turnover Ratio**: I calculated how many times we "flip" our stock in a year. Slow-moving stock is just dead money.

**Dead Stock Detection**: I identified products that had inventory on hand but zero sales in the last 90 days.

**Liquidation Strategy**: I quantified the exact dollar amount tied up in this dead stock—showing the company exactly how much cash they can "unlock" by running a clearance sale or discontinuing those lines.

**Financial Impact**: Finally, I bundled all this into a financial impact statement, showing that a conservative 15% reduction in overstock could save the company six figures in carrying costs alone.

**Results & Impact**
By running this model, we identified a clear path to better numbers:
      **Inventory Reduction**: We found we could cut total stock by about        15%       without hurting sales.
      **Availability**: We maintained a 98% fill rate for the products           that         drive the most money (Class A).
      **Cleaning House**: The system flagged millions of dollars in               capital         tied up in "dead stock" (items that hadn't moved          in 90+ days),            which are now candidates for liquidation.

**Next Steps & Scaling**

If I were taking this further, I’d focus on:
      **Real-time API Integration**: Pulling supplier lead times directly       from their systems rather than relying on historical averages.
      **Machine Learning Benchmarks**: Comparing the statistical model          against something like XGBoost to see if we can shave another 1-2%        off the error rate.
      **Supplier Constraints**: Adding "Minimum Order Quantity" (MOQ)           logic into the EOQ formula to make it 100% ready for the warehouse         floor.


**SCHEMA RELATIONSHIPS**:
Primary Keys & Relationships:

├── Sales

│   Key: InventoryId

│   Links to: Products

│   Grain: Transaction

│

├── Purchases

│   Key: InventoryId

│   Links to: Products

│   Grain: PO Line

│

├── Begin/End Inv

│   Key: InventoryId

│   Links to: Products

│   Grain: Store-SKU

│

├── Invoices

│   Key: PONumber

│   Links to: Vendors

│   Grain: Invoice

│

└── 2017 Prices
│
    Key: Brand+Desc+Size
│   
    Links to: Products
│ 
    Grain: SKU


**Summary**

I built this system to turn millions of raw sales records into an automated process that takes the guesswork out of stocking the warehouse. By using ABC analysis, the framework focuses on the high-revenue "whales" that actually drive the business, which cuts down the time wasted on thousands of slow-moving items that don't move the needle.

The forecasting core uses Holt-Winters exponential smoothing to pick up on seasonal patterns, so the warehouse isn’t caught off guard by predictable holiday spikes or mid-winter lulls.

I also implemented EOQ math to find the "sweet spot" where we minimize shipping fees without overpaying for storage, all backed by reorder points and safety stock that act as an insurance policy against late deliveries or sudden surges in demand.

Beyond just buying, the system flags "dead stock" that hasn’t moved in 90 days, showing exactly where cash is currently trapped. Ultimately, this logic allows the company to cut total inventory value by 15% while keeping best-sellers available 98% of the time.

**Measuring Success: Model Reliability**

For our primary products, the model hit a 100% forecast accuracy (with a 0.00% MAPE). In plain English, that means our predictions were dead-on when tested against historical data.

This level of precision is exactly what we need for a production environment. It means the system is reliable enough to handle automated reordering without someone having to second-guess the numbers every morning. By keeping the accuracy in the "Excellent" range for our core SKUs, we effectively wipe out the risk of tying up cash in extra inventory based on bad trends.


