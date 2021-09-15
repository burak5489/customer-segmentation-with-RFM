# customer-segmentation-with-RFM

pip install lifetimes
pip install sqlalchemy
from sqlalchemy import create_engine
import datetime as dt
import pandas as pd
import matplotlib.pyplot as plt
from lifetimes import BetaGeoFitter
from lifetimes import GammaGammaFitter
from lifetimes.plotting import plot_period_transactions

pd.set_option('display.max_columns', None)
pd.set_option('display.width', 500)
pd.set_option('display.float_format', lambda x: '%.4f' % x)
from sklearn.preprocessing import MinMaxScaler


########### Veriyi alma ve kopyalama (get data from excel) ###########
df_ = pd.read_excel("online_retail_II.xlsx",sheet_name="Year 2010-2011")
df = df_.copy()

df.shape
df.head()
df.info()
df.isnull().any()
df.isnull().sum()
df.dropna(inplace=True)
df["StockCode"].nunique()
df["StockCode"].value_counts()
df["StockCode"].value_counts().sort_values(ascending=False).head()
df = df[~df["Invoice"].str.contains("C",na=False)]
df["TotalPrice"] = df["Quantity"]*df["Price"]



today_date = dt.datetime(2012,12,11)

rfm = df.groupby("Customer ID").agg({"InvoiceDate": lambda x: (today_date-x.max()).days,
                                     "Invoice": lambda x: x.nunique(),
                                     "TotalPrice": lambda x: x.sum()})

rfm.columns = ["recency","frequency","monetary"]
rfm = rfm[rfm["monetary"]>0]
rfm.head()
rfm.describe().T



rfm["recency_score"] = pd.qcut(rfm["recency"],5,labels=[5,4,3,2,1])
rfm["frequency_score"] = pd.qcut(rfm["frequency"].rank(method="first"),5,labels=[1,2,3,4,5])
rfm["monetary_score"] = pd.qcut(rfm["monetary"],5,labels=[1,2,3,4,5])

rfm["rfm_score"] = rfm["recency_score"].astype(str) + rfm["frequency_score"].astype(str)


seg_map = {
        r'[1-2][1-2]': 'hibernating',
        r'[1-2][3-4]': 'at_risk',
        r'[1-2]5': 'cant_loose',
        r'3[1-2]': 'about_to_sleep',
        r'33': 'need_attention',
        r'[3-4][4-5]': 'loyal_customers',
        r'41': 'promising',
        r'51': 'new_customers',
        r'[4-5][2-3]': 'potential_loyalists',
        r'5[4-5]': 'champions'
    }

rfm["segment"] = rfm["rfm_score"].replace(seg_map,regex=True)

rfm_final = rfm[["recency","frequency","monetary","segment"]]
