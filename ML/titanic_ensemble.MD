# Titanic Disaster with Ensemble Learning

![image](https://github.com/hughiephan/DPL/assets/16631121/5597fb70-eeb6-4ec9-b1da-f48cd4c36a44)

By incorporating the diverse strengths of these three powerful machine learning techniques Artificial Neural Network, Random Forest, and SVM with ensemble majority voting, we can create a robust and reliable predictive model for determining the survival outcomes of Titanic passengers.

## Step 1: Import libraries
```python
import pandas as pd
import numpy as np
import keras
import sklearn
from sklearn import metrics
from sklearn.svm import SVC
from sklearn.ensemble import RandomForestClassifier
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split
from keras.models import Sequential
from keras.layers import Dense
```

## Step 2: Data

Download the dataset from `https://www.kaggle.com/competitions/titanic/data` . We only use `train.csv` for simplicity, but `test.csv` is also available if you want a more concrete approach.

```python
train_df = pd.read_csv("/kaggle/input/titanic/train.csv")
X = train_df.drop(['Survived'], axis=1)
Y = train_df['Survived']
```

![image](https://github.com/hughiephan/DPL/assets/16631121/6f4c6fcb-5cce-4740-9556-573678446770)

## Step 3: Fill missing values

For the `Embarked`, C stands for Cherbourg, Q stands for Queenstown, and S stands for Southampton, which indicates the port where each passenger boarded the Titanic. We use two methods for filling in missing values, where `mean` fills missing values with the mean of existing data, and `interpolate` fills missing values by generating new values based on the existing data's trend or pattern, you can also read about interpolate here `https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.interpolate.html` . For example, temperature increase is gradual and not sudden, so we cannot just copy the values from earlier recordings into the empty values. So in the time series data, we should use `interpolate`

```python
X['Embarked'] = X['Embarked'].fillna('C')
X['Age'] = X['Age'].fillna(X['Age'].mean())
X['Fare'] = X['Fare'].interpolate()
```

![image](https://github.com/hughiephan/DPL/assets/16631121/87a8e44e-ac1d-4236-bc6d-b064ab686ee7)

## Step 4: Encoding
```python
X['Sex'] = X['Sex'].map({'male': 0, 'female': 1})
X['Embarked'] = X['Embarked'].map({'S': 0, 'C': 1, 'Q': 2})
```

![image](https://github.com/hughiephan/DPL/assets/16631121/c12e7039-4341-4454-ae64-53bedf75e57c)

## Step 5: Scaling and split

StandardScaler is a preprocessing technique used to standardize features by removing the mean and scaling to unit variance. It's often applied to numerical features to ensure they have the same scale, which can be important for certain machine learning algorithms. We apply the StandardScaler transformation to the features in `X` after we have removed some features like `Name`, `Ticket`,... then transform the data using the `fit_transform()` and convert the resulting array back into a DataFrame, ensuring that all features in `X` is standardized.

```python
sc = StandardScaler()
X = X.drop(['Name', 'Ticket', 'Cabin'], axis=1)
X = pd.DataFrame(sc.fit_transform(X.values), index=X.index, columns=X.columns)
X_train, X_test, Y_train, Y_test = train_test_split(X, Y, test_size=0.2)
```

## Step 6: Neural network

![image](https://github.com/hughiephan/DPL/assets/16631121/d64723d0-b655-412a-9d16-e48928fbfa89)

```python
def build_ann():
    ann = Sequential()
    ann.add(Dense(units=32, activation='relu', input_shape=(X.shape[1],)))
    ann.add(Dense(units=64, activation='relu'))
    ann.add(Dense(units=1, activation='sigmoid'))
    ann.compile(optimizer='adam', loss='binary_crossentropy', metrics=['accuracy'])
    return ann
```

## Step 6: Ensemble classifier

![image](https://github.com/hughiephan/DPL/assets/16631121/2819e4a8-3ca7-4f5f-bbe6-420035a38099)

`(pred_ann > 0.5)*1` compares each element in the prediction array with 0.5. If the values are greater than 0.5 are classified as `1` (Survived), and values less than or equal to 0.5 are classified as `0` (Dead). If the combination of both `ann`, `rf`, and `svm` is larger than 2, it means that atleast 2 classfiers agree that the passenger survive, otherwise classify them as dead.

```python
class EnsembleClassifier:
    def __init__(self):
        self.ann = build_ann()
        self.rf = RandomForestClassifier()
        self.svm = SVC()
    def fit(self, X, y):
        self.ann.fit(X, y, epochs=2, verbose=0)
        self.rf.fit(X, y)
        self.svm.fit(X, y)
    def predict(self, X):
        predictions = list()
        pred_ann = self.ann.predict(X)
        pred_ann = (pred_ann > 0.5)*1
        pred_rf = self.rf.predict(X)
        pred_svm = self.svm.predict(X)
        for n in range(len(pred_ann)):
            combined = pred_ann[n] + pred_rf[n] + pred_svm[n] 
            if combined >= 2: 
                p = 1
            else:
                p = 0
            predictions.append(p)
        return predictions
```

## Step 6: Training

Train our EnsembleClassifier by calling `model.fit` on the train set and calling `model.predict` to get the prediction on the test set. Finally, compare the prediction of our classifier on the test set and the actual label `Y_test` to get the accuracy.

```python
model = EnsembleClassifier()
model.fit(X_train, Y_train)
prediction = model.predict(X_test)
score = metrics.accuracy_score(Y_test, prediction)
print("Accuracy:", score)
```

![image](https://github.com/hughiephan/DPL/assets/16631121/020abd22-fcf7-43cc-bc5f-02ed1d6015af)

## Reference
- https://pub.towardsai.net/introduction-to-ensemble-methods-226a5a421687
- https://www.numpyninja.com/post/interpolation-using-pandas
