# Data Journey - Vending machine analysis

# 1. Data
ค้นหา Data จาก Kaggle https://www.kaggle.com/datasets/awesomeasingh/vending-machine-sales 
รายละเอียดของข้อมูล ประกอบไปด้วย

**The location and machine data is as follows**


>(1) Gutten Plans - Frozen dough specialist company that operates 24/5 . Vending machine assigned is GuttenPlans x1367

>(2) EB Public Library - Public library that has high foot traffic 5-6 days a week. Vending machine : EB Public Library x1380

>(3) Brunswick Sq Mall - Mall with average foot traffic 7 days a week. Vending machine(s) : BSQ Mall x1364 - Zales & BSQ Mall x1366 - ATT

>(4) Earle Asphalt - A construction engineering firm that operates 5 days a week. Vending machine : Earle Asphalt x1371



**This file has the following attributes**

>Status : Represents if the machine data is successfully processed

>Device ID : Unique electronic identifier ( also called as ePort) for the vending machine. A machine is allocated a unique ePrt * device

>Location : Indicates location of the vending machine

>Machine : User-friendly machine name

>Product : Product vended from the machine

>Category : Carbonated / Food / Non-carbonated / Water

>Transaction : Unique identifier for every transaction

>TransDate : The Date & time of transaction

>Type : Type of transaction ( Cash / Credit )

>RCoil : Coil # used to vend the product

>RPrice : Price of the Product

>RQty : Quantity sold. This is usually one but machines can be configured to sell more items in a single transaction

>MCoil : Mapped coil # used to vend the product ( from toucan )

>MPrice : Mapped price of the Product

>MQty : Mapped quantity sold. This is usually one but machines can be configured to sell more items in a single transaction

>LineTotal : Total sale per transaction

>TransTotal : Represents total of all transactions that will show up on the Credit Card. A user could vend a drink for 3 and a snack for 1.5 making a total of 4.50

>Prcd Date : Date when the transaction was processed by SeedLive ( an entity that is used to aggregate all transactions electronically )

ซึ่งในขั้นตอนการหาข้อมูลนั้น ต้องหาข้อมูลที่สามารถนำไปใช้ประโยชน์ได้ และยังไม่มีคนหา insight ข้อมูลมากนัก อีกทั้งต้องเป็นข้อมูลที่เราสนใจ จึงใช้เวลาในขั้นตอนนี้ค่อนข้างนานเพื่อให้ได้ Data ที่เราต้องการอย่างแท้จริง

# 2. Import Library & Import Data
```
import sys
import pandas as pd
import numpy as np
import re # ในตอนแรกจะใช้ในการ replace ค่าวันที่ เป็นชื่อวัน แต่พบว่ามีวิธีที่ง่ายกว่าจึงไม่ได้ใช้แล้ว
import matplotlib.pyplot as plt 
import seaborn as sns
df = pd.read_csv('vending_machine_sales.csv') 
df.info()
```
 1. ตรวจสอบข้อมูลที่ได้มาว่า column ไหนสามารถใช้ได้บ้าง แล้วเป็น type อะไรบ้าง
 2. import file ซึ่งนำมาจาก kaggle

```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 6445 entries, 0 to 6444
Data columns (total 18 columns):
 #   Column       Non-Null Count  Dtype  
---  ------       --------------  -----  
 0   Status       6445 non-null   object 
 1   Device ID    6445 non-null   object 
 2   Location     6445 non-null   object 
 3   Machine      6445 non-null   object 
 4   Product      6441 non-null   object 
 5   Category     6260 non-null   object 
 6   Transaction  6445 non-null   int64  
 7   TransDate    6445 non-null   object 
 8   Type         6445 non-null   object 
 9   RCoil        6445 non-null   int64  
 10  RPrice       6445 non-null   float64
 11  RQty         6445 non-null   int64  
 12  MCoil        6445 non-null   int64  
 13  MPrice       6444 non-null   float64
 14  MQty         6445 non-null   int64  
 15  LineTotal    6445 non-null   float64
 16  TransTotal   6445 non-null   float64
 17  Prcd Date    6445 non-null   object 
dtypes: float64(4), int64(5), object(9)
memory usage: 906.5+ KB
```
3. ตรวจสอบข้อมูลที่ได้มาว่า column ไหนสามารถใช้ได้บ้าง แล้วเป็น type อะไรบ้าง มี column ใดบ้างที่เป็น N/A

```display(df)```

4. เลือกเฉพาะ column ที่ต้องการ

```df = df.loc[ : ,['Location','Machine','Product','Category','TransDate','Type','MPrice','MQty','LineTotal'] ]```

# 3. Cleansing Data
1. จากขั้นตอนการเช็คข้อมูล เราจะพบว่า column Product มี N/A จึงได้ทำการเลือกมาดู
```df[df[ 'Product'].isna()][['Product'] ]```
2. ตัด N/A ใน column Product ออกไป เนื่องจากไม่สามารถใช้ได้
```
df = df.dropna(subset=['Product'])
print(df.shape)
```


