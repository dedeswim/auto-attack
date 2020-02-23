# AutoAttack

We propose to use an ensemble of four diverse attacks to reliably evaluate robustness:
+ **APGD-CE**, our new step size-free version of PGD on the cross-entropy,
+ **APGD-DLR**, our new step size-free version of PGD on the new DLR loss,
+ **FAB**, which minimizes the norm of the adversarial perturbations [(Croce & Hein, 2019)](https://arxiv.org/abs/1907.02044),
+ **Square Attack**, a query-efficient black-box attack [(Andriushchenko et al, 2019)](https://arxiv.org/abs/1912.00049).

**Note**: we fix all the hyperparameters of the attacks, so no tuning is required to test every new classifier.

## How to use AutoAttack

### PyTorch models
Import and initialize AutoAttack with

```python
from autoattack import AutoAttack
adversary = AutoAttack(forward_pass, norm='Linf', eps=epsilon, plus=False)
```

where:
+ `forward_pass` returns the logits and takes input with components in [0, 1] (NCHW format expected),
+ `norm = ['Linf' | 'L2']` is the norm of the threat model,
+ `eps` is the bound on the norm of the adversarial perturbations,
+ `plus = True` adds to the list of the attacks the targeted versions of APGD and FAB (AA+).

To apply the standard evaluation, where the attacks are run sequentially on batches of size `bs` of `images`, use

```python
x_adv = adversary.run_standard_evaluation(images, labels, bs=batch_size)
```

To run the attacks individually, use

```python
dict_adv = adversary.run_standard_evaluation_individual(images, labels, bs=batch_size)
```

which returns a dictionary with the adversarial examples found by each attack.

To specify a subset of attacks add e.g. `adversary.attacks_to_run = ['apgd-ce']`.

### TensorFlow models
To evaluate models implemented in TensorFlow, use

```python
import utils_tf
model_adapted = utils_tf.ModelAdapter(logits, x_input, y_input, sess)

from autoattack import AutoAttack
adversary = AutoAttack(model_adapted, norm='Linf', eps=epsilon, plus=False, is_tf_model=True)
```

where:
+ `logits` is the tensor with the logits given by the model,
+ `x_input` is a placeholder for the input for the classifier (NHWC format expected),
+ `y_input` is a placeholder for the correct labels,
+ `sess` is a TF session.

The evaluation can be run in the same way as done with PT models.
