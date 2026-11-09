# Modeling_BlindBox: demand estimation and pricing for a blind box series

## Goal

Model demand and optimal pricing for a blind box series, and use the model to answer four questions: why a hot figure is priced the way it is, how prices should move when stock piles up, roughly how much profit the seller makes, and how a seller could find a better price.

This is a modeling project: demand estimation, discrete choice and revenue optimisation under inventory constraints. It uses real public prices but no real sales data, so it demonstrates the method and a decision tool rather than making empirical claims about this seller.

[Interactive page](https://arcadianlyric.github.io/Modeling_BlindBox/). 

## Results

The data are public prices from one Taobao seller: 6 regular figures at 142 / 136 / 88 / 89 / 84 / 84, 1 hidden figure at 335, a single blind box at 108, a confirmed-figure set (all 6 regular figures) at 618 and a sealed, unopened set at 732. Official prices are 129 per box and 774 per set, and the hidden figure probability is 1/72.

**1. Prices are internally consistent.** The 6 regular figures average 103.8. Including the hidden figure, the expected revenue from opening one box is about 107, almost exactly the 108 box price, so the seller appears to anchor on "expected value of confirmed figures ≈ box price". The confirmed set at 618 is only 5 below buying the 6 figures separately (623), so it carries almost no discount. The sealed set at 732 is 114 above the confirmed set; the hidden-figure chance explains only about 28 of that, leaving roughly 86 as a premium for sealed packaging, the outer case and the unboxing experience.

**2. What 142 implies about price sensitivity.** Under exponential demand D = a·e^(−bP), the revenue-maximising price is P* = 1/b (or c + 1/b with unit cost c). A price of 142 implies b ≈ 0.0070, while the cold figures at 84 to 89 imply b ≈ 0.0115, so buyers of the hot figure are about 60% as price sensitive as buyers of the cold ones. This relies on two assumptions, that the seller prices optimally and that the item sells, so it is circular: it explains a price but cannot test whether the price is right. The "recently hot" label suggests 142 may still be below the optimum; result 5 shows how to check.

**3. Inventory dynamics.** With the price set to P = ln(aT/Q)/b (sell out exactly by the end of the horizon), a hot figure with tight stock is priced above 1/b and rises as stock falls, while overstocked cold figures are priced below 1/b and drift down slowly. If the hot figures sell out first, the 618 bundle is delisted, cold figures can only be sold singly, and the overstock risk grows. The equilibrium looks like "hot figures up, cold figures down slowly, fewer boxes opened" rather than a simple "hot up, cold down".

**4. Seller profit.** Cost cannot be uniquely identified from prices, so only a range is possible. With the default parameters (purchase discount 75%, platform fee 1%, shipping 4 per order, cold sell-through 85%, unsold stock salvaged at 50% of list), profit per box is about:

| Channel | Profit per box (CNY) |
|---|---|
| Sell blind box at 108 | 6.2 |
| Open box, sell confirmed figures | 1.0 |
| Confirmed set at 618 | 4.6 |
| Sealed set at 732 | 23.4 |

Selling blind boxes at 108 breaks even at a purchase discount of about 80%. Opening boxes and selling confirmed figures earns almost nothing beyond the purchase discount, and is worse than selling the box when cold figures do not sell through. The sealed set is the highest-margin channel.

**5. Finding a better price needs a price test.** A second price point breaks the circularity of result 2: with daily sales D₀ at P₀ and D₁ at a test price P₁, the slope is b = ln(D₀/D₁) / (P₁ − P₀), the profit-maximising price is c + 1/b, and tight stock adds a floor at the clearance price P₀ + ln(D₀T/Q)/b. Three practical lessons came out of building this:

- When a test barely moves sales, b is small and 1/b explodes. A test from 142 to 225 with sales falling only from 5.0 to 4.7 implies an optimum near 1400, which is extrapolation, not a recommendation. The page caps the price at a willingness-to-pay ceiling and flags any optimum more than 30% beyond the tested range.
- If sales rise with price, b cannot be estimated. In real data this points to noise, a promotion, a stock-out or a demand shift between the two periods, not to "higher prices sell better".
- The trustworthy procedure is iterative: move the price part of the way towards the model optimum, test again, and re-estimate.

## Discussion

**Price endogeneity is the central identification problem.** The seller sets prices in response to demand, so popular figures have both a high αᵢ and a high price. Regressing shares on observed prices across figures therefore confounds popularity with price and biases β towards zero, which makes demand look less price sensitive than it is [2]. Two ways out are instruments for price, as in the discrete choice literature, or randomised price variation. Result 5 takes the second route: a price test is an A/B experiment on price, and the same tools apply, including enough days per arm for the sales difference to exceed day-to-day noise and checks that nothing else changed between arms.

- The profit table depends on the assumed purchase discount. The real value should be recovered by regressing on per-figure sales: estimate β and each αᵢ, then back out cost from c = Pᵢ − 1/(β(1−sᵢ)). The six estimates should roughly agree; large disagreement means "priced optimally" fails and prices may instead be anchored to competitors.
- The sales in the third module of the page (40 / 35 / 12 / 13 / 11 / 10) are illustrative placeholders, not real data.
- The third module iterates each figure's first-order condition independently. Under logit, strict multi-product monopoly pricing is a uniform markup; the gap between the two has not been quantified.
- The inventory module is a deterministic pricing rule with Poisson noise, not the optimal policy from dynamic programming.
- Competing sellers, official restocks and secondary-market premiums are not modelled, and all would change the demand curve. The model assumes genuine products; counterfeits have lower costs and would change the model.

## Material and methods

- Demand: exponential demand D = a·e^(−bP), with revenue-maximising price P* = 1/b.
- Competition across figures: multinomial logit, share sᵢ = exp(αᵢ − βPᵢ) / (1 + Σⱼ exp(αⱼ − βPⱼ)), optimal markup Pᵢ − c = 1/(β(1−sᵢ)) [1,2]. αᵢ is backed out from sales and current prices, then the model price is found by iteration.
- Inventory pricing: the price is recomputed each day from remaining stock Q and remaining days T, following the finite-horizon dynamic pricing literature [3].
- Profit: profit per box = revenue × (1 − platform fee) − shipping − 129 × purchase discount.
- Price test: b = ln(D₀/D₁) / (P₁ − P₀); optimal price = min(max(c + 1/b, P₀ + ln(D₀T/Q)/b), ceiling). The estimate is rejected when demand does not fall with price.

The interactive page is a single static HTML file with no external dependencies; charts are inline SVG. To host it, enable GitHub Pages in the repository settings with the main branch and the /docs folder.

## References

1. Train K. Discrete Choice Methods with Simulation. Cambridge University Press, 2009.
2. Berry S. Estimating discrete-choice models of product differentiation. RAND Journal of Economics, 1994.
3. Gallego G, van Ryzin G. Optimal dynamic pricing of inventories with stochastic demand over finite horizons. Management Science, 1994.
