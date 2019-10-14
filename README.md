# UCL_IPP-Dissertation
MSc dissertation code

Appendix F: Example of code used to generate preliminary results 

###Set up script and working directory###
rm(list = ls())
setwd("~/PUBL0055")

###Install folowing packages if neccessary###
install.packages("plm")
install.packages("tidyverse")
install.packages("ggmap")
install.packages("mcv")
install.packages('dplyr'

###Load in necessary packages####
library(texreg)
library(foreign)
library(lmtest)
library(plm)
library(boot)
library(car)
##For data cleaning, wrangling and easier manipulation:
library(mice)
library(tidyverse)
library(dplyr)
library(ggplot2)

########## Load in Data ##########
library("readxl")
Dataset2_ <- read_excel("~/Desktop/Dissertation/Dataset2 .xlsx")
Panel1<-Dataset2_

######### Wrangling and Data Management ##########
ls(Panel1)
#Take a look at the structure of the dataset, followed by a summary...
str(Panel1)
summary(Panel1)
dim(Panel1)
head(Panel1$Loans_Chinese)


#Coerce variables to correct types:
str(Panel1)
Panel1$IMF_loans <- as.integer(Panel1$IMF_loans)
Panel1$Received_Chinese_loans <- as.integer(Panel1$Received_Chinese_loans)
str(Panel1)

##Check for missing values
summary(Panel1[c('Govt_Exp','Inflation_annual','GDP_pc','GDP')])
#Count the missing values
sum(is.na(Panel1))
###Alternative to MICE is to replace missing values with average values
Panel1$Inflation_annual[is.na(Panel1$Inflation_annual)] <- mean(Panel1$Inflation_annual, na.rm = TRUE)
Panel1$GDP_pc[is.na(Panel1$GDP_pc)] <- mean(Panel1$GDP_pc, na.rm = TRUE)
Panel1$GDP[is.na(Panel1$GDP)] <- mean(Panel1$GDP, na.rm = TRUE)
Panel1$Govt_Exp[is.na(Panel1$Govt_Exp)] <- mean(Panel1$Govt_Exp, na.rm = TRUE)
Panel1$Central_govt_debt[is.na(Panel1$Central_govt_debt)] <- mean(Panel1$Central_govt_debt, na.rm = TRUE)
Panel1$Loans_WB[is.na(Panel1$Loans_WB)] <- mean(Panel1$Loans_WB, na.rm = TRUE)
sum(is.na(Panel1))
head(Panel1)

#Transformations
LogGDP_PC <- log(Panel1$GDP_pc)
LogGDP <- log(Panel1$GDP)
LogODA <- log(Panel1$Net_ODA)
LogGovExp <- log(Panel1$Govt_Exp)
LogWB <- log(Panel1$Loans_WB)


####GAM Regressions####
library(mgcv)
M1 <-gam(Good_gov ~ s(Inflation_annual, k = 10) 
         + Received_Chinese_loans
         + lag(Loans_Chinese)
         + LogGovExp
         + LogODA
         + LogGDP
         + Central_govt_debt
         + Natural_resource
         + Functioning_Gov
         + Jud_indep
         + Freedom_House
         + Media_censorship
         + Internet_censorship,
         data = Panel1,
         method = "REML", 
         family = "gaussian",
         select = TRUE)


#Model checking
summary.gam(M1)
gam.check(M1)
#Model fit
waldtest2 = wald.test(b = coef(M1), Sigma = vcov(M1), Terms = 1:13)
waldtest2

library(aod)
anova(M1, M2, test = "Chisq")

M2 <-gam(Good_gov ~ s(Urban_pop, k = 10) 
         + IMF_loans
         + lag(LogWB)
         + LogGovExp
         + LogODA
         + LogGDP
         + Central_govt_debt
         + Natural_resource
         + Jud_indep
         + Functioning_Gov 
         + Freedom_House
         + Media_censorship
         + Internet_censorship,
         data = Panel1,
         method = "REML", 
         family = "gaussian",
         select = TRUE)

plot(M2, shade = TRUE, shade.col = "lightblue")



M3 <-gam(Control_of_corruption ~ s(Inflation_annual, k = 10) 
         + Received_Chinese_loans
         + LogGDP 
         + LogGovExp
         + s(Central_govt_debt) 
         + Natural_resource
         + Jud_indep
         + Functioning_Gov 
         + Freedom_House
         + Media_censorship,
         data = Panel1,
         method = "REML", 
         family = "gaussian",
         select = TRUE)

M3.1 <-gam(Control_of_corruption ~ s(Inflation_annual, k = 10) 
           + IMF_loans
           + LogWB
           + LogGDP 
           + LogGovExp
           + s(Central_govt_debt) 
           + Natural_resource
           + Jud_indep
           + Functioning_Gov 
           + Freedom_House
           + Media_censorship,
           data = Panel1,
           method = "REML", 
           family = "gaussian",
           select = TRUE)
screenreg(list(M3, M3.1))
coef(M3)

M4 <-gam(Control_of_corruption ~ s(Inflation_annual, k = 10) 
         + Chinese_contracts
         + LogGDP 
         + LogGovExp
         + s(Central_govt_debt) 
         + Natural_resource
         + LogODA
         + Jud_indep
         + Functioning_Gov 
         + Freedom_House
         + Media_censorship
         + LogWB,
         data = Panel1,
         method = "REML", 
         family = "gaussian",
         select = TRUE)
screenreg(M4)












M5 <-gam(Reg_qual ~ s(Inflation_annual, k = 10) 
         + Chinese_contracts
         + LogGDP 
         + LogGovExp
         + s(Central_govt_debt) 
         + Natural_resource
         + LogODA
         + Jud_indep
         + Functioning_Gov 
         + Freedom_House
         + Media_censorship
         + LogWB,
         data = Panel1,
         method = "REML", 
         family = "gaussian",
         select = TRUE)

M5 <-gam(Voice ~ s(Inflation_annual, k = 10) 
         + Chinese_contracts
         + LogGDP 
         + LogGovExp
         + s(Central_govt_debt) 
         + Natural_resource
         + LogODA
         + Jud_indep
         + Functioning_Gov 
         + Freedom_House
         + Media_censorship,
         data = Panel1,
         method = "REML", 
         family = "gaussian",
         select = TRUE)
screenreg(M5)

screenreg(list(M3, M3.1,M4,M5))
htmlreg(list(M3,M3.1),file = "Chinese_corruption1.2.doc")

#Visualization 
ggplot(Panel1, aes(x = Contracts2, y = Control_of_corruption)) + 
  geom_point(
    mapping = aes(x = Contracts2, y = Control_of_corruption, color = Received_Chinese_loans)) +
  scale_x_log10() +
  stat_smooth(method = "gam", formula = y ~ s(x,k=10))


####(PLM) Fixed Effects Models####
Fixed_effects1 <- plm (Good_gov ~ Received_Chinese_loans
                       + Loans2
                       + LogGDP
                       + LogODA
                       + LogGovExp
                       + Inflation_annual
                       + Central_govt_debt
                       + Functioning_Gov
                       + Jud_indep
                       + Natural_resource
                       + Urban_pop
                       + Internet_censorship,
                       data = Panel1,
                       index = c("Country", "Year"),
                       effect = "individual")

summary(Fixed_effects1)
screenreg(Fixed_effects1)

Fixed_effects2 <- plm (Good_gov ~ IMF_loans
                       + lag(LogWB)
                       + lag(LogGDP)
                       + LogODA
                       + LogGovExp
                       + Inflation_annual
                       + Central_govt_debt
                       + Functioning_Gov
                       + Jud_indep
                       + Natural_resource
                       + Urban_pop
                       + Internet_censorship,
                       data = Panel1,
                       index = c("Country", "Year"),
                       effect = "individual")

Fixed_effects3 <- plm (Control_of_corruption ~ Received_Chinese_loans
                       + lag(Loans2)
                       + lag(LogGDP)
                       + LogGovExp
                       + Central_govt_debt
                       + Functioning_Gov
                       + Voice
                       + Jud_indep
                       + Freedom_House
                       + Natural_resource
                       + Urban_pop
                       + Internet_censorship,
                       data = Panel1,
                       index = c("Country", "Year"),
                       effect = "time")

Fixed_effects4 <- plm (Control_of_corruption ~ IMF_loans
                       + lag(LogWB)
                       + lag(LogGDP)
                       + LogGovExp
                       + Central_govt_debt
                       + Functioning_Gov
                       + Voice
                       + Jud_indep
                       + Freedom_House
                       + Natural_resource
                       + Urban_pop
                       + Internet_censorship,
                       data = Panel1,
                       index = c("Country", "Year"),
                       effect = "individual")













Fixed_effects5 <- plm (Control_of_corruption ~ Chinese_contracts
                       + lag(LogWB)
                       + lag(LogGDP)
                       + LogGovExp
                       + Central_govt_debt
                       + Functioning_Gov
                       + Voice
                       + Jud_indep
                       + Freedom_House
                       + Natural_resource
                       + Urban_pop
                       + Internet_censorship,
                       data = Panel1,
                       index = c("Country", "Year"),
                       effect = "time")
Fixed_effects6 <- plm (Reg_qual ~ Contracts3
                       + lag(LogWB)
                       + lag(LogGDP)
                       + LogGovExp
                       + Central_govt_debt
                       + Functioning_Gov
                       + Voice
                       + Jud_indep
                       + Freedom_House
                       + Natural_resource
                       + Urban_pop
                       + Internet_censorship,
                       data = Panel1,
                       index = c("Country", "Year"),
                       effect = "individual")

screenreg(list(Fixed_effects1, Fixed_effects2, Fixed_effects3, Fixed_effects4, Fixed_effects5, Fixed_effects6))
htmlreg(list(Fixed_effects1, Fixed_effects2, Fixed_effects3, Fixed_effects4, Fixed_effects5, Fixed_effects6),file = "Appendix_Fixed_effects.doc")

#Test for the prescence of country effects
plmtest(Fixed_effects1, effect ="individual")

#Test for serial correlation in the error term
pbgtest(Fixed_effects1)
#Correction for serial correlation 
FE_1_hac <- coeftest(
  Fixed_effects1, 
  vcov = vcovHC(Fixed_effects1, method = "arellano", type = "HC3")
)

#Test for cross-sectional dependence
pcdtest(Fixed_effects1)
#The cross-sectional and serial correlation (SCC) method by Driscoll and Kraay for obtaining heteroskedasticity and autocorrelation consistent errors that are also robust to cross-sectional dependence
FE_scc <- coeftest(
  Fixed_effects1,
  vcov = vcovSCC(Fixed_effects1, type="HC3", cluster = "group")
)
FE_scc

screenreg(
  list(Fixed_effects1, FE_1_hac, FE_scc),
  custom.model.names = c("Fixed Effects", "Fixed Effects (HAC)","Fixed Effects (SCC)")
