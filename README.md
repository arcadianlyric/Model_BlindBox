# Modeling_BlindBox: demand estimation and pricing for a blind box series

## Goal

Model demand and optimal pricing for a blind box series, and use the model to answer five questions: why a hot figure is priced the way it is, how prices should move when stock piles up, roughly how much profit the seller makes, how a seller could find a better price, and how many boxes to open given that opening a box, selling a figure and selling a set all draw on the same stock.

This is a modeling project: demand estimation, discrete choice and revenue optimisation under inventory constraints. It uses real public prices but no real sales data, so it demonstrates the method and a decision tool rather than making empirical claims about this seller.

[Interactive page](https://arcadianlyric.github.io/Modeling_BlindBox/). 

## Results

The data are public prices from one Taobao seller: 6 regular figures at 142 / 136 / 88 / 89 / 84 / 84, 1 hidden figure at 335, a single blind box at 108, a confirmed-figure set (all 6 regular figures) at 618 and a sealed, unopened set at 732. Official prices are 129 per box and 774 per set, and the hidden figure probability is 1/72.

**1. Prices are internally consistent.** The 6 regular figures average 103.8. Including the hidden figure, the expected revenue from opening one box is about 107, almost exactly the 108 box price, so the seller appears to anchor on "expected value of confirmed figures ≈ box price". The confirmed set at 618 is only 5 below buying the 6 figures separately (623), so it carries almost no discount. The sealed set at 732 is 114 above the confirmed set. A hidden figure replaces a regular one, so the hidden-figure chance adds only 6/72 × (335 − 103.8) ≈ 19.3, leaving about 94.7 as a premium for sealed packaging, the outer case and the unboxing experience.

**2. What 142 implies about price sensitivity.** Under exponential demand D = a·e^(−bP), the profit-maximising price is P* = c + 1/b, so an observed price implies b = 1/(P − c). With c = 0, 142 implies b ≈ 0.0070 and the cold figures at 84 to 89 imply b ≈ 0.0115, so hot-figure buyers would be about 60% as price sensitive as cold-figure buyers. This is not a measurement: it follows from assuming that the prices are optimal and that cost is zero, and the ratio changes as c rises. It explains a price but cannot test whether the price is right. The "recently hot" label suggests 142 may still be below the optimum; result 7 shows how to check.

**3. Who owns the figures changes the optimal prices.** With a common price sensitivity β, independent sellers (one per figure) each satisfy Pᵢ − c = 1/(β(1−sᵢ)), so more popular figures get higher markups. One seller who owns all six maximises total profit instead, and the multinomial logit optimum is then the same markup on every figure. At the page defaults (β = 0.02, c = 60) the independent-seller prices are about 124 / 121 / 112 / 112 / 111 / 111 and the single-seller price is 144 for all six. For a single reseller, popularity alone therefore does not justify charging 142 for a hot figure and 84 for a cold one; the premium has to come from scarcity (result 6) or from different price sensitivities.

**4. A rolling clearance rule keeps prices flat, not rising.** Resetting each day to P = ln(a·T_left/Q_left)/b, the price at which expected demand uses up the remaining stock, keeps the ratio of stock to remaining days constant. Without noise the price stays at about 182 for the hot figure and 81 for the cold one at the defaults. The rising and falling paths seen with random sales come from noise, whole-unit sales and the last few days. This is a heuristic, not the optimal dynamic policy, which needs a value function over stock and time [3].

**5. Seller profit.** Cost cannot be uniquely identified from prices, so only a range is possible. With the default parameters (purchase discount 75%, platform fee 1%, shipping 4 per order, cold sell-through 85%, unsold stock salvaged at 50% of list), profit per box is about:

| Channel | Profit per box (CNY) |
|---|---|
| Sell blind box at 108 | 6.2 |
| Open box, sell confirmed figures | 1.0 |
| Confirmed set at 618 | 4.6 |
| Sealed set at 732 | 23.4 |

Selling blind boxes at 108 breaks even at a purchase discount of about 80%. Opening boxes and selling confirmed figures earns almost nothing beyond the purchase discount, and is worse than selling the box when cold figures do not sell through. The sealed set is the highest-margin channel.

**6. Opening boxes, figure prices and set prices are one decision.** Opening a box turns one sealed box into one random figure, the confirmed set uses one of each regular figure, and the sealed set uses six sealed boxes. This is the structure of network revenue management, with sets as multi-leg itineraries and box opening as random capacity [4]. The page solves it with a fluid approximation: for each number of boxes to open, it prices ten options (sealed box, sealed set, confirmed set, six figures, hidden figure) under a multinomial logit with option-specific price sensitivity [5], subject to expected sales not exceeding supply, and tests the plan in a stochastic simulation. At the defaults (240 sealed boxes, 60 days, 20 visitors a day, elasticity 3 at observed prices, salvage 50%; all demand parameters are assumptions):

- The plan opens 170 of 240 boxes. That earns 10,858 more than selling everything sealed and 3,200 more than opening everything. At the optimum, an opened box and a sealed box have the same shadow value (94.3), which is the opening rule in exact form: open another box only while its expected value as a random figure exceeds its value sealed.
- With one seller and one elasticity for every option, the markup over effective cost is the same for every option. The planned hot-figure prices (198 to 206) are far above the cold ones (94 to 100) because the hot figures have shadow prices of about 76 to 79 against 16 for the cold ones. This is result 3 again from the other side: here the hot premium comes from scarcity.
- In simulation (20 runs each), re-solving the plan daily from the stock actually left earned 31,549 in revenue plus salvage. The static plan earned 30,541, and keeping the observed prices while opening a box whenever any figure runs out earned 25,805. Opening the planned boxes gradually to keep options open earned 29,819, 1,730 less than opening them up front, because the confirmed set needs all six figures in stock and was possible on only 27 of 60 days instead of 56.

**7. Finding a better price needs a price test.** A second price point breaks the circularity of result 2: with daily sales D₀ at P₀ and D₁ at a test price P₁, the slope is b = ln(D₀/D₁) / (P₁ − P₀). With Q units and T days left, profit over the horizon is (P − c)·min(D(P)·T, Q), which peaks at c + 1/b without a stock limit and at no lower than the clearance price P₀ + ln(D₀T/Q)/b when stock is tight. At the page defaults the recommended price is 200 (the willingness-to-pay ceiling), which earns 8,700 more over 60 days than 142 but still sells out around day 47. Three practical lessons came out of building this:

- When a test barely moves sales, b is small and 1/b explodes. A test from 142 to 225 with sales falling only from 5.0 to 4.7 implies an optimum near 1400, which is extrapolation, not a recommendation. The page caps the price at a willingness-to-pay ceiling and flags any optimum more than 30% beyond the tested range.
- If sales rise with price, b cannot be estimated. In real data this points to noise, a promotion, a stock-out or a demand shift between the two periods, not to "higher prices sell better".
- The trustworthy procedure is iterative: move the price part of the way towards the model optimum, test again, and re-estimate.

## Discussion

**Price endogeneity is the central identification problem.** It applies at two levels. Across figures, β is not identified at all: each figure has its own αᵢ and only one price–sales point, so any β fits the observed sales exactly once αᵢ is backed out. Over time, the price of one figure is still not exogenous. The seller sets prices in response to demand, so popular figures have both a high αᵢ and a high price. Regressing shares on observed prices therefore confounds popularity with price and biases β towards zero, which makes demand look less price sensitive than it is [2]. Two ways out are instruments for price, as in the discrete choice literature, or randomised price variation. Result 7 takes the second route: a price test is an A/B experiment on price, and the same tools apply, including enough days per arm for the sales difference to exceed day-to-day noise and checks that nothing else changed between arms.

- The profit table depends on the assumed purchase discount. The real value should be recovered by regressing on per-figure sales: estimate β and each αᵢ, then back out cost from c = Pᵢ − 1/(β(1−sᵢ)). The six estimates should roughly agree; large disagreement means "priced optimally" fails and prices may instead be anchored to competitors.
- The sales in the third module of the page (40 / 35 / 12 / 13 / 11 / 10) are illustrative placeholders, not real data.
- The joint model in result 6 rests on assumed demand parameters (elasticity, visitor numbers, utility offsets for sets and the sealed box). Its numbers show the mechanism, not this seller's optimum. Its demand parameters are also known to the seller, so waiting to open boxes has no learning value; in practice, keeping boxes sealed until it is clear which figure is hot could be worth more than the simulation shows.
- Opened boxes yield independent random figures. Real sealed sets often contain one of each regular figure, which would reduce the stock imbalance.
- The fluid plan is solved by shadow-price descent on a grid of opening amounts, so the planned optimum is approximate to the grid step.
- Competing sellers, official restocks and secondary-market premiums are not modelled, and all would change the demand curve. The model assumes genuine products; counterfeits have lower costs and would change the model.

## Material and methods

- Demand: exponential demand D = a·e^(−bP), with revenue-maximising price P* = 1/b.
- Demand across figures: multinomial logit, share sᵢ = exp(αᵢ − βPᵢ) / (1 + Σⱼ exp(αⱼ − βPⱼ)) [1,2]. αᵢ is backed out from sales and current prices. Independent sellers: Pᵢ − c = 1/(β(1−sᵢ)), solved by damped iteration. Single seller: Pⱼ − cⱼ = 1/βⱼ + Π, where Π solves Π = Σⱼ exp(αⱼ − βⱼcⱼ − 1 − βⱼΠ)/βⱼ (Newton's method); with a common β this is a uniform markup.
- Inventory pricing: a rolling clearance heuristic that resets the price each day from remaining stock and remaining days.
- Joint opening and pricing: fluid approximation over 8 stocks (6 regular figures, hidden figure, sealed boxes) and 10 options, with option-specific βⱼ = ε / observed priceⱼ. For a given number of boxes opened, the shadow prices μ of the stocks are found by projected descent on the dual; given μ, the prices are the single-seller optimum with effective cost salvage + μ for every unit an option uses. The number of boxes to open is chosen on a grid. The simulation uses Poisson daily visitors, logit choice among options in stock, and four policies on common random numbers.
- Profit: profit per box = revenue × (1 − platform fee) − shipping − 129 × purchase discount.
- Price test: b = ln(D₀/D₁) / (P₁ − P₀); horizon profit (P − c)·min(D(P)·T, Q); optimal price = min(max(c + 1/b, P₀ + ln(D₀T/Q)/b), ceiling). The estimate is rejected when demand does not fall with price.

The interactive page is a single static HTML file with no external dependencies; charts are inline SVG. To host it, enable GitHub Pages in the repository settings with the main branch and the /docs folder.

## References

1. Train K. Discrete Choice Methods with Simulation. Cambridge University Press, 2009.
2. Berry S. Estimating discrete-choice models of product differentiation. RAND Journal of Economics, 1994.
3. Gallego G, van Ryzin G. Optimal dynamic pricing of inventories with stochastic demand over finite horizons. Management Science, 1994.
4. Gallego G, van Ryzin G. A multiproduct dynamic pricing problem and its applications to network yield management. Operations Research, 1997.
5. Gallego G, Wang R. Multiproduct price optimization and competition under the nested logit model with product-differentiated price sensitivities. Operations Research, 2014.
