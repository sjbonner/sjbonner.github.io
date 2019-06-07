---
id: 89
title: First Attempts with the parallel Package
date: 2014-11-26T02:27:31+00:00
author: Simon Bonner
layout: revision
guid: http://simon.bonners.ca/bonner-lab/wpblog/?p=89
permalink: /?p=89
---
The last time I worked with parallel processing in R I used the package Rmpi. This ws very flexible, but took a lot of setup to handle passing jobs, code, and data between the master and the slave. I have used the package snow for some simple projects, but I have new need for parallel computing and recently found out about the parallel package (something I should have known about earlier). This is my first attempt at using the mcparallel function from the parallel package.

As a simple test, I have used mcparallel to implement multiple chains for a Metropolis-Hastings sampler of the mean parameter in a multivariate log-normal distribution. That is, to sample from the posterior distribution of<img src='http://s0.wp.com/latex.php?latex=+%5Cmu+&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt=' \mu ' title=' \mu ' class='latex' /> given a an iid sample of random p-vectors<img src='http://s0.wp.com/latex.php?latex=+X_1%2C%5Cldots%2CX_n+&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt=' X_1,\ldots,X_n ' title=' X_1,\ldots,X_n ' class='latex' /> such that<img src='http://s0.wp.com/latex.php?latex=+%5Clog%28X_i%29+%5Csim+N_p%28%5Cmu%2C%5CSigma%29+&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt=' \log(X_i) \sim N_p(\mu,\Sigma) ' title=' \log(X_i) \sim N_p(\mu,\Sigma) ' class='latex' /> . In the simulation I have set<img src='http://s0.wp.com/latex.php?latex=+p%3D2+&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt=' p=2 ' title=' p=2 ' class='latex' /> ,<img src='http://s0.wp.com/latex.php?latex=+%5Cmu%3D%281%2C1%29%5ET+&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt=' \mu=(1,1)^T ' title=' \mu=(1,1)^T ' class='latex' /> , and<img src='http://s0.wp.com/latex.php?latex=+%5CSigma%3D.5+%2B+.5+I_2&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt=' \Sigma=.5 + .5 I_2' title=' \Sigma=.5 + .5 I_2' class='latex' /> .

The sampler I have implemented generates proposals using a symmetric multivariate normal proposal distribution with variance Sigma.prop. I have manually tuned the proposal variance to achieve acceptance rates just above .5 and visually pleasing traceplots.

The use of the parallel package comes in three steps:

<p style="padding-left: 30px;">
  1. Initialize the random number stream:
</p>

<p style="padding-left: 30px;">
  This part of the code selects and initializes the L&#8217;Ecuyer random number generator so that each chain uses a different RNG.
</p>

<pre class="brush: plain; title: ; notranslate" title="">## Initialize random number generator to obtain different chains from
## each parallel run
RNGkind(&quot;L'Ecuyer-CMRG&quot;)
set.seed(7777)
mc.reset.stream()

</pre>

<p style="padding-left: 30px;">
  2. Start the parallel processes:
</p>

<p style="padding-left: 30px;">
  The second part of the code starts the parallels processes running. I have used the lapply function here so that the number of parallel processes is not hardwired and can be changed easily.
</p>

<pre class="brush: r; title: ; notranslate" title="">## Start parallel processes
niter &lt;- 1000
Sigma.prop &lt;- .03*Sigma&lt;/p&gt;
MHprocesses &lt;- lapply(1:nchains,function(i){
mcparallel(MHalg(mu.init[i,],x,Sigma,n.iter,Sigma.prop))
})
</pre>

<p style="padding-left: 30px;">
  3. Collect results:
</p>

<p style="padding-left: 30px;">
  The final chunk uses the mccollect function to wait for the parallel jobs to finish and collect the output.
</p>

<pre class="brush: r; title: ; notranslate" title="">## Wait and collect output
MHout &lt;- mccollect(MHprocesses)
</pre>

In this case, the chains could have been run in parallel using the mcapply function. This would avoid the need to set the RNG and to collect results manually. Both functions use forks and share memory until objects are modified, so they should not differ in speed or memory use. However, mcparallel seems to be more

Here is the full code. Any comments are welcome!

<pre class="brush: r; title: ; notranslate" title="">############################################################
#
# parallel_1.R
#
# First test with the parallel package.
#
# In this example I run parallel chains for a Metroposlis-
# Hastings sampler for the multivariate log-normal
# distribution using the mcparallel() function to
# parallelize the code.
#
############################################################

##### Preliminaries #####
## Load packages
library(parallel)
library(mvtnorm)

## Define functions
dmvlnorm &lt;- function(x,mu,Sigma,log=FALSE){
 ## Multivariate log-normal density
 dmvnorm(log(x),mu,Sigma,log=log)
}

rmvlnorm &lt;- function(n,mu,Sigma){
 ## Multivariate log-normal random generator

## Generate normal random variates
 logx &lt;- rmvnorm(n,mu,Sigma)

## Exponentiate to get log normal random variates
 x &lt;- exp(logx)

return(x)
}

MHstep &lt;- function(mu.curr,x,Sigma,Sigma.prop){
 ## Metropolis-Hastings step

## Generate proposal
 mu.prop &lt;- mu.curr + rmvnorm(1,rep(0,length(mu.curr)),Sigma.prop)

## Compute Hastings ratio (on log scale)
 log.alpha &lt;- sum(dmvlnorm(x,mu.prop,Sigma,log=TRUE)) -
 sum(dmvlnorm(x,mu.curr,Sigma,log=TRUE))

## Accept or reject proposal
 if(-rexp(1) &lt; log.alpha)
 return(mu.prop)
 else
 return(mu.curr)
}

MHalg &lt;- function(mu,x,Sigma,niter,Sigma.prop){
 ## Initialize storage
 mu.store &lt;- matrix(nrow=niter + 1,ncol=length(mu))
 mu.store[1,] &lt;- mu

## Initialize acceptance monitor
 accept &lt;- rep(0,niter)

## Run MH algorithm
 for(k in 1:niter){
 ## Perform one MH step
 mu &lt;- MHstep(mu,x,Sigma,Sigma.prop)

## Store values
 mu.store[k+1,] &lt;- mu

## Track acceptance
 if(all(mu.store[k+1,] != mu.store[k,]))
 accept[k] &lt;- 1
 }

 ## Return sampled values
 return(list(mu=mu.store,accept=accept))
}

##### Run Samplers #####

## Generate some random data
mu &lt;- c(1,1)
Sigma &lt;- .5 + .5 * diag(2)

n &lt;- 25

x &lt;- rmvlnorm(n,mu,Sigma)

## Parameters
n &lt;- 25
mu &lt;- c(1,.5)
Sigma &lt;- .5 + .5*diag(2)

## Contour plot of true density
xgrid &lt;- seq(0,30,length=100)
Z &lt;- sapply(xgrid,function(x1){
 dmvlnorm(cbind(x1,xgrid),mu,Sigma)
})

contour(xgrid,xgrid,Z)

## Generate data
x &lt;- rmvlnorm(n,mu,Sigma)

## Generate initial values
nchains &lt;- 4
mu.init &lt;- rmvnorm(nchains,apply(log(x),2,mean),2*Sigma)

## Initialize random number generator to obtain different chains from
## each parallel run
RNGkind(&quot;L'Ecuyer-CMRG&quot;)
set.seed(7777)
mc.reset.stream()

## Start parallel processes
niter &lt;- 1000
Sigma.prop &lt;- .03*Sigma

MHprocesses &lt;- lapply(1:nchains,function(i){
 mcparallel(MHalg(mu.init[i,],x,Sigma,n.iter,Sigma.prop))
})

## Wait and collect output
MHout &lt;- mccollect(MHprocesses)

##### Examine Output #####

## One-dimensional traceplots
col &lt;- rainbow(nchains)
par(mfrow=c(2,1))

trace1 &lt;- sapply(1:nchains,function(i) MHout[[i]]$mu[,1])
matplot(trace1,type=&quot;l&quot;,lty=1,col=col)
abline(h=mu[1],lty=2,col=&quot;grey&quot;)

trace2 &lt;- sapply(1:nchains,function(i) MHout[[i]]$mu[,2])
matplot(trace2,type=&quot;l&quot;,lty=1,col=col)
abline(h=mu[2],lty=2,col=&quot;grey&quot;)

## Two-dimensional traceplot
par(mfrow=c(1,1))
lim &lt;- range(sapply(MHout,function(out) range(out$mu)))
plot(NA,NA,xlim=lim,ylim=lim)
for(i in 1:nchains)
 lines(MHout[[i]]$mu[,1],MHout[[i]]$mu[,2],col=col[i])

abline(v=mu[1],lty=2,col=&quot;grey&quot;)
abline(h=mu[2],lty=2,col=&quot;grey&quot;)

## Acceptance rates
sapply(1:nchains,function(i) mean(MHout[[i]]$accept))

## Density plots
par(mfrow=c(2,1))
dens1 &lt;- lapply(1:nchains,function(i){
 density(MHout[[i]]$mu[,1],from=lim[1],to=lim[2])
})
ylim1 &lt;- c(0,max(sapply(1:nchains,function(i) max(dens1[[i]]$y))))

plot(NA,NA,xlim=lim,ylim=ylim1)
for(i in 1:nchains)
 lines(dens1[[i]],col=col[i])

abline(v=mu[1],lty=2,col=&quot;grey&quot;)

dens2 &lt;- lapply(1:nchains,function(i){
 density(MHout[[i]]$mu[,2],from=lim[1],to=lim[2])
})
ylim2 &lt;- c(0,max(sapply(1:nchains,function(i) max(dens2[[i]]$y))))

plot(NA,NA,xlim=lim,ylim=ylim2)
for(i in 1:nchains)
 lines(dens2[[i]],col=col[i])

abline(v=mu[2],lty=2,col=&quot;grey&quot;)

</pre>