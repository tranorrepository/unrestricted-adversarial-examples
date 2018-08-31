# Unrestricted Adversarial Examples Challenge

In the Unrestricted Adversarial Examples Challenge, attackers submit arbitrary adversarial inputs, and defenders are expected to assign low confidence to difficult inputs while retaining high confidence and accuracy on a clean, unambiguous test set.

You can learn more about the motivation and structure of the contest in our recent paper:

**Unrestricted Adversarial Examples**<br>
*Authors*<br>
[http://arxiv.org/](http://arxiv.org/)

This repository contains code for the warm-up to the challenge, as well as [the public proposal for the contest](https://github.com/google/unrestricted-adversarial-examples/blob/master/contest_proposal.md).

![image](https://user-images.githubusercontent.com/306655/44686400-f0b74800-aa02-11e8-8967-fa354244813f.png)



## Warm-up phase
### <a name="leaderboard"></a>Leaderboard


| Defense               | Submitted by  | Spatial acc.<br>at 80% cov. | SPSA acc.<br>at 80% cov. | L2-ball acc.<br>at 80% cov. |  Submission Date |
| --------------------- | ------------- | ------------ |--------------- |--------------- | --------------- |
| [Worst-of-10-Spatial Baseline](#)  |  ?? |    **??**    |     **??**   |     **??**     |  Aug 28th, 2018 |
| [Undefended ResNet Baseline](https://github.com/google/unrestricted-adversarial-examples/tree/master/unrestricted_advex/pytorch_resnet_baseline)   |  Google Brain   |    0.0%    |     0.0%    |     0.0%     |  Aug 27th, 2018 |


We include three attacks in the warm-up phase of the challenge

- 1000 Linfinity-ball adversarial examples generated by SPSA
- 1000 spatial adversarial examples (via grid search)
- 100 L2-ball adversarial examples generated by a decision-only attack

### Implementing a defense

First install the requirements (assuming you already have working installation
of Tensorflow or pytorch)
```bash
git clone git@github.com:google/unrestricted-adversarial-examples.git
cd unrestricted-adversarial-examples

apt-get install imagemagick

pip install -e tcu-images
pip install -e unrestricted-advex
```

Confirm that your setup runs correctly by training and evaluating an MNIST model.
```bash
cd unrestricted-advex/unrestricted_advex/mnist_baselines
CUDA_VISIBLE_DEVICES=0 python train_tcu_mnist.py --total_batches 10000
# Outputs look like (specific numbers may vary)
# 0 Clean accuracy 0.046875 loss 2.3123064
# 100 Clean accuracy 0.9140625 loss 0.24851117
# 200 Clean accuracy 0.953125 loss 0.1622512
# ...
# 9800 Clean accuracy 1.0 loss 0.004472881
# 9900 Clean accuracy 1.0 loss 0.00033166306

CUDA_VISIBLE_DEVICES=0 python evaluate_tcu_mnist.py
# Outputs look like (specific numbers may vary)
# Executing attack: null_attack
# Fraction correct under null_attack: 1.000
# Executing attack: spsa_attack
# Fraction correct under spsa_attack: 0.016
# Executing attack: spatial
# Fraction correct under spatial: 0.117
```

#### To be evaluated against our fixed warm-up attacks, your defense must implement the following API

It must be a function that takes in batched images, and returns two scalar (e.g. logits) between `(-inf, inf)`. These correspond to the likelihood the image corresponds to each of the two classes (e.g. the bicycle and bird class)

```python
import numpy as np

batch_size = 32

def my_very_robust_model(images_nchw):
  """This function implements a valid unrestricted advex defense"""
  logits = np.random.randn(batch_size, 2)
  return logits

from unrestricted_advex import eval_kit
from unrestricted_advex.mnist_baselines.evaluate_tcu_mnist import iter_mnist_testset
eval_kit.evaluate_tcu_mnist_model(
    my_very_robust_model, iter_mnist_testset(
        num_datapoints=batch_size, batch_size=batch_size))
```

For ease of evaluation, your model must also maintain a throughput of at least **100 images per second** when evaluated on a P100 GPU on TCU-Images

##### Your defense will be evaluated with the following mechanism

- The test dataset is passed through the model and converted to logits.
- `confidence` is defined as `max(bicycle_logit, bird_logit)` for each image.
- The 20% of images that resulted in logits with the lowest `confidence` are abstained on by the model and are discarded.
- The model’s score is the **accuracy on points that were not abstained on**.

## Contest phase

The contest phase will begin after the warm-up attacks have been conclusively solved. We have published the [contest proposal](https://github.com/google/unrestricted-adversarial-examples/blob/master/contest_proposal.md) and are soliciting feedback from the community.

## Authors
