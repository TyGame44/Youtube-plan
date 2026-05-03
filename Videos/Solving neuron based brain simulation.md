# Solving neuron based brain simulation

## Issue summary
A more complete neuron model would combine LIF’s efficient soma-level spike abstraction with a dendritic active-circuit layer.

Concise summary

Standard leaky integrate-and-fire models treat a neuron as a single RC circuit: inputs charge a membrane, voltage leaks, and a spike occurs at threshold. This captures basic spiking cheaply, but it ignores the neuron’s spatial structure.

A real neuron is better modeled as a branching active transmission network. Dendrites are not passive wires; they contain nonlinear, voltage-sensitive mechanisms that can filter, delay, amplify, suppress, or locally spike before signals reach the soma.

So the missing layer is:

LIF soma + distributed nonlinear dendritic circuit

This means modeling dendrites as branch compartments with:

local capacitance, resistance, and leak;
distance-dependent delay and attenuation;
nonlinear voltage-gated conductances;
branch-level thresholds;
local dendritic spikes;
backpropagating action potentials;
refractory or excitability state;
spike interaction between incoming synaptic signals and outgoing/backpropagating signals.

The key upgrade is that a neuron should not be treated as one threshold bucket. It should be treated as a small active electrical tree feeding a LIF-like spike generator.

In one sentence:

A complete efficient neuron model should keep LIF for soma firing, but add dendrites as distributed, nonlinear, stateful transmission branches where signals can amplify, collide, backpropagate, and re-trigger firing.

## Based on How close are we to solving P versus NP
## Made with AI

# Plan: Hybrid LIF + Active Dendrite Model Optimized with EML Symbolic Compression

## Purpose

Build a neuron model that keeps the speed and scalability of leaky integrate-and-fire (LIF) neurons while adding the missing dendritic computation layer: branch-local nonlinear integration, dendritic spikes, backpropagating action potentials, refractory branch state, and spike-interaction effects such as apparent double firing.

Then use the Exp-Minus-Log (EML) operator,

```text
eml(x, y) = exp(x) - ln(y)
```

as a symbolic compression and optimization tool for the dendritic layer.

The goal is not to replace biophysical simulation everywhere. The goal is to use detailed dendritic models as a teacher, learn compact branch-level surrogates, convert those surrogates into simplified symbolic expressions, and compile them into fast runtime kernels.

---

## Core Thesis

A complete efficient neuron model should be:

```text
active dendritic circuit surrogate + LIF soma/axon spike engine
```

The dendrites handle spatial, nonlinear, stateful signal processing. The soma handles efficient spike generation.

EML is useful as a uniform symbolic representation for compressing learned dendritic dynamics, not as a magic shortcut that solves all optimization or P versus NP problems.

---

## Background Assumptions

### 1. Standard LIF is a point-neuron approximation

A basic LIF neuron models the cell as a single leaky RC node:

```text
dV/dt = -(V - V_rest) / tau + input_current
if V >= threshold:
    spike
    reset
```

This captures simple spike timing but collapses dendrites, branch structure, backpropagation, local spikes, and branch-level nonlinearities into one soma voltage.

### 2. Real neurons are distributed active circuits

A more realistic neuron is closer to a branching nonlinear transmission network:

```text
synapses → dendritic branches → branch nonlinearities → soma → axon spike
```

Each dendritic branch may filter, delay, amplify, suppress, locally spike, or store state.

### 3. EML gives a uniform symbolic basis

The EML operator was proposed as a single binary operator that can generate a wide scientific-calculator basis of elementary functions when combined with the constant `1`. This creates a uniform binary-tree grammar:

```text
S → 1 | eml(S, S)
```

This is potentially useful for symbolic regression and expression compression, but it should be treated as a representation and optimization tool, not as a proof that hard computational problems become easy.

---

## High-Level Architecture

```text
Input spike trains
      ↓
Synapse models
      ↓
Active dendrite layer
      ↓
Branch output currents / local dendritic spikes
      ↓
LIF soma
      ↓
Axon spike output
      ↓
Backpropagating action potential trace sent back into dendrite layer
```

The model should have three levels:

1. **Synapse layer**
2. **Dendritic branch layer**
3. **Soma/axon LIF layer**

---

## Model Components

### A. Synapse Layer

Each synapse should have at least:

```text
input_spike
synaptic_weight
synaptic_time_constant
synaptic_conductance_trace
branch_location
excitatory_or_inhibitory_type
```

Minimal update:

```text
g_syn[t+1] = decay * g_syn[t] + spike_input[t] * weight
```

Optional extensions:

```text
short-term plasticity
NMDA-like slow conductance
inhibitory shunting conductance
synapse clustering
```

---

### B. Dendritic Branch Layer

Each dendritic branch should maintain a compact local state:

```text
V_branch       branch voltage
g_syn          synaptic conductance/input trace
Ca_branch      slow calcium-like memory
R_branch       refractory or recovery state
bAP_branch     backpropagating action potential trace
D_branch       local dendritic spike/plateau state
```

Example conceptual update:

```text
V_branch[t+1] =
    leak(V_branch[t])
  + synaptic_drive[t]
  + neighbor_coupling[t]
  + bAP_effect[t]
  - refractory_suppression[t]
```

Branch event rule:

```text
if V_branch + synaptic_drive + bAP_trace > branch_threshold:
    emit local dendritic spike
    increase branch output to soma
    update calcium/refractory state
```

This is where missed LIF effects are represented.

---

### C. Soma / Axon LIF Layer

The soma remains a fast LIF-style unit:

```text
I_soma[t] = sum(branch_output_i[t])

V_soma[t+1] =
    V_soma[t]
  + dt * (-(V_soma[t] - V_rest) / tau_soma + I_soma[t])
```

Spike rule:

```text
if V_soma[t+1] >= threshold and not refractory:
    spike[t+1] = 1
    V_soma[t+1] = reset_voltage
    soma_refractory = refractory_period
    send bAP trace into dendritic branches
```

---

## Explicitly Modeling the Double-Firing / Backfire Problem

The key missed event is not necessarily a literal reflection like a wave bouncing off a wall. It is better modeled as nonlinear pulse interaction:

```text
incoming synaptic event
+ local near-threshold dendritic state
+ backpropagating action potential
+ partial refractory/excitability state
→ local dendritic spike or plateau
→ additional soma drive
→ possible second soma spike
```

Minimal local interaction term:

```text
interaction_i =
    f(V_branch_i, syn_input_i, bAP_i, Ca_i, R_i)
```

Double-fire condition:

```text
if soma recently spiked
and bAP_trace is active
and a branch receives clustered input
and branch is not fully refractory
and local dendritic threshold is crossed:
    branch emits a regenerative event
    soma receives delayed extra current
    second spike probability increases
```

Runtime pseudocode:

```python
for branch in dendritic_branches:
    branch.bAP *= bap_decay
    branch.Ca *= ca_decay
    branch.R *= refractory_decay

    branch.V = (
        leak(branch.V)
        + synaptic_drive(branch)
        + coupling_from_neighbors(branch)
        + bap_gain * branch.bAP
        - refractory_gain * branch.R
    )

    if branch.V > branch.threshold:
        branch.local_spike = 1
        branch.Ca += ca_increment
        branch.R += refractory_increment
        soma_current += branch.spike_gain
    else:
        branch.local_spike = 0
        soma_current += branch.passive_output_gain * branch.V

if soma_voltage + soma_current > soma_threshold:
    soma_spike = 1
    soma_voltage = reset_voltage

    for branch in dendritic_branches:
        branch.bAP += branch.bap_coupling
else:
    soma_spike = 0
```

---

## EML Optimization Strategy

### Key Idea

Do not directly replace every runtime equation with raw EML unless benchmarking proves it is faster. Direct EML trees may be expensive because each node uses `exp` and `ln`.

Instead, use EML for:

1. symbolic regression;
2. equation discovery;
3. compression of dendritic branch dynamics;
4. interpretable surrogate construction;
5. hardware-specific compilation when custom EML cells are available.

---

## Teacher–Student Training Pipeline

### Step 1: Build a detailed teacher model

Use a biologically richer multi-compartment model as the reference.

Teacher model includes:

```text
dendritic compartments
branch morphology
synaptic conductances
voltage-gated sodium channels
calcium dynamics
potassium/recovery currents
NMDA-like nonlinearities
backpropagating action potentials
local dendritic spikes
```

The teacher does not need to be real-time. Its purpose is to generate high-quality training data.

---

### Step 2: Generate training data

Run the teacher model across many input conditions:

```text
random spike trains
clustered synaptic bursts
distal versus proximal input
inhibitory shunting input
recent soma spike timing
bAP timing windows
near-threshold dendritic states
refractory branch states
```

Record:

```text
branch voltage traces
local dendritic spike events
branch output current to soma
soma voltage
soma spike timing
double-fire / burst events
calcium-like traces
energy proxy variables
```

Important: oversample rare events such as local dendritic spike interactions and double-firing conditions.

---

### Step 3: Define a compact dendritic student model

Student model state:

```text
x_branch = [
    V_branch,
    syn_fast,
    syn_slow,
    Ca,
    refractory,
    bAP,
    neighbor_input
]
```

Student output:

```text
branch_current_to_soma
local_spike_probability
branch_refractory_update
calcium_update
```

Student update:

```text
branch_output = surrogate_function(x_branch)
```

Initially, this surrogate can be a small differentiable model:

```text
tiny MLP
polynomial model
rational approximation
spline
EML tree
hybrid DNN-EML head
```

---

### Step 4: Fit an EML symbolic surrogate

Represent the branch update function as a bounded-depth EML tree:

```text
branch_output ≈ EML_tree(V, syn_fast, syn_slow, Ca, R, bAP, neighbor)
```

Possible outputs:

```text
EML_tree_current
EML_tree_local_spike_score
EML_tree_refractory_update
EML_tree_calcium_update
```

Training targets:

```text
minimize branch current error
minimize local spike classification error
minimize soma spike timing error
minimize burst/double-fire miss rate
minimize long-run stability error
```

Loss function:

```text
L =
    λ1 * branch_current_MSE
  + λ2 * local_spike_BCE
  + λ3 * soma_spike_timing_loss
  + λ4 * burst_event_loss
  + λ5 * stability_penalty
  + λ6 * expression_complexity_penalty
```

---

### Step 5: Snap, simplify, and verify

After training:

```text
snap constants to simple values
remove redundant subtrees
merge repeated motifs
replace unstable subexpressions
factor common traces
bound output ranges
check domain constraints for ln(y)
```

Simplification objectives:

```text
fewer EML nodes
smaller tree depth
stable numeric behavior
same spike timing behavior
same rare-event behavior
easy bytecode compilation
```

---

### Step 6: Compile into runtime form

The final simulator should not necessarily evaluate a pure raw EML tree. It should compile to the fastest verified representation.

Possible compiled forms:

```text
direct classical arithmetic
lookup tables
piecewise polynomial approximation
vectorized CPU code
GPU kernel
neuromorphic instruction sequence
FPGA EML-cell implementation
analog EML-inspired circuit
```

Decision rule:

```text
Use EML as the discovery/compression language.
Use the fastest numerically stable equivalent form at runtime.
```

---

## Bytecode Strategy

### A. Intermediate Representation

Represent each dendritic branch update as a small bytecode graph:

```text
LOAD_STATE V
LOAD_STATE syn_fast
LOAD_STATE syn_slow
LOAD_STATE Ca
LOAD_STATE R
LOAD_STATE bAP
EML a b
ADD a b
MUL a b
CLAMP min max
THRESHOLD value
STORE_STATE V
OUTPUT_CURRENT
```

Even if EML is the symbolic basis, the bytecode should allow optimized substituted operations after simplification.

---

### B. Bytecode Execution Modes

Support multiple backends:

```text
Interpreter mode:
    easiest to debug

Vectorized CPU mode:
    useful for many neurons with same morphology class

GPU mode:
    useful for large network simulation

FPGA/ASIC mode:
    useful if custom EML cells exist

Neuromorphic mode:
    useful if branch updates can be mapped to event-driven local circuits
```

---

### C. Compiler Passes

Recommended compiler passes:

```text
1. domain validation
2. constant folding
3. common-subexpression elimination
4. tree-depth reduction
5. range analysis
6. numerical-stability rewrite
7. branch-output clamping
8. vector packing
9. event-sparsity optimization
10. backend-specific lowering
```

---

## Numerical Stability Guardrails

The EML function has domain and precision issues:

```text
eml(x, y) = exp(x) - ln(y)
```

Risks:

```text
y must stay positive
exp(x) can overflow
ln(y) can become unstable near zero
subtraction can lose precision
deep trees can explode or collapse numerically
```

Guardrails:

```text
clamp or softplus-transform y inputs
use expm1/log1p where appropriate
perform range analysis before compilation
use bounded-depth trees
use mixed precision carefully
validate over adversarial input ranges
fallback to classical stable arithmetic when needed
```

---

## Validation Plan

### A. Unit Tests

Test each branch model on:

```text
single synaptic pulse
paired pulse
clustered burst
distal input
proximal input
inhibition plus excitation
recent bAP plus input
refractory branch plus input
```

---

### B. Rare-Event Tests

Specifically test:

```text
backpropagation-assisted local spike
branch-local plateau event
spike suppression from refractory state
double-fire condition
burst initiation
branch collision-like interaction
```

---

### C. Network-Level Tests

Compare hybrid model against:

```text
standard LIF
adaptive LIF
multi-compartment teacher
experimental spike data, if available
```

Metrics:

```text
soma spike timing accuracy
burst event recall
false double-fire rate
synaptic clustering sensitivity
distal/proximal input difference
energy cost per simulated second
simulation speed
memory footprint
stability over long runs
```

---

## Development Milestones

### Milestone 1: Baseline Hybrid LIF

Deliverable:

```text
LIF soma with passive dendritic compartments
```

Success criteria:

```text
faster than teacher model
captures distance-dependent attenuation
stable over long simulation
```

---

### Milestone 2: Active Dendrite Events

Deliverable:

```text
branch-level thresholds
local dendritic spikes
bAP trace
branch refractory state
```

Success criteria:

```text
can reproduce local dendritic spike cases
can produce bAP-assisted second-spike events
avoids runaway firing
```

---

### Milestone 3: Teacher Dataset

Deliverable:

```text
dataset from detailed compartment simulations
```

Success criteria:

```text
contains normal cases
contains rare nonlinear branch events
contains double-fire / burst examples
has train/validation/test split
```

---

### Milestone 4: Learned Dendritic Surrogate

Deliverable:

```text
compact branch surrogate trained against teacher model
```

Success criteria:

```text
matches branch current traces
matches local spike events
improves over basic LIF on burst/double-fire cases
```

---

### Milestone 5: EML Symbolic Compression

Deliverable:

```text
bounded-depth EML-tree approximation of branch dynamics
```

Success criteria:

```text
smaller or more interpretable than black-box surrogate
similar accuracy to learned surrogate
stable on validation and adversarial tests
```

---

### Milestone 6: Compiler and Runtime

Deliverable:

```text
compiled branch update kernel
```

Success criteria:

```text
faster than teacher model
competitive with enhanced LIF
supports CPU/GPU backend
preserves rare dendritic event behavior
```

---

## Practical Minimal Version

If building a first prototype, start with this simplified model:

```text
Each neuron has:
    1 soma LIF unit
    4 to 16 dendritic branches

Each branch has:
    V_branch
    syn_fast
    syn_slow
    bAP_trace
    refractory_trace

Each branch computes:
    passive current to soma
    optional local spike event

The soma:
    sums branch outputs
    fires with LIF
    sends bAP traces back to branches
```

Minimal branch equation:

```text
V_branch[t+1] =
    a * V_branch[t]
  + b * syn_fast[t]
  + c * syn_slow[t]
  + d * bAP[t]
  - e * refractory[t]
```

Minimal nonlinear event:

```text
local_spike =
    1 if V_branch[t+1] > branch_threshold else 0
```

Minimal soma input:

```text
I_soma =
    sum(passive_gain * V_branch)
  + sum(local_spike_gain * local_spike)
```

This is enough to study the main ignored phenomenon before adding full EML compression.

---

## Research Questions

1. How many dendritic branches are needed before behavior meaningfully improves over LIF?
2. Which branch state variables are essential: voltage, calcium, refractory, bAP, NMDA-like slow trace?
3. Can EML trees recover compact expressions for local dendritic spike probability?
4. Does EML compression preserve rare double-fire events better than a generic MLP?
5. Is EML useful only for symbolic discovery, or can it also improve hardware execution?
6. What is the best runtime target: CPU vectorization, GPU kernels, FPGA, ASIC, or neuromorphic hardware?
7. How much biological detail is needed before network behavior changes meaningfully?

---

## Main Risks

### Risk 1: Overfitting to teacher simulations

Mitigation:

```text
use multiple teacher morphologies
train across varied input regimes
hold out entire event classes for validation
```

### Risk 2: EML trees become numerically unstable

Mitigation:

```text
bounded depth
range analysis
domain guards
stable rewrites
fallback kernels
```

### Risk 3: Runtime is slower than ordinary LIF

Mitigation:

```text
event-driven branch updates
shared morphology templates
lookup tables for expensive branch functions
compile EML to classical optimized arithmetic
```

### Risk 4: Rare double-fire events are still missed

Mitigation:

```text
oversample rare events
add explicit burst/double-fire loss
validate on adversarial timing windows
```

### Risk 5: Model becomes biologically plausible but not useful

Mitigation:

```text
define application metrics early
compare against LIF, adaptive LIF, and teacher model
measure simulation cost, spike accuracy, and network-level behavior
```

---

## Recommended First Experiment

Build a single-neuron simulation with:

```text
1 soma
8 dendritic branches
100 synapses
branch-local threshold events
bAP trace
branch refractory trace
```

Run three scenarios:

```text
1. normal random input
2. clustered branch input
3. clustered branch input shortly after soma spike
```

Compare:

```text
basic LIF
hybrid dendrite-LIF
teacher compartment model
```

Primary outcome:

```text
Does the hybrid model reproduce second-spike/burst events that basic LIF misses?
```

Then fit an EML-tree surrogate to the branch-output rule and compare accuracy/speed.

---

## Final System Design

```text
Detailed biophysical teacher
        ↓
Synthetic dendritic event dataset
        ↓
Compact active-dendrite student model
        ↓
EML symbolic compression
        ↓
Simplification and numerical verification
        ↓
Compiled dendrite-LIF runtime
        ↓
Large-scale spiking brain simulation
```

The final model should preserve LIF efficiency while adding enough dendritic realism to capture branch-local computation, backpropagation, and spike-interaction effects that are usually ignored.

---

## References

- Andrzej Odrzywołek, “All elementary functions from a single binary operator,” arXiv, 2026. https://arxiv.org/abs/2603.21852
- Eymen Ipek, “Hardware-Efficient Neuro-Symbolic Networks with the Exp-Minus-Log Operator,” arXiv, 2026. https://arxiv.org/abs/2604.13871
- Iczelia, “Numerically approximating Exp-Minus-Log is surprisingly...,” 2026. https://iczelia.net/posts/eml-approx/
- Eymen Ipek, “Evaluating the Exp-Minus-Log Sheffer Operator for Battery Characterization,” arXiv, 2026. https://arxiv.org/abs/2604.13873

