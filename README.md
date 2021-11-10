# BinanceAPI
This is a Python 3.7 Jupyter Notebook demonstrating direct use of the Binance API to Get Market Data.

## Background – 
Crypto blockchains enable highly distributed financial transactions which hold the promise of economic freedom, and the allure of easy access to many exotic trading strategies, even at low-dollar thresholds, with demonstrated gainfulness.

Binance holds the top spot among cryptocurrency exchanges for volume of trades.  Crypto exchanges pair buyers who bid on crypto with sellers who ask for a particular price.  The exchange enables the direct blockchain transaction between the buyer and seller.  This is different from a brokerage, which transacts as a liaison between the buyers and sellers.

Binance has a nice dashboard for analytics and an interface for transactions but accessing the exchange via an API has significant benefits.  With the API, traders can fully customize their inquiries to gain focused insights and automate their transactions.  

Two popular Python libraries are available to abstract the API interactions, but this wiki will show you how to access the Binance API from scratch.  Read on to get started on your journey to make money with data.

## HTTP methods:
This tutorial is focused on getting Market data from Binance using the Python requests library and ‘GET’ method.  With the Binance API, users can also POST, PUT, and DELETE.  The POST method allows users to make trades that buy and sell cryptocurrency, including with margin transactions.  The PUT method is used to extend streaming data timeframes, which timeout after 30 minutes, and the DELETE method will close the streaming data stream. https://binance-docs.github.io/apidocs/ 

## About the Endpoints:

There are more API endpoints on Binance than I’d like to count, organized into categories and sub-categories.  The four categories are Options, Coin futures, US Dollar futures, and Spot/Margin.  This wiki is focused on Spot/Margin, which is direct trades. There are 15 subcategories of endpoints within Spot/Margin, including many that tap into user-specific data, such as the user’s wallet and transactions with different currencies.  This wiki will look at a few of the endpoints under the Market Data subcategory.  There are 12 endpoints within Market Data.

The Market Data endpoints show current and historical trade and order data.  This section allows traders to do broad market research and determine which assets they may want to buy and sell soon!

Binance RESTful API endpoints provide JSON objects or arrays.  Access is rate limited by Binance at 20 requests per second. This wiki won’t get anywhere near that speed because we’ll be getting the data with human fingers.  However, if you want streaming data or are concerned you may hit these limits using a bot, it’s recommended to use the websockets protocol, which comply with the limitations.

The APIs are available with three different security protocols: None, API-Key only, and API-Key + Signature.  https://binance-docs.github.io/apidocs/spot/en/#endpoint-security-type 

Very basic endpoints, such as the ‘ping’ to verify a connection and the ‘server time’ require no key at all.  Market data requires a valid API-Key, so you must have a Binance account to access these APIs.  In addition to the API-Key, Trade, Margin and UserData endpoints a signature.  The one-time-use signature is created for each data request.  Creating a signature requires the secret key proving that the sender was in possession of the secret key at the moment the transaction occurred.  The signatures are encoded with HMAC SHA256, providing the key to decode the data.

## Authentication:

To generate your API-Key, you’ll need a Binance account.  If you’re in the USA, you can only sign up for Binance at Binance.US https://www.binance.us/en/home.  Creating an account requires a significant security procedure, including uploading your identification and recording facial-recognition biometric data.  This is part of a process called know-your-customer (KYC), which most exchanges are performing to meet or prepare for regulatory requirements.

Once you have an account, you can easily set up your API-Key and Secret-Key from your dashboard.  In the center of the screen is and ‘API – Manage’ block.  Simply name the API and click ‘Create.’  Save the Secret-Key somewhere safe because it will only be displayed one time.  

Each API can be configured to enable read, trade, and withdrawal transactions or limit the IP addresses that can use the API or greater security. 

#### Responses:

The GET requests we are going to access will return JSON formatted data.  Dictionaries store each key-value pair, and the values often return as lists of lists.  

There are no ‘headers’ included in the response dataset, so Binance provides a dictionary to interpret the lists where needed.  These can be organized into Pandas dataframes by looping over the dictionary and joining them into the dataframe with the concat method.  
Tutorial:

#### Once you have your API-Key, and Secret-Key, store these as variables in a separate file (SecretFile.py) to protect them from being visible in the analysis file.

Begin the analysis by importing any modules needed.  At a minimum, import your SecretFile to grab your keys, the requests module to access the API, and pandas to organize the data.

import requests

import SecretFile

import pandas as pd

Connect to the API by adding the endpoint_url to the base_url.  The base url is https://api.binance.com, and there are three others if this gets jammed up: https://api1.binance.com, https://api2.binance.com, and https://api3.binance.com. The first endpoint to test is the server time, which doesn’t require any headers or parameters.  This endpoint url is /api/v3/time.

base_url = "https://api.binance.com"

url_end =  "/api/v3/time"

url = base_url + url_end

Then, set up the headers.  Headers must include the accepted format as JSON and the accepted agents as the browsers that will be used.
headers = {'Accept': 'application/json', 'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36',}

Request using the get method, url, and headers, and view the response in JSON format.
response = requests.request("GET", url, headers=headers)
response.json()

This will return a dictionary with the key ‘serverTime’ and the POSIX timestamp.  This can be decoded into UTC with the datetime class and method (though it appears as Eastern Standard Time in my analysis, so this may be a custom default based on my location).

If all of this worked well, move on to Market data, which requires the API-Key to authenticate the request.  The endpoint url for orders data is /api/v3/depth.  Add this key-value pair to the headers dictionary: headers['X-MBX-APIKEY'] = SecretFile.clientcode (clientcode is the API-Key variable name stored in SecretFile.py).  

Then, add the required ‘symbol’ parameter with the TradeCoin/QuoteCoin pair you want to evaluate.  The example is Bitcoin in USD Tether coin; USDT is a stable-coin that tracks 1:1 with the US Dollar.

The following code will return the JSON-formatted Depth data for Bitcoin in USD Tether:

base_url = "https://api.binance.com"

url_end = "/api/v3/depth" #### Doesn't work, need symbol

url = base_url + url_end

headers = {
            'Accept': 'application/json',
            'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36', 
        } # ref def _get_headers https://github.com/sammchardy/python-binance/blob/master/binance/client.py

#### Set API-Key
headers['X-MBX-APIKEY'] = BinanceK.clientcode # ref Endpoint Security Type https://binance-docs.github.io/apidocs/spot/en/#endpoint-security-type

#### Set Parameters
parameters = {'symbol':'BTCUSDT'}

response = requests.request("GET", url, headers=headers, params=parameters)

print(response.status_code) # Check the status of the endpoint

Depth = response.json()

Depth

This data is most useful when displayed in a Pandas dataframe.  Thanks to Eryk Lewinson who provided the following code in his blog on depth charts referenced below.

df_list = []

for side in ["bids", "asks"]:
    df = pd.DataFrame(depth[side], columns=["price", "quantity"], dtype = float)
    df["side"] = side
    df_list.append(df)

depth = pd.concat(df_list).reset_index(drop=True)

Now, you can evaluate the most recent bids and asks for Bitcoin!

## More info:
Now, check out my YouTube video demonstrating the above Binance API tutorial, and trying out some useful analytics you can do in a Jupyter Notebook.  The video is here: xxxx

The Jupyter Notebook from that video is available in my GitHub repository here: https://github.com/aprilcrompy/BinanceAPI

Be sure to visit these blogs, by the original authors of the analytics you saw in the video:
•	Depth Chart: https://towardsdatascience.com/learn-what-a-depth-chart-is-and-how-to-create-it-in-python-323d065e6f86 
•	Simple Moving Average Chart: https://towardsdatascience.com/making-a-trade-call-using-simple-moving-average-sma-crossover-strategy-python-implementation-29963326da7a  

Dig deeper into the Binance API with the official documentation, and the unofficial and official Python libraries:
•	Binance API Documentation: https://binance-docs.github.io/apidocs/ 
•	Binance-Connector formal python API library: https://github.com/binance/binance-connector-python 
•	Python-Binance informal python API library by Sam McHardy: https://github.com/sammchardy/python-binance/blob/master/binance/client.py 
•	More Advanced interactions using python-binance library: https://algotrading101.com/learn/binance-python-api-guide/ 

Good reads about Cryptocurency:
•	Cryptocurrency exchanges by volume of trades: https://www.statista.com/statistics/864738/leading-cryptocurrency-exchanges-traders/ 
•	Why crypto enables economic freedom: https://blog.coinbase.com/how-crypto-enables-economic-freedom-7dee35d8f0e0 
•	How to make money with crypto: https://www.stilt.com/blog/2021/06/how-to-make-money-with-cryptocurrency/#6_Strategies_for_Making_Money_with_Crypto 
