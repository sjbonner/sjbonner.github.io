---
id: 16
title: A Weakly Informative Default Prior Distribution for Logistic and Other Regression Models
date: 2014-10-15T20:10:40+00:00
author: Simon Bonner
layout: revision
guid: http://simon.bonners.ca/bonner-lab/wpblog/?p=16
permalink: /?p=16
---
We continued our discussion of prior distributions for mark-recapture models including covariates in lab today by looking at the paper <a href="http://projecteuclid.org/euclid.aoas/1231424214" target="_blank">A Weakly Informative Default Prior Distribution for Logistic and Other Regression Models</a> published by Andrew Gelman et al in the Annals of Applied Statistics in 2008. Surprise, surprise, the paper proposes a new, weakly informative prior distribution that the authors argue can be used as a default when fitting logistic regression models. Specifically, the authors recommend using independent, scaled Cauchy priors with mean 0 and scale 10 for the intercept, 2.5 all predictors where it is assumed that continuous predictors are standardized to have mean 0 and standard deviation .5 and binary predictors are standardized to have mean 0 and a difference of 1 between the upper and lower values.

Gelman&#8217;s argument for this prior is that we should expect that, in most cases, a difference of one standard deviation a predictor should change the linear predictor by at most 5 units &#8220;which would move the probability from 0.01 to 0.50 or from 0.50 to 0.99&#8221;. The paper also provides a very nice extension of the Iteratively Weighted Least Squares algorithm that gives a fast approximation of the posterior without the need for MCMC or other sampling methods.

One thing that did perplex me is the appliation to the biassay data from Racine et al. (1986). The plot in Figure 3 does not convince me that the new prior is doing any better. The fitted curve from glm just looks like a better fit! Also, it seems that the posterior is very sensitive to the prior posterior mean is almost halved. I think that I am thrown off by the size of the data &#8212; n=20 &#8212; and it would have been helpful to me to see credible envelopes for the two curves to judge the magnitude of uncertainty. I&#8217;d be more convinced if I saw that the fitted curve from glm lie inside the envelope from bayesglm and vice versa.

&nbsp;