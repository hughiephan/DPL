# Popularity Bias with LightFM

The model’s prediction for user u and item i is then given by the dot product of user and item representations, adjusted by user and item feature biases. If the model recommends the same popular items to all users, the easiest solution is to set the item bias vectors to all zeros.

## Step 1: Import libraries

```python
!pip install lightfm
import numpy as np
import pandas as pd
from lightfm import LightFM
from sklearn.model_selection import train_test_split
from scipy.sparse import coo_matrix
```

## Step 2: Dataset

Download `Instacart Market Basket Analysis` from https://www.kaggle.com/datasets/psparks/instacart-market-basket-analysis . Then we use Pandas to read two CSV files `products.csv` and `order_products__train.csv`, representing product information and order details respectively. The `order_products__train.csv` file is read with a restriction on the number of rows `nrows=1000` as the score operation on the full matrix often results in out-of-memory notebook. After that, we construct a sparse matrix called `order_product_matrix` using the COO (Coordinate List) format from the `order_id'` and `product_id` columns, with `reordered` serving as the data values. Finally, the data is split into 80% data for training and 20% data for testing.

```python
df_products = pd.read_csv('/kaggle/input/instacart-market-basket-analysis/products.csv')
df_orders = pd.read_csv('/kaggle/input/instacart-market-basket-analysis/order_products__train.csv', nrows=1000)
order_product_matrix = coo_matrix((df_orders['reordered'], (df_orders['order_id'], df_orders['product_id']))).tocsr()
train_data, test_data = train_test_split(order_product_matrix, test_size=0.2)
```

## Step 3: Helper function

We create a dictionary to map the `product id` to their `product name` 

```python
product_id_to_name = df_products.set_index('product_id')['product_name'].to_dict()
```

## Step 4: Training

Start training our LightFM model in 10 epochs

```python
model = LightFM()
model.fit(train_data, epochs=10)
```

## Step 5: Score

![image](https://github.com/hughiephan/DPL/assets/16631121/98fd531b-4fbf-430a-bf87-34c3c188754e)

`get_item_representations()` and `get_user_representations()` functions extract the learned biases and embeddings for products and orders, respectively. We will be computing the recommendation scores using these representations. The scores are calculated by taking the dot product of the order embeddings and the transpose of the product embeddings, adding the product biases, and then adding the order biases. This score matrix represents the predicted affinity of each order for each product. We also calculate `scores_without_popularity_bias` like the previous score, but without adding the product biases so the model won't recommend popular products to the same order

```python
product_biases, product_embeddings = model.get_item_representations()
order_biases, order_embeddings = model.get_user_representations()
scores = ((order_embeddings @ product_embeddings.T + product_biases).T + order_biases).T
scores_without_popularity_bias = ((order_embeddings @ product_embeddings.T).T + order_biases).T
```

## Step 6: Make recommendations

Comparing the recommendations between the base model and the model without popularity bias

```python
order_id = 1
print(f'\nTop 5 Recommendations for Order {order_id}:')
for i in np.argsort(scores[order_id])[::-1][:5]:
    product_name = product_id_to_name.get(i, f'Unknown Product {i}')
    print(f'Product: {product_name}')

print(f'\nTop 5 Recommendations for Order {order_id} (without popularity bias):')
for i in np.argsort(scores_without_bias[order_id])[::-1][:5]:
    product_name = product_id_to_name.get(i, f'Unknown Product {i}')
    print(f'Product: {product_name}')
```

![image](https://github.com/hughiephan/DPL/assets/16631121/41502cee-3717-413e-8efa-76d6c5b3a371)

## Reference
- https://www.kaggle.com/code/niyamatalmass/lightfm-hybrid-recommendation-system
- https://www.stepbystepdatascience.com/hybrid-recommender-lightfm-python
- https://www.datarevenue.com/en-blog/building-a-production-ready-recommendation-system
- https://github.com/lyst/lightfm/issues/395
