---
layout: distill
title: Why to Keep Learning
# description: an example of a distill-style blog post and main elements
tags: continual-learning, curiosity, plasticity, stability
giscus_comments: false
date: 2026-08-25
featured: false
mermaid:
  enabled: true
  zoomable: true
code_diff: true
map: true
chart:
  chartjs: true
  echarts: true
  vega_lite: true
tikzjax: true
typograms: true

authors:
  - name: Jan Šochman
    url: "https://jansochman.github.io/"
    affiliations:
      name: CMP, CUT Prague

bibliography: 2026-08-26_why_to_keep_learning.bib

# Below is an example of injecting additional post-specific styles.
# If you use this post as a template, delete this _styles block.
_styles: >
    .figure {
      margin:0;
    }
#   .fake-img {
#     background: #bbb;
#     border: 1px solid rgba(0, 0, 0, 0.1);
#     box-shadow: 0 0px 4px rgba(0, 0, 0, 0.1);
#     margin-bottom: 12px;
#   }
#   .fake-img p {
#     font-family: monospace;
#     color: white;
#     text-align: left;
#     margin: 12px 0;
#     text-align: center;
#     font-size: 16px;
#   }
---

I have spent most of my machine learning research career studying (self-)supervised problems of
various complexity and I feel quite competent there after so many years. In this post I would like to
introduce you my new interest, which turns some of my usual assumptions upside-down. Let me tell you why I
started to like the continual learning problem.

*Let's assume a set of independent and identically distributed (i.i.d.) measurements...* If you have
ever built an algorithm for classification or regression, deep or shallow, trained end-to-end or
partially hand-crafted, you have most likely started with this assumption in mind. And it is a
useful assumption. It allows us to use all available statistical methods and results developed over
centuries and apply them directly to our problem. And thanks to that we were able to solve many
real-world problems using machine learning.

*We have a new production line and your algorithm fails to detect...* And maybe you have already met
a real-life situation where the i.i.d. assumption failed. In practice, if you develop for instance a
handwriting recognition system there is a good chance that it will not work well for a subset of
users. We call such data points out-of-distribution (OOD) samples -- they are from a different
distribution than the training data. There is a whole research field with many methods for detecting
these samples. I myself do see it as an important problem and have spent some time publishing related 
papers<d-cite key="vojir2023calibrated,PixOOD_TPAMI2026"></d-cite>.

Obviously, the i.i.d. assumption is just a useful simplification of the real-world. When we are
careful, it is a good one and it opens a wide range of useful methods for solving the problem. Yet,
it is a simplification only. Think of degrading optics in cameras, unexpected change of lighting,
shift in language statistics over time, new products introduced, ... What should we do about all
these cases? Well, let's build a "world model" or an LLM with all the data in the world in the
training set! Right? By collecting all possible samples we are reducing the chance of meeting an OOD
sample.

But, is it really necessary? And is it the best option?

Let me look at the problem from yet another angle. With the i.i.d. assumption we expect all the training data as
well as every test sample to come from the same (usually unknown) distribution. The aforementioned
examples pointed mostly to the mismatch between the training and the test distributions. However,
consider a reinforcement learning (RL) problem. In RL, the samples are not collected i.i.d.
beforehand, but interactively through the interaction of our current (only partially trained and
still evolving) policy and the environment. With the current policy we decide what to do next and
the environment provides respective observation. There is no i.i.d. stream of data here! If our
policy still cannot reach some state, we will not observe a measurement at that state. So, in RL we
cannot rely on the i.i.d. assumption even for the training data stream.

But wait, there is a solution to this problem which works magic and everybody uses it! It is called
imitation learning<d-cite key="imitation-survey-2024"></d-cite>. Similarly to what we do when
training LLMs, we collect trajectories in the problem space (e.g. a robot arm performing some task)
from human operators, save them in a training set and train the agent to behave similarly. Voilà, we
are back in the i.i.d. case with i.i.d. training set and if we are careful, also the test cases from
the same distribution.

Right? Problem solved! Or maybe there is another way? Maybe there are consequences of this
particular setup? And there is still that unpleasant covariate shift problem<d-cite
key="alvinn1988"></d-cite>...

What I would like to argue for in this post is that, yes, we are able to train very powerful
models, but maybe while overly focusing on one particular simplification, we are missing a chance to
study the model's **ability to adapt**, to keep learning. Something, which is very natural to us
humans. 

We are not forgetting it completely, there is some research on continual learning<d-cite
key="cl-survey-2024"></d-cite>, but remove clear task boundaries, limit the memory for replay and
fix the model size and you arrive at quite difficult task for today's algorithms, so called online
continual learning. I have been testing some of the relevant CL state-of-the-art algorithms and so
far I have not found any, which would be convincing enough for me. Yet, I do find this setup
interesting and very important. 

I think that removing the separation of the test phase from the training is the key for
really intelligent systems, which are able to continuously adapt with new information and
(hopefully) not to forget everything learned before. I do not see the adaptation as an ex-post
process, but as a necessary ingredient of every inference/training. I do not have a solution to this
problem yet, but I would like to share with you three ingredients I find important in these problems
-- curiosity, plasticity, and (in)stability.

## Curiosity 

The non-stationarity of the RL problem leads to the exploration-exploitation dilemma<d-cite
key="wiki:Exploration–exploitation_dilemma"></d-cite>: How much to explore and try new things
and how much to strengthen already known skills given limited budget? For a simple problem, random
exploration (for instance $\epsilon$-greedy) is sufficient to drive the learning towards good
scores, without getting stuck in the already known part of the state space. 

In more complex scenarios with sparse reward and huge state spaces, the exploration-exploitation
dilemma can be also expressed as the sample efficiency problem. The random exploration alone may
need too many steps to stumble upon an interesting new state space part and to discover a policy
which reaches any reward training signal. It keeps sampling uniformly at random even in the parts of
the state space already explored extensively by previous training. It does not distinguish what is
known and what is not. This makes the training computationally inefficient. 

We need some means to recognize what is new and what is already known, to be able to
explore the state space efficiently. If you think of it, this is like integrating OOD detection
inside the training loop. In RL, the methods attempting to distinguish these two regions of state
space are often referred to as curiosity-driven. They are trying to spend the random exploration
compute more on the unknown than the known, at least to some extent.

There is a whole set of approaches defining some curiosity signal. I am not going into detail here.
One may achieve better scores on difficult exploratory problems (e.g. games with very sparse
reward)<d-cite key="ngu2020"></d-cite>. It is possible to ignore part of the signal the agent cannot
influence (TV-noise problem)<d-cite key="rnd2019"></d-cite>. And there is even some evidence that
the approaches are often complementary, so combining several of
them is better than using just one<d-cite key="rlexplore2025"></d-cite>. 
Curiosity also adds an extra non-stationarity to the training. So one has to be extra careful when
adding it to one's approach.

For me, the most important aspect of curiosity is that it allows training with continuously changing
distributions. Without search for novel, it is easy to feel cozy with the already known strategy.

## Plasticity

There is another aspect of the ability to adapt I find interesting. Neural networks do have fixed
size parameter vectors but from our experiments with large i.i.d. training sets we know that
increasing this size together with the dataset size is beneficial. But the training dynamics in RL
is different. We may scale up the models as we wish, but the collection of the data is sequential,
non i.i.d. and typically limited by the number of interactions we are able to perform. Most RL
methods, even non-continual, thus keep their models rather small.

But, wouldn't it be nice if we could use big models with all their capacity and would not need to
worry about overfitting? 

With enough capacity in our model, maybe we could avoid catastrophic forgetting<d-cite
key="catastrophic-interference-1989"></d-cite>, i.e. overwriting the capabilities learned on previous
tasks just because we are learning something new. And maybe we could also avoid primacy
bias<d-cite key="primacy-bias-2022"></d-cite>, the tendency to learn well (overfit) early on and lose the
ability to adapt later on because of the weights being too big and difficult to change with small
gradients. Both of these are two sides of model plasticity. Both, too plastic and not plastic
enough, could be bad. What we are looking for is balance. 

Recently, I have found an interesting middle ground alternative, local plasticity<d-cite
key="elephant-2025"></d-cite>. The paper achieves this by employing activation function with not
only sparse activations (like ReLU), but also sparse gradients. The paper does not seem to scale too
well to large architectures, but that may change with future iterations. Below is a figure
from the paper demonstrating what happens to a neural network updated by a single new data point
with and without local plasticity.

{% include figure.liquid path="assets/img/local_plasticity.png" class="img-fluid z-depth-0" %}
<!-- <div class="caption">
Source: Elephant Neural Networks: Born to be a Continual Learner paper
</div> -->

That is a nice property to have in continual learning, isn't it?

## Training (In)stability

But the real question is: Does RL actually work?

The most striking difference to the i.i.d. setup for a fresh RL practitioner is the training
instability. With all the distribution shifts, sparse rewards and complex training algorithms trying
to handle that, it is often quite challenging to tame the training numerically. I believe, this is
the main reason people often say "RL does not work". They have tried, changed something and it does
not work. I have been there many times too. For this reason, the RL community has built a
set of stable baselines<d-cite key="stable-baselines3"></d-cite>, which tend to work in many cases.
If you do not dare to change them, of course. I remember reading my first RL tutorials and they
often start by advising "do not be a hero", start with a well tested baseline algorithm.

Yet, I want to be a hero. I want to understand the mechanics and the principles which lead to
success instead of failures. For me, this is an interesting adventure at the moment. I do not have an
exact goal here yet. Instead I am getting my hands dirty again and again and hopefully I will be able
to add something to this one day even in the continual learning setup with its extra non-stationarities.

--

Let me stop here, even though there is, of course, much more to continual learning. The curiosity,
plasticity and (in)stability aspects are just three of many I do find interesting at the moment.  I
am sure, I will return to this topic in another post later on as I will keep learning.
