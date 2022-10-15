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
2. ตัด N/A ใน column Product ออกไป เนื่องจากไม่สามารถใช้ได้
3. สร้าง df2 ขึ้นมาใหม่ เพื่อคัดเฉพาะ category ที่เป็น N/A
4. เราจะทำการเพิ่ม category ใน N/A เราจึง list ข้อมูล product ออกมาดู เพื่ออ้างอิงว่าเป็น category อะไร
**(ในตอนแรกตั้งใจจะใช้ fillna แต่เนื่องจากข้อมูลใน แต่ละ row ต้องอ้างอิง column อื่น และต้องมีเงื่อนไขด้วย จึงเปลี่ยนมาสร้าง def เพื่อเขียนเงื่อนไขแทน)**
```
array(['Doritos Dinamita Chile Lemon', 'Doritos Spicy Nacho',
       'Mini Chips Ahoy - Go Paks', 'Oreo Mini - Go Paks',
       'Teddy Grahams - Go Paks', 'Starbucks Doubleshot Energy - Coffee',
       'Canada Dry - Ginger Ale & Lemonde', 'Canada Dry - Ginger Ale'],
      dtype=object)
```
5. สร้าง def ขึ้นมาเพื่อใส่ค่า category โดยกำหนดเงื่อนไขดังด้านล่างนี้ใน df2
```
def assign_Result(product):
    if product == 'Canada Dry - Ginger Ale & Lemonde' or product == 'Canada Dry - Ginger Ale':
        result = 'Carbonated'
    elif product == 'Starbucks Doubleshot Energy - Coffee':
        result = 'Non Carbonated'
    else:
        result = 'Food'

    return result
df2['Category'] = df2['Product'].apply(assign_Result)
```
6. ทำการ drop row ที่ category เป็น N/A ทิ้ง บน df เพื่อเตรียม concatenate กับ df2 ที่สร้างใหม่
7. Concatenate ระหว่าง df และ df2 จะได้เป็น df_vending
```
df_vending = pd.concat( [df, df2], ignore_index=True  )
df_vending
```
8. เนื่องจาก TransDate เป็น object จึงทำการเปลี่ยน Dtype ให้เป็น datetime เพื่องานต่อการจัดการข้อมูล
```
df_vending['TransDate'] = pd.to_datetime( df_vending['TransDate'] )
df_vending.info()
df_vending
```

9. ทำการเพิ่ม column Day เพื่อระบุชื่อวันในแต่ละ row
**(ในตอนแรกจะใช้ regex เพื่อ replace ค่าวัน แต่เนื่องจากเรายังต้องการใช้ข้อมูลที่เป็นวันที่ในการวิเคราะห์ จึงใช้วิธีการนี้แทน  และเขียนโค้ดสั้นกว่าด้วย เพราะหาก replace จะต้องทำ 7 รอบ แทนชื่อวันทุกวัน)**
```
df_vending['Day'] = df_vending['TransDate'].dt.day_name()
df_vending
```
# 4. Descriptive Statistics
1. ดูข้อมูลทางสถิติพื้นฐาน เช่น mean max min ของภาพรวม
2. ดูข้อมูลแยกตาม Attribute ต่าง ๆ (ดูเพิ่มเติมได้ในโค้ดค่ะ)

# 5. Visualization
นำข้อมูลที่ได้มาหาคำตอบที่เราตั้งไว้
1. สถานที่ตั้ง Vending machine ที่ไหนขายดีที่สุด 

![image](https://user-images.githubusercontent.com/115805661/195896616-91b400b1-b6b8-497b-aa1b-91a9fc136111.png)

- จากภาพจะเห็นได้ว่า Histogram นี้ได้ show ข้อมูลจำนวนที่ขายได้ของ vending machine แต่ละสถานที่ พบว่าสถานที่ที่มีลูกค้าเยอะสุดคือ 1. GuttenPlans 2. EB Public Library 3. Brunswick Sq Mall 4. Earle Asphalt

**ข้อเสนอแนะ : อาจเสนอให้ลงทุนตู้กดเพิ่มในสถานที่ GuttenPlans**

2. สินค้าที่ขายดีในแต่ละสถานที่ตั้งเครื่อง Vending Machine มีความแตกต่างกันหรือไม่

![image](https://user-images.githubusercontent.com/115805661/195898011-cebb5764-9d84-4ead-ad77-51c8a388c59e.png)

จากภาพจะเห็นได้ว่า Heatmap ที่ได้มา จะเป็นข้อมูลของสินค้า 3 อันดับแรกในแต่ละสถานที่ ซึ่งพบว่า
- ที่ GuttenPlans มีการซื้อ สินค้าประเภทเอเนอจี้ดริ้งเยอะกว่าที่อื่นอย่างเห็นได้ชัด
- ที่ Earle Asphalt 3 อันดับแรกจะเป็น Food ทั้งหมด
- EB Public Library ซื้อ Coca Cola - Zero Sugar มากที่สุด รองลงมาเป็น Poland Springs Water และไม่มีอาหารอยู่ใน 3 อันดับแรก
- Brunswick Sq Mall ซื้อ Poland Springs Water มากที่สุด และไม่มีอาหารอยู่ใน 3 อันดับแรก

**ข้อเสนอแนะ : GuttenPlans ควรมี stocks Energy drinks เยอะ เนื่องจากเป็นสถานที่โรงงานทำ Frozen dough ลูกค้าจึงเป็นคนจากโรงงาน ต้องการความกระปี้ประเป่าเยอะกว่าที่อื่น**

3. ประเภทการชำระเงินแบบไหนที่ผู้คนนิยม

![image](https://user-images.githubusercontent.com/115805661/195979863-3677ec3d-07b2-4cc8-a769-01b9378b8675.png)


- พบว่าภาพรวม ผู้คนนิยมใช้จ่ายโดย Cash มากกว่าใช้ Credit
- พบว่าที่ GuttenPlans มีการชำระโดยวิธี Cash เยออะกว่า Credit อย่างเห็นได้ชัด

**ข้อเสนอแนะ : อาจต้องมีการเพิ่มเงินทอนในตู้ที่ตั้งที่ GuttenPlans เพื่อรองรับการจ่ายเงินสด**

### นอกจากคำถามแล้ว ยังมีข้อมูลที่สามารถหา Insight เพิ่มได้ ดังนี้
1. กราฟเส้นเแสดงข้อมูลตั้งแต่อดีตจนถึงปัจจุบันของแต่ละสถานที่ว่ามีรายได้เป็นอย่างไร
![image](https://user-images.githubusercontent.com/115805661/195899883-d6507437-b1bd-4cc3-ad3f-201aab11ef06.png)
- ข้อมูลดูยากเนื่องจากข้อมูลในแต่ละวันละเอียด และรายได้ในแต่ละวันค่อนข้างเหวี่ยง แต่เมื่อพิจารณาแล้วพบว่า ข้อมูลของ EB Public Library และ GuttenPlans มีความใกล้เคียงกัน แต่ EB Public Library เริ่มตั้งขายทีหลัง จึงทำให้ภาพรวมของ EB Public Library ต่ำกว่า 

**ข้อเสนอแนะ : อาจจะปรับเปลี่ยนการวิเคราะห์ โดยเริ่มจากวันแรกที่ EB Public Library เริ่มเปิดใช้งาน เพื่อความแม่นยำยิ่งขึ้นของข้อมูล**

2. สร้างกราฟ Location กับ category เพื่อดูความสัมพันธ์โดยดูจากรายได้ เพื่อดูว่าในแต่ละสถานที่ตั้งตู้กด กับหมวดหมู่สินค้า ลูกค้าเลือกซื้อสินค้าแตกต่างกันหรือไม่

![image](https://user-images.githubusercontent.com/115805661/195900127-d1d5a056-afc9-4c78-a2ec-2ef2a9815982.png)
- ที่ GuttenPlans รายได้จากลูกค้าที่ซื้อ carbonated และ Food พอ ๆ กัน
- ที่ Earle Asphalt และ EB Public Library รายได้จากลูกค้าที่ซื้อ Food มากที่สุดอย่างเห็นได้ชัด
- ที่ Brunswick Sq Mall	รายได้จากลูกค้าที่ซื้อสินค้าแต่ละอย่างพอ ๆ กัน

**ข้อเสนอแนะ : อาจจะปรับเปลี่ยนการวิเคราะห์ โดยเริ่มจากวันแรกที่ EB Public Library เริ่มเปิดใช้งาน เพื่อความแม่นยำยิ่งขึ้นของข้อมูล**

3. สร้างกราฟเพื่อดูข้อมูลระหว่าง วันต่าง ๆ และรายได้ โดยเส้นจะแบ่งตาม Location ของ vending machine
- เพื่อให้ชัดเจนขึ้น จึงนำมาทำ heatmap เพื่อดูข้อมูลที่ cross ระหว่าง Day และ Location
- สร้าง dataframe อันใหม่ โดย pivot ข้อมูลระหว่าง Day และ Location แลให้ข้อมูลในตารางเป็น LineTotal($)
- ในขั้นตอนนี้ ตอนแรกชื่อวันจะเรียงตามตัวอักษร ทำให้ดูยาก จึงมีการ sort Day ใหม่ให้เรียงเป็นวันอาทิตย์-วันเสาร์
```
df_vending['Day'] = pd.Categorical(df_vending['Day'], ["Sunday", "Monday", "Tuesday","Wednesday","Thursday","Friday","Saturday"])
piv = df_vending.pivot_table(index="Location", columns="Day", values="LineTotal",aggfunc='sum')
display(piv)

sns.set(rc={'figure.figsize':(12,6)})
ax = sns.heatmap(piv,annot=True,fmt='.1f', cmap="YlGnBu")
```
![image](https://user-images.githubusercontent.com/115805661/195906058-e6725475-113d-41bf-b458-ef07ffd698c7.png)
- พบว่าที่ GuttenPlans มีรายได้สูงสุดระหว่างวันจันทร์-วันศุกร์
- Earle Asphalt มีรายได้ต่ำสุดเมื่อเทียบกับที่อื่น ๆ

**ข้อเสนอแนะ : เมื่อต้องการเติม stock สินค้าหรือเงินทอน ควรทำวันเสาร์อาทิตย์**

# 6. Challenge
1.	การหา Data ที่ต้องการนำมาวิเคราะห์ ต้องเป็น Data ที่ดูน่าสนใจ และไม่มีคนขุด insight เยอะแล้ว เพื่อให้เราได้เป็นผู้ค้นหาเอง ซึ่งมีไม่ค่อยมาก ทำให้ใช้เวลากับค่อนข้างนานในการหา
2.	การ Clean Data ข้อมูลที่ได้ต้องมีการจัดเตรียม Data ให้พร้อมต่อการวิเคราะห์ และ Visualize ทำให้ใช้เวลากับค่อนข้างนานในการ Clean เพื่อให้ข้อมูลเตรียมพร้อม
3.	การเขียน Code มีการ Error บ่อย จึงต้องค้นหาว่า Error เพราะอะไร โดยค้นหาจาก google
4.	การเขียน Code บางวิธีไม่เหมาะกับข้อมูลเรา จึงแก้ปัญหาโดยเปลี่ยนวิธีการเขียน Code หรือ ปรับแต่ง Data ให้เหมาะกับ Code 
5.	ยังต้องฝึกการวิเคราะห์ข้อมูลต่อไป เนื่องจากยังไม่ชินกับข้อมูลว่าต้องใช้กับกราฟประเภทไหน
