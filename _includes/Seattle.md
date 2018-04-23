
***You're about to get on a plane to Seattle. You want to know if you should bring an umbrella. You call 3 random friends of yours who live there and ask each independently if it's raining. Each of your friends has a 2/3 chance of telling you the truth and a 1/3 chance of messing with you by lying. All 3 friends tell you that "Yes" it is raining. What is the probability that it's actually raining in Seattle?***

&emsp;&emsp;<a href="https://www.glassdoor.com/Interview/You-re-about-to-get-on-a-plane-to-Seattle-You-want-to-know-if-you-should-bring-an-umbrella-You-call-3-random-friends-of-y-QTN_519262.htm" target="_blank">Facebook interview question posted on Glassdoor</a>.

Most answers posted on the link above are:

$$\left(\frac{2}{3}\right)^3 = \frac{8}{27}$$

which is the probability that all friends say it is raining provided that it's true... but we are interested in the opposite, the probability that it is raining provided that all friends say that.

Another common answer is based on a correct idea:

$$\begin{aligned}
\Pr\left(H \mid E\right) & = \frac{\Pr\left(H, E\right)}{\Pr\left(E\right)} \\ 
& = \frac{\Pr\left(H, E\right)}{\displaystyle\sum_{h \in \mathbb{H}} \Pr\left(h, E\right)}
\end{aligned}$$

where

$$\begin{aligned}
H & : \text{it is raining in Seattle} \\ 
E & : \text{all friends say it is raining}
\end{aligned}$$

$$h$$ is a binary event, so:

$$\Pr\left(H \mid E\right) = \frac{\Pr\left(H, E\right)}{\Pr\left(H, E\right) + \Pr\left(\bar{H}, E\right)}$$

But those answers then mix joint and conditional probabilities, getting yet a wrong result (which is correct only if the probability of rain in Seattle is $$0.5$$):

$$\frac{8/27}{8/27 + 1/27} = \frac{8}{9}$$

$$\left(2/3\right)^3 = 8/27$$ is not the joint probability that all friends say the same thing (it is raining) **and** that is true, but the probability that all friends say that it is raining **conditional** on the fact that it is raining.

Let's consider the two extreme cases to confirm that the solution above is not always correct. If it is always raining in Seattle,

$$\begin{aligned}H = \Omega & \Rightarrow \Pr(H) = 1 \\
& \Rightarrow
\left\{\begin{matrix}
\Pr(H, E) & = & \Pr(E) & = & 8/27 \\ 
\Pr\left(\bar{H}, E \right) & = & 0
\end{matrix}\right. \\
& \Rightarrow \Pr(H \mid E) = 1
\end{aligned}$$

If it never rains in Seattle,

$$\begin{aligned}H = \varnothing & \Rightarrow \Pr(H) = 0 \\
& \Rightarrow
\left\{\begin{matrix}
\Pr(H, E) & = & 0  \\ 
\Pr\left(\bar{H}, E \right) & = & \Pr(E) & = & 1/27 
\end{matrix}\right. \\ 
& \Rightarrow \Pr(H \mid E) = 0
\end{aligned}$$

The tricky part of the problem, since we may be biased to think that the solution is fixed, is that there is an unknown parameter ($$p$$, the probability of rain in Seattle), which the solution depends on. We need to apply the **Bayes' theorem**, which states that the posterior probability (the probability of an hypothesis $$H$$ conditional on a given body of data (the evidence, $$E$$) can be calculated as:

$$\begin{aligned}
\Pr\left(H \mid E\right) & = \frac{\Pr\left(E \mid H\right) \cdot \Pr\left(H\right)}{\Pr\left(E\right)} \\ 
& = \frac{\Pr\left(E \mid H\right) \cdot \Pr\left(H\right)}{\displaystyle\sum_{h \in \mathbb{H}} \Pr\left(E \mid h\right) \cdot \Pr\left(h\right)} \\ 
& = \frac{\Pr\left(E \mid H\right) \cdot \Pr\left(H\right)}{\Pr\left(E \mid H\right) \cdot \Pr\left(H\right) + \Pr\left(E \mid \bar{H}\right) \cdot \Pr\left(\bar{H}\right)}
\end{aligned}$$

Let $$q$$ the probability that one friend tells the truth. Then, if there are $$N$$ friends:

$$\Pr\left(H \mid E\right) = \frac{q^N \cdot p}{q^N \cdot p + (1 - q)^N \cdot (1 - p)}$$

We are told that $$3$$ is the number of friends and that $$q = 2/3$$. Hence:

$$\begin{aligned} \Pr\left(H \mid E\right) & = \frac{\left(\frac{2}{3}\right)^3 \cdot p}{\left(\frac{2}{3}\right)^3 \cdot p + \left(\frac{1}{3}\right)^3 \cdot (1 - p)} \\ & = \frac{8p}{8p + (1-p)} \\ & = \mathbf{\frac{8p}{7p + 1}} \end{aligned}$$

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
8/11 & \approx & 0.73 \\ 
8/9 & \approx & 0.89 \\ 
24/25 & = & 0.96 \\ 
1
\end{matrix}\right.$$

Also, note that the posterior reaches a value of $$1/2$$ for a prior as low as $$1/9 \approx 0.11$$.

Let's confirm that the solution above is right, simulating the problem in R (for all possibles values of $$p$$ from $$0$$ to $$1$$, in steps of $$0.05$$):

<!--
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
-->


<div class="highlight highlight-source-r"><pre>library(<span class="pl-smi">dplyr</span>)
set.seed(<span class="pl-c1">12345</span>)
<span class="pl-c"># Simulate N cases for each probability of rain in Seattle (p)</span>
<span class="pl-c"># in the whole [0, 1] range, in steps of 0.05</span>
<span class="pl-smi">N</span> <span class="pl-k">&lt;-</span> <span class="pl-c1">5e3</span> <span class="pl-c"># simulations (per value of p)</span>
<span class="pl-smi">q</span> <span class="pl-k">&lt;-</span> <span class="pl-c1">2</span><span class="pl-k">/</span><span class="pl-c1">3</span> <span class="pl-c"># prob of a friend telling the truth</span>
<span class="pl-smi">data</span> <span class="pl-k">&lt;-</span> <span class="pl-k">data.frame</span>(<span class="pl-v">p</span> <span class="pl-k">=</span> rep(seq(<span class="pl-v">from</span> <span class="pl-k">=</span> <span class="pl-c1">0</span>, <span class="pl-v">to</span> <span class="pl-k">=</span> <span class="pl-c1">1</span>, <span class="pl-v">by</span> <span class="pl-k">=</span> <span class="pl-c1">0.05</span>), <span class="pl-v">each</span> <span class="pl-k">=</span> <span class="pl-smi">N</span>))
<span class="pl-smi">data</span> <span class="pl-k">&lt;-</span> <span class="pl-smi">data</span> <span class="pl-k">%&gt;%</span> rowwise() <span class="pl-k">%&gt;%</span> mutate(<span class="pl-v">r</span> <span class="pl-k">=</span> as.logical(rbinom(<span class="pl-c1">1</span>, <span class="pl-c1">1</span>, <span class="pl-smi">p</span>))) <span class="pl-k">%&gt;%</span> 
  mutate(<span class="pl-v">A</span> <span class="pl-k">=</span> as.logical(ifelse(<span class="pl-smi">r</span> <span class="pl-k">==</span> <span class="pl-c1">1</span>, rbinom(<span class="pl-c1">1</span>, <span class="pl-c1">1</span>, <span class="pl-smi">q</span>), rbinom(<span class="pl-c1">1</span>, <span class="pl-c1">1</span>, <span class="pl-c1">1</span> <span class="pl-k">-</span> <span class="pl-smi">q</span>))), 
         <span class="pl-v">B</span> <span class="pl-k">=</span> as.logical(ifelse(<span class="pl-smi">r</span> <span class="pl-k">==</span> <span class="pl-c1">1</span>, rbinom(<span class="pl-c1">1</span>, <span class="pl-c1">1</span>, <span class="pl-smi">q</span>), rbinom(<span class="pl-c1">1</span>, <span class="pl-c1">1</span>, <span class="pl-c1">1</span> <span class="pl-k">-</span> <span class="pl-smi">q</span>))), 
         <span class="pl-v">C</span> <span class="pl-k">=</span> as.logical(ifelse(<span class="pl-smi">r</span> <span class="pl-k">==</span> <span class="pl-c1">1</span>, rbinom(<span class="pl-c1">1</span>, <span class="pl-c1">1</span>, <span class="pl-smi">q</span>), 
                               rbinom(<span class="pl-c1">1</span>, <span class="pl-c1">1</span>, <span class="pl-c1">1</span> <span class="pl-k">-</span> <span class="pl-smi">q</span>)))) <span class="pl-k">%&gt;%</span> ungroup()
<span class="pl-c"># p: probability that it is raining in Seattle</span>
<span class="pl-c"># r: it is raining in Seattle</span>
<span class="pl-c"># A, B, C: friend A, B, C says it is raining</span>
  <span class="pl-c"># (not to be confused with he or she is telling the truth)</span>
<span class="pl-smi">data</span> <span class="pl-k">%&gt;%</span> sample_n(<span class="pl-c1">10</span>) <span class="pl-k">%&gt;%</span> print(<span class="pl-v">n</span> <span class="pl-k">=</span> <span class="pl-c1">10</span>) <span class="pl-c"># show 10 of the 21*N rows</span></pre></div>

<div class="highlight highlight-source-r"><pre><span class="pl-r">Source: local data frame [</span><span class="pl-c1">10</span><span class="pl-r"> x </span><span class="pl-c1">5</span><span class="pl-r">]

       p     r     A     B     C
   </span><span class="pl-k">&lt;</span><span class="pl-r">dbl</span><span class="pl-k">&gt; &lt;</span><span class="pl-r">lgl</span><span class="pl-k">&gt; &lt;</span><span class="pl-r">lgl</span><span class="pl-k">&gt; &lt;</span><span class="pl-r">lgl</span><span class="pl-k">&gt; &lt;</span><span class="pl-r">lgl</span><span class="pl-k">&gt;</span><span class="pl-c1">
1   0.70  TRUE  TRUE  TRUE  TRUE
2   0.65  TRUE  TRUE FALSE  TRUE
3   0.85  TRUE  TRUE  TRUE  TRUE
4   0.90  TRUE FALSE  TRUE FALSE
5   0.80  TRUE FALSE  TRUE  TRUE
6   0.75  TRUE  TRUE  TRUE  TRUE
7   0.95  TRUE  TRUE  TRUE FALSE
8   0.70  TRUE FALSE  TRUE  TRUE
9   0.55  TRUE  TRUE FALSE  TRUE
10  0.60 FALSE  TRUE FALSE  TRUE
</span></pre></div>

<div class="highlight highlight-source-r"><pre><span class="pl-c"># Check proportions of r (compared to p)</span>
<span class="pl-smi">data</span> <span class="pl-k">%&gt;%</span> group_by(<span class="pl-smi">p</span>) <span class="pl-k">%&gt;%</span> summarize(<span class="pl-v">Real_Prob_Rain</span> <span class="pl-k">=</span> mean(<span class="pl-smi">r</span>)) <span class="pl-k">%&gt;%</span> 
  rename(<span class="pl-v">Prob_Rain</span> <span class="pl-k">=</span> <span class="pl-smi">p</span>) <span class="pl-k">%&gt;%</span> print(<span class="pl-v">n</span> <span class="pl-k">=</span> <span class="pl-c1">Inf</span>)</pre></div>

<div class="highlight highlight-source-r"><pre><span class="pl-r">Source: local data frame [</span><span class="pl-c1">21</span><span class="pl-r"> x </span><span class="pl-c1">2</span><span class="pl-r">]

   Prob_Rain Real_Prob_Rain
       </span><span class="pl-k">&lt;</span><span class="pl-r">dbl</span><span class="pl-k">&gt;          &lt;</span><span class="pl-r">dbl&gt;</span><span class="pl-c1">
1       0.00         0.0000
2       0.05         0.0460
3       0.10         0.0978
4       0.15         0.1494
5       0.20         0.1938
6       0.25         0.2466
7       0.30         0.2956
8       0.35         0.3390
9       0.40         0.3934
10      0.45         0.4524
11      0.50         0.4994
12      0.55         0.5430
13      0.60         0.5900
14      0.65         0.6586
15      0.70         0.6908
16      0.75         0.7406
17      0.80         0.8004
18      0.85         0.8504
19      0.90         0.8942
20      0.95         0.9492
21      1.00         1.0000
</span></pre></div>

<div class="highlight highlight-source-r"><pre><span class="pl-c"># Check proportions of friends telling the truth (compared to q)</span>
<span class="pl-smi">data</span> <span class="pl-k">%&gt;%</span> rename(<span class="pl-v">Raining</span> <span class="pl-k">=</span> <span class="pl-smi">r</span>) <span class="pl-k">%&gt;%</span> group_by(<span class="pl-smi">Raining</span>) <span class="pl-k">%&gt;%</span> 
  summarize(<span class="pl-s"><span class="pl-pds">"</span>%Cases_All_Friends_Say_Raining<span class="pl-pds">"</span></span> <span class="pl-k">=</span> mean(<span class="pl-smi">A</span> <span class="pl-k">+</span> <span class="pl-smi">B</span> <span class="pl-k">+</span> <span class="pl-smi">C</span>) <span class="pl-k">/</span> <span class="pl-c1">3</span>) <span class="pl-k">%&gt;%</span> 
  print(<span class="pl-v">n</span> <span class="pl-k">=</span> <span class="pl-c1">Inf</span>)
<span class="pl-smi">data</span> <span class="pl-k">%&gt;%</span> rename(<span class="pl-v">Prob_Rain</span> <span class="pl-k">=</span> <span class="pl-smi">p</span>, <span class="pl-v">Raining</span> <span class="pl-k">=</span> <span class="pl-smi">r</span>) <span class="pl-k">%&gt;%</span> 
  group_by(<span class="pl-smi">Prob_Rain</span>, <span class="pl-smi">Raining</span>) <span class="pl-k">%&gt;%</span> 
  summarize(<span class="pl-s"><span class="pl-pds">"</span>#Cases<span class="pl-pds">"</span></span> <span class="pl-k">=</span> n(), 
            <span class="pl-s"><span class="pl-pds">"</span>%Cases_All_Friends_Say_Raining<span class="pl-pds">"</span></span> <span class="pl-k">=</span> mean(<span class="pl-smi">A</span> <span class="pl-k">+</span> <span class="pl-smi">B</span> <span class="pl-k">+</span> <span class="pl-smi">C</span>) <span class="pl-k">/</span> <span class="pl-c1">3</span>) <span class="pl-k">%&gt;%</span> 
  print(<span class="pl-v">n</span> <span class="pl-k">=</span> <span class="pl-c1">11</span>)</pre></div>

<div class="highlight highlight-source-r"><pre><span class="pl-r">Source: local data frame [</span><span class="pl-c1">2</span><span class="pl-r"> x </span><span class="pl-c1">2</span><span class="pl-r">]

  Raining %Cases_All_Friends_Say_Raining
    </span><span class="pl-k">&lt;</span><span class="pl-r">lgl</span><span class="pl-k">&gt;                          &lt;</span><span class="pl-r">dbl</span><span class="pl-k">&gt;</span><span class="pl-c1">
1   FALSE                      0.3322547
2    TRUE                      0.6649218</span>

<span class="pl-r">Source: local data frame [</span><span class="pl-c1">40</span><span class="pl-r"> x </span><span class="pl-c1">4</span><span class="pl-r">]
Groups: Prob_Rain [?]

   Prob_Rain Raining #Cases %Cases_All_Friends_Say_Raining
       </span><span class="pl-k">&lt;</span><span class="pl-r">dbl</span><span class="pl-k">&gt;   &lt;</span><span class="pl-r">lgl</span><span class="pl-k">&gt;  &lt;</span><span class="pl-r">int</span><span class="pl-k">&gt;                          &lt;</span><span class="pl-r">dbl</span><span class="pl-k">&gt;</span><span class="pl-c1">
1       0.00   FALSE   5000                      0.3332667
2       0.05   FALSE   4770                      0.3243885
3       0.05    TRUE    230                      0.6826087
4       0.10   FALSE   4511                      0.3302298
5       0.10    TRUE    489                      0.6659850
6       0.15   FALSE   4253                      0.3359981
7       0.15    TRUE    747                      0.6604195
8       0.20   FALSE   4031                      0.3304391
9       0.20    TRUE    969                      0.6584107
10      0.25   FALSE   3767                      0.3342182
11      0.25    TRUE   1233                      0.6580157
..       ...     ...    ...                            ...
</span></pre></div>

<div class="highlight highlight-source-r"><pre><span class="pl-c"># Filter the evidence: all 3 say it rains</span>
<span class="pl-c"># And group by each value of the prior Prob(it rains), p</span>
<span class="pl-smi">data_interest</span> <span class="pl-k">&lt;-</span> <span class="pl-smi">data</span> <span class="pl-k">%&gt;%</span> filter(<span class="pl-smi">A</span> <span class="pl-k">*</span> <span class="pl-smi">B</span> <span class="pl-k">*</span> <span class="pl-smi">C</span> <span class="pl-k">==</span> <span class="pl-c1">TRUE</span>) <span class="pl-k">%&gt;%</span> group_by(<span class="pl-smi">p</span>)
<span class="pl-c"># For each value of the prior (p), what is the posterior?</span>
<span class="pl-c"># (i.e., the mean number of cases where it rain, r == 1)</span>
<span class="pl-smi">data_interest</span> <span class="pl-k">%&gt;%</span> summarize(<span class="pl-v">Posterior</span> <span class="pl-k">=</span> round(mean(<span class="pl-smi">r</span>), <span class="pl-c1">3</span>)) <span class="pl-k">%&gt;%</span> 
  mutate(<span class="pl-v">Posterior_Theoretical</span> <span class="pl-k">=</span> round(<span class="pl-c1">8</span><span class="pl-k">*</span><span class="pl-smi">p</span> <span class="pl-k">/</span> (<span class="pl-c1">1</span> <span class="pl-k">+</span> <span class="pl-c1">7</span><span class="pl-k">*</span><span class="pl-smi">p</span>), <span class="pl-c1">3</span>)) <span class="pl-k">%&gt;%</span> 
  rename(<span class="pl-v">Prior</span> <span class="pl-k">=</span> <span class="pl-smi">p</span>) <span class="pl-k">%&gt;%</span> print(<span class="pl-v">n</span> <span class="pl-k">=</span> <span class="pl-c1">Inf</span>)</pre></div>

<pre><code><span class="pl-r">Source: local data frame [</span><span class="pl-c1">21</span><span class="pl-r"> x </span><span class="pl-c1">3</span><span class="pl-r">]
    
       Prior Posterior Posterior_Theoretical
       </span><span class="pl-k">&lt;</span><span class="pl-r">dbl</span><span class="pl-k">&gt;     &lt;</span><span class="pl-r">dbl</span><span class="pl-k">&gt;                 &lt;</span><span class="pl-r">dbl</span><span class="pl-k">&gt;</span><span class="pl-c1">
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
</span></code></pre>

Now let's consider a more generic case, where not only $$p$$, but also the number of friends, $$N$$, and the probability that any of them tells you the truth, $$q$$, are not fixed. This analysis will give us the *morality* of this problem, which could be: "Always trust your friends (especially the more you have!), provided that they tell the truth more often than not."

Without loss of generality, let's assume that $$q$$ is a positive rational number, and hence can be written as $$a/b$$, where $$a,b \in \mathbb{N}; b \geq a$$. We want to focus on the case where our friends are more likely to tell the truth, so the following condition must hold:

$$q \geq \frac{1}{2} \Rightarrow \frac{b}{a} \leq 2 \Rightarrow \frac{b}{a} - 1 \leq 1$$

We can write the posterior probability that we want to calculate as:

$$\begin{aligned}\Pr\left(H \mid E\right) & = \frac{\left(\frac{a}{b}\right)^N \cdot p}{\left(\frac{a}{b}\right)^N \cdot p + \left(1 - \frac{a}{b}\right)^N \cdot (1 - p)} \\ 
& = \frac{a^N \cdot p}{a^N \cdot p + (b - a)^N \cdot (1-p)} \\
&  = \frac{1}{1 + \left(\frac{b}{a} - 1\right)^N \cdot \frac{1-p}{p}} \\
\end{aligned}$$

Since $$b/a - 1 \leq 1$$, the limit of the posterior probability as $$N$$ approaches infinity is $$1$$, regardless of the value of $$p$$ (for $$p > 0$$).

I.e., if a sufficiently large number of friends tell you that it is raining in the desert, and the chances that each one of them is messing with you are less than $$1/2$$, bring an umbrella with you. For $$N = 10$$ (and $$q = 2/3$$), the posterior is greater than $$0.9$$ for a prior as low as $$0.009$$.

Let's finish by plotting the posterior against the prior for a few possible values of $$q$$ and $$N$$ (our case of interest, $$N=3, q=2/3$$ corresponds to the light blue line in the upper-right graph). As expected, if $$N = 0$$ (i.e., we have no evidence), the posterior equals the prior.

<!--
```R
posterior_prob <- function(q, N, p) {
  q^N * p / (q^N * p + (1 - q)^N * (1-p))
}
p <- seq(0, 1,  0.01)
library(MASS)
df <- data.frame(Prior = rep(p, each = 24), 
                 N = rep(c(0:3, 5, 10), each = 4), 
                 q = c(0.51, 2/3, 3/4, 4/5)) %>% 
  mutate(Posterior = posterior_prob(q, N, Prior), 
         Friends = as.factor(N), 
         Q = factor(as.character(fractions(q)), 
                    levels = as.character(fractions(unique(q)))))
library(ggplot2)
ggplot(data = df, aes(x = Prior, y = Posterior, colour = Friends)) + 
  geom_line() + scale_color_hue(c = 240) + 
  labs(title = paste('Posterior vs. Prior for 4 possible\nvalues of', 
                     'the (individual) Likelihood')) + 
  facet_wrap( ~ Q, nrow = 2) + coord_fixed()
options(repr.plot.width = 8, repr.plot.height = 8)
```
-->

<div class="highlight highlight-source-r"><pre><span class="pl-en">posterior_prob</span> <span class="pl-k">&lt;-</span> <span class="pl-k">function</span>(<span class="pl-smi">q</span>, <span class="pl-smi">N</span>, <span class="pl-smi">p</span>) {
  <span class="pl-smi">q</span><span class="pl-k">^</span><span class="pl-smi">N</span> <span class="pl-k">*</span> <span class="pl-smi">p</span> <span class="pl-k">/</span> (<span class="pl-smi">q</span><span class="pl-k">^</span><span class="pl-smi">N</span> <span class="pl-k">*</span> <span class="pl-smi">p</span> <span class="pl-k">+</span> (<span class="pl-c1">1</span> <span class="pl-k">-</span> <span class="pl-smi">q</span>)<span class="pl-k">^</span><span class="pl-smi">N</span> <span class="pl-k">*</span> (<span class="pl-c1">1</span><span class="pl-k">-</span><span class="pl-smi">p</span>))
}
<span class="pl-smi">p</span> <span class="pl-k">&lt;-</span> seq(<span class="pl-c1">0</span>, <span class="pl-c1">1</span>,  <span class="pl-c1">0.01</span>)
library(<span class="pl-smi">MASS</span>)
<span class="pl-smi">df</span> <span class="pl-k">&lt;-</span> <span class="pl-k">data.frame</span>(<span class="pl-v">Prior</span> <span class="pl-k">=</span> rep(<span class="pl-smi">p</span>, <span class="pl-v">each</span> <span class="pl-k">=</span> <span class="pl-c1">24</span>), 
                 <span class="pl-v">N</span> <span class="pl-k">=</span> rep(c(<span class="pl-c1">0</span><span class="pl-k">:</span><span class="pl-c1">3</span>, <span class="pl-c1">5</span>, <span class="pl-c1">10</span>), <span class="pl-v">each</span> <span class="pl-k">=</span> <span class="pl-c1">4</span>), 
                 <span class="pl-v">q</span> <span class="pl-k">=</span> c(<span class="pl-c1">0.51</span>, <span class="pl-c1">2</span><span class="pl-k">/</span><span class="pl-c1">3</span>, <span class="pl-c1">3</span><span class="pl-k">/</span><span class="pl-c1">4</span>, <span class="pl-c1">4</span><span class="pl-k">/</span><span class="pl-c1">5</span>)) <span class="pl-k">%&gt;%</span> 
  mutate(<span class="pl-v">Posterior</span> <span class="pl-k">=</span> posterior_prob(<span class="pl-smi">q</span>, <span class="pl-smi">N</span>, <span class="pl-smi">Prior</span>), 
         <span class="pl-v">Friends</span> <span class="pl-k">=</span> as.factor(<span class="pl-smi">N</span>), 
         <span class="pl-v">Q</span> <span class="pl-k">=</span> <span class="pl-k">factor</span>(as.character(fractions(<span class="pl-smi">q</span>)), 
                    <span class="pl-v">levels</span> <span class="pl-k">=</span> as.character(fractions(unique(<span class="pl-smi">q</span>)))))
library(<span class="pl-smi">ggplot2</span>)
ggplot(<span class="pl-v">data</span> <span class="pl-k">=</span> <span class="pl-smi">df</span>, aes(<span class="pl-v">x</span> <span class="pl-k">=</span> <span class="pl-smi">Prior</span>, <span class="pl-v">y</span> <span class="pl-k">=</span> <span class="pl-smi">Posterior</span>, <span class="pl-v">colour</span> <span class="pl-k">=</span> <span class="pl-smi">Friends</span>)) <span class="pl-k">+</span> 
  geom_line() <span class="pl-k">+</span> 
  scale_color_hue(<span class="pl-v">c</span> <span class="pl-k">=</span> <span class="pl-c1">240</span>) <span class="pl-k">+</span> 
  labs(<span class="pl-v">title</span> <span class="pl-k">=</span> paste(<span class="pl-s"><span class="pl-pds">'</span>Posterior vs. Prior for 4 possible<span class="pl-cce">\n</span>values of<span class="pl-pds">'</span></span>, 
                     <span class="pl-s"><span class="pl-pds">'</span>the (individual) Likelihood<span class="pl-pds">'</span></span>)) <span class="pl-k">+</span> 
  facet_wrap( <span class="pl-k">~</span> <span class="pl-smi">Q</span>, <span class="pl-v">nrow</span> <span class="pl-k">=</span> <span class="pl-c1">2</span>) <span class="pl-k">+</span> coord_fixed()
options(<span class="pl-v">repr.plot.width</span> <span class="pl-k">=</span> <span class="pl-c1">8</span>, <span class="pl-v">repr.plot.height</span> <span class="pl-k">=</span> <span class="pl-c1">8</span>)</pre></div>

![svg](/img/output_8_0.png){:class="img-responsive"}
