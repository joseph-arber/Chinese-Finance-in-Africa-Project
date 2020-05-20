
# Do Chinese Loans facilitate a worsening in good governance metrics at the state level?


Paper: https://www.researchgate.net/publication/338580894_A_Multi-Method_Analysis_of_the_impact_of_Chinese_non-concessional_finance_on_state-level_governance_in_Africa



Over the past decade, Chinese financial flows to Africa have increased considerably. As part of these flows, China has distributed non-concessional loans totaling $300bn (USD) across the continent. The scale of this financing has encouraged academics and policymakers alike to question the motivations behind Chinese lending, and more specifically whether the distribution of non-concessional finance reflects a similar set of conditions to the institutionalized multilateral-lending of the IMF and the World Bank. Simultaneously, attention has been directed to the protracted impact of Chinese lending on recipient states.

### Strategy:

Using a manually coded dataset with 1000 observations, I measured the impact of Chinese non-concessional loans on corruption and governance standards across 35 African states between 2006 and 2016. 

After fitting several **Panel Linear Models (PLMs)**, to analyse the non-linear relationship i programmed several **Generalized Additive Boosted Models (GAMs)** in R. 

https://cran.r-project.org/web/packages/gam/gam.pdf
https://cran.r-project.org/web/packages/plm/plm.pdf

I feature-engineered several variables that captured:

1. Whether a state had received both multilateral loans and Chinese loans in a given year
2. A good governance feature that captured the weighted average of a state's corruption level, regulatory quality and respect for property rights
3. The volume of PPPs (public-private partnerships) infrastructure contracts within a recipient state.

### Insights:

Results from the regressions provided no evidence for an association between greater levels of Chinese lending and worsening governance metrics. Albeit, after introducing a new variable for whether states had received Chinese loans in a given year, the models indicated that the receipt of Chinese loans is associated with a decline in good governance. In contrast the effect of receiving IMF loans is far less impactful on governance metrics. Overall, the empirical results produced by the panel and GAM regressions suggested that the relationship was far more complex than first assumed.
