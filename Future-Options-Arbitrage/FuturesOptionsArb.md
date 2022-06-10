---
title: "Futures-Options Arbitrage Trade"
author: "Sanjaya J Shetty"
date: '2022-06-02'
output:
  html_document:
    keep_md: yes
---

### What are Futures and Options?

------------------------------------------------------------------------

A Futures is a financial contract that obligates the participant to transact an asset at a predetermined future date and price. A seller of the Future contract would be obligated to sell the underlying stock/index/commodity at a predetermined price and date, where as the buy would be obligated to buy the same.

In Options, the obligation part is removed. Instead, it has two types of contracts,

-   Call Option
-   Put Option

In a call options, The buyer would have a right, but not the obligation to buy the underlying at the predetermined date and price.

And in a Put options, the buy would have a right, but not the obligation to sell the underlying at the predetermined date and price.

The predetermined price in option lingo is `Strike Price` and the date is `Expiry date`.

*Another* important term you need to be aware is ITM, OTM and ATM options

-   ITM: The options whose Strike price is less than the price of the underlying
-   OTM: The options whose Strike price is more than the price of the underlying
-   ATM: The options whose Strike price is equal to the price of the underlying (or could be near to the underlying)

### What about Arbitrage? What is it??

------------------------------------------------------------------------

An Arbitrage in trading is a trade which is inherently a risk-free trade arising due to price deviations among the asset classes.

A simple example could be when the price of a share (same company) listed in two different exchanges have a some kind of price deviation. Then we could short (sell) the shares in the exchange where the price is listed higher and buy the same shares in the exchange where the price is quoted lower.

### Now What is the Arbitrage here?

------------------------------------------------------------------------

We would look at the Ask price of all the ITM (in the Money) Options. If the Asking Price (For Call Option) of the Options plus the Strike price is less than the Future price of the underlying, then we have an arbitrage opportunity.

In simpler terms, Imagine we have an underlying stock ABC with a price of \$120. The call option price would be \$22 for the strike price of \$100. The Future is trading at \$125. Now with this example, what would happen if you sell the Future and buy the options at \$100 strike price for \$22? (you will hold it till the expiry)

When you sell the Future: You promise someone that you would sell them ABC at \$125. When you buy the Call Option: You can buy the same underlying at \$100. Since Options and Futures have the same lot size, we could sell the same underlying we bought at the Options counter to the one at the Future counter. Hence any difference would be our profit.

Does that mean we have a \$25 profit in the above example? Well no, since we had to pay \$22 for the options premium, hence \$25-\$22 = \$3 would be our profit ( incl. other brokerage charges).

### Find the arbitrage programmatically using R.

------------------------------------------------------------------------

We would use S&P500 Options and S&P500 Futures (E Mini Futures)

#### Load the Library

------------------------------------------------------------------------

we would load `quantmod` to load all the required data.


```r
library(quantmod)
library(dplyr)
library(kableExtra)
```

#### Setting up the expiry date

------------------------------------------------------------------------

Since the project was done on June 6 2022, we would pick the nearest expiry contract of the futures, which would be June 17, 2022. We would be picking the Futures expiry date since we need both options and futures to expire on the same day.


```r
date = '20220617'
```

#### Get the Option Chain data.

------------------------------------------------------------------------

A Option Chain would contain all the necessary information regrading the option at every strike price. It basically gives you a birds eye view of the same. Here we would use `quantmod` to extract Option chain from Yahoo Finance.


```r
optionData = getOptionChain('^SPX', src = 'yahoo', Exp = date)
```

#### Separate call and Put data into to dataframes

------------------------------------------------------------------------

Since the class of `optionData` is list with two dataframes, we need to make them datarframe.


```r
callData = optionData$calls
putData = optionData$puts
```

#### Getting the Futures Data.

------------------------------------------------------------------------

We would use the same `quantmod` package to pull the futures quoted price of S&P500 (ticker symbol : `ES=F`)


```r
future = getQuote('ES=F')
FutData = future$Last
```

#### Filter the ITM options

------------------------------------------------------------------------

We would only look into ITM options, because these are the options whose strike is lower than the price of the underlying (for call options), which in turn would be lower than the futures price (in ideal cases).


```r
callData = callData[callData$ITM == 'TRUE',]
putData = putData[putData$ITM == 'TRUE',]
```

#### Get the Arbitrage prices

------------------------------------------------------------------------

Calculating the arbitrage price for the options has at least a single ask price offer.


```r
callData$ArbAmt = ifelse(callData$Ask == 0, 0, 
                         FutData - (callData$Strike + callData$Ask))

putData$ArbAmt =  ifelse(putData$Ask == 0, 0, 
                         (putData$Strike - putData$Ask )) - FutData
```

#### Can we find the any opportunity right now?

------------------------------------------------------------------------

We would try to extract the Options with the best Opportunity


```r
callMax = which.max(callData$ArbAmt)
putMax = which.max(putData$ArbAmt)

bestCallStrike = callData$Strike[callMax]
bestPutStrike = putData$Strike[putMax]

callArbPrice = callData$ArbAmt[callMax]
putArbPrice = putData$ArbAmt[putMax]
```

#### The Best Opportunity in the Call Options?

------------------------------------------------------------------------


```
## [1] "The best Arbitarage is $120.90 per contract at 2250CE strike"
```

#### The Best Opportunity in the Put Options?

------------------------------------------------------------------------


```
## [1] "The Arbitarage is $470.30 per contract at the 5200PE strike"
```

#### Other Opportunity in Call Options

------------------------------------------------------------------------

<table class="table" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:center;"> Strike Price </th>
   <th style="text-align:center;"> Arbitrage Price </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:center;"> 3425 </td>
   <td style="text-align:center;"> 38.1 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 1000 </td>
   <td style="text-align:center;"> 4.7 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 1300 </td>
   <td style="text-align:center;"> 4.7 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 200 </td>
   <td style="text-align:center;"> 3.5 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 900 </td>
   <td style="text-align:center;"> 3.4 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 1600 </td>
   <td style="text-align:center;"> 3.4 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2050 </td>
   <td style="text-align:center;"> 3.4 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2100 </td>
   <td style="text-align:center;"> 3.3 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2800 </td>
   <td style="text-align:center;"> 3.3 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2850 </td>
   <td style="text-align:center;"> 3.1 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 3500 </td>
   <td style="text-align:center;"> 3.1 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2400 </td>
   <td style="text-align:center;"> 2.9 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 3300 </td>
   <td style="text-align:center;"> 2.5 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 1650 </td>
   <td style="text-align:center;"> 2.3 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 3200 </td>
   <td style="text-align:center;"> 2.3 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 3000 </td>
   <td style="text-align:center;"> 2.1 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 3350 </td>
   <td style="text-align:center;"> 2.0 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 3400 </td>
   <td style="text-align:center;"> 1.9 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 3550 </td>
   <td style="text-align:center;"> 1.6 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 3600 </td>
   <td style="text-align:center;"> 1.4 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 3430 </td>
   <td style="text-align:center;"> 1.3 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 3560 </td>
   <td style="text-align:center;"> 0.8 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 3570 </td>
   <td style="text-align:center;"> 0.8 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 3580 </td>
   <td style="text-align:center;"> 0.7 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 3620 </td>
   <td style="text-align:center;"> 0.3 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 3650 </td>
   <td style="text-align:center;"> 0.2 </td>
  </tr>
</tbody>
</table>

#### Other Opportunity in Put Options

------------------------------------------------------------------------

<table class="table" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:center;"> Strike Price </th>
   <th style="text-align:center;"> Arbitrage Price </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:center;"> 5600 </td>
   <td style="text-align:center;"> 430.4 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 5180 </td>
   <td style="text-align:center;"> 404.8 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 5220 </td>
   <td style="text-align:center;"> 404.4 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 5210 </td>
   <td style="text-align:center;"> 404.2 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 5130 </td>
   <td style="text-align:center;"> 381.4 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 6100 </td>
   <td style="text-align:center;"> 380.5 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 5030 </td>
   <td style="text-align:center;"> 372.6 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 5090 </td>
   <td style="text-align:center;"> 324.9 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 5160 </td>
   <td style="text-align:center;"> 312.1 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 5010 </td>
   <td style="text-align:center;"> 306.8 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 6000 </td>
   <td style="text-align:center;"> 298.9 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 4645 </td>
   <td style="text-align:center;"> 196.0 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 4620 </td>
   <td style="text-align:center;"> 91.8 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 4760 </td>
   <td style="text-align:center;"> 90.0 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 5350 </td>
   <td style="text-align:center;"> 85.1 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 7200 </td>
   <td style="text-align:center;"> 82.0 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 4695 </td>
   <td style="text-align:center;"> 79.5 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 5900 </td>
   <td style="text-align:center;"> 64.4 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 5700 </td>
   <td style="text-align:center;"> 64.0 </td>
  </tr>
</tbody>
</table>

**Note: The large difference between the closing prices of Future and Options is due to the time difference between the closing price of options and the Future contract of S&P 500. (`ES=F` trade 24x7 but the options trade during the market hour.)**

*Disclaimer: This is not a financial advice. So please do contact your financial Advisor before taking any positions.*
