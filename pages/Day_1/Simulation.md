---
layout: page
title: Practical Exercise - Simulation
schemadotorg:
  "@context": http://schema.org/
  "@type": CreativeWork
  "genre": TrainingMaterial
  isPartOf:
      url: "https://gtpb.github.io/ABSTAT18/"
      name: "ABSTAT18 - Advanced Biostatistics for Bioinformatics Tool Users"
---

The follow exercises will cover the concepts of:

-   Simulation
-   Parameters
-   Monte Carlo simulation
-   Parametric Bootstrap estimation
-   Non-parametric Boostrap estimation
-   Boostrap Confidence Intervals
-   Bootstrap Hypothesis Tests for one sample
-   Booststrap Hypothesis Tests for two independent samples

### Introduction

One of the greatest powers of using a computer lies in the ability to simulate things. What if you don't have any data, but you know the probability model that you want to work with? Through the power of simulation, you can use the computer to generate sample values for you. It's like doing an experiment, only in the virtual world instead of the real world. Simulation is an essential technique in computational statistics.

-   Some distributions  has a corresponding function in R that starts with "r" for random. For example, if you whant to simulate a sample with normal distribution with mean value 2 and standard deviation 1 with size 20, you can use the above R code :

```r
x<-rnorm(20,mean=2,sd=1)
```

### What is simulation

-   (Pseudo) random numbers generated from a computer, since
    the method is completely deterministic. Thus, this numbers
    have a similar behaviour from the random numbers.

-   Random number generator is an algorithm that can generate x\_{i+1} from x_i.

-   Require a start called "seed", i.e., a number that initiates the
    deterministic/iterative process.

-   The "seed" associated to a generator method (algorithm),
    always produce the same sequence.

```r
set.seed(1234)
```

-   This is an important characteristic, because that gives to the
    user the possibility to reproduce exactly the same results.


-   Basically the uniform distribution is simulated in this way.

> **Uniform Distribution X ~ U(a,b), a&lt;b**

The [Uniform distribution](https://en.wikipedia.org/wiki/Uniform_distribution_(continuous)) assigns equal probabilities to all possible ranges of equal length within which the r.v. can fall.
This distributions is widely used in bioinformatics, Bayesian analysis, quantitative genetics and so on.

One of the most important applications of the uniform distribution is in the generation of random numbers. That is, almost all random number generators generate random numbers on the (0,1) interval.

**Exercise 1.** Generate random numbers from an Uniform
Distribution  with different size samples and construct
histograms and analyze them.

<details><summary>Click Here to see the answer</summary><p>
<div markdown="1">

```r
# Get a vector of 4 numbers
runif(4)  #by default is generated an uniform in (0,1)


# Get a vector of 3 numbers from 0 to 100
runif(3, min=0, max=100)


# Get 3 integers from 0 to 100
# Use max=101 because it will never actually equal 101
floor(runif(3, min=0, max=101))  

# This will do the same thing
sample(1:100, 3, replace=T)

# To generate integers WITHOUT replacement:
sample(1:100, 3, replace=F)


#comparison of several samples generated from U(0,1)

x<-runif(100)
y<-runif(200)
z<-runif(300)
k<-runif(400)
w<-runif(500)
j<-runif(600)

par(mfrow=c(3,2))

hist(x,main="n=100",prob=T,xlab="")
abline(h=1,col="red")
hist(y,main="n=200",prob=T,xlab="")
abline(h=1,col="red")
hist(z,main="n=300",prob=T,xlab="")
abline(h=1,col="red")



hist(k,main="n=400",prob=T,xlab="")
abline(h=1,col="red")
hist(w,main="n=500",prob=T,xlab="")
abline(h=1,col="red")
hist(y,main="n=600",prob=T,xlab="")
abline(h=1,col="red")
```

</div>
</p></details>
<br/>
<br/>

**Exercise 2** Generate one sample of size 200 from each of the discrete Binomial and Poisson distributions. Compare the distributions with suitable plots. Change the parameters in each of the models to see how they infuence the distributions.

<details><summary>Click Here to see the answer</summary><p>
<div markdown="1">

```r
bin<-rbinom(200,20,0.5) # B(20,0.5)
pois<-rpois(200,2)      # P(2)
bin2<-rbinom(200,200,0.9) # B(20,0.9)
pois2<-rpois(200,30)      # P(30)


par(mfrow=c(2,2))
hist(bin, main="B(20,0.5)",col="blue",prob=T)
lines(density(bin),col="red")

hist(pois, main="P(2)",col="blue",prob=T)
lines(density(pois),col="red")

hist(bin2, main="B(200,0.9)",col="blue",prob=T)
lines(density(bin2),col="red")

hist(pois2, main="P(30)",col="blue",prob=T)
lines(density(pois2),col="red")
```

</div>
</p></details>
<br/>
<br/>

## Monte Carlo Simulation

**1.1**  Generate 1000 MC replicates from samples of size 15 of a normal distribution with mu=1 and sigma=1.7.
**1.2** Get the following MC estimates: MC mean, MC standard
deviation, MC bias and MC MSE, of the true parameter mu,
considering the sample mean, 20% trimmed mean and median
estimators.
**1.3** Compare the results and choose the best estimator.

<details><summary>Click Here to see the answer</summary><p>
<div markdown="1">

```r
set.seed(3) #  is used to set the random number seed,
#  if we want the results reproducible we use
#  set.seed before generate the number

S <- 1000   #  number of MC simulations
n <- 15     #  sample dimension
mu <- 1     # true mean value
sigma <- sqrt(5/3) # standard deviation

out<-t(replicate(S,rnorm(n,mu,sigma))) # 1000 MC replicates
str(out)
class(out)
trimmean <- function(Y){mean(Y,0.2)} # function to calculate 20% trimmean

outsampmean <- apply(out,1,mean)   
outtrimmean <- apply(out,1,trimmean)
outmedian <- apply(out,1,median)
summary.sim <- data.frame(mean=outsampmean,trim=outtrimmean,
                          median=outmedian)

meanMC<-function(Y){   # MC mean estimate
  sum(Y)/length(Y)
}

MCbias<-function(Y,T){ # MC bias  T=true value of a parameter
  meanMC(Y)-T
}



#labels:
#mean=1,  20% trimmean=2,  median=3

MC_mean.1<-meanMC(outsampmean)
MC_mean.2<-meanMC(outtrimmean)
MC_mean.3<-meanMC(outmedian)

MCsd.1<-sd(outsampmean)
MCsd.2<-sd(outtrimmean)
MCsd.3<-sd(outmedian)


MCbias.1<-MCbias(outsampmean,1)
MCbias.2<-MCbias(outtrimmean,1)
MCbias.3<-MCbias(outmedian,1)

MC_MSE.1<-(MCsd.1)^2-(MCbias.1)^2
MC_MSE.2<-(MCsd.2)^2-(MCbias.2)^2
MC_MSE.3<-(MCsd.3)^2-(MCbias.3)^2

results<-matrix(c(1,1,1,1000,1000,1000,MC_mean.1,MC_mean.2,MC_mean.3,
                  MCsd.1,MCsd.2,MCsd.3,MCbias.1,MCbias.2,MCbias.3,MC_MSE.1,MC_MSE.2,MC_MSE.3),
                ncol=3,byrow=T,dimnames=list(c("True value","# replicates","MC mean","MC sd","MC bias","MC MSE"),
                                             c("Sample mean","Trimmed mean","Median")))

print(round(results,3))
```

</div>
</p></details>
<br/>
<br/>

## **Bootstrap**

> Bootstrap techniques are generally categorized as either nonparametric or parametric. Parametric bootstrap techniques assume that the data are generated
> from a standard parametric probability model. Nonparametric bootstrap techniques are more versatile. Because of their versatility, nonparametric bootstrap techniques are the more popular type of bootstrap applications.
>
> R has two specific packages devoted to the bootstrap: _bootstrap_ and _boot_.
> Both packages are worth looking at and include various applications of the
> bootstrap and further examples. In addition, many other packages, such as the _genetics_ package, contain application specific bootstrap functionality.

**Exercise 2** Suppose it was drawn a sample of 300 observations with sample mean(x) = 2 from an exp(lambda) distribution.

**2.1** Estimate boostrap lambda and bootstrap standard error of hat(lambda) using B=1000.

**2.2** Estimate lambda using a 95%
parametric bootstrap confidence interval for lamda.

<details><summary>Click Here to see the answer</summary><p>
<div markdown="1">

```r
# 95% CI for exponential parameter lambda
# Given 300 data points with mean 2.
# Assume the data is exp(lambda)

# We are given the number of data points and mean
n<-300  
xbar<-2  


# The MLE for lambda is 1/xbar  
lambda_hat<-1/xbar


# Generate the bootstrap samples
# Each column is one bootstrap sample (of 300 resampled values)
nboot = 1000
# Here's the key difference with the empirical
# bootstrap. We draw the bootstrap sample from Exponential(hat lambda)

x<-rexp(n*nboot,lambda_hat)
bootstrap_sample<-matrix(x, nrow=n, ncol=nboot)  

# Compute the bootstrap lambda star
lambda_star = 1./colMeans(bootstrap_sample)

#2.1 Boostrap estimation of lambda
mean_lambda<-mean(lambda_star)
cat(mean_lambda)

# Boostrap estimation of the standard error of lambda hat

num<-NULL
i<-1
for (i in 1:1000){
  num[i]<-(lambda_star[i]-mean_lambda)^2
}

se.B.lambda<-sqrt(sum(num)/1000)

cat(se.B.lambda)

# 2.2 Find the .025 and .975 quantile for delta star
d = quantile(lambda_star, c(.025,.975))  

cat(d)
```

</div>
</p></details>
<br/>
<br/>

**Exercise 3** The following measurements were given for weights (Kg) of 11 children with ages between 8 and 10 years old with renal disfunction: 38.43, 38.43, 38.39, 38.83, 38.45, 38.35, 38.43, 38.31, 38.32, 38.48, 38.50.

Find the 95% parametric bootstrap confidence interval for the mean value (mu) assuming the normal distribution for the observations and sigma = 0.57 (s.d.). Compare with the classical analytic approach based on the t-distribution.

**Note**:Use B=1000 bootstrap samples (each sample hence consisting
of 11 measurements).

<details><summary>Click Here to see the answer</summary><p>
<div markdown="1">

```r
x<-c(38.43, 38.43, 38.39, 38.83, 38.45, 38.35, 38.43, 38.31, 38.32, 38.48, 38.50)

xbar<-mean(x)
sd<-0.57
nx<-11
B<-1000

boot_sample<-replicate(B,rnorm(nx,xbar,sd))
dim(boot_sample)
str(boot_sample)
boot_sample[,1]  # 1st bootsample
mean(boot_sample[,1]) # 1st mean
boot_means<-apply(boot_sample,2,mean)
length(boot_means)

ci_mu<-quantile(boot_means,c(0.025,0.975))
print(ci_mu)

hist(boot_means, col="blue", nclass=30)
```

```r
# Classical parametric CI
hist(x)
```

```r
plot(density(x))
```

```r
shapiro.test(x)
t.test(x)
```

</div>
</p></details>
<br/>
<br/>

**Exercise 4** To illustrate the bootstrap procedure, let's bootstrap a small random sample:
3.12 0.00 1.57 19.67 0.22 2.20

**4.1** Create 1000 replicates of size 6 with replacement.

**4.2** Calculate the sample mean for each of the replicates.

**4.3** Make a histogram and a normal quantile plot of the 1000 means. Make the density plot of the 1000 replicates. This is the bootstrap distribution.

**4.4** Calculate the bootstrap estimates of the mean and the standard error.

<details><summary>Click Here to see the answer</summary><p>
<div markdown="1">

```r
set.seed(1234)
x<-c(3.12,0.00,1.57,19.67, 0.22, 2.20)

x.star<-matrix(NA,1000,6)
i<-1
for (i in 1:1000){
  x.star[i,]<-sample(x,length(x),replace=T)
}

class(x.star)
dim(x.star)

mean.1000<-apply(x.star,1,mean)

meanB<-mean(mean.1000)  #bootstrap estimate of the mean

biasB<-meanB-mean(x)   #bias

hist(mean.1000)
```

```r
qqnorm(mean.1000)
```

```r
num<-NULL
i<-1
for (i in 1:1000){
  num[i]<-(mean.1000[i]-meanB)^2
}

se.B<-sqrt(sum(num)/999)


#------------  95% confidence interval for mean

CI<-quantile(mean.1000,c(0.025,0.975))

#using boot package

library(boot)

#we need a function of the parameter that we whant to estimate
#second argument "indeices" is the indices of the observations for bootstrap sample

my.mean = function(x, indices){
  return( mean( x[indices] ) )
}

data.boot<-boot(x,my.mean,1000)
mode(data.boot)
str(data.boot)
ci.boots<-boot.ci(data.boot,0.95,type="perc")
print(ci.boots)
```

</div>
</p></details>
<br/>
<br/>

**Exercise 5** Confidence interval for a correlation coefficient (Adapted from Applied Statistics for Bioinformatics using R, Wim P. Krijnen).

Consider two sets of expression values of the MCM3 gene of the Golub et al. (1999) data. This data set is a gene expression data (3051 genes and 38 tumor mRNA samples) from the leukemia microarray study. This gene encodes for highly conserved mini-chromosome maintenance proteins (MCM) which are involved in the initiation of eukaryotic genome replication.

**5.1** Obtain a bootstrap sample from (x; y), and compute the correlation coeficient for the bootstrap sample.

**5.2** Repeat the procedure [5.1] B=1000 times.

**5.3** From the sample of size n = 38 of the bootstraped correlation coeficients obtain the 0.025 and 0.975 percentiles.

**5.4** This pair is a bootstrap 95% confidence interval for the correlation coefficient parameter.

<details><summary>Click Here to see the answer</summary><p>
<div markdown="1">

```r
source("https://bioconductor.org/biocLite.R")
biocLite("multtest")

library(multtest);
data(golub)
x <- golub[2289,];
y <- golub[2430,]
cor(x,y)

xy<-matrix(c(x,y),ncol=2) #matrix containig the data

B <- 1000 #number of bootstrap samples
cor.star <- 0 #matrix containing correlation coefficients
for (i in 1:B){
  z<-sample(1:nrow(xy),replace=TRUE)
  cor.star[i] <- cor(xy[z,1],xy[z,2])
}

mean(cor.star) #bootstrap estimate of the correlation coefficient

plot(density(cor.star)) #bootstrap sampling distribution

quantile(cor.star,c(0.025,0.975))#bootstrap 95% confidence interval

#parametric approach

install.packages("psychometric") # install package with function
library(psychometric) # load package with function

# The following command calculates lower and upper
# 95% confidence intervals (level)
# sample size (n) is

CIr(r=cor(x,y), n = 38, level = .95)
```

</div>
</p></details>
<br/>
<br/>

**Exercise 6** Gdf5 gene from the Golub et al. (1999) data.
(Adapted from Applied Statistics for Bioinformatics using R, Wim P. Krijnen).

-   The corresponding expression values are contained in row 2058.

-   A quick search through the NCBI site makes it likely that this
    gene is not directly related to leukemia.

-   Hence, we may hypothesize that the population mean of the
    ALL expression values equals zero.

-   Accordingly, we test H0 : mu = 0 vs. H1 : mu > 0.

-   How can we use bootstrap to test the present hypothesis?

<details><summary>Click Here to see the answer</summary><p>
<div markdown="1">

```r
library(multtest)
data(golub)
?golub
#golub.cl is a numeric vector indicating the tumor class,
#27 acute lymphoblastic leukemia (ALL) cases (code 0)
#and 11 acute myeloid leukemia (AML) cases (code 1).

gol.fac <- factor(golub.cl,levels=0:1, labels= c("ALL","AML"))

x <- golub[2058,golub.cl==0] # getting ALL expression levels


#non-parametric approach:bootstrap
n<-length(x)
mu0<-0

t.obs<-(mean(x)-mu0)*sqrt(n)/sd(x)
t.obs

z<-x-mean(x)+mu0
z
hist(z)


m <- 10000

t.star <- 0
for(j in 1:m)
{
  z.star <- sample(z,n,replace=T)
  t.star[j] <- (mean(z.star)-mu0)*sqrt(n)/sd(z.star)
}

p<-length(t.star[t.star>t.obs])/m  # one-sided
p


#parametric approach: t-test

hist(x,probability=T)
lines(density(x))
shapiro.test(x)  #normality test
t.test(x,alternative = "greater",mu = 0)
```

</div>
</p></details>
<br/>
<br/>

**Exercise 7** Gene CCND3 Cyclin D3

-   Golub et al. (1999) argue that gene CCND3 Cyclin D3 plays an important role with respect to discriminating ALL from AML patients.

-   We are interested in testing the null hypothesis of equal means, ie, we want to test: H0: meanvalue(ALL)=meanvalue(AML)  vs. H1: meanvalue(ALL)=~meanvalue(AML).

-   Implement a boostrap test for this problem.

<details><summary>Click Here to see the answer</summary><p>
<div markdown="1">

```r
golub.cl
x1<-golub[1042,golub.cl==0]
x2<-golub[1042,golub.cl==1]


plot(density(x1),xlim=c(-2,3))  #empirical densities
lines(density(x2),col=2)
legend(-1,0.6,legend=c("ALL","AML"),col=1:2,lty=1)

n1<-length(x1)
n2<-length(x2)
xb1<-mean(x1)
xb2<-mean(x2)
vb1<-var(x1)
vb2<-var(x2)
t.obs<-(xb1-xb2)/sqrt(vb1/n1+vb2/n2)
t.obs
xb<-mean(c(x1,x2)) #combined mean of the two samples
z1<-x1-xb1+xb
z2<-x2-xb2+xb
mean(z1)
mean(z2)
xb
t.star<-0
B<-1000
for(i in 1:B){
  z1.star<-sample(z1,n1,replace=T)
  z2.star<-sample(z2,n2,replace=T)
  zb1<-mean(z1.star)
  zb2<-mean(z2.star)
  vz1<-var(z1.star)
  vz2<-var(z2.star)
  t.star[i]<-(zb1-zb2)/sqrt(vz1/n1+vz2/n2)
}
pvalue<-(sum(abs(t.star)>abs(t.obs))/B)
pvalue

#parametric approach
shapiro.test(x1)
shapiro.test(x2)
var.test(x1,x2)
test<-t.test(golub[1042,] ~ golub.cl, var.equal=T,alternative="two.sided")
test
```

</div>
</p></details>

<br>

### Back

Back to [main page](../../index.md).
