```python

# 1. download the datasets from the coingeko
# 2. send mail
# 3. schedule task


Libs for email part of the task
import smtplib                                  #for Sending email
from email.mime.text import MIMEText            #text for email
from email.mime.multipart import MIMEMultipart  #multipart to combine subject and body
from email.mime.base import MIMEBase            #to enable attachments in email
import email.encoders                           #to ensure smooth flow of email

#Libs for data fetching and workaround
import requests
import schedule
from datetime import datetime
import time
import pandas as pd


#---------------------------------------------------------
# TASK 2 : # 2. send mail
#---------------------------------------------------------

#Setting up SMTP server for mail connection

def send_mail(subject, body,filename):
        smtp_server = "smtp.gmail.com"
        smtp_port = 587
        sender_email = "gaga8204@gmail.com"
        email_password = "qwyykgjptksbvydm"      #unique 16 digit generated password via settings on Gmail
        receiver_email = "waranganya@gmail.com"


        #Compose email
        message = MIMEMultipart()
        message['From'] = sender_email
        message['To'] = receiver_email
        message['Subject'] = subject

        #Attach body of mail
        message.attach(MIMEText(body,'Plain'))

        #Attach CSV file
        with open(filename, "rb") as attachment:
            part = MIMEBase("application", "octet-stream")
            part.set_payload(attachment.read())
            email.encoders.encode_base64(part)  
            part.add_header("Content-Disposition", f"attachment; filename={filename}")
            message.attach(part)


        #Start server

        try:
             with smtplib.SMTP(smtp_server,smtp_port) as server:
                  server.starttls()                     #Secure connection
                  server.login(sender_email,email_password)     #Login

                  #Sending email
                  server.sendmail(sender_email,receiver_email,message.as_string())

                  print("email sent successfully!")

        except Exception as e:
             print(f"Unable to send mail {e}")


#---------------------------------------------------------
# TASK 1 : # 1. download the datasets from the coingeko
#---------------------------------------------------------


#We convert this block of code into a function so we can use it whenever we want
# and perform the same activities again

def get_crypto_data():

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

        #Filtering columns for purpose
        df = df[['id','name','current_price','price_change_24h',
                'price_change_percentage_24h',
                'high_24h','low_24h','ath','atl']]
        
        #Adding an extra column - timestamp in the dataframe
        today = datetime.now().strftime("%d-%m-%y_%H-%M-%S")
        df['time_stamp'] = today
        print(df.shape)

        #Fetching and sorting top 10 values based on 24 hour price changes in different way

        #[negative = df.sort_values(by='price_change_24h',ascending=True)]
        negative_10 = df.nsmallest(10,'price_change_24h')
        print(negative_10)

        #[positive = df.sort_values(by='price_change_24h',ascending=False)]
        positive_10 = df.nlargest(10,'price_change_24h')
        print(positive_10)

        #Exporting these 10 values as a csv file

        #negative_10.to_csv(f"negative_10_{today}.csv",index=False)
        #positive_10.to_csv(f"positive_10_{today}.csv",index=False)


        #saving data
        file_name = f'crypto_data_{today}.csv'
        df.to_csv(file_name,index=False)

        print(f"Data saved successfully by {file_name}")

        #Calling email function to send reports

        subject = "Top_10_Cryptos for today"
        body = f"""

                Hello sir, \n
                Please find attached crypto reports for {today}.\n

                Crypto with highest price inc in last 24 hours. \n
                {positive_10}\n\n\n
                Crypto with highest decrease inc in last 24 hours.
                {negative_10}\n\n\n

                Attached top 250 crypto reports.

                Regards,\n
                Aniruddha
        """
        send_mail(subject,body,file_name)

    else:
        print(f"Connectiion has failed: {response.status_code}")

#This executes only if we run the function
if __name__ == "__main__":

    schedule.every().day.at('14:13').do(get_crypto_data)

    while True:
         schedule.run_pending()

```
