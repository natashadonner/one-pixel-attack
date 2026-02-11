
# Attacking and Defending Deep Learning Models

**Secure AI – Adversarial ML Project**

This project explores something we find genuinely interesting:

> How fragile are deep neural networks — and can we make them more robust?

We worked through the fundamentals of machine learning and neural networks, and then moved on to implementing our own adversarial attack and defense mechanism from scratch.
For the attack, we based our implementation on “One Pixel Attack for Fooling Deep Neural Networks” by Su, Vargas & Sakurai (2019), where a single-pixel modification is optimized using Differential Evolution to cause misclassification.
For the defense, we implemented an approach presented in “Adaptive Pixel Resilience: A Novel Defence Mechanism Against One-Pixel Adversarial Attacks on Deep Neural Networks” by Srivastava (2024). 

The highlight:

* Successfully fooled a pretrained ResNet-18 by changing **just one pixel**
* Reduced attack success rate (ASR) from **70% down to 10%** using our own defense

---

# Part 1 – ML & Neural Network Fundamentals

For the first part of the lab, the base code was provided by the instructor.
Our task was to analyze, extend, debug, and experiment with it.

## Preprocessing & kNN

We worked with MNIST and implemented:

* Mean normalization
* Standardization
* Whitening + ZCA
* Manhattan (L1) and Euclidean (L2) distances
* k-Nearest Neighbors extension

While testing, we discovered a major issue:

The dataset was stored as `uint8`, causing overflow in distance calculations.

After converting to float and properly normalizing:

* L1 accuracy improved from **26% → 81%**
* L2 accuracy improved from **19% → 82.9%**

It was a great reminder that model performance can collapse due to small implementation details.

We then extended the nearest neighbor classifier to full **kNN**, tested different k values, and analyzed why performance degraded with small training sets.


## Backpropagation & Neural Networks

Using the provided neural network skeleton, we:

* Stepped through forward propagation
* Analyzed backpropagation and gradient flow
* Experimented with activation functions (Sigmoid, ReLU, Softmax)
* Tested learning rate sensitivity

Key observations:

* η = 0.5 → training diverged (~9% accuracy)
* η = 0.05 → stable (~97% test accuracy)
* ReLU required smaller η to behave well

This part was less about writing everything from scratch and more about deeply understanding how gradient-based learning behaves in practice.

---

# One Pixel Attack

This is where things got interesting.

We implemented the **One Pixel Attack** ourselves using Differential Evolution.

### Setup:

* Target model: ResNet-18 (pretrained on ImageNet)
* Dataset: CIFAR-10 (resized to 224×224)
* Attack type: Untargeted
* Population size: 400
* Max iterations: 50–100

### Mutation Strategy

We used a population-based Differential Evolution scheme:

```
mutant = best + F * (r2 − r3)
```

Where:

* F = 0.5
* r2 and r3 are random population members

### Fitness Function

```
maximize (1 - P(original_class))
```

The attack stopped early once misclassification occurred or the model’s confidence in the original class dropped below 5%

### Results

* **70% attack success rate (7/10 images)**
* Most successful attacks occurred in the first few generations
* Low-confidence images were significantly easier to attack

---

# Adaptive Pixel Resilience Defense

After breaking the model, we built a defense mechanism inspired by Adaptive Pixel Resilience (APR).

Our defense combined:

### Adversarial Training

* 50% clean images
* 50% adversarially generated one-pixel images

### Pixel-wise Attention Layer

We added a learnable 3×224×224 attention tensor that:

* Learns which pixels are important
* Downweights suspicious local perturbations
* Is applied before feeding input to ResNet-18

### Gradient Regularization

We added a gradient penalty term:

```
R = β ||∇x f(x)||²
```

This enforces smoother decision boundaries and reduces sensitivity to tiny perturbations.

---

# Defense Results

After training the defense:

* Attack success rate reduced from **70% → 10%**
* Training time: ~8 minutes 

However:

We also analyzed how an adaptive attacker could break this defense by:

* Expanding beyond 1-pixel (L₀ norm expansion)
* Increasing population size
* Increasing iterations

Security is always a moving target.

---


