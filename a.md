```python
import matplotlib.pyplot as plt
import pandas as pd

df=pd.read_csv('가구주의_성__연령_및_세대구성별_가구_일반가구___시군구_20251126185337.csv',encoding='cp949')
df=df.set_index('행정구역별(시군구)')
df=df.iloc[1:,2:]

df1=df.iloc[:,::2].astype(int)
df2=df.iloc[:,1::2].astype(int)
df2.columns=df1.columns
df3=df2/df1*100
df3=df3.transpose()
df4=df3[['서울특별시','부산광역시','대구광역시','인천광역시']]


from sklearn.linear_model import LinearRegression
import numpy as np

df4.index = df4.index.astype(int)


# 예측 연도 범위

future_years = list(range(2025, 2035+1))
pred_dict = {region: [] for region in df4.columns}

#{'서울특별시': [], '부산광역시': [], '대구광역시': [], '인천광역시': []}

for region in df4.columns:
    y = df4[region].values# 각 지열별1인가구 비율
    X = df4.index.values.reshape(-1, 1)#2차원 년도

    model = LinearRegression()
    model.fit(X, y)

    preds = model.predict(np.array(future_years).reshape(-1, 1))
    pred_dict[region] = preds

df_future = pd.DataFrame(pred_dict, index=future_years)
# 실제+ 예측 합치기
df_all = pd.concat([df4, df_future])
print(df_all)



plt.rc('font',family='Malgun Gothic')

for column in df_all.columns:
    plt.plot(df_all.index,df_all[column],marker='o',label=column)

    for x,y in zip(df_all.index,df_all[column]):
        plt.text(x,y,f'{y:.2f}',ha='center',va='bottom', fontsize=9)

plt.title('행정구역별 연도별 데이터')
plt.xlabel('연도')
plt.ylabel('1인가구 비율')
plt.legend(loc='upper left')
plt.grid()
plt.show()

```
