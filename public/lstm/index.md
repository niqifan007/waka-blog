# 使用LSTM对航班延误进行预测

# 使用LSTM预测航班延误

## 介绍

航班延误是航空公司和乘客都不愿意面对的问题。在某些情况下，延误可能会导致航空公司的巨额损失，也会影响旅客的日常生活。因此，预测航班延误对于航空公司和旅客来说都非常重要。在本文中，我们将介绍如何使用LSTM来预测航班延误。

## 数据收集和预处理

要预测航班延误，我们需要收集航班数据。我们可以从航空公司、机场网站或第三方数据提供商那里获取数据。在本文中，我们使用开放数据集“航班延误和取消数据”来预测航班延误。在收集数据后，我们需要对数据进行预处理。这包括清洗数据、处理缺失值和异常值等。我们还需要将数据转换为适合LSTM模型的形式。

这个是本文的例子中所用的数据集的来源：

## LSTM模型构建和训练

LSTM是一种适合于序列数据的深度学习模型，常用于时间序列预测。我们可以使用LSTM来预测航班延误。在构建LSTM模型之前，我们需要将数据集分为训练集和测试集。训练集用于训练模型，测试集用于评估模型的性能。在训练LSTM模型时，我们需要确定一些参数，如LSTM层数、神经元个数、迭代次数等。我们还需要选择合适的损失函数和优化器。在训练完模型后，我们可以使用测试集来评估模型的性能。

1.导入包

```python
import pandas as pd
from tensorflow.keras import layers,models,optimizers
from keras.models import Sequential
from tensorflow.keras.layers import Dropout
from keras.layers import LSTM,Dense,Flatten
from tensorflow.keras.losses import MeanSquaredError
from sklearn.preprocessing import LabelEncoder
from sklearn.model_selection import train_test_split
from keras.callbacks import EarlyStopping
import numpy as np
import matplotlib.pyplot as plt
import tensorflow as tf
from os.path import exists
!nvidia-smi
import os
for dirname, _, filenames in os.walk('/kaggle/input'):
    for filename in filenames:
        print(os.path.join(dirname, filename))
os.environ['CUDA_VISIBLE_DEVICES'] = "0"
```

```
Thu Apr 13 16:28:46 2023
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 470.161.03   Driver Version: 470.161.03   CUDA Version: 11.4     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Tesla P100-PCIE...  Off  | 00000000:00:04.0 Off |                    0 |
| N/A   38C    P0    27W / 250W |      0MiB / 16280MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
/kaggle/input/us-flight-delay-from-january-2017-july-2022/Airline_Delay_Cause.csv
/kaggle/input/airlinedelay/Airline%20Delay.csv
/kaggle/input/airline-delay-cause/Airline_Delay_Cause.csv
```

2.导入数据集

```python
#导入数据集，分为大中小
# df=pd.read_csv('/kaggle/input/us-flight-delay-from-january-2017-july-2022/Airline_Delay_Cause.csv')#mid
# df=pd.read_csv('/kaggle/input/airline-delay-cause/Airline_Delay_Cause.csv')#big
df=pd.read_csv('/kaggle/input/airlinedelay/Airline%20Delay.csv')#small
df.dropna(inplace=True,axis=0)
```

3.进行简单的数据预处理，这里我使用了中位数进行对每个特征求出对应的值

```python
# 首先通过 df['arr_cancelled']==0 这个条件筛选掉了所有 arr_cancelled 列中值为 1 的行，也就是已经取消的航班。
df=df[df['arr_cancelled']==0]

# 然后将 carrier_ct、weather_ct、nas_ct、security_ct 和 late_aircraft_ct 这五列重新赋值，除以 arr_flights 列对应行的值，计算每种延误类型对总到达航班数的占比
arrivingFlights=df['arr_flights']
df['carrier_ct']=df['carrier_ct']/df['arr_flights']
df['weather_ct']=df['weather_ct']/df['arr_flights']
df['nas_ct']=df['nas_ct']/df['arr_flights']
df['security_ct']=df['security_ct']/df['arr_flights']
df['late_aircraft_ct']=df['late_aircraft_ct']/df['arr_flights']
# 接着，将 carrier_delay、weather_delay、security_delay 和 late_aircraft_delay 分别除以 arr_flights 对应行的值，计算每种延误类型对总延误时间的占比。
df['carrier_delay']=df['carrier_delay']/df['arr_flights']
df['weather_delay']=df['weather_delay']/df['arr_flights']
df['security_delay']=df['security_delay']/df['arr_flights']
df['late_aircraft_delay']=df['late_aircraft_delay']/df['arr_flights']
# 将 arr_delay 列除以 arr_flights 对应行的值，计算平均每个到达航班的延误时间。
df['arr_delay']=df['arr_delay']/df['arr_flights']

# 计算 arr_delay 列的标准差和均值，然后求出阈值（threshhold），这里将阈值设为平均值加两倍标准差。
std=df['arr_delay'].std()
mean=df['arr_delay'].mean()
threshhold=mean+2*std
filter=((df['arr_delay']<threshhold))

# 最后根据阈值和 arr_delay 列的值进行筛选，将 arr_delay 列的值小于阈值的行保留，其余行删除，即剔除过度偏离平均水平的异常值，返回一个新的数据帧 df。
df=df[filter]
```

```python
# 画图，延误的分钟，横坐标为时间序
df['carrier_delay'].plot(legend='Carrier')
df['weather_delay'].plot(legend='Weather')
df['security_delay'].plot(legend='Security')
df['late_aircraft_delay'].plot(legend='Late')
plt.title('delay in minutes')
plt.legend()
plt.show()
# 从这里我们可以看到，大多数的延误时间在10-15分钟，所以使用这个来判断航班是否延误
```

{{< image src="https://s1.imagehub.cc/images/2023/07/25/__results___3_0.png" width=900px loading=lazy >}}

```python
# 差不多也是中位数
df['carrier_ct'].plot()
df['nas_ct'].plot()
df['security_ct'].plot()
df['late_aircraft_ct'].plot()
plt.show()
```

{{< image src="https://s1.imagehub.cc/images/2023/07/25/__results___4_0.png" width=900px loading=lazy >}}

```python
# 航空公司计数
fig, ax = plt.subplots(figsize=(15,4))
carrier=df[df.groupby("carrier_name")['carrier_name'].transform('size') > 9]["carrier_name"]

carrier.value_counts().plot(kind="bar",ax=ax)
plt.title("Carrier Flight counts")
plt.show()
```

{{< image src="https://s1.imagehub.cc/images/2023/07/25/__results___5_0.png" width=900px loading=lazy >}}

```python
# 机场计数
fig, ax = plt.subplots(figsize=(15,4))
flights=df[df.groupby("airport")['airport'].transform('size') > 9]["airport"]

flights.value_counts().plot(kind="bar",ax=ax)
plt.title("Airport Flight counts")
plt.show()
```

{{< image src="https://s1.imagehub.cc/images/2023/07/25/__results___6_0.png" width=900px loading=lazy >}}

```python
encoder=LabelEncoder()
# 删掉没有用的列
df['carrier']=encoder.fit_transform(df['carrier'])
df['airport']=encoder.fit_transform(df['airport'])
if "airport_name" in df.columns:
    df.drop(["arr_del15","arr_cancelled","year", "airport_name","carrier_name","nas_delay",],axis=1,inplace=True)

# 对数据进行二分处理，若某条数据中的延误时间大于 10 分钟，这一列对应的值被标记为 1，否则标记为 0。这样处理之后，这三列数据变成了二分类变量。
def arr_delay_columns(row):
    if row['arr_delay'] > 10 :
        return 1
    else:
        return 0
    
df['label'] = df.apply(arr_delay_columns, axis=1)

def late_aircraft_delay_columns(row):
    if row['late_aircraft_delay'] > 10 :
        return 1
    else:
        return 0
    
def carrier_delay_columns(row):
    if row['carrier_delay'] > 10 :
        return 1
    else:
        return 0
    
# 画图 可以看到延误的航班大约占30%
print("delay mean:",df['arr_delay'].mean(),df['arr_delay'].max())
print("late_aircraft_delay mean:",df['late_aircraft_delay'].mean(),df['late_aircraft_delay'].max())
print("carrier_delay mean:",df['carrier_delay'].mean(),df['carrier_delay'].max())

df['late_aircraft_delay'] = df.apply(late_aircraft_delay_columns, axis=1)
df['carrier_delay'] = df.apply(carrier_delay_columns, axis=1)

counts = df["label"].value_counts()
total = df["label"].count()

percents = counts / total

plt.pie(percents, labels=percents.index, autopct='%1.1f%%')
plt.title('{} distribution'.format("label"))
plt.show()
delay mean: 8.184618800493773 28.467741935483872
late_aircraft_delay mean: 2.7714400135202206 24.285714285714285
carrier_delay mean: 3.2955479971538186 26.5
```

{{< image src="https://s1.imagehub.cc/images/2023/07/25/__results___7_1.png" width=900px loading=lazy >}}

```python
# 分割数据集，训练集和验证集
X_columns=[x for x in df.columns if not x in['label','arr_delay']]

X=df[X_columns]
y=df['label']

print(X_columns)

X_train, X_test, y_train, y_test = train_test_split( X, y, test_size=0.22, random_state=42)
['month', 'carrier', 'airport', 'arr_flights', 'carrier_ct', 'weather_ct', 'nas_ct', 'security_ct', 'late_aircraft_ct', 'arr_diverted', 'carrier_delay', 'weather_delay', 'security_delay', 'late_aircraft_delay']
```

```python
#建立LSTM模型
lookback=6
n_features=len(X_columns)

# inputs = layers.Input((lookback, n_features), dtype="float32")
# x = layers.LSTM(200, activation='tanh', return_sequences=True)(inputs)
# x = layers.BatchNormalization()(x)
# x = layers.Dropout(0.25)(x)
# x = layers.LSTM(200, activation='tanh', return_sequences=False)(x)
# x = layers.BatchNormalization()(x)
# x = layers.Dropout(0.25)(x)
# x = layers.Dense(100, activation='relu')(x)
# outputs = layers.Dense(1, activation='sigmoid')(x)

inputs = layers.Input((lookback, n_features),dtype="float32")
lstm1 = layers.LSTM(128, activation='tanh', return_sequences=True)
lstm2 = layers.LSTM(64, activation='tanh', return_sequences=False)
batch1 = layers.BatchNormalization()
dropout1 = layers.Dropout(0.2)
dense1 = layers.Dense(100)
dense2 = layers.Dense(50)
output = layers.Dense(1,activation="sigmoid")

out = lstm1(inputs)
out = lstm2(out)
out = batch1(out)
out = dense1(out)
out = dropout1(out)
out = dense2(out)
outputs = output(out)

#model = models.Model(inputs=inputs, outputs=outputs)
```

```python
model = models.Model(inputs, outputs)

optimizer = optimizers.Adam()
model.compile(
    optimizer=optimizer,
    loss='binary_crossentropy',
    #loss=MeanSquaredError(reduction="auto", name="mean_squared_error"),
    weighted_metrics=["acc"],
)

model.summary()

es = EarlyStopping(monitor="val_loss", mode="min", verbose=1, patience=20)

# 转换数据格式，输入数据 X 和标签 y 分别转换为 np.array 格式，并使用 .astype(np.float32) 将数据类型转为浮点数类型。
X2_train = np.asarray(X_train).astype(np.float32)
X2_train = np.resize(X2_train,(X2_train.shape[0],lookback,X2_train.shape[1]))
y2_train = np.asarray(y_train).astype(np.float32)

X2_test = np.asarray(X_test).astype(np.float32)
X2_test = np.resize(X2_test,(X2_test.shape[0],lookback,X2_test.shape[1]))
y2_test = np.asarray(y_test).astype(np.float32)

path_to_file="/kaggle/working/lstm3.h5"
file_exists = exists(path_to_file)
if(file_exists):
    model.load_weights(path_to_file)

# Fit data
history = model.fit(
    X2_train,
    y2_train,
    epochs=200,
    validation_data=(X2_test, y2_test),
    # verbose=0,
    shuffle=False,
    #callbacks=[es],
)

model.save_weights(path_to_file)
```

```python
# 训练loss
plt.plot(history.history["loss"],label="Train Loss")
plt.plot(history.history["val_loss"],label="Validation Loss")
plt.title("Loss vs Epochs")
plt.xlabel("Epochs")
plt.ylabel("Loss")
plt.legend()
plt.show()
```


{{< image src="https://s1.imagehub.cc/images/2023/07/25/__results___11_0.png" width=900px loading=lazy >}}

```python
y_pred=model.predict(X2_train)
y_pred=[float(x) if x>0.5 else 0 for x in y_pred]
```

43/43 [==============================] - 1s 3ms/step

```python
fig, ax = plt.subplots(figsize=(15,8))
legend_labels=["Prediction","Train"]
colors=["Blue","Red"]
units=np.arange(0,100)
ax.stackplot(units,y_pred[0:100],[1 if y>0.5 else 0 for y in y_train][0:100],labels=legend_labels,colors=colors)
print(np.max(y_train))
```

{{< image src="https://s1.imagehub.cc/images/2023/07/25/__results___13_1.png" width=900px loading=lazy >}}

```python
def accuracy(predictions, labels):
    correct=0
    for index,x in enumerate(predictions):
        if predictions[index] == labels[index]:
            correct+=1
    return (100.0 * correct
           / len(predictions))
print("accuracy: ",accuracy(y_pred[0:100],[1 if y>0.5 else 0 for y in y_train][0:100]))
```

accuracy: 90.0

## 模型预测和评估

在模型训练完成后，我们可以使用该模型来预测航班延误。我们可以将实时数据输入到模型中，模型将输出延误的概率。我们还可以使用各种指标来评估模型的性能，如平均绝对误差、均方误差等。如果模型的性能不够好，我们可以尝试调整模型参数、增加训练数据等方法来提高模型性能。

## 总结

本文介绍了如何使用LSTM来预测航班延误。我们需要收集和预处理数据，构建和训练LSTM模型，使用模型预测和评估模型性能。预测航班延误对于航空公司和旅客都非常重要，希望这篇文章能够帮助你了解如何使用LSTM来预测航班延误。
<!--more-->


---

> 作者: map[avatar:/images/avatar.jpg email:<nil> link:<nil> name:waka]  
> URL: http://blog.579878700.xyz/lstm/  

