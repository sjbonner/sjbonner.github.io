---
id: 102
title: Simulating zero-truncated Poisson
date: 2014-12-14T19:25:35+00:00
author: Simon Bonner
layout: post
guid: http://simon.bonners.ca/bonner-lab/wpblog/?p=102
permalink: /?p=102
categories:
  - Uncategorized
---
I have been working on simulating data from Bob Dorazio&#8217;s version of the Royle N-mixture model for repeated count data. This version of the model assumes that the abundance at site<img src='http://s0.wp.com/latex.php?latex=+N_i+&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt=' N_i ' title=' N_i ' class='latex' /> is generated from a Hurdle model such that<img src='http://s0.wp.com/latex.php?latex=+P%28N_i%3D0%29%3D%281-%5Cpsi%29+&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt=' P(N_i=0)=(1-\psi) ' title=' P(N_i=0)=(1-\psi) ' class='latex' /> and<img src='http://s0.wp.com/latex.php?latex=+P%28N_i%3Dn%29+%3D+%5Cpsi+e%5E%7B-%5Clambda%7D+%5Clambda%5E%7Bn%7D%2F%28n%21+%281-e%5E%7B-%5Clambda%7D%29+&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt=' P(N_i=n) = \psi e^{-\lambda} \lambda^{n}/(n! (1-e^{-\lambda}) ' title=' P(N_i=n) = \psi e^{-\lambda} \lambda^{n}/(n! (1-e^{-\lambda}) ' class='latex' /> ,<img src='http://s0.wp.com/latex.php?latex=+n%3D1%2C2%2C3%2C+&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt=' n=1,2,3, ' title=' n=1,2,3, ' class='latex' /> . This can be interpreted as specifying an occupancy rate of<img src='http://s0.wp.com/latex.php?latex=+%5Cpsi+&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt=' \psi ' title=' \psi ' class='latex' /> with abundance given by a zero-truncated Poisson with mean<img src='http://s0.wp.com/latex.php?latex=+%5Clambda+&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt=' \lambda ' title=' \lambda ' class='latex' /> at the occupied sites.

I was originally generating the zero-truncated Poisson values using a while loop to simulate from a Poisson with mean<img src='http://s0.wp.com/latex.php?latex=+%5Clambda+&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt=' \lambda ' title=' \lambda ' class='latex' /> until a non-zero value was generated. This is inefficient and runs into problems if<img src='http://s0.wp.com/latex.php?latex=+%5Clambda+&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt=' \lambda ' title=' \lambda ' class='latex' /> is sufficiently small that the probability of a non-zero value is below computer precision.

To remedy this I found [this nice solution from Peter Daalgard](https://stat.ethz.ch/pipermail/r-help/2005-May/070683.html) in response to a question from Spencer Graves on the R-help list. This solution essentially uses the inverse probability transform but truncates the CDF so that the values generated are guaranteed to be greater than 1. The same could be done for any distribution (that has a distribution function implemented in R). My code below implements this trick but adds a tolerance that avoids problems if the value of lambda is too small.

<a href="http://www.codecogs.com/eqnedit.php?latex=P(N_i=n)&space;=&space;\psi&space;\frac{e^{-\lambda}\lambda^{n}}{n!&space;(1-e^{-\lambda})},\quad&space;n=1,2,3,..." target="_blank"><img src="http://latex.codecogs.com/gif.latex?P(N_i=n)&space;=&space;\psi&space;\frac{e^{-\lambda}\lambda^{n}}{n!&space;(1-e^{-\lambda})},\quad&space;n=1,2,3,..." title="P(N_i=n) = \psi \frac{e^{-\lambda}\lambda^{n}}{n! (1-e^{-\lambda})},\quad n=1,2,3,..." /></a>

<pre class="brush: r; title: ; notranslate" title="">rtpois &lt;- function(n, lambda,tol=1e-10){
## Simulate from zero-truncated Poisson

## Initialize output
x &lt;- rep(NA,n)

## Identify lambda values below tolerance
low &lt;- which(lambda &lt; tol)
nlow &lt;- length(low)

if(nlow &gt; 0){
x[low] &lt;- 1

if(nlow &lt; n)
x[-low] &lt;- qpois(runif(n-nlow, dpois(0, lambda[-low]), 1), lambda[-low])
}
else
x &lt;- qpois(runif(n-nlow, dpois(0, lambda), 1), lambda)

return(x)
}
</pre>