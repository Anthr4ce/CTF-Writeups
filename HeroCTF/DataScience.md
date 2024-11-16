Dans ce challenge, nous devons trier des données et obtenir certains nombres.
Aprés plusieurs essays, je suis arrivé à ce code:

```python
import pandas as pd  
import math  
  
df = pd.read_csv("orders.csv", parse_dates=["date"])  
  
df = df[df["date"] < "2023-01-01"]  
  
initial_balance = 10000  
user_balances = {}  
  
unique_users = pd.concat([df["buyer_id"], df["seller_id"]]).unique()  
user_balances = {}
for user_id in unique_users:
    user_balances[user_id] = initial_balance

total_discount_spared = 0  
  
for _, row in df.iterrows():  
    buyer_id = row["buyer_id"]  
    seller_id = row["seller_id"]  
    price = row["price"]  
    discount = row["discount"]  
  
    discounted_price = price * (1 - discount / 100)  
    amount_spared = price - discounted_price  
    total_discount_spared += amount_spared  
  
    user_balances[buyer_id] -= discounted_price  
    user_balances[seller_id] += discounted_price  
  
#resp 1  
richest_user = max(user_balances, key=user_balances.get)  
  
#resp 2  
total_discount_spared = math.floor(total_discount_spared)  
  
#resp 3  
negative_balance_count = 0
for balance in user_balances.values(): 
        if balance < 0:
                    negative_balance_count += 1
  
#print du flag  
flag = f"Hero{{{richest_user}_{total_discount_spared}_{negative_balance_count}}}"  
print(flag)
```

```t
Hero{732669_188098001_3468}
```