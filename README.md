
# Tradier .NET Client

Tradier .NET Client is a .NET Library for the [Tradier API](https://documentation.tradier.com/) to interact with the Tradier Brokerage platform to get account information, market data, place orders, and create watchlists. 

In order to use this client you will need to have an Access Token from Tradier for either the [Developer Sandbox](https://developer.tradier.com/user/sign_up) or the [Brokerage Account](https://documentation.tradier.com/brokerage-api).

## Usage

To implement the Library into your project, install [NuGet package]() into your solution/project by running in the Package Manager Console.
````
PM> Install-Package TradierDotNetClient
````
Or by searching TradierDotNetClient in the Package Manager searching bar.

### Client
To implement the client into your project, include the `Tradier.Client` namespace:
```csharp
using Tradier.Client;
```

To instantiate a new client in your program, include the following:

```csharp
TradierClient client = new TradierClient("<TOKEN>");
```
**On an important note**, the client's constructor default setting is to use the Developer Sandbox. To set the client to Brokerage Account, you need to explicitly set it to Production by including the property `useProduction` and set it to true:

```csharp
TradierClient client = new TradierClient("<TOKEN>", useProduction: true);
```

*Note that it is not needed to pass a `HttpClient` to the client. TradierDotNetClient already creates one using the `HttpClientFactory` helping it to keep control over the requests.*

<br/>
<div align="right">
    <b><a href="#tradier-net-client">↥ back to top</a></b>
</div>
<br/>

### Account
Fetch positions, balances and other account related details.
For all the account-related usages, make sure you include the Account Model namespace:
```csharp
using Tradier.Client.Models.Account;
```
#### Get User Profile
The user’s profile contains information pertaining to the user and his/her accounts. In addition to listing all the accounts a user has, this call should be used to create a personalized experience for your customers (i.e. displaying their name when they log in).
```csharp
Profile userProfile = await client.Account.GetUserProfile();
```

#### Get Balances
Get balances information for a specific user account. Account balances are calculated on each request during market hours. Each night, balance figures are reconciled with our clearing firm and used as starting point for the following market session.

*The account number is found in the Profile model*.
```csharp
Balances balance = await client.Account.GetBalances(accountNumber);
```

#### Get Positions
Get the current positions being held in an account. These positions are updated intraday via trading.

*The account number is found in the Profile model*.
```csharp
Positions positions = await client.Account.GetPositions(accountNumber);
```

#### Get History
Get historical activity for an account. This data originates with our clearing firm and inherently has a few limitations:
-   Updated nightly (not intraday)
-   Will not include specific time (hours/minutes) a position or order was created or closed
-   Will not include order numbers

*The account number is found in the Profile model*.
```csharp
History history = await client.Account.GetHistory(accountNumber);
```

#### Get Gain/Loss
Get cost basis information for a specific user account. This includes information for all closed positions. Cost basis information is updated through a nightly batch reconciliation process with our clearing firm.

*The account number is found in the Profile model*.
```csharp
GainLoss gainLoss = await client.Account.GetGainLoss(accountNumber);
```

#### Get Orders
Retrieve orders placed within an account. Without additional parameters, this API will return orders placed for the market session of the present calendar day.

*The account number is found in the Profile model*.
```csharp
Orders orders = await client.Account.GetOrders(accountNumber);
```

#### Get an Individual Order
Get detailed information about a previously placed order.

*The account number is found in the Profile model*.
```csharp
Order order = await client.Account.GetOrder(accountNumber, orderId);
```

<br/>
<div align="right">
    <b><a href="#tradier-net-client">↥ back to top</a></b>
</div>
<br/>

### Market Data
Fetch quotes, chains and historical data.
For all the market data related usages, make sure you include the MarketData Model namespace:
```csharp
using Tradier.Client.Models.MarketData;
```

#### Get Quotes
Get a list of symbols using a keyword lookup on the symbols description. Results are in descending order by average volume of the security. This can be used for simple search functions.

You can call GetQuotes passing a comma-separated `string` with the symbols:
```csharp
Quotes quotes = await client.MarketData.GetQuotes("AAPL, NFLX");
```
Or you can call GetQuotes passing a `List<string>` with the symbols:
```csharp
List<string> symbols = new List<string>();
symbols.Add("AAPL");
symbols.Add("NFLX");

Quotes quotes = await client.MarketData.GetQuotes(symbols);
```
#### Get Quotes (POST method)
Get a list of symbols using a keyword lookup on the symbols description. Results are in descending order by average volume of the security. This can be used for simple search functions.

You can call PostGetQuotespassing a comma-separated `string` with the symbols:
```csharp
Quotes quotes = await client.MarketData.PostGetQuotes("AAPL, NFLX");
```
Or you can call PostGetQuotes passing a `List<string>` with the symbols:
```csharp
List<string> symbols = new List<string>();
symbols.Add("AAPL");
symbols.Add("NFLX");

Quotes quotes = await client.MarketData.PostGetQuotes(symbols);
```

#### Get Option Chains
Get all quotes in an option chain. Greek and IV data is included courtesy of [ORATS](https://info.orats.com/dataapi?utm_campaign=Tradier&utm_source=email). Please check out their APIs for more in-depth options data.
```csharp
Options options = await client.MarketData.GetOptionChain("AAPL", "2020-05-27");
```

#### Get Options Expirations
Get expiration dates for a particular underlying.

Note that some underlying securities use a different symbol for their weekly options (RUT/RUTW, SPX/SPXW). To make sure you see all expirations, make sure to send the includeAllRoots parameter. This will also ensure any unique options due to corporate actions (AAPL1) are returned.
```csharp
Expirations expirations = await client.MarketData.GetOptionExpirations("AAPL");
```

#### Get Option Strikes
Get an options strike prices for a specified expiration date.

You can call GetStrikes passing the expiration as a `string`:
```csharp
Strikes strikes = await client.MarketData.GetStrikes("UNG", "May 15, 2020");
```
Or you can call GetStrikes passing the expiration as a `DateTime`
```csharp
Strikes strikes = await client.MarketData.GetStrikes("UNG", DateTime.Now);
```

#### Get Historical Quotes
Get historical pricing for a security. This data will usually cover the entire lifetime of the company if sending reasonable start/end times. You can fetch historical pricing for options by passing the OCC option symbol (ex. AAPL220617C00270000) as the symbol.

You can call GetStrikes passing the start/end dates as a `string`:
```csharp
HistoricalQuotes historicalQuotes = 
	await client.MarketData.GetHistoricalQuotes("AAPL", "daily", "January 1, 2020", "May 15, 2020");
```

Or you can call GetStrikes passing the start/end dates as a `DateTime`
```csharp
HistoricalQuotes historicalQuotes = 
	await client.MarketData.GetHistoricalQuotes("AAPL", "daily", DateTime.Today.AddDays(-1), DateTime.Today);
```
#### Get Time and Sales
Time and Sales (timesales) is typically used for charting purposes. It captures pricing across a time slice at predefined intervals.

Tick data is also available through this endpoint. This results in a very large data set for high-volume symbols, so the time slice needs to be much smaller to keep downloads time reasonable.
|Interval|Data Available (Open)|Data Available (All)|
|--|--|--|
|tick|5 days|N/A|
|1min|20 days|10 days|
|5min|40 days|18 days|
|15min|40 days|18 days|

You can call GetStrikes passing the start/end dates as a `string`:
```csharp
Series timeSales = 
	await client.MarketData.GetTimeSales("AAPL", "1min", "May 1, 2020", "May 15, 2020");
```

Or you can call GetStrikes passing the start/end dates as a `DateTime`
```csharp
Series timeSales = 
	await client.MarketData.GetTimeSales("AAPL", "1min", DateTime.Today.AddDays(-1), DateTime.Today);
```

#### Get ETB Securities
The ETB list contains securities that are able to be sold short with a Tradier Brokerage account. The list is quite comprehensive and can result in a long download response time.

```csharp
Securities securities = await client.MarketData.GetEtbSecurities();
```

#### Get Clock
Get the intraday market status. This call will change and return information pertaining to the current day. If programming logic on whether the market is open/closed – this call should be used to determine the current state.

```csharp
Clock clock = await client.MarketData.GetClock();
```

#### Get Calendar
Get the market calendar for the current or given month. This can be used to plan ahead regarding strategies. However, the [Get Intraday Status](https://documentation.tradier.com/brokerage-api/markets/documentation/brokerage_api/markets/get-clock) should be used to determine the current status of the market.

```csharp
Calendar calendar = await client.MarketData.GetCalendar();
```

#### Search Companies
Get a list of symbols using a keyword lookup on the symbols description. Results are in descending order by average volume of the security. This can be used for simple search functions.

```csharp
Securities securitiesFilter = await client.MarketData.SearchCompanies("NY");
```

#### Lookup Symbol
Search for a symbol using the ticker symbol or partial symbol. Results are in descending order by average volume of the security. This can be used for simple search functions.

```csharp
Securities lookup = await client.MarketData.LookupSymbol("goog");
```

<br/>
<div align="right">
    <b><a href="#tradier-net-client">↥ back to top</a></b>
</div>
<br/>

### Trading
Place equity and complex option trades including advanced orders.
For all the trading related usages, make sure you include the Trading Model namespace:
```csharp
using Tradier.Client.Models.Trading;
```

#### Place Equity Order
Place an order to trade an equity security.
```csharp
OrderReponse order = 
	await client.Trading.PlaceEquityOrder(accountNumber, "equity", "AAPL", "buy", "10", "market", "day", "1.00", "1.00");
```

#### Place Option Order
Place an order to trade a single option.
```csharp
IOrder order = 
	await client.Trading.PlaceOptionOrder(accountNumber, "option", "SPY", "SPY140118C00195000", "buy_to_open", "10", "market", "day", "1.00", "1.00", preview: true);
```

#### Place Multileg Order
Place a multileg order with up to 4 legs.
```csharp
List<string> optionSymbols = new List<string>();
optionSymbols.Add("SPY190605C00282000");
optionSymbols.Add("SPY190605C00286000");

List<string> sides = new List<string>();
sides.Add("buy_to_open");
sides.Add("buy_to_close");

List<string> quantities = new List<string>();
sides.Add("10");
sides.Add("10");

OrderReponse order = 
	await client.Trading.PlaceMultilegOrder(accountNumber, "option", "AAPL", "market", "day", "1.00", optionSymbols, sides, quantities);
```

#### Modify Order
Modify an order. You may change some or all of these parameters. You may not change the session of a pre/post market session order. Send only the parameters you would like to adjust.
```csharp
OrderReponse order = 
	await client.Trading.ModifyOrder(accountNumber, orderId, "limit", "day", "1.00", "1.00");
```

#### Cancel Order
Cancel an order.
```csharp
OrderReponse order = await client.Trading.CancelOrder(accountNumber, orderId);
```

<br/>
<div align="right">
    <b><a href="#tradier-net-client">↥ back to top</a></b>
</div>
<br/>

## Authors

* **[Henrique Tedeschi](https://github.com/htedeschi)**
* **[Vitali Karmanov](https://github.com/vitali-karmanov)**

## Disclaimer

This Wrapper is NOT an official .NET Tradier Library. This library is provided "as is" without expressed or implied warranty of any kind. Please use at your own risk and discretion.

## License
This Library is provided under the [MIT License](https://raw.github.com/sta/websocket-sharp/master/LICENSE.txt).
