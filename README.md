## BitTensor Network Daemon

<img src="assets/mycellium.jpeg" width="1000" />

> Decentralized Machine Intelligence

## Table of Contents

- [Overview](#overview)
- [To-Run-Locally](#to-run-locally)
- [To-Run-Testnet](#to-run-testnet)
- [Why](#why)
- [How](#how)
- [Market](#market)
- [Incentive](#incentive)
- [Organization](#organization)
  - [Nucleus](#nucleus)
  - [Dendrite](#dendrite)
  - [Synapse](#synapse)
  - [Metagraph](#metagraph)
- [Word-Embeddings](#word-embeddings)
- [License](#license)

---

## Overview

BitTensor allows a new class of Machine Learning models to train across a peer-to-peer network. It enables any computer and any engineer in the world to contribute in training.

The nature of trust-less computing necessitates that these contributions are combined through incentive rather than direct control from any one computer. We use a digital token to carry that incentive signal through the network: where the magnitude of this incentive is derived from a p2p collaborative filtering technique similar to Google's Page rank algorithm.  

The lack of centrality allows the structure to grow to arbitrary size across the internet. Both the cost and control of the system is distributed. And the network's informational product is priced into the reward token's value.

When run, this software folds your computing power into a p2p network and rewards you with an EOS based digital token for your contribution.

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

<img src="assets/brain.png" width="1000" />

## Why

We believe Machine Intelligence, like Human Intelligence, is an a priori good. And yet, intelligence is power and power, if held in the hands of the few, will corrupt. Machine Intelligence should be democratized and made open source. Unfortunately, companies like OpenAI who claim this goal have failed in their mandate, opening up the algorithms but not access to the _intelligence_ itself.

This technology is being built to do this, while democratizing its ownership, and sharing its profit with any computer or any individual who deems it worthwhile to contribute.

Moreover, although democratization and openness are ethical values, we are relying on their practical use here: A free and open system with a large number of stake holders is also the most direct path towards our goal of producing Strong Machine Intelligence. The scale of the AI problem in front of us necessitates that we build it this way.

Why is this? Because decentralized computing approaches can harness the largest pool of computing power and the largest pool of collaborators: Every computer and every engineer can contribute to this system. We've seen how this worked for Bitcoin, the largest super computer in the world, BitTorrent, at one time, the largest bandwidth user across the globe, and open source Linux, the most widely used operating system in use today.

<img src="assets/Lightning.png" width="1000" />

Above: Bitcoin Lightning network nodes from late 2018.

## Introduction

In standard Machine learning setting, the training mechanism uses Back-propagation to minimize the loss on a dataset with respect to the weights, and at each step the model parameters wait for a signal from the global loss function before the next update.

This loss function centrality is prohibitive when the scale of those networks reach the scale desired by modern machine learning  -- or biological scale -- and necessarily so, when we are attempting to train a system which spans multiple computers connected across the web, as we are doing here.

To avoid this problem, each node within the p2p network is training against its unique 'local' loss function. Each participant  has it own dataset and it hypothetically working on problems which are disjoint. They do not wait for a global error signal to propagate backwards from a far off computer's loss. The local models can be split width-wise in each node, across compute hardware with rapid communication, while the local losses allow depth-wise, lateral, expansion of the network.

<img src="assets/kgraphbittensor.png" width="1000" />
Above: Local loss function training in a k-graph shaped NN organization.

## Training Method

To begin, we follow a standard training scheme within each component.  Node i, contains a dataset M, with targets X_i and labels Y_i, and is attempting to ﬁt a function that predicts the output from the input, yˆ = f_i(x), by minimizing the loss on the output of the model,

  <p style="text-align: center;"> L_i(ˆy, y) = Ep[ -logQ(f_i(x), x)]. (1) </p>

Where Q is the cross entropy between targets and outputs and Ep is the expectation over a subset P of our dataset M, our training set. Component i is also composing its inputs with downstream components in the network f_i = (d1 ◦ d2 ... dn ) and serving its model to upstream components, u1, u2 ... un = ( ... f_i, ... ) where the output of our model is the input to that model.

We are continuously receiving streams of gradient information from our local loss L_i and from upstream components u1, u2 ... un. These signals are combined using a in-feed queue which is last-in ﬁrst-out cyclic. Our component pulls greedily from this queue in an online fashion and by applying these gradients is optimizing its parameters θ by moving them in a direction which minimizes a combination of losses, namely L' = (L_i + Lu1 + Lu1 + Lu2 ... Lun).

Simultaneously we are sending streams of gradients downstream, to components (d1 ◦ d2 ... dn ), which carry information to receiving nodes on how to update their parameters θ by moving them in the direction of L'. -- intuitively, the structure mirrors the  differential graph architecture of TensorFlow itself, but for asynchronous operations stored on computers across the web.

## P2P Training.

We are extending previous work in local loss function training by moving the training process from a datacenter into a decentralized computing domain where no computer is privileged, there is no single user of the network, and some computers may be incompetent, offline, or malicious. In lieu of these constraints we must use _incentive_ to draw our compute nodes into line. That incentive should drive them to stay online, and to learn well, and to train in alignment with a useful network product.

We begin by defining our network problem. The global objective for the entire network, *L* is a summation over each local objective *L* = Σ Li. Our goal is to incent each component towards optimizing this global loss function. i.e. towards minimizing *L*.

To do this, we first augment our global loss with a stake vector *S*, e.g. *L* = *S* ◦ *L* such that the global loss function is scaled towards computers holding stake. This binds the concept of value into the network training process -- holding more stake directly changes the network's global loss.

Further more, we represent stake quantities in the form of a digital token using a decentralized compute and storage network known as a blockchain. The tokens can be transferred and bought by computers who wish to attain more power over the network's global loss.

## Incentive

In what follows we will explain how the network determines how these finite tokens are emitted. We wish to mint new tokens to compute nodes in-proportion to their contribution optimizing the global loss. This statement is equivalent to asking what it would cost, in terms of loss, to prune a single component, f_k, from the network.

We can understand that question using a 2nd order approximation to the loss function, L, around the current parameter values θ, with respect to a change in parameters d, reflecting the removal of single component.

  <p style="text-align: center;">g = ∇L(θ),    H = ∇2L(θ), (3) </p>
  <p style="text-align: center;">L(θ + d) − L(θ) ≈ g T d + 0.5 d H d (4) </p>

Where g is the gradient of the loss and H is the Hessian. Following this approximation, dropping the kth parameter (setting θk = 0) would lead to the following increase in loss:

  <p style="text-align: center;"> L(θ − θkek) − L(θ) ≈ − gkθk + 0.5 * Hkkθ^2k (5) </p>

where ek is the unit vector which is zero everywhere except at its kth entry, where it is 1. And if we assume that the current set of parameters is at a local optimum the 1st term vanishes leaving us with the Hessian of the loss:

<p style="text-align: center;"> Hkk ≈ EP [∂ / ∂θk log Qθ(fi(x) | x)^2] (6)</p>

This approximation becomes exact assuming the distribution of P and M are close and Eqn. (6) can be viewed as an empirical estimate of the Fisher information of θk. We can use N data points to estimate the increase in loss by removing our parameter θk:

  <p style="text-align: center;"> ∆k = 1/2N  θk Σgnk^2 (7)</p>

where gnk is the gradient of the parameter with respect to the nth data point.

## Composition Ranking.

For our compositional architecture, we wish to understand this pruning signal with respect to the inputs from neighboring components instead of individual parameters. Consider a single component, f_i, in our network and let ankj be the activation/input of the kth downstream component, dk, at activation location j for the nth datapoint. The gradient of the loss for the nth datapoint with respect to a removal of the kth component is,

<p style="text-align: center;">gnk = −Σ ankj ∂/∂ankj log Q(f(x)| x) (9) </p>

and the pruning signal between the ith component and the kth component is, Aik = 1/2N Σ gnk^2. Where gnk is now the nth gradient with respect to the kth input activations from component i. This information is available during the backward pass of computing the network’s gradient and the pruning signal can therefore be computed at little extra computational cost.

## Global Attribution

It would be sufficient at this point to sum each local attribution across each component and at depth one to derive a proxy score for each node. However, the network is recursively connected, and we are interested in the attribution scores for every pair-wise path through the graph -- each component has multiple children who have multiple children and we wish to know their contributions as well.

This follows immediately from the  notion of transitive value: If a component i values component j, it should also value the components contributing to j since component j is a composition of its neighbors. Applying the chain rule (9) we find the following recursive relation for transitive attribution scores: 

 <p style="text-align: center;"> Aik = Aij * Ajk. (10) </p>

We have each component i calculate the local weight importance Aij for all its neighbors (sub components).
posting them to a centralized contract running on a decentralized append only database (blockchain). This contract stores normalized weights as a directed weighted graph G = [V, E] where for each edge _eij_ in E we have a value _Aij_ associated with the connection between component _fi_ and _fj_. G is updated continuously by transaction calls from each working component as they train and as they calculate attribution scores for their neighbors in the network.

Global attribution scores for component _fi_ are calculated using a modified EigenTrust algorithm which iteratively updates the component use vector to an attractor point through multiple multiplications by the adjacency matrix described by G. i.e. a(t+1) = G * a(t). The attribution vector a, converges to G's largest Eigen Vector. 

## Emission System

We emit new tokens within the graph to components with high contribution scores. The entire calculation is done using a consensus engine which ensures that the specifics of token emission stay fixed and the state of G is held global so that every node can see how they are attaining token emissions.

below is an approximate method written below using python-numpy:
```
def attribution_simulation():

    # Stake vector.
    S = [9.  8.  7.  6.  5.  4.  3.  2.  1.  0.]

    # Loop-in edges.
    N = [0.6 0.9 0.4 0.5 0.5 0.5  1.  1.  1.  1. ]

    # Outgoing edges.
    M =[[0.  0.1 0.3 0.2 0.  0.  0.  0.  0.  0. ]
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
    n_blocks = 100
    for _ in range(n_blocks):        

        # Attribution calculation.
        depth = 100
        A = np.multiply(S, N)
        T = np.matmul(M, S)
        for _ in range(depth):
            A += np.multiply(T, N)
            T = np.matmul(M, T)

        # Emission calculation.
        tokens_per_block = 50
        A = A / np.linalg.norm(A, 1)
        E = A * tokens_per_block
        S = S + E
```


## Market Analysis

----

## Organization

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




---

## Word-Embeddings

A word-embedding is a projection from a word to a continuous vector space representation of that word, which attempts to maintain the semantics under the projection, For instance, 'King' --> [0,1, 0,9, ..., -1.2], such that 'King' - 'Queen' = 'Male'.

Word-embeddings are highly useful initital projections for a large number of Machine Learning problems. This makes them an ideal product for our network. They can also be trained in an un-supervised fashion which is a requirment for the local-loss approach described above.

During training we use a language corpus and find our projection by training a classifier to predict words in context. For example, the sentence 'The queen had long hair', may produce a number of supervised training examples ('queen' -> 'had'), or ('hair' -> 'long'). The ability to predict context requires an understanding of the relationships between words ('meaning' is a relational quality) -- a highly successful assumption in practice.

In the prototype node above, we train each NN using a standard skip gram model, to predict the following word from the previous, however any other embedding producing method is possible -- indeed, the goal should be diversity.

## License

MIT
