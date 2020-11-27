CFDs are complex instruments and come with a high risk of losing money rapidly due to leverage. 

**75.38% of retail investor accounts lose money when trading CFDs with this provider because more retail investors are fucking idiots.**

You should consider whether you understand how CFDs work and whether you can afford to take the high risk of losing your money.

Change 1: This line is change 1. 

Change 2: This line is change 2. 



# RestAPI

Our REST API is a web-based API using a Websocket connection and was developed with algorithmic trading in mind. 

Developers and investors can create custom trading applications, integrate into our platform, back test strategies and build robot trading. Calls can be made in any language that supports a standard HTTP. 

We utilize the new OAuth 2.0 specification for authentication via token. This allows for a more secure authorization to access your application and can easily be integrated with web applications, mobile devices, and desktop platforms

Change 3: This line is change 3.


With the use of the socket.io library, the API has streaming capability and will push data notifications in a JSON format. Your application will have access to our real-time streaming market data, subscribe in real time access to trading tables and place live trades. You can also retrieve market data price history. You can find instructions on documentation and sample code.

## FXCM Trading hours:
FXCM's trading hours vary by product. For forex, trading opens on Sundays between 5:00 PM ET and 5:15 PM ET and closes on Fridays around 4:55 PM ET. for CFD, please check [CFD Product Guide.](http://docs.fxcorporate.com/user-guide/ug-cfd-product-guide-ltd-en.pdf)

## How to start:
1.	Python wrapper fxcmpy at [here](https://www.fxcm.com/fxcmpy/) 
2.	Sample code in Node.js at [here](https://github.com/fxcm/RestAPI/tree/master/fxcm-api-rest-nodejs-example) 
3.	Quick start guide in Java at [here](https://github.com/fxcm/RestAPI/blob/master/FXCM%20JAVA%20REST%20API%20QuickStart.pdf)
4.  Java sample code at [here](https://apiwiki.fxcorporate.com/api/RestAPI/JavaRestClient.zip)
5.  C# sample code at [here](https://apiwiki.fxcorporate.com/api/RestAPI/RestAPI_CSharp_sample_code.zip)
6.	Apply for a [demo account](https://www.fxcm.com/uk/forex-trading-demo/)  Generate access token. You can generate one from the <a href="https://tradingstation.fxcm.com/">Trading Station web</a>. Click on User Account > Token Management on the upper right hand of the website. For Live account, please send your username to api@fxcm.com, we will need to enable Rest API access. For demo account, Rest API access was enabled by default.
7.	Download Rest API pdf documents at <a href="https://apiwiki.fxcorporate.com/api/RestAPI/Socket%20REST%20API%20Specs.pdf">here</a>
8. Start coding.  You will need to reference the <a href="https://socket.io/docs/client-api/">socket.io library</a> in your code. 
   a.	Using Javascript, click <a href="https://www.npmjs.com/package/socket.io">here</a>
   b.	 Using Python, click <a href="https://github.com/miguelgrinberg/python-socketio">here</a>


## How to connect:
Clients should establish a persistent WebSocket connection using socket.io library. All non-solicited updates will be sent over this connection. Client requests are to be sent via normal HTTP messages. Every HTTP message must contain following parameters

| Header | Description | Values | Req’d |
| --- | --- |--- |--- |
| HTTP version | Version of HTTP used | HTTP/1.1 | Y |
| User-Agent | Identification of the client software | xxxxxx_software | Y |
| Accept | Acceptable response MIME type | application/json | Y |
| Content-Type | Media type of the request | application/x-www-form-urlencoded | Y |
| Authorization | Authorization string containing “Bearer “, ID of socket.io connection and persistent token| 'Bearer ' + socket_id + api_token | Y |

      Sample Request
      GET /socket.io/?access_token=cj5wedhq3007v61fe935ihqed&EIO=3&transport=polling&t=Lsd_lZY&b64=1 
      HTTP/1.1 
      User-Agent: node-XMLHttpRequest 
      Accept: */* 
      Host: api.fxcm.com 
      Connection: close

## What 't' means
"t" is the table id: 

| t | table ID |
| --- | --- |
| 1 | Open Position |
| 2 | Closed Position |
| 3 | Order |
| 5 | Summary |
| 6 | Account |

## Subscribe vs snapshot:
FXCM Rest API provides two ways to deliever data. susbcribe vs snapshot.

After susbcribe, data will be pushed to your socket whenever there is an update. You can susbcribe Market data stream /susbcribe or live table update /trading/susbcribe. You can also unsubscribe.
You can request a snapshot of trading tables via /trading/get_model. 

      Model choices: 'Offer', 'OpenPosition', 'ClosedPosition', 'Order', 'Summary', 'LeverageProfile', 'Account', 'Properties'.   

## How to place market order:
This function is very similar to our market order in TSweb2.0. Since it is a market order, order will be filled at market by system. Client don’t need to specify the price,  you can set rate=0. Order_type=AtMarket tells the server it is a market order, not MarketRange order. Stop and limit are the price you need to specify when you want to exit the market in stoploss/take profit price. Is_in_pips=true means stop/limit is in pips, otherwise it is in real value. Stop should be negative value. 

      Placing Market order:
      POST /trading/open_trade
      account_id=1537581&symbol=EUR%2FUSD&is_buy=false&rate=0&amount=5&order_type=AtMarket&time_in_force=GTC

## OrderID vs TradeID:
OrderID and TradeID are different.
In Market order, an order id is created straightaway and it is in callback immediately. 

      {"response":{"executed":true},"data":{"type":0,"orderId":81712802}}

A trade id is not generated until after order is executed. You have to subscribe the order table and listing the live update and look up the trade id. You will not get trade id in snapshot, because that information was gone when you submit the request. 
to link the orderID to TradeID, you need to subscribe order table. Open position table, you will get order id immediately from response. You will see insert in order table, follow by update and delete on order table. In Open position, at the same time you will see same order ID + trade ID been insert in open position table then followed by update. Now you know what order combined to what trade ID


      Examples:
      Subscribing for Orders table:
      POST /trading/subscribe
      models=Order
      
      Placing Market order:
      POST /trading/open_trade
      account_id=1537581&symbol=EUR%2FUSD&is_buy=false&rate=0&amount=5&order_type=AtMarket&time_in_force=GTC

      Response from server:
      {"executed":true}{"type":0,"orderId":390285837}

      Received Order record from /trading/subscribe with order_id and trade_id:
      {"t":3,"ratePrecision":5,"orderId":"390285837","tradeId":"170162801","time":"04252018120716391","accountName":"01537581","accountId":"1537581","timeInForce":"GTC","expireDate":"","currency":"EUR/USD","isBuy":false,"buy":0,"sell":1.21818,"type":"OM","status":2,"amountK":5,"currencyPoint":0.5,"stopMove":0,"stop":0,"stopRate":0,"limit":0,"limitRate":0,"isEntryOrder":false,"ocoBulkId":0,"isNetQuantity":false,"isLimitOrder":false,"isStopOrder":false,"isELSOrder":false,"stopPegBaseType":-1,"limitPegBaseType":-1,"range":0,"action":"I"}


Furthermore, a single market order can have many TradeIDs, if they are partial fills or closing of other orders. in this case, its more approriate to provide the OrderID which ties back to that spcific market order request, from there you can join this OrderID to any associated.

In entry order, an order ID is in callback function. You can also see it on order table sanpshot. but you will not get TradeID until order been executed. 

## limitation on historical candle download per request:

| Time-frame | max days back | max num |
| --- | --- | --- |
| m1 | 16 | 10,000 |
| m5 | 56 | 10,000 |
| m15 | 212 | 10,000 |
| m30 | 316 | 10,000 |
| h1 | 624 | 10,000 |
| h2 | 1224 | 10,000 |
| h3 | 2056 | 10,000 |
| h4 | 2664 | 10,000 |
| h6 | 3632 | 10,000 |
| h8 | 5128 | 10,000 |
| D1, W1, M1 | no limit | no limit |

## Max calls limitation:
| Error code ORA-20301, Cmd= | Max calls | Seconds | Function description |
| --- | --- | --- | --- |
| 8 | 2 | 600 | GET_TRADER_ORDERS |
| 24 | 500 | 3600 | SET_INSTRUMENTS |
| 44 | 50 | 3600 | Get Open positions by Account |
| 46 | 1500 | 3600 | Get Orders by Account |
| 79 | 500 | 3600 | Get all accounts per acct id |
| 117 | 10 | 60 | GetLastOrderMsg |
| 118 | 10 | 60 | getCustomClsdTrade |

## How to place trailing stop 

The fixed trailing stop should be 10 or above, for dynamic trailing stop = 1, number between 2-9 will be rejected. also the parameter is trailing_stop_step
      
      Example Entry order with trailing stop of 10 pips:
      POST /trading/create_entry_order account_id=1537581&symbol=EUR%2FUSD&is_buy=true&rate=1.1655&amount=3&order_type=Entry&time_in_force=GTC&stop=-50&trailing_stop_step=10&is_in_pips=true

## Difference between account name and account ID

There is a difference bewteen account name and account id. usually removing the heading zeros are account ID. and you need to pass account_id when you place orders. You can retrieve this information from /trading/get_model/Accounts.

      Wrong:
      {"is_buy":false,"account_id":"00654061","symbol":"EUR/USD","rate":1.15,"amount":11,"stop":-40,"is_in_pips":true,"order_type":"AtMarket","time_in_force":"GTC"}

      ERR noExec: /trading/create_entry_order
      {"code":3,"message":"Amount should be divisible by 10","parameters":["10"]}
 
      Correct:
      {"is_buy":false,"account_id":"654061","symbol":"EUR/USD","rate":1.15,"amount":11,"stop":-40,"is_in_pips":true,"order_type":"AtMarket","time_in_force":"GTC"}
      
      request # 2  has been executed: {
      "response": {"executed": true}, "data": {"type": 0,"orderId": 194963057}}


## Offer ID
| Symbol | OfferId |
| ------ | ------- |
| EUR/USD | 1 |
| USD/JPY | 2 |
| GBP/USD | 3 |
| USD/CHF | 4 |
| EUR/CHF | 5 |
| AUD/USD | 6 |
| USD/CAD | 7 |
| NZD/USD | 8 |
| EUR/GBP | 9 |
| EUR/JPY | 10 |
| GBP/JPY | 11 |
| CHF/JPY | 12 |
| GBP/CHF | 13 |
| EUR/AUD | 14 |
| EUR/CAD | 15 |
| AUD/CAD | 16 |
| AUD/JPY | 17 |
| CAD/JPY | 18 |
| NZD/JPY | 19 |
| GBP/CAD | 20 |
| GBP/NZD | 21 |
| GBP/AUD | 22 |
| AUD/NZD | 28 |
| USD/SEK | 30 |
| EUR/SEK | 32 |
| EUR/NOK | 36 |
| USD/NOK | 37 |
| USD/MXN | 38 |
| AUD/CHF | 39 |
| EUR/NZD | 40 |
| USD/ZAR | 47 |
| USD/HKD | 50 |
| ZAR/JPY | 71 |
| USD/TRY | 83 |
| EUR/TRY | 87 |
| NZD/CHF | 89 |
| CAD/CHF | 90 |
| NZD/CAD | 91 |
| TRY/JPY | 98 |
| USD/ILS | 100 |
| USD/CNH | 105 |
| AUS200 | 1001 |
| ESP35 | 1002 |
| FRA40 | 1003 |
| GER30 | 1004 |
| HKG33 | 1005 |
| JPN225 | 1007 |
| NAS100 | 1008 |
| SPX500 | 1010 |
| UK100 | 1012 |
| US30 | 1013 |
| Copper | 1016 |
| CHN50 | 1020 |
| EUSTX50 | 1035 |
| USDOLLAR | 1058 |
| US2000 | 1060 |
| USOil | 2001 |
| UKOil | 2002 |
| SOYF | 2003 |
| NGAS | 2015 |
| WHEATF | 2020 |
| CORNF | 2021 |
| Bund | 3001 |
| XAU/USD | 4001 |
| XAG/USD | 4002 |
| XAU/EUR | 4005 |
| XAU/AUD | 4006 |
| XAU/TRY | 4007 |
| BTC/USD | 8001 |
| BCH/USD | 8002 |
| ETH/USD | 8003 |
| LTC/USD | 8004 |

## Real Case Study:

1. Learn how to run BT backtest on FXCM historical data via RestAPI at <a href="https://apiwiki.fxcorporate.com/api/StrategyRealCaseStudy/RestAPI/BT strategy on FXCM data.zip">here</a>. 
What is <a href="http://pmorissette.github.io/bt/">bt?</a> 
2. Learn how to run QSTrader on FXCM data via RestAPI at <a href="https://apiwiki.fxcorporate.com/api/StrategyRealCaseStudy/RestAPI/QSTrader on FXCM data.zip">here</a>. 
what is <a href="https://www.quantstart.com/qstrader">QSTrader?</a>
3. Building/back testing RSI strategy via RestAPI at <a href="https://apiwiki.fxcorporate.com/api/StrategyRealCaseStudy/RestAPI/RsiStrategy.zip">here</a>.
4. Building/back testing Moving Average Crossover strategy via RestAPI at [Here](https://apiwiki.fxcorporate.com/api/StrategyRealCaseStudy/RestAPI/Moving_Average_Crossover_Strategy.zip)
5. One video demonstrate how to backtest strategies in Visual Studio via FXCM data On QuantConnect LEAN platform at <a href="https://www.youtube.com/watch?v=m6llfznP4d4">here</a>

## Release: 
#### 01/12/2020:
We did a release on demo on 1/12/2020 to improve the Rest API.
With that said, Our REST API wrapper fxcmpy has been updated, you need to install the latest fxcmpy version at 1.2.6.
Here is the link where you can find the [library:](https://pypi.org/project/fxcmpy/#files)
Please have in mind that just with pip install fxcmpy it might not work as it won’t update the library, please use below command.

pip install python-socketio

pip install –U fxcmpy

#### 01/24/2020:
We will release to live account coming weekend at 01/24/2020. 
Please refer to the above release and make sure your code is working on demo and get prepared.
Please contact us at api@fxcm.com if you have any questions. 

## Note:
o	This is for personal use and abides by our [EULA](https://www.fxcm.com/uk/forms/eula/)

o	For more information, you may contact us: api@fxcm.com

## Disclaimer:

High Risk Investment Notice: 

Trading Forex/CFDs on margin carries a high level of risk and may not be suitable for all investors. The products are intended for retail, professional, and eligible counterparty clients. Retail clients who maintain account(s) with Forex Capital Markets Limited (“FXCM LTD”) could sustain a total loss of deposited funds but are not subject to subsequent payment obligations beyond the deposited funds but professional clients and eligible counterparty clients could sustain losses in excess of deposits. Clients who maintain account(s) with FXCM Australia Pty. Limited (“FXCM AU”), FXCM South Africa (PTY) Ltd (“FXCM ZA”) or FXCM Markets Limited (“FXCM Markets”) could sustain losses in excess of deposits. Prior to trading any products offered by [FXCM LTD](https://www.fxcm.com/uk/), inclusive of all EU branches, [FXCM AU](https://www.fxcm.com/au/), [FXCM ZA](https://www.fxcm.com/za/), any affiliates of aforementioned firms, or other firms within the FXCM group of companies [collectively the “FXCM Group”], carefully consider your financial situation and experience level. If you decide to trade products offered by FXCM AU (AFSL 309763), you must read and understand the [Financial Services Guide](https://docs.fxcorporate.com/financial-services-guide-au.pdf), [Product Disclosure Statement](https://www.fxcm.com/au/legal/product-disclosure-statements/), and [Terms of Business](https://docs.fxcorporate.com/tob_au_en.pdf). Our Forex/CFD prices are set by FXCM, are not made on an Exchange and are not governed under the Financial Advisory and Intermediary Services Act. The FXCM Group may provide general commentary, which is not intended as investment advice and must not be construed as such. Seek advice from a separate financial advisor. The FXCM Group assumes no liability for errors, inaccuracies or omissions; does not warrant the accuracy, completeness of information, text, graphics, links or other items contained within these materials. Read and understand the Terms and Conditions on the FXCM Group’s websites prior to taking further action. 
