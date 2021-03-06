---
layout: article
title: Estimating Gradients of expectations
comments: true
categories: data_science
image:
  teaser: jupyter-main-logo.svg
---


Back-propagation (Rumelhart & Hinton, 1986), computes exact gradients for deterministic and differentiable objective functions but is not applicable if there is stochasticity or non-differentiable functions involved. Two sources of non-differentiability in computational graphs are:
1. Stochastic nodes:
    - Stochastic nodes can be reparameterized to decouple the stochastic part (needs a re-parameterizable distributions)
        + exponential distributions can be reparameterized easily with Normal distribution as the noise source. 
        + via the Gumbel trick (choosing a category, after injecting Gumbel noise), any categorical distribution is re-parameterizable.

2. Discrete nodes:
    - we can enable backpropagation in the discrete nodes by a continuous relaxation \citep{Bengio2013,Jang2016,Maddison2016,Tucker2017}. The idea is that a hard node in the computational graph is replaced with a soft one.

Stochasticity is the case when we want to calculate the gradient of an expectation of a function with respect to parameters $$\theta$$ i.e. $$ \nabla_\theta (E_q(z) [f(z)])=\nabla_\theta( \int q(z)f(z))$$ . An example is ELBO where gradient is difficult to compute since the expectation integral is unknown or the ELBO is not differentiable. If we have discrete distributions, both of above situations co-occur. These two are used in Gumbel-Softmax to make a latent variable model with discrete latent variables. 
    - Authors reparameterize a categorical latent variable with Gumbel trick to solve stochasticity. Then since the re-parameterization is still dicrete, they relax it by sampling under the softmax approximation.

- a few gradient estimation techniques: (REINFORCE [williams1992], NVIL [mnih2014], VIMCO [mnih2016]

# permuations:
## with Gumbel-Sinkhorn 

Although continuous relaxations of discrete variables work well for categorical variables, they do not directly apply to discrete objects possessing well defined combinatorial structure, such as a graphs or permutations

- They simply treat ordering the peices of the puzzle as classification of peices to the number of possible slots in a way that there aren't any conflicts. If there are N pieces, there are N spots. So it comes down to doing an N-class classification problem N times in a way that each piece is put in just one place. 

- If we make a softmax vector of N-dim for classifying each puzzle peice to a location, there might be peices that are assigned to the same location. 

- The matching operator solves this by forming a matrix of sofmaxes where both rows and cols sum to 1. It is simply a softmax applied to the stack of logits from all pieces of the puzzle.  Basically, first normalizing every row of the matrix of all peices (assigning each piece to a spot) and then normalizing applied to columns (removing inconsistent assignments) using  softmax operator. 

- This can be extended to a stochastic case of having a distribution over the permutations via re-parameterization of Gumbel-Sinkhorn operator to perform computations with stochastic nodes. If we map this to the case of a semi-supervised VAE, instead of using Gumbel-Softmax for a single discrete latent variable(classification label Y), we now have a bunch of N latent variables (the number of puzzle pieces). Instead of sampling from Gumble distribution and applying softmax for one discrete variable as in Gumbel-softmax, we sample from Gumble, add to logits and then apply the matching operator to softly sample from a set of discrete variables in a way that assigns each peice to only a single spot.

Learning permutation latent variable models requires an intractable marginalization over the combinatorial objects. 

The paper approximates discrete maximum-weight matching using the continuous Sinkhorn operator. Sinkhorn operator is attractive because it functions as a simple, easy-to-implement analog of the softmax operator. Gumbel-Sinkhorn is an extension of the Gumbel-Softmax to distributions over latent matchings.
https://openreview.net/pdf?id=Byt3oJ-0W

Notice that choosing a category can always be cast as a maximization problem (e.g. argmax of a softmax on categories). Similarly, one may parameterize the choice of a permutation $$P$$ through a square matrix $$X$$, as the solution to the linear assignment problem with $$P_N$$ denoting the set of permutation matrices. The matching operator can parameterize the hard choice of permutations with an argmax on the inner product of the matrix $$X$$ and the set of $$P_N$$ matrices i.e. $$M(X) = argmax <P,X>$$. They approximate $$M(X)$$ with the Sinkhorn operator. Sinkhorn normalization, or Sinkhorn balancing iteratively normalizes rows and columns of a matrix.

for a sentence of words:
- an LSTM classifies every token in the sequence to one of n location as 1-of-n. 
- we put all the outputs (each 1xn) of the LSTM corresponding to the $$n$$ input token (1-of-n vectors) into a list that forms an $$nxn$$ matrix of logits called log_alpha. 
- This matrix should be one-hot vectors that assign each input to only one location. Since such a matrix is categorical and non-differentiable, it is replaced with its continuous relaxation.
    + one can re-parameterize any categorical distribution by adding noise to the distribution parameter and then choosing a category. However, the marginalization  |Y| = N!.
        * $$\epsilon$$ : sample a few nxn noise matrices from Gumbel distribution (i.e. sample from uniform and apply 2 consecutive $$-log$$) 
        * noisy_log_alpha =  average ( $$\epsilon$$ + log_alpha. 
        * apply the sinkhorn operator to noisy_log_alpha for N_iter times to slove inconsistencies and approximate a permutation matrix.
        * sinkhorn simply applies softmax first to rows and then to columns for n_iterations. softmax on a row is $$\frac{exp(noisy_log_alpha_{i})}{sum(exp(noisy_log_alpha_{i}))}$$.
    + re-order original input using the permutation matrix.
- Feed this sampled factorization matrix to an LSTM decoder to reconstruct input tokens but with new ordering. 
    + we compare the re-ordered and reconstructed input sequence with target sequence ordering to calculate loss. 

- Note that the matching operator is not differentiable! It searches over all  n!n!  possible permutation matrices and returns the one with the maximum Frobinius inner product. Sinkhorn operator is the differentiable approximation of the matching operator. to be able to perform approximate posterior inference to enable learning of a probabilistic latent representation of permutations, rather than categories. Argmax function is reparameterized with Gumbel-softmax and matching operator is reparameterized using Gumbel-Sinkhorn. 

## Learning to compose words into sentences using RL
- composing word embeddings to sentence embeddings. 

- convert a liearized sentence to a tree representation by a sequence of a set of actions using Reinforce. 

- a list of actions(shift-> put word into stack as a node, reduce -> combine top two nodes in stack into a new node and put into stack)

## generative models for graphs:
- n^2 adjacency matrix for a graph
- the number of nodes vary between graphs
- ordering is factorial possibilities

- decomposes the process to two RNNs
    + one keeps track of graphs
    + one generates nodes to insert
    + 

## Variational permutation inference
with Reparameterizing the Birkhoff polytope for variational permutation inference



## message passing neural network (MPNN)
- MPNN is an unstructured graph-networks, where the nodes are neurons and the edges are like a synapse between two neurons with a particular (and modifiable) strength
- When the connection strengths are updated in a rule-based manner until they converge, these graph-based networks can be used to perform complex computation with very high data throughout


## [Grammer VAE](https://arxiv.org/pdf/1703.01925.pdf)
key observation: frequently, discrete data can be represented as a parse tree (i.e. a sequence of production rules) using a context-free grammar. why not directly encode/decode to and from these parse trees, ensuring the generated outputs are always valid. 
    1. Take a valid sequence, parse it into a sequence of ordered and reversible production rules (i.e. a parse tree).
    2. Assign a one-hot representation to each production rule, feed to an LSTM encoder, learn a hidden distribution on the latent space of a VAE. 
    3. LSTM decoder generates a sequence of production rules.
    4. Do an offline semantic check to weed out nonsensical sequences. 
    5. applying sequence of production rules in order to convert to grammatically correct strings. 

Grammars exist for a wide variety of discrete domains such as symbolic expressions, standard programming languages such as C, and chemical structures. A context-free grammar (CFG) is traditionally defined as a 4-tuple G = (V, Σ, R, S): V is a finite set of non-terminal symbols; the alphabet Σ is a finite set of terminal symbols, disjoint from V ; R is a finite set of production rules; and S is a distinct non-terminal known as the start symbol. The rules R are formally described as α → β for α ∈ V and β ∈ (V ∪ Σ)* with * denoting the Kleene closure. In practice, these rules are defined as a set of mappings from a single left-hand side non-terminal in V to a sequence of terminal and/or non-terminal symbols, and can be interpreted as a rewrite rule. Note that natural language is not context-free. Application of a production rule to a non-terminal symbol defines a tree with symbols on the right-hand side of the production rule becoming child nodes for the left-hand side parent. The grammar G thus defines a set of possible trees extending from each non-terminal symbol in V. produced by recursively applying rules in R to leaf nodes until all leaf nodes are terminal symbols in Σ. 

## [Syntax-Directed VAE for Structured Data](https://openreview.net/forum?id=SyqShMZRb)
The idea is to add an 'attribute grammar', called 'stochastic lazy attribute',  to convert the step 4 from grammer VAE (the offline semantic check) into online guidance for stochastic decoding.


### project ideas:
- taking models with non-differentiable part (i.e. DRAW, Neural Turing Machine, RL, etc) and applying a continuous relaxation like RELAX/REBAR

# VQ-VAE:
- Embed observation into continuous space
- Transform the Z into a discrete variable over k categories
    - make a lookup table embedding. 
    - find the nearest neigbour categorical embedding
- Take the embedding and feed to decoder
    + KL will become a constant and can be removed from objective.
    + gradients are straight through gradient estimator (pretend the non-diffrentiable part doesn't exist)
- The latent is a matrix of categorical variables
    + if we sample each categorical independently, the reconstruction won't have a coherent structure. 
    + Therefore, they used an autoregressive model (pixelCNN) to sample the categorical latent for generation. 

# DRAW
- use RNNs as encoder/decoder
- use spatial attention mechanism
    + main challenge is where to look
        * 1. using REINFORCE
        * 2. build a fully differentiable attention (like soft attention)
        * 3. what they did was to to sample from decoder, send it to encoder, 


# Attend infer repeat
- difference from DRAW is it's focus on learning an understandable representation compared to focus of DRAW on reconstruction
- Objective is ELBO. challenges:
    + the size of latent space is a random variable itself
    + mix of continuous and discrete latent random variables (presence, where, what)

# learning hard alignment with variational inference
- a seq2seq that can work in online setting. 
    + where the input seq comes in, in real time. 
    + TIMIT dataset (speech) 
- we want the model to attend to different parts of speach and decide if it maps to a phoneme (uses hard attention which is not differentiable)
- A bernouli decides at each input if we sould output or not. Therefore, the seq will be variable size. 
- used VIMCO gradient estimator. 

# Thinking fast as slow with deep learning and tree seach (AlphaGo Zero)
- why not just use REINFORCE (why use MCTS)?
    + reinforce is just average of reward times gradient of log of some policy.
    + we can only use differentiable policies
    + reinforce has high variance
    + 

- alpha-beta tree search is expensive. Monte carlo tree seach (MCTS) is and approaximation. 
    + selece nodes according to a huristic probability function (UCP function)
    + traverse the tree to get at leaf node:
        * if this node has not been explored, run simulation get reward
        * if you have, add child node to tree, run simulation from a random child
    + update upper confidence bound(UCP) values of node along path from leaf to node
- MCTS in acion:
    + selection 
    + expansion (add it's child)
    + simulation (get reward)
    + backprop


# GANs for text
- Adversarial loss (also happens in supervised learning when we optimize hyperparams)
    + convert the minmax optimization into a nested Bilevel optimization 
    + Actor-critic method combines policy and value learning and learns them simultaneously
    + formulate a GAN as an actor-critic method (generator-> plicy, discriminator -> value, state -> real/fake image, environment-> randomly gives real/fake image, reward -> real image label)
        * the critic cannot learn the causal structure of the environment


- Adversarial Autoencoder. For each minibactch:
    + First train a VAE
    + then take the encoder as generator of a GAN and compare it with samples from a prior you want to impose. The discriminator will try to tell apart these two
    + can be done on SSVAE to encourage the categorical be closer to one-hot. 
        * train auto-encoder
        * train the two discriminators (one for continuous, one for discrete)
        * additional classification loss

# Hierarchical Multiscale RNNs (HM-RNN)

- It's a stacked LSTM where instead of performing a cell UPDATE at every time step, and at each layer, we choose from 3 operations to do on cell state (memory) at each time step and each layer. This learns a multiscale and hierarchical representation (e.g. characters, words, etc). Each layer has a parametrized binary boundary detector, Z, that is learned to decide the boundaries at each layer and choose an operation on the cell state (memory).
    + UPDATE: update the previous hidden state, $$C_t = f*C_{t-1} + i*C^{~}_{t}$$, as soon as the boundary in the below layer is seen i.e. $$Z^{l-1}=1, Z^{l}=0$$.
    + COPY: copies previous hidden state to present, $$C_t = C_{t-1}, h_t = h_{t-1}$$, until the boundary in layer below is seen i.e. $$Z^{l-1}=0, Z^{l}=0$$. 
    + FLUSH: zeros out the memory of the present hidden state, $$C_t = i*C^{~}_{t}$$, if the boundary in the current layer is seen i.e. $$Z^{l-1}=0, Z^{l}=0$$
- In HM-RNN, at each time step, (1) operation is selected based on the hidden states of the below layer at the same time or the previous time step of the same layer and (2) operation executed (e.g., UPDATE, COPY, FLUSH). 
- an UPDATE at a layer can happen only after *at least* one UPDATE is performed at its previous layer
- The binary boundary parameter is not differentiable, so they used straight through gradient.
https://arxiv.org/pdf/1609.01704.pdf



# project idea:
- Take [DRAW](http://kvfrans.com/what-is-draw-deep-recurrent-attentive-writer/), [apply](https://github.com/chenzhaomin123/draw_pytorch) it to text instead of a seq2seq with attention.
    + Usually seq2seq with attention is used for text translation. Why not use DRAW instead?
    + DRAW uses a set of NxN Gaussians filters (N neighboring horizontal Gaussinas $$F^x$$ and N neighboring vertical Gaussians $$F^y$$ ) to map an AxB image into an attended NxN image patch $$Patch_{NxN} = (F^x_{NxA} . IMG_{AxB} . F^y_{BxN}) $$.

- Compare it with only attention networks i.e. [transformer network](https://mchromiak.github.io/articles/2017/Sep/12/Transformer-Attention-is-all-you-need/).
    + The Transformer model uses soft attention with softmax. If we could instead use linear attention (which can be converted into an RNN that uses fast weights), we could use the resulting model for RL. Specifically, an RL rollout with a transformer over a huge context would be impractical, but running an RNN with fast weights would be very feasible. Your goal: take any language modeling task; train a transformer; then find a way to get the same bits per character/word using a linear-attention transformer with different hyperparameters, without increasing the total number of parameters by much. Only one caveat: this may turn out to be impossible. But one potentially helpful hint: it is likely that transformers with linear attention require much higher dimensional key/value vectors compared to attention that uses the softmax, which can be done without significantly increasing the number of parameters.

- Can we also attend to different dimensions of the embeddings?
    + Conceptually, if we are able to attend to different parts of the embedding vector, we may be able to more precisely modulate the embedding vectors.
    + For example, imagine a 2d embedding vector where one dimension has learned to embed gender while the other has learned to embed age. If we attend to the first dimension and only change that, we can effectively only change the gender of the word without changing the age. 

- Attention is essentially a distribution over the context. Why do we need to use an MLP to find the attention weights? Why can't we use a parameterized distribution instead? This distribution can be scaled and quantized to desired length of context. 
    + This way, the length of the context is decoupled from the number of parameters required for the attention mechanism.
    + Simplest case is a normal distribution. but normal PDF is uni-modal, which only admits local attention. If we want to attend to points far away in the context, we need a multi-modal distribution. Can we transform a normal distribution to arbitrary shape using autoregressive flow. 

- Use it for surface realization with the [Sinkhorn operator](https://github.com/google/gumbel_sinkhorn/blob/master/sinkhorn_ops.py) on the attention matrix.
    + Sinkhorn operator normalizes the attention matrix on both rows and columns.

- taking models with non-differentiable part (i.e. DRAW, Neural Turing Machine, RL, etc) and use a continuous relaxation like [RELAX/REBAR](https://github.com/pemami4911/REBAR-pytorch/blob/master/rebar_toy.ipynb) for gradient estimator. 
    + Use a continuous relaxation of the hard attention in DRAW.

- Similar to [multiscale hierarchical LSTM](https://github.com/HanqingLu/MultiscaleRNN) perform one of UPDATE, COPY, or FLUSH on the LSTM cell based on a learned boundary variable. 
    + This way, the encoder can put words far from each other into a single level while the original model only can combine neighbouring words.

- Then try developing the [inverse DRAW](https://openai.com/requests-for-research/#inverse-draw) and apply it to text.
    + [Inverse DRAW](https://openai.com/requests-for-research/#inverse-draw) is similar to neural turing machine.
    + gradient of a custom [function](http://pytorch.org/tutorials/beginner/pytorch_with_examples.html#pytorch-defining-new-autograd-functions)