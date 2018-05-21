---
layout: post
title:  "Least-Squares Monte Carlo"
date:   2018-05-17 10:01:12
description: Longstaff–Schwartz least-squares Monte Carlo method.
categories:
- blog
---

This post presents a simple yet powerful approach for approximating the value of American options by simulation: the Longstaff–Schwartz least-squares Monte Carlo method. The key to this method is the use of least-squares to estimate the conditional expected payoff to the option holder from continuation. This makes this approach readily applicable in path-dependent and multifactor situations where traditional finite difference techniques cannot be used. From a practical perspective, it's well suited to parallel computing which allows significant gains in computational speed and efficiency.

To understand the intuition behind this approach, recall that at any exercise time, the holder of an American option optimally compares the payoff from immediate exercise with the expected payoff from continuation and then exercises if the immediate payoff is higher. Thus, the optimal exercise strategy is fundamentally determined by the conditional expectation of the payoff from continuing to keep the option alive. The key insight underlying our approach is that this conditional expectation can be estimated from the cross-sectional information in the simulation by using least-squares. The fitted value from this regression provides a pathwise approximation to the optimal stopping rule that maximizes the value of the American option. With this specification, American options can then be valued accurately by simulation.

Here's a simple implementation of the method in `Julia`.

```julia
function lsmc_am_put(S, K, r, σ, t, N, P)
    Δt = t / N
    R = exp(r * Δt)
    T = typeof(S * exp(-σ^2 * Δt / 2 + σ * √Δt * 0.1) / R)
    X = Array{T}(N+1, P)

    for p = 1:P
        X[1, p] = x = S
        for n = 1:N
            x *= R * exp(-σ^2 * Δt / 2 + σ * √Δt * randn())
            X[n+1, p] = x
        end
    end

    V = [max(K - x, 0) / R for x in X[N+1, :]]

    for n = N-1:-1:1
        I = V .!= 0
        A = [x^d for d = 0:3, x in X[n+1, :]]
        β = A[:, I]' \ V[I]
        cV = A' * β
        for p = 1:P
            ev = max(K - X[n+1, p], 0)
            if I[p] && cV[p] < ev
                V[p] = ev / R
            else
                V[p] /= R
            end
        end
    end

    return max(mean(V), K - S)
end

lsmc_am_put(100, 90, 0.05, 0.3, 180/365, 1000, 10000)
```

**Reference**

Francis Longstaff and Eduardo Schwartz. *Valuing American Options by Simulation: A Simple Least-Squares Approach*. The Review of Financial Studies, Vol. 14, No. 1, 2001.