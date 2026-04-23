# ⚡ Power Grid Demand Forecasting (Predictive Paradox)

**Project Goal :** Predict hourly power demand for 2024. To do this, I trained a model on previous year grid usage (2016-2023) and I also factored in the weather and economic data of these years and how they affect the demand.

**Final Selected Model :** LightGBM Regressor  
**MAPE of Selected Model :** 2.46% (Mean Absolute Percentage Error)

---
## How I Built It (Methodology)

### 1. Cleaning the Data & Fixing Spikes
Power grids have weird spikes from sensor errors or extreme events. Instead of just guessing what was an outlier, I used the **IQR (Interquartile Range) method**. This sets "invisible fences" based on how the data normally behaves and flags anything completely outside those fences.
* For the missing values and the bad data I removed, I used **time-based interpolation**. This is much better than just plugging in an average, because it draws a smooth, logical line between the valid points before and after the gap.

### 2. Feature Engineering
ATree-based classical ML models (XGBoost/LightGBM) cannot inherently comprehend the flow of time. I engineered specific features to give the model temporal awareness:
* **Lags:** I created columns for the demand 1 hour ago (`lag_1h`) and 24 hours ago (`lag_24h`). This gives the model "short-term memory."
* **Time Cycles:** I pulled out the exact hour, day of the week, and month. This helps the model realize that electricity demand follows human habits (like using less power on weekends or more in the evenings).

### 3. Including Weather and Economic datasets
* **Weather:** I merged hourly temperature data to account for things like sudden AC usage during heatwaves.
* **Economics:** I pulled in World Bank data (like GDP). Since their data is yearly, I had to transpose it to match my hourly timestamps. I also used a **forward-fill (`.ffill()`)** strategy because real-world economic data is usually reported late. This made sure my 2024 predictions had the most recent 2023 data to lean on.

*Economic features showed relatively low significance in the Feature Importance charts compared to `lag_1h` and `hour`. This aligns with domain logic: macro-economics dictate the yearly baseline, but short-term lags dictate the hour-to-hour variance*

## 📊 Model Benchmarking & Selection

I didn't just pick one algorithm; I benchmarked three different ones on a strict Train/Test split (Train: <= 2023, Test: 2024) split to see which handled the grid patterns the best

| Model | Architecture Type | Test MAPE (2024) | Notes |
| :--- | :--- | :--- | :--- |
| **LightGBM** | Boosting (Leaf-wise) | **~2.46%** | **Winner!!!** It was the fastest to train and caught the hourly trends perfectly. |
| **Random Forest** | Bagging | ~2.73% | Very stable and ignored the noisy data well, but it missed some of the sharper daily peaks. |
| **XGBoost** | Boosting (Level-wise) | ~2.77% | A solid baseline, but LightGBM's leaf-wise tree growth worked a bit better for this specific data. |


---

## 🔬 The Failed Experiment (Ablation Study)

After hitting a 2.46% error rate with LightGBM, I tried to make it even better. I added a 168-hour (1 week) lag and slowed down the learning rate to `0.01` with also increasing the trees to 2,000 and early stopping.

**It actually got worse (2.46% MAPE).** Why? The early stopping triggered at just 183 trees, meaning the model *underfit* the data. Plus, the 1-week lag probably confused the model whenever a holiday or weird weather event didn't line up perfectly with the week before. 

So, I went back to my original simpler LightGBM model. It proved that for this grid data, a strong, simple baseline with 24-hour lags beats a wildly complex setup!

---
### 👤 Submitted By
**Mohammed Shameem** Roll Number: 250106044  
B.Tech, BSBE,
IIT Guwahati
