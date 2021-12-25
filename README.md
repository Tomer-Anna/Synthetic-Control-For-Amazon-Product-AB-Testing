# Synthetic-Control-For-Amazon-Product-AB-Testing

### Before I start,
I want to acknowledge that this notebook is heavily inspired by the content of [Causal Inference for The Brave and True](https://matheusfacure.github.io/python-causality-handbook/15-Synthetic-Control.html).\
A major part of this notebook's code (and explanations) is taken from his open-source book.\
Another main resource I used is [Causal Inference: The Mixtape](https://mixtape.scunning.com/synthetic-control.html)
and some of the explanations were taken from this [article](https://towardsdatascience.com/causal-inference-using-synthetic-control-the-ultimate-guide-a622ad5cf827).



### Objective:
In this notebook, I'm going to examine if a change we did in one of our products on Amazon helped increase sales or not. \
I'm going to use Synthetic Control Method with a few custom changes in the procedure.\
**It's Important to mention** that because I'm doing changes in the process this is **not** exactly Synthetic Control, but I'm using the same logic of this procedure.\
The main differences are that I'm using regression with regularization (Elastic-Net) without any restrictions on the weights like the author uses.\
In order to prevent overfitting, I'm doing hyperparameter optimization with cross-validation that is stable for a time series task. 
     



### The Expariment:
In May 2021, we took one of our products on Amazon and combine it with another product into one multi-list.\
Here is example to illustrate the case - **this is a random product, not ours** - \
imagine we took this single [chef knif](https://www.amazon.com/J-Henckels-International-Forged-16901-201/dp/B001S41SH6/ref=sr_1_59_sspa?keywords=one+chef+knife+and+set&qid=1637327356&sr=8-59-spons&psc=1&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUEyMlNRN0VLRkRST1QxJmVuY3J5cHRlZElkPUEwODg4MDA4MVlIVzA0OFNLVDFSQSZlbmNyeXB0ZWRBZElkPUEwNTE0MTI0MUhPQllMRERPT0RCQiZ3aWRnZXROYW1lPXNwX2J0ZiZhY3Rpb249Y2xpY2tSZWRpcmVjdCZkb05vdExvZ0NsaWNrPXRydWU=)
and combine it with other list of [chef knifs](https://www.amazon.com/Zwillilng-J-Henckels-8-Inch-Stainless-Steel/dp/B00004RFKS?ref_=ast_sto_dp&th=1&psc=1) with different lengths (but basically very similar). \
We assumed the history of sales and reviews of the single product will improve the other one and will create a better list in general. 



### The Problem:

The problem of checking our hypothesis, is that we can't do "classic" A\B testing. \
We can't make a change in a list and show it only to one part of the potential buyers while the other part will see the original product on Amazon,\
and make a comparison. In other words, we can't divide our observation into treatment and control groups. \
So we won't be able know if our intervention caused a change in the product's sales or it would have happened anyway.



### The Method - Synthetic Control:

The first appearance of the synthetic control estimator was a 2003 article where it was used to estimate the impact of terrorism on economic activity ([Abadie and Gardeazabal 2003](https://www.jstor.org/stable/3132164)). Since that publication, it has become very popular and later on, Abadie, Diamond, and Hainmueller ([2010](https://economics.mit.edu/files/11859)) expound on the method by using a cigarette tax in California called Proposition 99.


The Synthetic Control Method uses a weighted average of multiple cases from the “donor” pool to create an artificial control case.\
Here is a simplified process:
1. There are J + 1 units (products in our case).
2. The j(1) is the treated case units, from j(2) to j(j+1) are unexposed cases that constitute the “donor pool” (other products in our case).
3. pool from the donors and get a weighted average of the units.
pick the weighted value, W*, that minimizes the following loss function:
$||\pmb{X}_1 - \pmb{X}_0 \pmb{W}|| = \bigg(\sum^k_{h=1}v_h \bigg(X_{h1} - \sum^{J+1}_{j=2} w_j X_{hj} \bigg)^2 \bigg)^{\frac{1}{2}}$

    Subject to the restriction that w2,…,wJ+1 are non-negative and sum to one.
    

4. Multiply these weights with the X dataset values and receive the synthetic unit.

### The Data

The data I used is a daily sales (quantity) of 203 related products which also partially correlated with our target product, but also represent the market trends and behavior. These time series contain only quantities and dates from 2017 until 2021, because of confidential issues I won't share the dataset in this repo.
