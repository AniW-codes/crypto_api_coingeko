# crypto_api_coingeko

```python
#setup virtual environment and 
#Install lib in terminals - "pandas", "requests"


#Purpose - 
# 1. Fetch real time crypto prices at 8 am everyday
# 2. Save data as csv
# 3. Identify top 10 currencies and send mails

#import libraries

#Requests
import requests
#Pandas
import pandas as pd
#Datetime
from datetime import datetime

#API information
url = "https://api.coingecko.com/api/v3/coins/markets"
param = {
    'vs_currency' : 'usd',
    'order':'market_cap_desc',
    'per_page':250,
    'page':1
}

#Send connection request to website
response = requests.get(url,params=param)

#To check the response is alright or not
if response.status_code == 200:
    print(f"response code {response.status_code} is ok means we are able to connect to the website")
    print("Connection successfull, we can fetch data now")

    #Saving the response in a json file with parameter data
    data = response.json()

    #Forming a dataframe to store dict values found in json file as 
    #json file consists of items in dict format

    df = pd.DataFrame(data)

    print(df.columns)
    print(df.head())
    df = df[['id','name','current_price','market_cap','price_change_24h',
            'price_change_percentage_24h','ath','atl']]
    
    #Adding an extra column - timestamp in the dataframe
    today = datetime.now().strftime("%d-%m-%y")
    df['time_stamp'] = today
    print(df.shape)

    #Fetching and sorting top 10 values based on 24 hour price changes

    negative = df.sort_values(by='price_change_24h',ascending=True)
    negative_10 = negative.head(10)

    positive = df.sort_values(by='price_change_24h',ascending=False)
    positive_10 = positive.head(10)

    #Exporting these 10 values as a csv file

    print(negative_10)
    negative_10.to_csv(f"negative_10_{today}.csv",index=False)

    print(positive_10)
    positive_10.to_csv(f"positive_10_{today}.csv",index=False)


    #saving data
    df.to_csv(f"crypto_data_{today}.csv",index=False)
    print("Data saved successfully")

else:
    print(f"Connectiion has failed: {response.status_code}")

```
