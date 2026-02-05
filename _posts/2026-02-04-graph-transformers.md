---
title:  "What are Graph Transformers ?"
date:   2026-02-04 10:44:09 -0800
categories: projects
permalink: :categories/graph-transformers.html
hero_image: assets/images/2026-02-04-cover.png
---
{% include mathjs.html %}

From social networks to urban transit, graphs are everywhere. But beyond those obvious candidates, graphs can elegantly describe gridded data, where each cell's value is dependent on its neighbours. This perspective opens a tremendous range of applications in scientific computing. In computational fluid dynamics, models are based on differential equations, such as the Navier-Stokes equations, which describe the behavior of the fluid through advection, diffusion and reaction. Numerical models solve them by approximating local gradients, and the value of each cell is updated at each time step from the value of a finite number of neighbors. This approach, despite it computational load, works well in idealized cases. However, as soon as complexity in the form of viscosity or turbulence arises, equations fall short in describing the entire picture. Modelling relies on empirical parametrizations to describe viscosity coefficients for instance.

Such cases are perfect candidates for graph transformers. Cells can be represented by nodes, and their connectivity by edges, thus creating regular graphs. They can be trained to not only approximate the gradient contribution from neighboring nodes, but also account for phenomenon that were previously parametrized. But how do graph transformers work in the first place ?

Let’s first take it back to the classical transformer architecture. *Transformers* are a type of neural networks specialized in sequence modelling [^1]. They iteratively transform a sequence of data into another, while mixing information between elements, or tokens, of the sequence via _self-attention_ [^2]. A transformer typically has several attention heads through which the data goes in parallel. In each head, the input data is transformed by going through a _transformer block_ a given number of time. A _transformer block_ defines a series of deep learning layers. Typically, it includes layer normalization, self-attention and MLP layers. The output is given by concatenating the output of the different heads, before an output transformation can be applied. Graph Transformer use a variation of _self-attention_ referred to as _sparse attention_. [Figure 1](#fig-transformer) shows where sparse attention is implemented in a classical transformer architecture. 

<figure id="fig-transformer" style="text-align: center; max-width: min(1200px,90%); margin: 2em auto;">
  <img src="/assets/images/2026-02-04-transformer.svg" alt="Graph Transformer" style="display: block; margin: 0 auto; max-width: 100%;">
  <figcaption style="margin-top: 0.5em; font-size: 0.9em;">
    <strong>Figure 1.</strong> Schematic representation of a simple Graph Transformer. (a) The data is input to several heads in parallel, (b) In each head, the data goes through several transformer blocks, (c) The sparse attention, as employed in Graph Transformers, is incorporated inside the transformer block. Arrows on the right side represent the summation of a residual.
  </figcaption>
</figure>

During self-attention, the output <span>$$Z$$</span> is a weighted average of the <span>$$T_{in}$$</span> input tokens and is computed from the query <span>$$Q$$</span>, keys <span>$$K$$</span> and values <span>$$V$$</span> matrices :

<div>
<span>$$z_i = \sum_{j=1}^{T_{in}} p_{i,j} \cdot v_j$$</span> with  <span>$$\sum_{j=1}^{T_{in}} p_{i,j} = 1$$</span>
</div>

forming a valid probability distribution. This can be written in matrix form as:

<div>
    $$Z = PV = softmax(\frac{Q K^T}{\sqrt{d_k}}) \cdot V$$
</div>

The summation is taken over the entire input tokens sequence, making this multiplication very large, and potentially a computational bottleneck in the model. *Sparse attention* is a variant of the self-attention mechanism, where tokens only attend to a subset of other tokens. This subset can be defined in several manners, such as choosing a random subgroup, or attending to neighboring tokens only. Graph transformers employ sparse attention and define token attendance by their relative position on the graph. The softmax operation only sums the neighbouring tokens. This process is sometimes referred to as window attention. [Figure 2](#fig-sparse-attention) illustrates this process in the one-dimensional case.

<figure id="fig-sparse-attention" style="text-align: center; max-width: min(1200px,90%); margin: 2em auto;">
  <img src="/assets/images/2026-02-04-sparse-attention.svg" alt="Sparse Attention" style="display: block; margin: 0 auto; max-width: 100%;">
  <figcaption style="margin-top: 0.5em; font-size: 0.9em;">
    <strong>Figure 2.</strong> The pink shadow represents tokens respectively attended to by each other token for (a) Regular Attention, (b) Sparse Attention with a window of size 3.
  </figcaption>
</figure>

Additionally, the updated cell value also depends on edges attributes, such as length and direction, of the edge that links it to its neighbor. This method is referred to _edge-conditioned convolution_ [^3]. The attention computation becomes :

<div>
<span>$$z_i = \sum_{j \in \mathcal{N}_i} p_{i,j} \cdot v_j \cdot h_{\theta}(e_{i,j})$$</span> 
</div>

with <span>$$h_{\theta}(e_{i,j})$$</span> a function of edge attributes <span>$$e_{i,j}$$</span> between nodes <span>$$i$$</span> and<span>$$j$$</span>. This function can be a learnt function, such as a simple MLP. <span>$$\mathcal{N}_i$$</span> denotes the neighbourhood of cell <span>$$i$$</span>. This variant allows the transfer of information to be conditional on the graph structure.

In conclusion, graph transformers are constrained by which input fields are connected during the self-attention layer [^4]. This eliminates the need for positional encoding, as all positional information is contained in the existing edge connections and attributes. The final architecture induces a strong _relational inductive bias_ as it assumes the structure of the data [^5]. In the context of computational fluid dynamics, it is equivalent to stating that the fluid at a point directly depends on the local fluid. Repeated passes of the transformer block allow the account of longer range spatial dependencies. The crux here is to have sufficient high-quality data to train such model, and a reliable way to assess its performance. In such applications, graph transformers are a prime example of how prior knowledge about the problem can be leveraged to apply an inductive bias in deep neural networks, thus reducing the computational and data requirements.

[^1]: A. Vaswani et al., “Attention is all you need,” Advances in neural information processing systems, Vol. 30, 2017.
[^2]: M. Jaggi and N. Flammarion, Class notes from _CS-433 Machine Learning_. Computer Sciences, EPFL, 2023.
[^3]: M. Simonovsky and N. Komodakis, “Dynamic Edge-Conditioned Filters in Convolutional Neural Networks on Graphs.”. Available: <a href="https://arxiv.org/abs/1704.02901">https://arxiv.org/abs/1704.02901</a>
[^4]: C. K. Joshi, “Transformers are Graph Neural Networks.”. Available:<a href="https://arxiv.org/abs/2506.22084">https://arxiv.org/abs/2506.22084</a>
[^5]: P. W. Battaglia et al., “Relational inductive biases, deep learning, and graph networks.”. Available: <a href="https://arxiv.org/abs/1806.01261">https://arxiv.org/abs/1806.01261</a>