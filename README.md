# The Impact of Chinese Financing in Africa


## Do Chinese Non-concessional loans facilitate a worsening in good governance metrics at the state level?


Paper: https://www.researchgate.net/publication/338580894_A_Multi-Method_Analysis_of_the_impact_of_Chinese_non-concessional_finance_on_state-level_governance_in_Africa



Over the past decade, Chinese financial flows to Africa have increased considerably. As part of these flows, China has distributed non-concessional loans totaling $300bn (USD) across the continent. The scale of this financing has encouraged academics and policymakers alike to question the motivations behind Chinese lending, and more specifically whether the distribution of non-concessional finance reflects a similar set of conditions to the institutionalized multilateral-lending of the IMF and the World Bank. Simultaneously, attention has been directed to the protracted impact of Chinese lending on recipient states. Have Chinese lending programmes resulted in worse good governance metrics? Or are loans distributed on the conditionality of the award of private infrastructure contracts to the Chinese state?

#### Exploratory Analysis:

To understand the impact of Chinese financing in Africa, data, taken from the China in Africa Research Initative was explored. This entailed conducting preliminary analysis on Chinese non-concessional loans. A few questions were investigated:

- What types of state recieves the most finance from China 
- Which countries have the worst governance ratings?

The findings from these questions were used to inform the modelling phases from this analysis. An example of some of the exploratory analysis conducted is shown below:

The first plot shows the **trends in good governance metrics** in selected African states over the specified time period. 

![Image of Countries](https://github.com/JUA96/Chinese-Finance-in-Africa-Project/blob/master/Screenshot%202020-06-28%2016.28.40.png)

The second plot shows the **relationship between good governance trends and the forecasted loans size from China**. This graph was particuarly useful in informing the feature selection for the models.

![Image of Countries](https://github.com/JUA96/Chinese-Finance-in-Africa-Project/blob/master/images/Screenshot%202020-06-28%2016.10.21.png)

### Strategy:

Using a manually coded dataset with 1000 observations, I measured the impact of Chinese non-concessional loans on corruption and governance standards across 35 African states between 2006 and 2016. 

After fitting several **Panel Linear Models (PLMs)**, to analyse the non-linear relationship i programmed several **Generalized Additive Boosted Models (GAMs)** in R. 

https://cran.r-project.org/web/packages/gam/gam.pdf
https://cran.r-project.org/web/packages/plm/plm.pdf

The following code chunk is an example of the coding workflow that was used produce the models: 

``
library(mgcv)
M1 <-gam(Good_gov ~ s(Inflation_annual, k = 10) + Received_Chinese_loans + lag(Loans_Chinese) + LogGovExp + LogODA + LogGDP + Central_govt_debt + Natural_resource  Internet_censorship, data = Panel1, method = "REML", family = "gaussian", select = TRUE)``

``#Model Check
summary.gam(M1)
gam.check(M1)``

``#Model fit
library(aod)
waldtest2 = wald.test(b = coef(M1), Sigma = vcov(M1), Terms = 1:13)``

I then feature-engineered several variables that captured:

1. Whether a state had received both multilateral loans and Chinese loans in a given year
2. A good governance feature that captured the weighted average of a state's corruption level, regulatory quality and respect for property rights
3. The volume of PPPs (public-private partnerships) infrastructure contracts within a recipient state.

### Data Acquisition:

I scraped data from a number of sources to compile a single dataset with 1000 observations. These sources included:

Datasets:
1. Official Chinese Financial Flows Dataset: AidData Research and Evaluation Unit. 2017. Geocoding Methodology, Version 2.0. Williamsburg, VA: AidData at William &
Mary. https://www.aiddata.org/publications/geocoding-methodology-version-2-0
2. CARI: China Africa Research Initiative at International Studies. http://www.sais-cari.org/data
3. V-Dem: Coppedge, Michael, John Gerring,. 2018. "V-Dem [Country-Year/Country-Date] Dataset v8". Varieties of Democracy (V-Dem) Project. https://doi.org/10.23696/vdemcy18
4. Quality of Governance Data: Teorell, Jan, Stefan Dahlberg, Sören Holmberg, Bo Rothstein, Natalia Alvarado Pachon and Richard Svensson. 2019. The Quality of Government Standard Dataset, version Jan19. University of Gothenburg: The Quality of Government Institute, http://www.qog.pol.gu.se doi:10.18157/qogstdjan19
5. WBI: World Bank Indicators; economy, governance and geography https://data.worldbank.org/indicator?tab=all
6. WGI: Worldwide Governance Indicators (www.govindicators.org)
7. IMF Lending and Debt Data:
https://www.imf.org/external/datamapper/DEBT1


### Data and Features:

Dependent Variables:
- Good-Governance (basket average of the following four variables)
- Corruption Level
- Regulatory Quality
- Rule of law
- Property Rights

Features:
- Chinese loans (amount USD)
- Chinese contracts (amount USD)
- Chinese loans recieved (dummy variable)
- IMF loans
- World Bank Loans

Controls:
- GDP
- Debt level
- Oil rent and natural resource volumes
- Internet and media censorship

![Image of Countries](https://github.com/JUA96/Chinese-Finance-in-Africa-Project/blob/master/images/img5.png)

### Insights:

Results from the regressions provided no evidence for an association between greater levels of Chinese lending and worsening governance metrics. Albeit, after introducing a new variable for whether states had received Chinese loans in a given year, the models indicated that the receipt of Chinese loans is associated with a decline in good governance. In contrast the effect of receiving IMF loans is far less impactful on governance metrics. Overall, the empirical results produced by the panel and GAM regressions suggested that the relationship was far more complex than first assumed.

**Relationship between volume of private Chinese contracts and state-level corruption**

`` ggplot(Panel1, aes(x = Chinese_contracts, y = Control_of_corruption)) + 
geom_point(mapping = aes(x = Chinese_contracts, y = Control_of_corruption, color = Received_Chinese_loans)) +
scale_x_log10() + stat_smooth(method = "gam", formula = y ~ s(x,k=10))``


![Image of Yaktocat](https://github.com/JUA96/Chinese-Finance-in-Africa-Project/blob/master/images/img4.png)
