## OpenAI ChatGPT v1.2026.195 Dynamic.ANN.Topologies.md

**Q: Are there ANNs whose hidden layers expand and or deepen dynamically?**

**A:** Yes. Several Artificial Neural Network (ANN) families **grow their topology during training** or **execute variable effective depth during inference**.

| Family                                             | Dynamic behavior                                                                                                                                                                                                          | Persistent structural change? |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------: |
| **Cascade-Correlation**                            | Starts with no hidden layer and adds hidden neurons sequentially. Each new neuron can consume inputs and previous hidden outputs, progressively creating deeper dependency chains. ([NeurIPS Proceedings][1])             |                           Yes |
| **Dynamically Expandable Networks (DEN)**          | Adds, duplicates, or splits hidden units when new continual-learning tasks cannot be represented adequately by the existing network. ([arXiv][2])                                                                         |                           Yes |
| **NeuroEvolution of Augmenting Topologies (NEAT)** | Evolutionary mutations add connections and split connections with new neurons, increasing width and path depth over generations. ([Neural Network Research Group][3])                                                     |                           Yes |
| **Deep evolving networks**                         | Add and sometimes prune hidden neurons while processing data streams. DEVDAN, for example, adds and removes hidden units online. ([arXiv][4])                                                                             |                           Yes |
| **Gradient-guided growth**                         | Methods such as Firefly Neural Architecture Descent and neuron-growth algorithms determine when, where, and how many neurons to insert according to optimization signals. ([Proceedings of Machine Learning Research][5]) |                           Yes |
| **Adaptive Computation Time (ACT)**                | Repeats a recurrent computation until a learned halting condition is reached. Harder inputs receive more computational steps. ([arXiv][6])                                                                                |                            No |
| **Universal Transformer with ACT**                 | Repeatedly applies a shared Transformer block; different positions can undergo different numbers of refinement steps. ([arXiv][7])                                                                                        |                            No |

## Three distinct meanings of “dynamic”

### 1. Persistent width expansion

A layer initially containing (n) neurons becomes:

[
n \rightarrow n+k
]

The new neurons receive initialized parameters and become permanent parts of the model. DEN and constructive neuron-growth methods use this approach.

### 2. Persistent depth expansion

The network inserts an additional transformation:

[
h_{l+1}=f_{l+1}(h_l)
]

This can occur explicitly by inserting a layer or implicitly by adding a neuron whose inputs include previous hidden neurons, as in Cascade-Correlation.

### 3. Conditional effective depth

The physical parameter set remains fixed, but the network determines how many times to apply a block:

[
h_{t+1}=F_\theta(h_t,x)
]

until:

[
p_{\text{halt}}(h_t)>\tau
]

Thus, one input might receive three iterations and another twelve. This is **dynamic execution**, not structural growth.

## Important distinction

True architecture modification ordinarily occurs **during training or continual learning**, not during ordinary inference. Permanently adding a neuron requires:

* initializing its weights;
* connecting it into the computational graph;
* extending optimizer state;
* training or adapting the new parameters;
* preventing disruption of previously learned behavior.

Inference-time systems therefore more commonly use:

* adaptive recurrence;
* conditional routing;
* Mixture-of-Experts activation;
* early exits;
* shared-layer iteration.

These provide variable computation without physically reconstructing the network.

For an opcode-oriented learning architecture, the closest analogue would be a **constructive network**: begin with a minimal executable program and add an opcode, branch, register, or subroutine only when the current structure cannot reduce the residual error. This corresponds more closely to Cascade-Correlation or NEAT than to a conventional fixed multilayer perceptron.

[1]: https://proceedings.neurips.cc/paper/1989/hash/69adc1e107f7f7d035d7baf04342e1ca-Abstract.html?utm_source=chatgpt.com "The Cascade-Correlation Learning Architecture"
[2]: https://arxiv.org/abs/1708.01547?utm_source=chatgpt.com "Lifelong Learning with Dynamically Expandable Networks"
[3]: https://nn.cs.utexas.edu/?stanley%3Aec02=&utm_source=chatgpt.com "Evolving Neural Networks Through Augmenting Topologies"
[4]: https://arxiv.org/abs/1910.04062?utm_source=chatgpt.com "DEVDAN: Deep Evolving Denoising Autoencoder"
[5]: https://proceedings.mlr.press/v188/maile22a/maile22a.pdf?utm_source=chatgpt.com "When, where, and how to add new neurons to ANNs"
[6]: https://arxiv.org/abs/1603.08983?utm_source=chatgpt.com "Adaptive Computation Time for Recurrent Neural Networks"
[7]: https://arxiv.org/abs/1807.03819v1/?utm_source=chatgpt.com "[1807.03819v1] Universal Transformers"

**Q: I'm not familiar with any of these architectures. How effective are they at learning. How would you judge these, based upon their effectiveness?**

**A:** Dynamic-growth Artificial Neural Networks (ANNs) **can learn effectively, but no dynamic-growth method is generally superior to a well-designed fixed network**. Their principal advantage is selecting or allocating capacity automatically—not fundamentally improving gradient learning.

## Effectiveness assessment

| Architecture                              | Where it is effective                                                       | Main weakness                                                                   | Judgment                                     |
| ----------------------------------------- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------- | -------------------------------------------- |
| **Gradient-guided neuron growth**         | Static supervised learning when the correct width is unknown                | Growth decisions add training overhead and hyperparameters                      | **Most promising general approach**          |
| **Dynamically Expandable Networks (DEN)** | Continual learning across successive tasks                                  | Model size can continually increase; usually assumes identifiable tasks         | **Strongest for continual learning**         |
| **DEVDAN/evolving stream networks**       | Nonstationary data streams and concept drift                                | Limited evidence on large vision, language, or multimodal workloads             | **Strong specialist architecture**           |
| **NEAT**                                  | Small control systems, reinforcement learning, nondifferentiable objectives | Population-based evolution becomes computationally expensive at scale           | **Excellent niche method**                   |
| **Cascade-Correlation**                   | Small supervised problems and automatic topology construction               | Sequential construction, frozen features, increasing connectivity               | **Historically impressive, poorly scalable** |
| **Adaptive Computation Time (ACT)**       | Problems requiring different computation depths per input                   | Does not add persistent capacity; halting behavior can be difficult to optimize | **Useful but inconsistent**                  |

## 1. Gradient-guided growth

These are currently the most technically convincing versions of dynamic expansion.

The network might begin with:

[
[32,32,10]
]

and grow into:

[
[87,61,10]
]

based on gradient magnitude, activation independence, residual error, or another capacity signal.

The NORTH methods, for example, determine:

* **when** capacity is insufficient;
* **where** additional neurons should be placed;
* **how many** neurons to add;
* **how** their new weights should be initialized.

On VGG-11 image-classification experiments, one NORTH configuration exceeded a larger static network while using less than half as many parameters. However, on WideResNet-28, the growing networks tended to expand to nearly the permitted maximum size. The paper also reports higher training cost than predetermined static architectures. 

### Judgment

**Effective, but not consistently superior.**

Their strongest result is not necessarily higher absolute accuracy. It is obtaining comparable accuracy while discovering a reasonably compact architecture automatically.

The weakness is that the system now has two optimization problems:

[
\text{learn weights}
]

and

[
\text{learn architecture changes}.
]

A poor growth trigger can add redundant neurons, add them too late, or grow the wrong layer.

---

## 2. Dynamically Expandable Networks

A Dynamically Expandable Network is intended primarily for **continual learning**:

[
T_1 \rightarrow T_2 \rightarrow T_3 \rightarrow \cdots
]

When task (T_2) arrives, the system first attempts to reuse parameters learned for (T_1). If the existing representation is insufficient, it adds or duplicates units.

DEN combines:

* selective retraining;
* network expansion;
* unit splitting;
* task timestamps;
* parameter sharing between tasks.

Its original experiments reported substantially better continual-learning performance than contemporary alternatives, while reaching performance comparable to batch-trained networks with fewer parameters. ([arXiv][1])

### Judgment

**One of the most effective persistent-growth designs when the problem is lifelong learning.**

It directly addresses catastrophic forgetting:

[
\text{learn }T_2
\quad\not\Rightarrow\quad
\text{destroy performance on }T_1.
]

Its limitation is potentially unbounded accumulation:

[
N_{t+1}=N_t+\Delta N_t.
]

Unless expansion is balanced by pruning, compression, or module reuse, the network can become progressively larger.

---

## 3. DEVDAN and evolving data-stream networks

Deep Evolving Denoising Autoencoder Network (DEVDAN) is designed for data that arrives continuously:

[
x_1,x_2,x_3,\ldots
]

where the underlying distribution may change over time:

[
P_t(x,y)\neq P_{t+1}(x,y).
]

It can:

* add hidden units;
* remove hidden units;
* learn from labeled and unlabeled samples;
* operate in a single-pass setting;
* adapt to concept drift.

The original study reported competitive results over ten streaming classification datasets. ([arXiv][2])

### Judgment

**Effective for online sensors, industrial monitoring, telemetry and changing tabular data.**

It is less convincing as a general replacement for fixed convolutional networks or Transformers. Its evidence base is concentrated around streaming classification rather than very large static datasets.

---

## 4. NeuroEvolution of Augmenting Topologies

NeuroEvolution of Augmenting Topologies (NEAT) does not normally use backpropagation. It evolves populations of networks.

A mutation can:

* add a connection;
* split a connection with a new neuron;
* alter a weight;
* introduce recurrence.

The topology becomes more complex only when the additional structure improves fitness.

In its original pole-balancing experiments, NEAT learned approximately five times faster than the fixed-topology ESP evolutionary method and 25 times faster than Cellular Encoding. 

### Judgment

**Highly effective for small control policies and black-box objectives. Poorly suited to large-scale supervised learning.**

NEAT is attractive when:

* gradients are unavailable;
* the environment supplies only a fitness score;
* recurrent structure might be useful;
* the network is relatively small;
* simulation is inexpensive.

It becomes costly because an entire population must be evaluated:

[
\text{cost}
\approx
\text{population size}
\times
\text{generations}
\times
\text{evaluation cost}.
]

For a costly robotics or game simulation, this can dominate the entire learning process.

---

## 5. Cascade-Correlation

Cascade-Correlation starts with no hidden neurons. It trains the direct input-output mapping and then adds a hidden neuron specifically selected to correlate with the remaining error:

[
e(x)=y-\hat y.
]

After installation, that neuron's incoming weights are frozen. The next neuron receives inputs from both the original inputs and all previously installed hidden neurons.

On the historical two-spirals benchmark:

* Cascade-Correlation solved all 100 trials;
* it required an average of 1,700 epochs;
* Quickprop required approximately 8,000 epochs;
* standard backpropagation required approximately 20,000 epochs;
* the estimated connection-operation reduction was approximately (23\times) versus Quickprop and (50\times) versus standard backpropagation. ([NeurIPS Proceedings][3])

Those were excellent results for the benchmark and period.

### Judgment

**A strong demonstration of constructive learning, but not a strong modern large-scale architecture.**

Its weaknesses emerge as it grows:

1. **Frozen-feature problem**

   Once installed, an early hidden unit cannot revise its representation:

   [
   W_i^{\text{input}}=\text{constant}.
   ]

   A suboptimal early feature remains embedded permanently.

2. **Increasing fan-in**

   Each new neuron may connect to every preceding hidden neuron:

   [
   \operatorname{fanIn}(h_k)\propto d+k.
   ]

3. **Sequential construction**

   Neurons are added one at a time, limiting training parallelism.

4. **Residual greediness**

   Each new feature attacks the current error rather than jointly optimizing the entire representation.

I would consider Cascade-Correlation useful as inspiration for an opcode-growing architecture, but not as the final learning mechanism.

---

## 6. Adaptive Computation Time

Adaptive Computation Time does not physically add layers. It applies the same computational block repeatedly until a learned halting mechanism decides to stop:

[
h_{t+1}=F_\theta(h_t,x).
]

A simple input might use three iterations; a difficult input might use twelve.

ACT substantially improved performance on synthetic arithmetic, parity, sorting and logic tasks. However, its original language-modeling experiment produced no large gain. ([arXiv][4]) A later comparison also found that repeating the recurrent computation a fixed number of times could match ACT on the selected tasks. ([arXiv][5])

Universal Transformers extended this principle to self-attention and reported improvements on algorithmic tasks, language understanding and machine translation. ([arXiv][6])

### Judgment

**Useful when problem difficulty genuinely varies by input, but not evidence that architectural expansion itself improves learning.**

It dynamically allocates computation:

[
C(x)=\text{number of iterations for input }x,
]

rather than dynamically increasing parameter capacity.

---

## Overall ranking by use case

| Use case                                           | Preferred approach                                       |
| -------------------------------------------------- | -------------------------------------------------------- |
| Ordinary fixed training dataset                    | Static architecture or modern gradient-guided growth     |
| Unknown required layer width                       | NORTH, Firefly, GradMax or related neuron-growth methods |
| Successive identifiable tasks                      | Dynamically Expandable Networks                          |
| Continuous nonstationary sensor stream             | DEVDAN or another evolving stream network                |
| Small black-box control problem                    | NEAT                                                     |
| Algorithmic problems with variable reasoning depth | ACT or recurrent/shared-block models                     |
| Experimental constructive learning                 | Cascade-Correlation                                      |

## How effectiveness should be measured

Dynamic networks should not be judged merely by final accuracy. A useful evaluation should include:

[
J =
\alpha A
-\beta C_{\text{train}}
-\gamma C_{\text{infer}}
-\delta P
-\epsilon F,
]

where:

* (A): validation or task accuracy;
* (C_{\text{train}}): total training computation, including architecture search;
* (C_{\text{infer}}): inference cost;
* (P): final parameter count;
* (F): forgetting on previously learned tasks.

The critical comparison is:

> Does the growing network outperform the best fixed network at equal data, total computation, parameter count and tuning effort?

Under that standard, dynamic-growth systems are **useful and sometimes superior**, but the evidence does not establish them as universally better learners. Their clearest advantages are **continual adaptation, automatic capacity selection and resistance to under-capacity**, rather than universally higher learning ability.

[1]: https://arxiv.org/abs/1708.01547?utm_source=chatgpt.com "Lifelong Learning with Dynamically Expandable Networks"
[2]: https://arxiv.org/abs/1910.04062?utm_source=chatgpt.com "DEVDAN: Deep Evolving Denoising Autoencoder"
[3]: https://proceedings.neurips.cc/paper/1989/file/69adc1e107f7f7d035d7baf04342e1ca-Paper.pdf "The Cascade-Correlation Learning Architecture"
[4]: https://arxiv.org/abs/1603.08983?utm_source=chatgpt.com "Adaptive Computation Time for Recurrent Neural Networks"
[5]: https://arxiv.org/abs/1803.08165?utm_source=chatgpt.com "Comparing Fixed and Adaptive Computation Time for Recurrent Neural Networks"
[6]: https://arxiv.org/abs/1807.03819?utm_source=chatgpt.com "Universal Transformers"

**Q: Are neurons and or layers added arbitrarily? It seems reasonable that an artificial learning problem can be broken down mechanically into suitable features for training. Each neuron provides a purpose; each layer provides a purpose.**

**A:** They are usually **not added arbitrarily**, but the added neuron or layer is rarely assigned a precise human-readable purpose. Most growth methods add capacity according to an optimization signal, then allow training to determine what the new capacity represents.

## How additions are typically decided

A neuron may be added when the current network exhibits:

* persistent residual error;
* saturated or overloaded hidden units;
* insufficient gradient rank;
* high task interference;
* poor representation of new data;
* concept drift;
* failure to improve validation loss.

A simplified trigger is:

[
\text{if } L_t-L_{t-k}<\epsilon
\quad\text{and}\quad
L_t>L_{\text{target}},
\quad\text{add capacity}.
]

The system detects that learning has stalled, but it does not necessarily know *why*.

The new neuron is then initialized to address a measurable deficiency, such as:

[
h_{\text{new}}
==============

\arg\max_h
\left|
\operatorname{corr}(h(x),e(x))
\right|,
]

where (e(x)) is the current residual error. This is approximately the Cascade-Correlation strategy.

That neuron has a numerical purpose:

> Represent some function correlated with what the existing network still gets wrong.

It usually does not have an explicit semantic purpose such as:

> Detect wheels, identify negation, or calculate velocity.

## Three meanings of “purpose”

### 1. Architectural purpose

A layer can have an explicitly designed role:

| Component           | Designed purpose                  |
| ------------------- | --------------------------------- |
| Convolutional layer | Detect local spatial patterns     |
| Attention layer     | Relate distant tokens or features |
| Pooling layer       | Reduce spatial resolution         |
| Recurrent layer     | Preserve sequential state         |
| Normalization layer | Stabilize activation statistics   |
| Output head         | Produce task-specific predictions |

This purpose is imposed by the engineer.

### 2. Optimization purpose

A newly added neuron can be introduced to reduce a particular deficiency:

* capture unexplained residuals;
* separate two confused classes;
* represent a new task;
* increase rank;
* reduce interference;
* model a newly observed data region.

This purpose is imposed by the growth algorithm.

### 3. Learned semantic purpose

The actual internal function emerges from training.

For example, a neuron may respond to:

[
\text{vertical edge}
+
\text{specific texture}
-----------------------

\text{background brightness}.
]

That function may not correspond to any simple human concept. In large networks, meaning is usually distributed across many neurons rather than localized in one neuron.

Therefore:

[
\text{one neuron}
\not\equiv
\text{one feature}.
]

More commonly:

[
\text{feature}
==============

f(h_1,h_2,\ldots,h_n),
]

and each neuron participates in several features.

## Your proposed approach

Your idea is closer to **mechanically decomposed learning** than conventional neural-network training.

Suppose a problem is decomposed into explicit operations:

[
x
\rightarrow
\text{parse}
\rightarrow
\text{detect relation}
\rightarrow
\text{transform}
\rightarrow
\text{compare}
\rightarrow
\text{decide}.
]

Each stage has a defined contract:

[
M_i : S_i \rightarrow S_{i+1}.
]

A module is added only when a specific transformation is absent:

[
\exists x:
T(x)\neq
M_n\circ\cdots\circ M_1(x).
]

The system could then identify the missing capability:

[
\Delta_i
========

## S_{i+1}^{\text{required}}

S_{i+1}^{\text{produced}},
]

and instantiate a component designed to reduce that discrepancy.

This is more interpretable than saying merely:

> Add 128 neurons because validation loss stopped improving.

## Feature decomposition is reasonable, but difficult

The central challenge is determining a universally suitable decomposition.

For a visual task, explicit features may include:

* edges;
* corners;
* shapes;
* texture;
* depth;
* motion;
* object identity.

For language:

* token identity;
* syntax;
* semantic roles;
* reference resolution;
* temporal relationships;
* logical entailment.

For program synthesis:

* load;
* compare;
* branch;
* iterate;
* store;
* call;
* return.

The last case is especially compatible with your opcode architecture because the decomposition is already operational. Each instruction has explicit semantics and state effects:

[
S_{t+1}
=======

\operatorname{Opcode}_i(S_t).
]

A missing capability can therefore be expressed as:

[
\text{required state transition}
--------------------------------

\text{available state transitions}.
]

That is much clearer than an unidentified deficiency in a hidden vector.

## Why conventional ANNs do not normally work this way

Neural networks optimize a global objective:

[
\theta^*
========

\arg\min_\theta L(f_\theta(x),y).
]

They are not normally given an explicit decomposition of the solution. Backpropagation distributes credit throughout the network:

[
\frac{\partial L}{\partial \theta_i}.
]

A neuron changes because altering it reduces loss, not because the system has identified an explicit missing concept.

This gives ANNs flexibility, but weakens interpretability.

## Layers are harder to add meaningfully than neurons

Adding a neuron usually preserves the surrounding representation dimensions with limited disruption. Adding a layer inserts a new function:

[
h_{l+1}
=======

F_{\text{new}}(h_l),
]

which can alter everything downstream.

A safer method is function-preserving insertion:

[
F_{\text{new}}(h)=h
]

at initialization, followed by training. For example, a residual block can initially behave approximately like the identity:

[
h_{l+1}
=======

h_l+G(h_l),
\qquad
G(h_l)\approx 0.
]

The layer is then added with the initial purpose:

> Provide additional transform capacity without initially changing the existing network function.

But again, its final learned purpose remains emergent unless additional constraints are imposed.

## A more principled constructive architecture

A genuinely purpose-driven growth system could require every new component to define:

1. **Trigger**

   What measurable failure requires the component?

2. **Input contract**

   What state or representation does it consume?

3. **Output contract**

   What state transformation must it produce?

4. **Novelty test**

   Does it provide a function not already available?

5. **Utility test**

   Does it reduce error or search cost?

6. **Retention test**

   Does it remain useful after later components are added?

Formally, add component (C_k) only when:

[
\Delta L(C_k)> \tau_L,
]

[
\operatorname{Novelty}(C_k)>\tau_N,
]

and

[
\operatorname{Cost}(C_k)<\tau_C.
]

Then periodically remove components for which:

[
\Delta L(C_k)\approx 0.
]

This produces a system closer to **constructive program induction** than ordinary neural-network expansion.

## Core distinction

Your premise is valid:

> A learning problem can often be decomposed into mechanically meaningful features and transformations.

But conventional ANNs generally operate under a different premise:

> Provide sufficient differentiable capacity and allow optimization to discover an internal decomposition.

The first approach emphasizes:

* explicit structure;
* semantic roles;
* inspectability;
* modular reuse;
* controlled growth.

The second emphasizes:

* continuous optimization;
* distributed representation;
* tolerance for imperfect decomposition;
* broad statistical generalization.

For your opcode-based architecture, neurons should probably not be the primary units of growth. The primary units should be **typed operations, state transformations, predicates, branches, and reusable subprograms**, each added because a specific unresolved transformation has been identified. Neural components, when used, would serve as local estimators or selectors rather than as the architecture itself.

**Q: Each added neuron should capture a discrete correlating feature in the training data. This can be accomplished by batching discovered patterns into sets.**

**A:** Yes. That yields a coherent **constructive feature network**:

> Each neuron is created only after a recurring, predictive pattern has been discovered, grouped into a feature set, and shown to explain training residuals not already captured by existing neurons.

The important change is that neurons are no longer anonymous scalar parameters. Each neuron becomes a **registered feature detector** with provenance.

## Feature-set neuron

Let the training data be:

[
D={(x_i,y_i)}_{i=1}^{N}.
]

A pattern-discovery stage extracts candidate patterns:

[
P={p_1,p_2,\ldots,p_m}.
]

Correlated or structurally equivalent patterns are grouped into sets:

[
\mathcal{F}*k={p*{k1},p_{k2},\ldots,p_{kr}}.
]

A neuron (h_k) is then assigned to detect membership in, or similarity to, that feature set:

[
h_k(x)=\phi\left(
\operatorname{match}(x,\mathcal{F}_k)
\right).
]

The neuron therefore has a defined interpretation:

[
h_k \equiv \text{detector for feature family }\mathcal{F}_k.
]

This is much more disciplined than randomly initializing hundreds of neurons and allowing backpropagation to assign arbitrary internal roles.

## What should constitute a set

Patterns should not be grouped merely because they look similar. They should share a measurable functional relationship.

A feature set could be defined by:

[
\mathcal{F}_k=
\left{
p:
\operatorname{Effect}(p,y)
\approx
\operatorname{Effect}(p_k,y)
\right}.
]

Possible equivalence criteria include:

* similar correlation with the target;
* similar effect on prediction residuals;
* similar state transition;
* similar activation context;
* similar causal or conditional relationship;
* invariance under translation, scaling, rotation, substitution, or ordering;
* shared output behavior despite different surface forms.

For example, in visual data, several patterns may represent the same feature under transformation:

[
\mathcal{F}_{\text{vertical edge}}
==================================

{
\text{vertical edge at position }1,
\text{vertical edge at position }2,
\ldots
}.
]

In language:

[
\mathcal{F}_{\text{negation}}
=============================

{
\text{not},
\text{never},
\text{without},
\text{fails to}
}.
]

In program learning:

[
\mathcal{F}_{\text{comparison}}
===============================

{
a<b,,
b>a,,
\operatorname{compare}(a,b)<0
}.
]

These are different observations belonging to the same functional feature class.

## Residual-guided feature creation

The strongest version would discover patterns specifically from the cases the existing network fails to explain.

Given current prediction:

[
\hat y_i=f(x_i),
]

define residual:

[
r_i=y_i-\hat y_i.
]

Candidate feature (F_k) should be added only when its activation correlates with the residual:

[
\left|
\operatorname{corr}(h_k(x),r)
\right|

> \tau.
> ]

This gives every new neuron an explicit purpose:

> Explain a statistically significant portion of the remaining error.

The feature should also provide information not already represented by existing neurons:

[
I(h_k;Y\mid h_1,\ldots,h_{k-1})>\tau_I,
]

where (I) is conditional mutual information.

This prevents the system from adding duplicate neurons that rediscover an existing feature.

## Proposed growth procedure

### 1. Discover candidate patterns

Extract repeated structures from:

* raw inputs;
* intermediate states;
* misclassified examples;
* high-residual subsets;
* temporal sequences;
* co-occurrence relationships.

### 2. Canonicalize patterns

Map equivalent observations into a common representation:

[
p\rightarrow \operatorname{Canonical}(p).
]

Examples:

* normalize position;
* normalize scale;
* rename variables;
* sort commutative operands;
* remove irrelevant context;
* encode relative rather than absolute values.

Canonicalization is essential. Otherwise, the network may create separate neurons for superficially different instances of the same underlying feature.

### 3. Batch patterns into feature sets

Group patterns according to both structural and predictive similarity:

[
d(p_i,p_j)
==========

\alpha d_{\text{structure}}
+
\beta d_{\text{behavior}}
+
\gamma d_{\text{residual}}.
]

Patterns belong to the same set when:

[
d(p_i,p_j)<\epsilon.
]

### 4. Create one feature neuron per set

The neuron could compute:

[
h_k(x)
======

\max_{p\in\mathcal{F}_k}
\operatorname{similarity}(x,p),
]

or:

[
h_k(x)
======

\sum_{p\in\mathcal{F}_k}
w_p,\operatorname{match}(x,p).
]

The first behaves like a disjunction:

[
p_{k1}\lor p_{k2}\lor\cdots\lor p_{kr}.
]

The second represents graded evidence from multiple related patterns.

### 5. Validate usefulness

Add the neuron only if it improves held-out performance:

[
L_{\text{before}}-L_{\text{after}}>\tau_L.
]

Also evaluate:

* feature frequency;
* predictive value;
* stability across batches;
* redundancy;
* computational cost;
* resistance to spurious correlation.

### 6. Freeze the identity, not necessarily the parameters

The neuron's semantic contract should remain fixed:

[
h_k \equiv \mathcal{F}_k,
]

but its detector parameters may continue adapting within limits.

This avoids the conventional problem where a neuron initially represents one feature and later drifts into another.

A constrained update could be:

[
\theta_k^{t+1}
==============

\theta_k^t-\eta\nabla L
]

subject to:

[
\operatorname{Similarity}
\left(
h_k^{t+1},
h_k^{\text{reference}}
\right)

> \tau_S.
> ]

## Layers should represent composition depth

Your concept also gives layers a natural mechanical purpose.

### Layer 1: atomic features

[
h^{(1)}_i(x)=\text{primitive pattern detector}.
]

Examples:

* edge;
* token;
* local frequency;
* numeric comparison;
* temporal transition.

### Layer 2: feature combinations

[
h^{(2)}_j
=========

g_j\left(
h^{(1)}*{a},
h^{(1)}*{b},
h^{(1)}_{c}
\right).
]

Examples:

* corner from two edges;
* phrase from tokens;
* condition from comparisons;
* object part from local shapes.

### Layer 3: relational structures

[
h^{(3)}_k
=========

g_k\left(
h^{(2)}_u,
h^{(2)}_v,
\text{relation}
\right).
]

Examples:

* object;
* grammatical relation;
* causal sequence;
* control-flow structure.

Thus, depth corresponds to **feature composition order**:

[
\text{atomic}
\rightarrow
\text{compound}
\rightarrow
\text{relational}
\rightarrow
\text{task-level}.
]

A new layer should be added only when the required feature cannot be represented as a direct combination at the current compositional depth.

## Feature neurons may need internal structure

A single conventional neuron computes:

[
h(x)=\sigma(w^\top x+b).
]

This represents approximately one linear decision boundary followed by a nonlinearity. It may not be capable of detecting an arbitrary pattern set.

For example, if a feature set contains two disconnected regions:

[
\mathcal{F}
===========

R_1\cup R_2,
]

one linear neuron may not separate both regions from the rest of the input space.

Therefore, the term “neuron” may need to mean a small typed detector rather than a single weighted sum.

Possible feature-node implementations include:

* prototype matcher;
* radial basis unit;
* decision rule;
* small local neural network;
* finite-state recognizer;
* convolutional kernel;
* opcode sequence;
* logical predicate;
* learned similarity function.

The architecture remains neuron-like at the graph level, while each node is sufficiently expressive to detect its assigned feature.

## Main danger: combinatorial growth

The number of possible pattern combinations can become enormous:

[
2^m
]

for (m) primitive features.

The system therefore needs strict controls:

[
\operatorname{Add}(F_k)
\iff
\begin{cases}
\operatorname{Support}(F_k)>\tau_s\
\operatorname{Novelty}(F_k)>\tau_n\
\operatorname{PredictiveGain}(F_k)>\tau_g\
\operatorname{Complexity}(F_k)<\tau_c
\end{cases}
]

Feature sets should also be merged when they become behaviorally equivalent:

[
\operatorname{Effect}(F_i)\approx
\operatorname{Effect}(F_j)
\Rightarrow
F_i\cup F_j.
]

Unused features should be removed:

[
\operatorname{Utility}(F_k)<\tau_u
\Rightarrow
\operatorname{Prune}(F_k).
]

## Spurious correlation remains a problem

A discrete feature can correlate strongly with the training target without representing a durable rule.

For example:

[
\text{snowy background}
\leftrightarrow
\text{wolf}
]

may be highly predictive in one dataset but fail outside it.

A feature should therefore be tested across:

* held-out samples;
* counterfactual examples;
* different environments;
* temporal partitions;
* perturbed inputs;
* adversarially balanced subsets.

A more rigorous criterion is:

[
P(Y\mid F_k,E_1)
\approx
P(Y\mid F_k,E_2)
]

across environments (E_1,E_2).

Stable correlation across changing contexts is stronger evidence that the feature is structurally relevant.

## Resulting architecture

The system would resemble:

[
\text{Pattern discovery}
\rightarrow
\text{Feature-set formation}
\rightarrow
\text{Feature-node creation}
\rightarrow
\text{Compositional layering}
\rightarrow
\text{Residual analysis}
\rightarrow
\text{Further growth}.
]

Each node would store metadata such as:

```text
Feature ID
Feature-set members
Canonical representation
Input dependencies
Output meaning
Training examples supporting it
Residual reduction
Confidence
Creation time
Last-use time
Redundancy score
Validation stability
```

This converts the hidden network from an opaque matrix into an inspectable feature graph.

## Assessment

The principle is sound, with one qualification:

> A neuron should represent a discrete feature class, not necessarily one literal recurring pattern.

The feature class should batch together observations that are:

1. structurally related;
2. behaviorally equivalent;
3. predictive of the same residual;
4. stable across contexts;
5. not already represented.

That produces purposeful neuron growth and purposeful layer growth. It also moves the architecture toward a hybrid of:

* constructive neural networks;
* concept learning;
* prototype learning;
* program induction;
* feature lattices;
* residual boosting.

The closest mathematical analogy is not conventional multilayer neural learning. It is **incremental basis construction**, where each new neuron adds a validated basis function:

[
f_K(x)
======

\sum_{k=1}^{K}
\alpha_k h_k(x),
]

and every (h_k) corresponds to a documented, discrete feature set.
