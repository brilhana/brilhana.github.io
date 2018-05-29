---
layout: post
title:  "Black-Scholes"
date:   2018-05-29 10:01:12
description: "Black-Scholes model."
categories:
- blog
---

In this last post of a serie about options pricing models, we'll discuss the Black-Scholes model. Black-Scholes, named after its creators Fischer Black and Myron Scholes, was the first widely used model for option pricing. It's used to calculate the theoretical value of European-style options using current stock prices, expected dividends, the option's strike price, expected interest rates, time to expiration and expected volatility.

The key financial insight behind the model is that one can perfectly hedge the option by buying and selling the underlying asset in just the right way and consequently eliminate risk. This hedge, in turn, implies that there is only one right price for the option, as returned by the Black–Scholes formula. The value of a call option for a non-dividend-paying underlying stock in terms of the Black–Scholes parameters is:
![Black-Scholes](/assets/bs.png)

Where:
+ `N` is the cumulative distribution function of the standard normal distribution.
+ `T-t` is the time to maturity, expressed in years.
+ `S` is the spot price of the underlying asset.
+ `K` is the strike price.
+ `r` is the risk free rate, expressed in terms of continuous compounding.
+ `sigma`  is the volatility of returns of the underlying asset.

Under certain assumptions on the asset and market:
+ The option is European and can only be exercised at expiration.
+ No dividends are paid out during the life of the option.
+ Markets are efficient.
+ There are no transaction costs in buying the option.
+ The risk-free rate and volatility of the underlying are known and constant.
+ The returns on the underlying are normally distributed.

The model is essentially divided into two parts: the first part multiplies the price by the change in the call premium in relation to a change in the underlying price. This part of the formula shows the expected benefit of purchasing the underlying outright. The second part provides the current value of paying the exercise price upon expiration, remember the Black-Scholes model applies to European options that can be exercised only on expiration day. The value of the option is calculated by taking the difference between the two parts, as shown in the equation.

Here's a simple implementation in `F#`.

```fsharp
type Style = Call | Put

let cnd x =
    let pow x n = exp(n*log(x))
    let a1 = 0.31938153
    let a2 = -0.356563782
    let a3 = 1.781477937
    let a4 = -1.821255978
    let a5 = 1.330274429
    let pi = 4.0*atan 1.0
    let l = abs(x)
    let k = 1.0/(1.0+0.2316419*l)
    let w = ref(1.0-1.0/sqrt(2.0*pi)*
            exp(-l*l/2.0)*(a1*k+a2*k*k+a3*(pow k 3.0)+
            a4*(pow k 4.0)+a5*(pow k 5.0)))
    if (x < 0.0) then w := 1.0 - !w
    !w

// style: 'Call' if call option; otherwise 'Put'
// s: stock price
// k: strike price of option
// t: time to expiration in years
// r: risk free interest rate
// v: volatility
let blackscholes style s k t r v =
    let d1 = (log(s/k)+(r+v*v/2.0)*t)/(v*sqrt(t))
    let d2 = d1-v*sqrt(t)
    match style with
        | Call -> s*cnd(d1)-k*exp(-r*t)*cnd(d2)
        | Put -> k*exp(-r*t)*cnd(-d2)-s*cnd(-d1)

blackscholes Call 60.0 65.0 0.25 0.08 0.3;;
```

**Reference**

Fischer Black and Myron Scholes. *The Pricing of Options and Corporate Liabilities*. Journal of Political Economy, 1981.