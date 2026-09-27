# Live 3D Neural Network Lab — MLclass companion

A fully interactive, real-time, **3D visualisation of a neural network actually learning**,
built as a companion to [MLclass](https://github.com/erachelson/MLclass) **Section 6 —
Artificial Neural Networks**. Everything you see is real: a genuine multilayer perceptron
implemented from scratch in JavaScript, training live in your browser via mini-batch
backpropagation — no fake data, no mocked "AI".

## Run it

```bash
# option 1: just open it (needs internet once for the Three.js CDN)
open index.html

# option 2: serve locally
python3 -m http.server 8000
# then visit http://localhost:8000
```

Press **▶ Train** (or the spacebar) and watch the network learn.

## What you see

### The 3D network (Three.js)
- **Neurons** as glowing spheres arranged in layer planes: input (gold), hidden (sky blue),
  output (magenta). Sphere **brightness tracks the live activation** of the current sample.
- **Connections** coloured by weight — blue = positive, red = negative — with brightness
  proportional to |weight|, updating every training step.
- **Cyan pulse waves** travel layer-by-layer, input → output: the *forward pass*, the data
  flowing through the network for a real training sample.
- **Orange pulse waves** travel output → input right after — and they show **real gradients**:
  edges are re-coloured by the actual per-edge |∂L/∂w| of the latest step while the wave
  flows (brightest = learning fastest), pulses are ranked and scaled by gradient magnitude,
  and each layer's neuron flash intensity reflects its share of the gradient norm. Watching
  the orange wave fade layer-by-layer is the vanishing-gradient lesson made visible.
- A translucent **decision surface** plane below the network shows the model's current
  prediction over input space, evolving live as training progresses.
- Full orbit / zoom / pan controls, optional auto-rotate.

### The analytics dock (all live)
| Panel | What it shows |
|---|---|
| Live metrics | train loss, test loss, test accuracy, epoch |
| Loss curves | train vs test BCE loss over training steps, with y-axis ticks and a **log-scale toggle** |
| Decision boundary | 2D map of p(x, y) with the held-out test points |
| Gradient flow | ‖∂L/∂W‖ per layer (log-mapped bars — vanishing gradients stay visible) |
| Weight distribution | histogram of every weight in the network |
| Confusion matrix | TP/FP/FN/TN on the test set, with actual/predicted axis labels + precision & recall |
| Sample activations | per-neuron activation bars for the sample currently flowing through |
| Status bar | step, epoch, parameter count, learning rate, dataset, FPS |

A **training-phase chip** in the header (idle / learning / converged / plateaued / overfit /
diverging) tracks the run — the same state is appended to the accuracy readout so it is
always readable as text, never colour alone. Every chart carries a live screen-reader
summary (`aria-label`) and the whole app respects `prefers-reduced-motion` (fewer pulses,
no auto-rotate, no scale pulse animation).

### Keyboard shortcuts

| Key | Action |
|---|---|
| Space | start / pause training |
| S | single training step |
| R | regenerate dataset |
| 1–5 | switch dataset |
| F | toggle fullscreen |
| ? | help overlay (what am I looking at?) |
| Esc | close the overlay |

All settings (dataset, architecture, features, optimizer, learning rate, batch, speeds,
surface, log-scale) persist to `localStorage` between sessions — refreshing keeps your
experiment.

### Controls
- **Datasets**: spiral, two moons, concentric circles, XOR blobs, gaussians (+ noise slider,
  regenerate)
- **Architecture**: hidden layers as e.g. `8, 8` (up to 6 layers × 12 neurons)
- **Input features**: engineered (`x, y, x², y², x·y, sin x, sin y`) or raw `(x, y)` —
  switch to raw and watch how much *harder* the spiral becomes. This is the feature-engineering
  lesson made visible.
- **Optimizer**: Adam (default) or plain SGD, learning rate, batch size
- **Train speed** (steps per frame) and **wave speed** (pulse animation rate)
- Play / pause / single-step / reset weights; spacebar toggles training

## The engine (and why you can trust it)

The network lives in a DOM-free region of `index.html` marked
`/*ENGINE-START*/ … /*ENGINE-END*/`: a from-scratch MLP with tanh/ReLU hidden units, a sigmoid
output, binary cross-entropy loss, and hand-derived backprop. `test/engine.test.mjs` extracts
that exact code and verifies it in Node:

```bash
node test/engine.test.mjs
```

- **Numerical gradient check**: analytic backprop gradients match central finite differences
  to ~1e-10 relative error (tanh and ReLU).
- **Convergence**: spiral reaches >90% test accuracy (with engineered features); moons,
  circles, XOR and gaussians reach 85–100% (Adam and SGD paths both covered).
- **Dataset integrity**: every dataset is exactly class-balanced and bounded.

### A measured data fix worth knowing
The original spiral (inner radius 0.12, noise σ≈0.075) had genuinely ambiguous labels near
the centre — a 24×24 network overfit to 100% train while test accuracy capped at ~85%,
i.e. irreducible label noise, not an optimisation failure. The shipped spiral starts at
radius 0.25, making the two arms separable (verified: 93–96% test accuracy across seeds).

## Pedagogical pointers (Section 6 tie-ins)

1. **Forward pass** → cyan waves; **backpropagation** → orange waves. Two phases, one loop.
2. Turn train speed up and watch the **loss curves** and **decision boundary** co-evolve.
3. Watch **gradient flow**: early in training all layers have large ‖∂L/∂W‖; as the network
   fits, gradients shrink. Deep nets with raw features can show the classic
   vanishing-gradient pattern.
4. Set learning rate to 1 and watch Adam explode — a live demonstration of why LR matters.
5. Compare Adam vs SGD on the same spiral seed: SGD (lr 0.1) crawls where Adam converges.
6. Switch features raw ⇄ engineered: the single most dramatic lesson in the app.

## Tech notes

- Single self-contained `index.html`; Three.js r160 via CDN (import map + ES modules).
- No build step, no dependencies to install, no tracking, runs 100% locally.
- The visualisation samples the strongest |weights| per wave so dense layers stay legible.
