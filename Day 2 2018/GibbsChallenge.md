---
title: "Advanced Biostatistics 2018 - Practical Exercises"
#author: "lisete"
#date: "3 de abril de 2018"
output:
  html_document:
     keep_md: true
---

x <- rmarkdown::render("file.RMD", run_pandoc = FALSE, clean = FALSE)
knit_meta <- attr(x, "knit_meta") 
rmarkdown::render(input = 'file.knit.md', knit_meta = knit_meta )


<style type="text/css"> body, td { font-size: 18px; } code.r{ font-size: 18px; } pre { font-size: 16px }  </style>

\newcommand{\bteta}{\boldsymbol{\theta}}

## Gibbs Sampling

At the end of this challenge, you should be able to:

(1) understand the Gibbs Sampler mechanism,

(2) how to run it for the Multinomial-Dirichlet distributions with known **x**,

(3) implement the main functions in $\texttt{R}$.


**Step 1.** Simulate the data - Multinomial Distribution

* Learn about [Multinomial ditribution] (https://en.wikipedia.org/wiki/Multinomial_distribution). For the three-dimensional case we have:
$${\bf X}=(X_1,X_2,X_3)\frown Multinomial(n;\theta_1,\theta_2,\theta_3)$$

where $X_i\in\{0,1,2,\ldots\}$\: and\: $\theta_1+\theta_2+\theta_3=1\quad (\theta_i>0)$
$$P[(X_1,X_2,X_3)=(x_1,x_2,x_3)\ |\ \theta_1,\theta_2,\theta_3]=\dfrac{n!}{x_1!x_2!x_3!}\ \theta_1^{\;x_1}\theta_2^{\;x_2}\theta_3^{\;x_3}$$

* Search for the $\texttt{R}$ function which generates multinomially distributed random number vectors and computes multinomial probabilities.


```r
?rmultinom
```

```
## starting httpd help server ... done
```

* Simulate 1 random vector, ${\bf x}=(x_{1},x_{2},x_{3})$, following a Multinomial distribution with parameters $n=1000$ and $\bteta=(\theta_1,\theta_2,\theta_3)=(0.2,0.3,0.5)$. Store the simulated data in an object named $\texttt{data}$.



```r
theta<-c(0.2,0.3,0.5)
data<-rmultinom(1,1000,theta)
data
```

* Calculate the probability of observing the vector (220,350,430), that is, $P[(X_1,X_2,X_3)=(220,350,430)\ |\ \bteta]$.


```r
dmultinom(c(220,350,430),1000,theta)
```

**Step 2.** Dirichlet Distribution - Prior / Posterior Distribution

* Learn about [Dirichlet ditribution] (https://en.wikipedia.org/wiki/Dirichlet_distribution). For the three-dimensional case we have:

$$\bteta=(\theta_1,\theta_2,\theta_3)\frown Dirichlet(a_1,a_2,a_3)$$

where $a_1,a_1,a_3>0$ and $a=a_1+a_2+a_3$

$$p_{\bteta}(\bteta)=\dfrac{\Gamma(a)}{\Gamma(a_1)\Gamma(a_2)\Gamma(a_3)}\ \theta_1^{\;{\color{blue}a_1}-1}\theta_2^{\;{\color{blue}a_2}-1}\theta_3^{\;{\color{blue}a_3}-1}$$

* Search for the $\texttt{R}$ function which generates Dirichlet distributed random number vectors and computes Dirichlet probabilities.


```r
?rdirichlet
library(gtools)
```

* Simulate a random vector $\bteta^{(0)}=(\theta_1^{(0)},\theta_2^{(0)},\theta_3^{(0)})$, from a Dirichlet distribution with hyperparameters ${\bf a}^{(0)}=(a_1^{(0)},a_2^{(0)},a_3^{(0)})=(1,1,1)$. Store the simulated vector in \texttt{theta.0} and vector ${\bf a}^{(0)}$ in $\texttt{a.0}$.


```r
a.0<-c(1,1,1)
theta.0<-rdirichlet(1,a.0)
theta.0
```

* Calculate the probability density for the vector (0.15,0.25,0.6), that is, $p_{\bteta}[(0.15,0.25,0.6)]$.


```r
ddirichlet(c(0.15,0.25,0.6),a.0) # Note: prob density function > 0
```

* The Dirichlet distribution is the conjugate prior of the Multinomial distribution, by achiving the following result:   $p_{\bteta|\bf x}(\bteta)\propto \theta_1^{\;{\color{blue}a_1+x_1}-1}\theta_2^{\;{\color{blue}a_2+x_2}-1}\theta_3^{\;{\color{blue}a_3+x_3}-1}$.

  Simulate $\bteta^{(1)}$ knowing **x** and $\bteta^{(0)}$,
directly from de posterior distribution (Dirichlet).


```r
data<-t(data) # need a row vector
updated.theta<-rdirichlet(1,a.0+data) # Note: prob density function > 0
updated.theta
```

**Step 3.** The Gibbs Sampler.

* Develop a function in $\texttt{R}$ for the Gibbs Sampler. Consider 5000 iterations ($\texttt{nr.iter}=5000$).

* Store the updated parameters for each iteration in a  matrix of order $\texttt{nr.iter}\times 3$.


```r
nr.iter<-5000
theta.all.iter<-matrix(0,5000,3)

a<-a.0
for(i in 1:nr.iter)
{
  theta.all.iter[i,]<-rdirichlet(1,a) 
  a<-a+data
  # cat(i,"\n")
}
tail(theta.all.iter)  # last 6 rows
```

**Step 4.** Trace / Parameters Estimation

* Represent graphically the trace for each parameter along the 5000 iterations.


```r
par(mfrow=c(1,3))

plot(1:nr.iter,theta.all.iter[,1],ylim=c(0,0.5),main=expression(paste("Trace for ",theta[1])),ylab=expression(theta),xlab="iteration")
plot(1:nr.iter,theta.all.iter[,2],ylim=c(0,0.5),main=expression(paste("Trace for ",theta[2])),ylab=expression(theta),xlab="iteration")
plot(1:nr.iter,theta.all.iter[,3],ylim=c(0,0.5),main=expression(paste("Trace for ",theta[3])),ylab=expression(theta),xlab="iteration")
```

* Evaluate the need of setting a period of burn-in.

* Find estimates of the parameters according to the decisions made in (b).


```r
theta.final<-apply(theta.all.iter[1001:5000,],2,mean) # 2: 'by column'
round(theta.final,2)
```