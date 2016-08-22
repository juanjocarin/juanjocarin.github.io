
***You're about to get on a plane to Seattle. You want to know if you should bring an umbrella. You call 3 random friends of yours who live there and ask each independently if it's raining. Each of your friends has a 2/3 chance of telling you the truth and a 1/3 chance of messing with you by lying. All 3 friends tell you that "Yes" it is raining. What is the probability that it's actually raining in Seattle?***

&emsp;&emsp;<a href="https://www.glassdoor.com/Interview/You-re-about-to-get-on-a-plane-to-Seattle-You-want-to-know-if-you-should-bring-an-umbrella-You-call-3-random-friends-of-y-QTN_519262.htm" target="_blank">Facebook interview question posted on Glassdoor</a>.

Most answers posted on the link above are $$\left(\frac{2}{3}\right)^3 = \frac{8}{9}$$, which is the probability that all friends say it rains provided that it actually rains... but we are interested in the opposite, the probability that it rains provided that all friends say it does!

Thus, we need to apply **Bayes' theorem**, which states that the posterior probability (the probability of an hypothesis $$H$$ conditional on a given body of data (evidence, $$E$$) can be calculated as:

$$\begin{aligned} \Pr\left(H \mid E\right) & = \frac{\Pr\left(E \mid H\right) \cdot \Pr\left(H\right)}{\Pr\left(E\right)} \\ & = \frac{\Pr\left(E \mid H\right) \cdot \Pr\left(H\right)}{\displaystyle\sum_{h \in \mathbb{H}} \Pr\left(E \mid h\right) \cdot \Pr\left(h\right)} \end{aligned}$$

In this problem:

$$\begin{aligned} H & :  \text{it rains in city }C \\ E & : \text{all friends say that it rains} \end{aligned}$$

$$H$$ is a binary event, so:

$$\Pr\left(H \mid E\right) = \frac{\Pr\left(E \mid H\right) \cdot \Pr\left(H\right)}{\Pr\left(E \mid H\right) \cdot \Pr\left(H\right) + \Pr\left(E \mid \bar{H}\right) \cdot \Pr\left(\bar{H}\right)}$$

Let $$p$$ be the probability of rain in $$C$$, and $$q$$ be the probability that one friend tells the truth. Then, if there are $$N$$ friends:

$$\Pr\left(H \mid E\right) = \frac{q^N \cdot p}{q^N \cdot p + (1 - q)^N \cdot (1 - p)}$$

We are told that $$3$$ is the number of friends and that $$q = \frac{2}{3}$$. Hence:

$$\begin{aligned} \Pr\left(H \mid E\right) & = \frac{\left(\frac{2}{3}\right)^3 \cdot p}{\left(\frac{2}{3}\right)^3 \cdot p + \left(\frac{1}{3}\right)^3 \cdot (1 - p)} \\ & = \frac{8p}{8p + (1-p)} \\ & = \frac{8p}{7p + 1} \end{aligned}$$

The tricky part of the problem is this unknown parameter, $$p$$, which the solution depends on: we may be biased to think that the solution to this kind of problem is fixed, a particular number.

Let's see what the posterior probability would be for a few possible values of the prior probability:

$$prior = \begin{Bmatrix}
0 \\ 
1/4 \\ 
1/2 \\ 
3/4 \\ 
1
\end{Bmatrix}
\Rightarrow
posterior = \left\{\begin{matrix}
0 \\ 
8/11 \\ 
8/9 \\ 
24/25 \\ 
1
\end{matrix}\right.$$

Let's check that the solution above is right, simulating the problem in R (for all possibles values of $$p$$ from $$0$$ to $$1$$, in steps of $$0.05$$):

```R
library(dplyr)
set.seed(12345)
N <- 5e3 # simulations (per value of p)
q <- 2/3 # prob of a friend telling the truth
data <- data.frame(p = rep(seq(from = 0, to = 1, by = 0.05), each = N))
data <- data %>% rowwise() %>% mutate(r = rbinom(1, 1, p)) %>% 
  mutate(A = ifelse(r == 1, rbinom(1, 1, q), rbinom(1, 1, 1 - q)), 
         B = ifelse(r == 1, rbinom(1, 1, q), rbinom(1, 1, 1 - q)), 
         C = ifelse(r == 1, rbinom(1, 1, q), rbinom(1, 1, 1 - q))) %>% 
  mutate(ABC = paste0(A, B, C)) %>% select(p, r, ABC) %>% ungroup()
# A, B, C: 1 rains, 0 not
# r: 1 rains, 0 not
data %>% sample_n(10) # show 10 of the 21*N rows
```

```R
Source: local data frame [10 x 3]

       p     r   ABC
   <dbl> <int> <chr>
1   0.70     1   111
2   0.65     1   101
3   0.85     1   111
4   0.90     1   010
5   0.80     1   011
6   0.75     1   111
7   0.95     1   110
8   0.70     1   011
9   0.55     1   101
10  0.60     0   101
```

```R
# Filter the evidence: all 3 say it rains
# And group by each value of the prior Prob(it rains), p
data_interest <- data %>% filter(ABC == '111') %>% group_by(p) 
# For each value of the prior (p), what is the posterior?
# (i.e., the mean number of cases where it rain, r == 1)
data_interest %>% summarize(posterior = round(mean(r), 3)) %>% 
  mutate(posterior_theoretical = round(8*p / (1 + 7*p), 3)) %>% 
  rename(prior = p) %>% print(n = Inf)
```

```R
Source: local data frame [21 x 3]

   prior posterior posterior_theoretical
   <dbl>     <dbl>                 <dbl>
1   0.00     0.000                 0.000
2   0.05     0.279                 0.296
3   0.10     0.498                 0.471
4   0.15     0.587                 0.585
5   0.20     0.643                 0.667
6   0.25     0.705                 0.727
7   0.30     0.778                 0.774
8   0.35     0.814                 0.812
9   0.40     0.845                 0.842
10  0.45     0.850                 0.867
11  0.50     0.900                 0.889
12  0.55     0.903                 0.907
13  0.60     0.928                 0.923
14  0.65     0.949                 0.937
15  0.70     0.933                 0.949
16  0.75     0.952                 0.960
17  0.80     0.974                 0.970
18  0.85     0.977                 0.978
19  0.90     0.986                 0.986
20  0.95     0.995                 0.993
21  1.00     1.000                 1.000
```

Now let's consider a more generic case, where not only $$p$$, but also the number of friends, $$N$$, and the probability that any of them tells you the truth, $$q$$, are not fixed. This analysis will give us the *morality* of this problem, which could be "Always trust your friends (especially the more you have!), provided that they tell the truth more often than not" :)

Without loss of generality, let's assume that $$q$$ is a positive rational number, and hence can be written as $$\frac{a}{b}$$, where $$a,b \in \mathbb{N}$$. We want to focus on the case where our friends are more likely to tell the truth, so the following conditions must hold:

$$q \leq 1 \Rightarrow b \geq a$$

$$q \geq \frac{1}{2} \Rightarrow \frac{b}{a} \leq 2 \Rightarrow \frac{b}{a} - 1 \leq 1$$

We can write the posterior probability that we want to calculate as:

$$\begin{aligned}\Pr\left(H \mid E\right) & = \frac{\left(\frac{a}{b}\right)^N \cdot p}{\left(\frac{a}{b}\right)^N \cdot p + \left(1 - \frac{a}{b}\right)^N \cdot (1 - p)} \\ 
& = \frac{a^N \cdot p}{a^N \cdot p + (b - a)^N \cdot (1-p)} \\
&  = \frac{1}{1 + \left(\frac{b}{a} - 1\right)^N \cdot \frac{1-p}{p}} \\
\end{aligned}$$

Since $$\frac{b}{a} - 1 \leq 1$$, the limit of the posterior probability as $$N$$ approaches infinity is $$1$$, regardless of the value of $$p$$.

I.e., if a sufficiently large number of friends tell you that it rains in the desert, and the chances that each one of them is messing with you are less than 1/2, bring an umbrella with you. For $$N = 10$$ (and $$q = 2/3$$), the posterior is greater than $$0.9$$ for a prior as low as $$0.009$$.

Let's finish by plotting the posterior against the prior for a few possible values of $$q$$ and $$N$$:

```R
posterior_prob <- function(q, N, p) {
  q^N * p / (q^N * p + (1 - q)^N * (1-p))
}
p <- seq(0, 1,  0.025)
df <- data.frame(Prior = rep(p, each = 24), 
                 N = rep(c(0:3, 5, 10), each = 4), 
                 q = c(0.51, 2/3, 3/4, 4/5)) %>% 
  mutate(Posterior = posterior_prob(q, N, Prior), 
         Friends = as.factor(N), 
         Q = factor(as.character(fractions(q)), 
                    levels = as.character(fractions(unique(q)))))
library(ggplot2)
library(MASS)
ggplot(data = df, aes(x = Prior, y = Posterior, colour = Friends)) + 
  geom_line() + 
  labs(title = paste('Posterior vs. Prior for different\nvalues of', 
                     'the (individual) Likelihood')) + 
  facet_wrap( ~ Q, nrow = 2) + coord_fixed()
options(repr.plot.width = 8, repr.plot.height = 8)
```
![svg](/output_8_0.png){:class="img-responsive"}
