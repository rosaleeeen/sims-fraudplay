⠀⠀⠀⠀⢀⣤⣶⣶⣿⣿⣶⣶⣤⣀⠀⠀⠀⠀  ⠀⠀⠀⠀⢀⣤⣶⣶⣿⣿⣶⣶⣤⣀⠀⠀⠀⠀  
⠀⠀⢀⣴⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣷⡄⠀⠀  ⠀⠀⢀⣴⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣷⡄⠀⠀  
⠀⢠⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡄⠀  ⠀⢠⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡄⠀  
⠀⣼⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⠀  ⠀⣼⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⠀  
⣾⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣷  ⣾⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣷  
⠈⠉⠉⠉⣉⣉⠉⠉⠉⠉⠉⠉⣉⣉⡉⠉⠉⠉  ⠈⠉⠉⠉⣉⣉⠉⠉⠉⠉⠉⠉⣉⣉⡉⠉⠉⠉  
⢀⣴⣿⣿⣿⣿⣿⣷⣶⣶⣾⣿⣿⣿⣿⣿⣦⡀  ⢀⣴⣿⣿⣿⣿⣿⣷⣶⣶⣾⣿⣿⣿⣿⣿⣦⡀  
⢸⣿⣿⣅⠀⢀⣨⣿⣿⣿⣿⣇⡀⠀⣈⣿⣿⡿  ⢸⣿⣿⣅⠀⢀⣨⣿⣿⣿⣿⣇⡀⠀⣈⣿⣿⡿  
⠀⠙⠿⢿⣿⣿⡿⠿⠛⠛⠿⢿⣿⣿⡿⠿⠛⠁  ⠀⠙⠿⢿⣿⣿⡿⠿⠛⠛⠿⢿⣿⣿⡿⠿⠛⠁   
</br>
# -`,🕵️ sims-fraudplay 💸₊˚💲
💳/)/) </br>
( ^>^) </br>
( づ💰  </br>
**a playground for simulating credit card transactions and teaching models to spot fraud.**  
> generates fake customers and terminals to produce fake transactions, injects weird fraud behaviours (according to three common scenarios), then see which models actually notice. 

---

## ❓ functionality ❓

- spins up **synthetic customers**, **terminals**, and a **baseline transaction stream**  
- injects three fraud scenarios that mirror real world patterns  
- annotates useful **augmented features** (time since last txn, night activity, spending drift, etc.)  
- trains a few **lightweight models** and compares them with **ROC AUC** under heavy class imbalance  

loosely follows the following lovely guide: [fraud detection handbook – model selection](https://fraud-detection-handbook.github.io/fraud-detection-handbook/Chapter_5_ModelValidationAndSelection/ModelSelection.html).

---

## 🚨 fraud scenarios 🚨

fraudulent rows are labeled with `IS_FRAUD = 1`.

- **go big or go home 🎰🤑** 
  - stolen card used for a few huge purchases.  
  - once per day: pick a customer and create a new high value txn if they did not swipe that day.

- **under wraps rapid fire ⌛️💨**  
  - lots of tiny charges to fly under the radar. 
  - each day: pick 3 to 5 customers and inject 10 small transactions per day for the next 3 days, with amounts between 1 and 5.

- **night owl 🦉🌚**  
  - amounts look normal, timing does not (... perhaps a thief who enjoys a midnight snack).  
  - every 4 days: pick 2 customers and inject several transactions between 00:00 and 04:00 over the current and following day.



## 🫆 features 🫆

transactions are annotated before modeling to add augmented features. each transaction includes the following features:

- `TRANSACTION_AMOUNT`   :: raw amount, captures both micro charges and huge spikes.  
- `TXN_DAYS_ELAPSED`     :: day index relative to the simulation start.  
- `HOUR`, `DAY_OF_WEEK`  :: basic time of day and weekday context.  
- `IS_NIGHT`             :: flag for transactions between midnight and 4am.  
- `SEC_SINCE_PREV`       :: seconds since the previous transaction for that customer.  
- `CUST_TXN_COUNT`       :: running count of how many times this customer has transacted.  
- `CUST_RUNNING_MEAN_AMT`:: rolling average amount per customer up to the previous txn.  
- `CUST_NIGHT_RATE`      :: fraction of this customer’s history that happened at night.  
- `MEAN_PURCHASE_AMOUNT`, `STD_PURCHASE_AMOUNT` :: baseline spend profile from the customer table.


## 📈 models 📈

training, validation, and testing occurs with the following models:
- **logistic regression** in a `Pipeline` w/ `StandardScaler` and `class_weight="balanced"`.  
- **SGDClassifier** w/ logistic loss for streaming style linear baseline.  
- **ExtraTreesClassifier** w/ shallow trees for cheap non linear structure.  
- **GradientBoostingClassifier** w/ a small number of trees for extra separation.

evaluation is mostly:
- **ROC AUC** on a held out test set  
- classification report for the fraud class  
- basic feature importances for tree based models  

-----

## 🧱 prerequisites 🧱

- python 3.11 or similar  
- a virtual environment tool you like  
- packages (roughly):  

```text
pandas
numpy
scikit-learn
matplotlib
seaborn
jupyter
