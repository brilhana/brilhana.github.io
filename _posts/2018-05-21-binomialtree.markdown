---
layout: post
title:  "Binomial Tree"
date:   2018-05-21 10:01:12
description: "Cox-Ross-Rubinstein binomial tree model."
categories:
- blog
---

In this post I will present another simple option pricing model: the Cox-Ross-Rubinstein model, also known as the binomial tree model, which is a variation of the original Black-Scholes model that I will cover in my next post. This method is popular because it considers the underlying instrument over a period of time instead of at just one point in time. It does this by using a lattice-based tree, which takes into account expected changes in various parameters over the option's life, thereby producing a more accurate estimate of option prices than created by models that consider only one point in time. The Cox-Ross-Rubinstein model uses a risk-neutral valuation method. Its underlying principal affirms that when determining option prices, it can be assumed that the world is risk neutral and that all individuals (and investors) are indifferent to risk.

The Cox-Ross-Rubinstein model is a two-state model in that it assumes the underlying price can only either increase or decrease with time until expiration. Valuation begins at each of the final nodes (at expiration) and iterations are performed backwards through the binomial tree up to the first node (date of valuation). In very basic terms, it's a bottom-up approach and the model involves three steps:
1. Create the binomial price tree.
2. Compute the option value at each final node.
3. Compute the option value at each preceding node.

Here's a simple implementation of the method in `Julia`.

```julia
function crr_am_put(S, K, r, σ, t, N)
    Δt = t / N
    U = exp(σ * √Δt)
    D = 1 / U
    R = exp(r * Δt)
    p = (R - D) / (U - D)
    q = (U - R) / (U - D)
    Z = [max(0, K - S * exp((2 * i - N) * σ * √Δt)) for i = 0:N]
    
    for n = N-1:-1:0
        for i = 0:n
            x = K - S * exp((2 * i - n) * σ * √Δt)
            y = (q * Z[i+1] + p * Z[i+2]) / R
            Z[i+1] = max(x, y)
        end
    end

    return Z[1]
end

crr_am_put(100, 90, 0.05, 0.3, 180/365, 1000)
````

Alternatively, in a more esoteric programming language: `F#`.

```fsharp
let crr_am_put s k r o t n =
    let deltaT = t/float n
    let up = exp(o*sqrt(deltaT))
    let p0 = (up*exp(-r*deltaT))*up/((up*up)-1.0)
    let p1 = exp(-r*deltaT)-p0
    let value = Array.zeroCreate(n+1)

    for i = 0 to n do
      value.[i] <- (k-s) * (up**(float(2*i-n)))
      if (value.[i] >= 0.0) then value.[i] <- 0.0
    
    for j = n - 1 downto 0 do
      for i = 0 to j do
        value.[i] <- p0*value.[i]+p1*value.[i+1]
        let exercise = k-s*(up**float(2*i-j))
        if value.[i] < exercise then value.[i] <- exercise
    
    value.[0]

crr_am_put 100.0 90.0 0.05 0.3 180.0/365.0 1000
```

Next up, we'll cover the most-known pricing model: Black-Scholes.

**Reference**

John Cox, Stephen Ross and Mark Rubinstein. *Option Pricing: A Simplified Approach*. Journal of Financial Economics 7, 1979.