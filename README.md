## BitTensor Network Daemon

<img src="assets/mycellium.jpeg" width="1000" />

## Table of Contents

- [Overview](#overview)
- [To-Run-Locally](#to-run-locally)
- [To-Run-Testnet](#to-run-testnet)
- [Method](#method)
  - [Representation](#representation)
  - [Incentive](#incentive)
  - [Attribution](#attribution)
  - [Emission](#emission)
  - [Market](#market)
- [BitTensor1](#bitTensor1)
  - [Nucleus](#nucleus)
  - [Dendrite](#dendrite)
  - [Synapse](#synapse)
  - [Metagraph](#metagraph)
- [References](#references)
- [License](#license)

---

## Overview

BitTensor allows a new class of Machine Learning model which trains across a peer-to-peer network. It enables any computer and any engineer in the world to contribute to its training.

The nature of trust-less computing necessitates that these contributions are driven by incentives rather than by direct control from any one computer. We use a digital token to carry that incentive signal through the network. The magnitude of this incentive is derived from a p2p collaborative filtering technique similar to Google's Page Rank algorithm.  

As a network product we focus on learning unsupervised multi-task representations, starting from language and extending the network to image and speech. The result is a sufficiently general product which is useful to a large number of downstream stake holders.

The lack of centralization allows the structure to grow to an arbitrary size across the internet. Both the cost and control of the system is distributed and the network's informational product is priced into the reward token's value.

When run, this software integrates your computing power into a p2p network and rewards you with an EOS-based digital token for your contribution.

## To-Run-Locally
1. [Install Docker](https://docs.docker.com/install/)

```
$ git clone https://github.com/unconst/BitTensor
$ cd BitTensor

# Run a test EOS blockchain.
$ ./start_eos.sh

# Run Node 1.
$ ./bittensor.sh

# Run Node 2.
$ ./bittensor.sh

...

# Run Node N.
$ ./bittensor.sh

```

## To-Run-Testnet

1. [Install Docker](https://docs.docker.com/install/)
1. [Make account on Digital Ocean](https://www.digitalocean.com/)
1. [Make a Digital Ocean API key](https://cloud.digitalocean.com/account/api/tokens)

```
$ git clone https://github.com/unconst/BitTensor
$ cd BitTensor

# Run a Remote Node
$ ./bittensor.sh --remote --token $DIGITAL_OCEAN_TOKEN --eosurl http://142.93.177.245:8888


# Run a Local Node
$ python src/upncp.py --port 9091  // To punch a hole in your router.
$ ./bittensor.sh --port 9091 --eosurl http://142.93.177.245:8888

```

## Method
<img src="assets/brain.png" width="1000" />


### Representation

We require a Machine Intelligence problem which is general enough to interest a diverse set of stake holders. The problem should be sufficiently difficult to warrant a collaborative approach and the data used to train it should be ubiquitous and cheap.

For our purposes, we choose unsupervised representation learning [5, 6, 7, 9, 10, 14, 24, 25], where the network trains on large-scale unlabeled corpora to learn a feature basis ('representation') of inputs. These representations are a form of product, which uniquely identifies and disentangles the underlying explanatory factors of an input. This is a widely used product and arguably a fundamental task in the development of an AI which understands the world around it. [26]

We initially focus on Language Representation from text, where components build an understanding of natural language and will respond to queries in pure unicode strings with a vector representation. For the sake of generality, we leave tokenization and parsing to each component and limit outputs across the network to fixed length vectors.

<p align="center"> "raw natural language text"  ---> [ f(x) ]  --->  [fixed length representation] </p>

This high-level paradigm is shared between a range of training methods which can be combined and shared across the network. These include:
<ul>
<li>BERT [5] which uses multi-word masking strategies.</li>

<li>MT-DNN [14] which combines pre-training with multi-task knowledge transfer.</li>

<li>GPT-2 [7] which adds task information from an independently sourced question-answering dataset.</li>

<li>XLM which uses language embeddings to improve performance for cross-lingual tasks.</li>

<li>ERNIE [10] which adds entity/phrase level masking and</li>

<li>XLNet [9] which implements learning across all mask permutations.</li>
</ul>

We intend on running network components which use these algorithmic approaches and others. We will also take advantage of differing training sets which range from freely available datasets, translation corpuses, reddit crawls, wikipedia entries and books. Thankfully langauge is ubiquitous and cheap so there is no need to worry about protecting datasets. In addition, the domain arguably contains the total of articulated human knowledge, so it is sufficiently deep.

### Incentives

We are extending previous work in Neural Network training by moving the training process from a datacenter into a decentralized computing domain where no computer is privileged, there is no single user of the network, and some computers may be incompetent, offline, or malicious. Because of these constraints, we must use _incentive_ to draw our compute nodes into line. That incentive should drive them to stay online, to learn well, and to train in alignment with a useful network product.

To begin, we assume a standard training scheme for the ith p2p component. It contains a dataset M, with targets X and labels Y, and is attempting to ﬁt a function that predicts the output from the input, yˆ = f(x), by minimizing the loss on the output of the model,

  <p align="center"> ith loss = Li = Ep[ Q(f(x), x) ]. (1) </p>

Where Q is a loss calculation between the targets and outputs and Ep is the expectation over a subset P of our full dataset M. Then a global objective for the entire network, G, should be to minimize the sum over each local objective.

Furthermore, it makes sense to scale our global objective with a stake vector S. This binds the concept of value into the network training process -- attaching more stake towards a loss function directly changes the global objective. Then for Si, the stake attached to the ith loss, we have:

  <p align="center"> global objective = G = min Σ Si ◦ Li . (2) </p>

### Attribution
We wish to mint new tokens to components in-proportion to their contribution to optimizing the _Global Objective_. We do this by asking what it would cost, in terms of loss, to prune each node from the network.

<p align="center"> ∆Lj = the change in global objective w.r.t removal of single component j. </p>

An approximation of ∆Lj can be attained by working first with each local estimation, ∆Lij, between a single loss Li, and a connected component j, and then transitively deriving all pair-wise paths through the network. We can calculate ∆Lij using a 2nd order approximation of the loss with respect to its input activations aj, and a change ∆aj reflecting the removal of the component j.

<p align="center"> ∆Lij = L(aj + ∆aj) − L(aj) ≈ g' ∙ ∆aj  +  1/2 ∆aj ∙ H ∙ ∆aj (4) </p>

Where g is the gradient of the loss which vanishes if we assume the loss is at a local optimum and the remaining term is the Hessian which can be approximated using an expectation over our training subset P:

<p align="center"> H ≈ Ep [ ( ∂L(x)/∂aj ) ^2] (6)</p>

This approximation becomes exact when P and M are close and Eqn. (6) can be viewed as an empirical estimate of the Fisher information of our activations. [17] We can use N data points to estimate our pruning signal *∆Lij*.

<p align="center"> gn = ( ∂L(xn)/∂aj )^2. </p>
<p align="center"> ∆Lij = 1/2N   ∆aj   Σn gn^2 (7)</p>

This information is available during the backward pass of computing the network’s gradient and the pruning signal can therefore be found at little extra computational cost.

### Emission

The totality of ∆Lij scores describe a directed weighted graph G = [V, E] where for each edge eij in E we have a the weight *∆Lij* associated with the connection between component i and j. ∆Lij is a local attribution and we would like to determine the global attribution for a node i, ∆Li. This score should be a sum over every pair-wise path through the graph weighted by stake *Si*.

<p align="center"> ∆Lj = Σi Si x ∆Lij </p>

We can derive all pair-wise paths by applying the chain rule to (7) to find the following transitive relation:

<p align="center"> Given ∆Lij and ∆Ljk </p>
<p align="center"> ∆Lik = ∆Lij x ∆Ljk (8) </p>

Which is intuitive, following immediately from the notion of transitive contribution: If a component i contributes to component j, it should multiplicatively contribute to the components using j since they are compositions of its parent.

As a corollary of (8) global attribution scores for component _i_ can be calculated with a Power Iteration over the adjacency matrix described by G.

<p align="center">  ∆L(t+1) = G ∙ ∆L (t). (9) </p>

This is similar to the EigenTrust algorithm [23] or Google Page Rank [1], but with Fishers Information scores instead of recommendations or web links. We emit new tokens within the graph to components with high contribution scores proportionally. The entire calculation is done using a consensus engine which ensures that the specifics of token emission stay fixed and where the state of G is held global so that every node can see how they are attaining token emissions.

We note that this form of local attribution emission scheme may be similar in nature to the propagation of the neurotrophic factor BDNF in the brain.[2]

Below is an approximate method written using python-numpy:
```
N_BLOCKS = 100
DEPTH = 100
TOKENS_PER_BLOCK = 50

def get∆L ():
  ∆L = np.multiply (S, Lii)
  T = np.matmul (Lij, S)
  for _ in range(DEPTH):
      ∆L += np.multiply(T, Lii)
      T = np.matmul (Lij, T)

  ∆L = ∆L / np.linalg.norm(∆L, 1)
  return ∆L

def emit():
  # Stake vector.
  S = [9.  8.  7.  6.  5.  4.  3.  2.  1.  0.]

  # i to i attributions
  Lii = [0.6 0.9 0.4 0.5 0.5 0.5  1.  1.  1.  1. ]

  # i to j attributions
  Lij = [[0.  0.1 0.3 0.2 0.  0.  0.  0.  0.  0. ]
    [0.1 0.  0.  0.1 0.  0.  0.  0.  0.  0. ]
    [0.1 0.  0.  0.2 0.  0.  0.  0.  0.  0. ]
    [0.2 0.  0.3 0.  0.  0.  0.  0.  0.  0. ]
    [0.  0.  0.  0.  0.  0.5 0.  0.  0.  0. ]
    [0.  0.  0.  0.  0.5 0.  0.  0.  0.  0. ]
    [0.  0.  0.  0.  0.  0.  0.  0.  0.  0. ]
    [0.  0.  0.  0.  0.  0.  0.  0.  0.  0. ]
    [0.  0.  0.  0.  0.  0.  0.  0.  0.  0. ]
    [0.  0.  0.  0.  0.  0.  0.  0.  0.  0. ]]

  # Loop over blocks.
  for _ in range(N_BLOCKS):        
    ∆L = get∆L()                          // Modified EigenTrust
    E = ∆L * TOKENS_PER_BLOCK             // Emission this block
    S = S + E                             // Stake update.
```


### Market

Our p2p network is composed of a set of functions, the ith of which produces an output yi given an input x as follows: yi = fi(x, d0(x), d1(x), ..., dn(x)), where d0 to dn are downstream compositions of the ith. Inputs to each function are variable length unicode strings and outputs are fixed length representation vectors. Reflexively, the ith function is one compositional component to a set of upstream components, u0, u1, ... uj ... uN. It may also be the compositional client to its own loss function, L(y, x).


<p align="center"> <img src="assets/UpDn.png" width="500" /> </p>


A finite quantity of stake, sj is being allocated to each loss term in the network. This is global knowledge which when combined with local ∆Lij calculations and the deterministic emission system described above, produces a stream of newly minted tokens for each component in the network. The magnitude of the stream from upstream jth component to the downstream ith is wji.

The ith component is making discrete SGD steps to its parameters θ. These updates are made w.r.t gradient information passed from upstream components. We assume SGD steps are being made in proportion to individual stream scores wji such that the next parameter update is a scaled summation of the upstream gradients ∆θ1, ∆θ2 ... ∆θN:

<p align="center"> ∆θ = ∆θ1 (w1i/Σjwji) + ∆θ2 (w2i/Σjwji) + ... + ∆θN (wNi/Σj wji) (10) </p>  

For simplicity we imagine this as a single update over a single example, but in reality these updates are over batches of different size and w.r.t different text inputs. This construction is a proportional allocation game. Where from (4) we know the utility for the jth component:

<p align="center"> ∆Lj= L(θ + ∆θ) − L(θ) ≈ g' ∙ ∆θ  +  1/2 ∆θ ∙ H ∙ ∆θ (11) </p>

and ∆Lj is strictly increasing and continuous in wji. We have by [--] that there exists a unique optimal solution to bids wij where the sum of weighted losses (global loss) is minimized. Omitting the utility function overlap i.e. gradient steps may harm or help each other. We know from [--] that the market is at least 75% percent efficient. And if we introduce a Vicker Clark Groves (VCG) pricing method then the Nash Equilibrium is optimal. With VCG pricing the optimal bidding for each component is its raw utility, or exactly ∆Lij.

----


## BitTensor 1.

<img src="assets/brain_engineering_diagram.png" width="1000" />

Above: An Engineering diagram of the brain. For inspiration.

```

                                     [EOS]
                                       |
                                  [Metagraph]
                               /       |       \
                    ----------------------------------------
                  |                  Neuron                  |
                  |                                          |
                  | [Dendrite] ---> [Nucleus] ---> [Synapse] |
                  |                                          |
                  |                                          |
                    ----------------------------------------
                               \       |       /
                                     [Main]
```

###### Nucleus
The main Tensorflow graph is defined and trained within the Nucleus object. As is, the class is training a self supervised word-embedding over a dummy corpus of sentences in text8.zip. The result is a mapping which takes word to a 128 dimension vector, representing that word while maintaining its semantic properties.

Although subject to future change, this problem serves as a good starting place because its generality and ubiquity within Artificial intelligence. In future versions of this code, this will be expanded to include sentence and paragraph embeddings, speech, image and video embeddings with the goal of training the network for general multitask.

###### Dendrite
During training the Nucleus interacts with the rest of the network through its Dendrite. The Dendrite maintains connections to upstream nodes making asynchronous calls using GRPC, and passing serialized Tensor protocol buffers along the wire.

During validation and inference the Dendrite is cut from the model and replaced by submodules which have been trained through distillation to approximate the incoming signals from the rest of the network.

###### Synapse
This inference graphs being produced in training are served by the Synapse object. The Synapse is responsible for upstream connections. It is responsible for rate limiting, and through this,  negotiating for higher attribution within the Metagraph.

Since the Synapse object is merely serving the inference graph, it is mostly detached from the Nucleus and Dendrite during training, only communicating with these objects by pulling the latest and best inference graph from the storage directory.

###### Metagraph
The Metagraph object acts as an interface between the EOS blockchain and the rest of the neuron. Through the Metagraph, this node can post updated attributions and call timed token emission (which releases newly mined tokens) The Metagraph object also serves as a de-facto DHT which removes the need for a gossip protocol used by many standard p2p applications Bitcoin and BitTorrent not withstanding.

###### EOS
The EOS contract is separate from Dendrite. Nucleus, Synapse and Metagraph objects during execution. During testing, this class is run on a local EOS instance, but during production the contract is running in a decentralized manner across the EOS network.  


## Control Flow
```
 Upstream           Downstream           Synapse            Dendrite              Nucleus
    +                   +                   +                   +                   +
    |      FSpike       |                   |                   |                   |
    +-------------------------------------> X                   |                   |
    |                   |                   |                   |                   |
    |                   |                   |                   |                   |
    |                   |      FSpike       |                   |                   |
    |                   X <-----------------+                   |                   |
    |                   X - - - - - - - - - +                   |                   |
    |                   X <-----------------+                   |                   |
    |                   |                   |                   |                   |
    |                   |                   |                   |                   |
    |                   |      RSpike       |                   |                   |
    |                   +-----------------> o ----------------> X                   |
    |                   + - - - - - - - - - o - - - - - - - - - X (Concatenation)   |
    |                   +-----------------> o ----------------> X                   |
    |                   |                   |                   |                   |
    |                   |                   |                   |                   |
    |                   |                   |                   |      Spike        +
    |                   |                   |                   +-----------------> X
    |                   |                   |                   |                   |
    |                   |                   X <-------------------------------------+
    |      RSpike       |                   |                   |                   |
    X <-------------------------------------+                   |                   |
    |                   |                   |                   |                   |
    |                   |                   |                   |                   |
    |      FGrade       |                   |                   |                   |
    +-------------------------------------> X                   |                   |
    |                   |                   |                   |      Grade        |
    |                   |                   +-------------------------------------> X
    |                   |                   |                   |                   |
    |                   |                   X <-------------------------------------+
    |                   |                   |                   |                   |
    |                   |      FGrade       |                   |                   |
    |                   X <-----------------+                   |                   |
    |                   X - - - - - - - - - +                   |                   |
    |                   X <-----------------+                   |                   |
    |                   |                   |                   |                   |
    |                   |                   |                   |                   |
    |                   |      RGrade       |                   |                   |
    |                   +-------------------o-----------------> X                   |
    |                   + - - - - - - - - - o - - - - - - - - - X (Summation)       |
    |                   +-------------------o-----------------> X                   |
    |                   |                   |                   |                   |
    |                   |                   |                   |                   |
    |                   |                   X <-----------------+                   |
    |      RGrade       |                   |                   |                   |
    X <-------------------------------------+                   +                   +

```



## References

[1] The PageRank Citation Ranking <br/>
http://ilpubs.stanford.edu:8090/422/1/1999-66.pdf

[2] Brain-derived neurotrophic factor and its clinical implications <br/>
https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4697050/

[3] Attention is all you need <br/>
https://arxiv.org/abs/1706.03762

[4] Universal Language Model Fine-Tuning for Text Classification <br/>
https://arxiv.org/abs/1801.06146

[5] Bi-directional Encoder Representations from Transformers <br/>
https://arxiv.org/abs/1810.04805

[6] Googles Transformer-XL <br/>
https://arxiv.org/abs/1901.02860

[7] Open AI GPT2 <br/>
https://openai.com/blog/better-language-models/

[9] XLNet <br/>
https://arxiv.org/abs/1906.08237

[10] ERNIE: Enhanced Representation through Knowledge Integration <br/>
https://arxiv.org/abs/1904.09223

[11] RoBerta: A Robustly Optimized Bert Pre-training Approach <br/>
https://arxiv.org/abs/1907.11692

[12] Cross-lingual Language Model Pre-training <br/>
https://arxiv.org/pdf/1901.07291.pdf

[13] Outrageously Large Neural Networks: The Sparsely-Gated Mixture-of-Experts Layer<br/>
https://arxiv.org/abs/1701.06538

[14] One Model To Learn Them All<br/>
https://arxiv.org/abs/1706.05137

[15] AHaH Computing–From Metastable Switches to Attractors to Machine Learning<br/>
https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0085175

[16] Distilling the Knowledge in a Neural Network <br/>
https://www.cs.toronto.edu/~hinton/absps/distillation.pdf

[17] Faster Gaze Prediction With Dense Networks and Fisher Pruning <br/>
https://arxiv.org/pdf/1801.05787.pdf

[18] Overcoming catastrophic forgetting in neural networks <br/>
https://arxiv.org/abs/1612.00796

[19] Bitcoin: A Peer-to-Peer Electronic Cash System <br/>
https://bitcoin.org/bitcoin.pdf

[20] IPFS - Content Addressed, Versioned, P2P File System <br/>
https://arxiv.org/abs/1407.3561

[21] Self-Attention with Relative Position Representations <br/>
https://arxiv.org/pdf/1803.02155.pdf

[22] Generating Wikipedia by Summarizing long sequences <br/>
https://arxiv.org/pdf/1801.10198.pdf

[23] The EigenTrust Algorithm for Reputation Management in P2P Networks <br/>
http://ilpubs.stanford.edu:8090/562/1/2002-56.pdf

[24] Distributed Representations of Words and Phrases and their Compositionality <br/>
https://arxiv.org/pdf/1310.4546.pdf

[25] Skip-Thought Vectors <br/>
https://arxiv.org/abs/1506.06726

[26] Representation Learning: A Review and New Perspectives <br/>
https://arxiv.org/pdf/1206.5538.pdf

[27] Optimal Brain Damage  <br/>
http://yann.lecun.com/exdb/publis/pdf/lecun-90b.pdf

## License

MIT
