Quick Start
===========

..  toctree::
    :hidden:

    Notebook Example <compression_pipeline_example>


Model compression usually consists of three stages: 1) pre-training a model, 2) compress the model, 3) fine-tuning the model. NNI mainly focuses on the second stage and provides very simple APIs for compressing a model. Follow this guide for a quick look at how easy it is to use NNI to compress a model. 

A `compression pipeline example <./compression_pipeline_example.rst>`__ with Jupyter notebook is supported and refer the code :githublink:`here <examples/notebooks/compression_pipeline_example.ipynb>`.

Model Pruning
-------------

Here we use `level pruner <../Compression/Pruner.rst#level-pruner>`__ as an example to show the usage of pruning in NNI.

Step1. Write configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^

Write a configuration to specify the layers that you want to prune. The following configuration means pruning all the ``default``\ ops to sparsity 0.5 while keeping other layers unpruned.

.. code-block:: python

   config_list = [{
       'sparsity': 0.5,
       'op_types': ['default'],
   }]

The specification of configuration can be found `here <./Tutorial.rst#specify-the-configuration>`__. Note that different pruners may have their own defined fields in configuration. Please refer to each pruner's `usage <./Pruner.rst>`__ for details, and adjust the configuration accordingly.

Step2. Choose a pruner and compress the model
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

First instantiate the chosen pruner with your model and configuration as arguments, then invoke ``compress()`` to compress your model. Note that, some algorithms may check gradients for compressing, so we may also define a trainer, an optimizer, a criterion and pass them to the pruner.

.. code-block:: python

   from nni.algorithms.compression.pytorch.pruning import LevelPruner

   pruner = LevelPruner(model, config_list)
   model = pruner.compress()

Some pruners (e.g., L1FilterPruner, FPGMPruner) prune once, some pruners (e.g., AGPPruner) prune your model iteratively, the masks are adjusted epoch by epoch during training.

So if the pruners prune your model iteratively or they need training or inference to get gradients, you need pass finetuning logic to pruner.

For example:

.. code-block:: python

   from nni.algorithms.compression.pytorch.pruning import AGPPruner

   pruner = AGPPruner(model, config_list, optimizer, trainer, criterion, num_iterations=10, epochs_per_iteration=1, pruning_algorithm='level')
   model = pruner.compress()

Step3. Export compression result
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

After training, you can export model weights to a file, and the generated masks to a file as well. Exporting onnx model is also supported.

.. code-block:: python

   pruner.export_model(model_path='pruned_vgg19_cifar10.pth', mask_path='mask_vgg19_cifar10.pth')

Plese refer to :githublink:`mnist example <examples/model_compress/pruning/naive_prune_torch.py>` for example code.

More examples of pruning algorithms can be found in :githublink:`basic_pruners_torch <examples/model_compress/pruning/basic_pruners_torch.py>` and :githublink:`auto_pruners_torch <examples/model_compress/pruning/auto_pruners_torch.py>`.


Model Quantization
------------------

Here we use `QAT  Quantizer <../Compression/Quantizer.rst#qat-quantizer>`__ as an example to show the usage of pruning in NNI.

Step1. Write configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   config_list = [{
       'quant_types': ['weight', 'input'],
       'quant_bits': {
           'weight': 8,
           'input': 8,
       }, # you can just use `int` here because all `quan_types` share same bits length, see config for `ReLu6` below.
       'op_types':['Conv2d', 'Linear'],
       'quant_dtype': 'int',
       'quant_scheme': 'per_channel_symmetric'
   }, {
       'quant_types': ['output'],
       'quant_bits': 8,
       'quant_start_step': 7000,
       'op_types':['ReLU6'],
       'quant_dtype': 'uint',
       'quant_scheme': 'per_tensor_affine'
   }]

The specification of configuration can be found `here <./Tutorial.rst#quantization-specific-keys>`__.

Step2. Choose a quantizer and compress the model
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   from nni.algorithms.compression.pytorch.quantization import QAT_Quantizer

   quantizer = QAT_Quantizer(model, config_list)
   quantizer.compress()


Step3. Export compression result
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

After training and calibration, you can export model weight to a file, and the generated calibration parameters to a file as well. Exporting onnx model is also supported.

.. code-block:: python

   calibration_config = quantizer.export_model(model_path, calibration_path, onnx_path, input_shape, device)

Plese refer to :githublink:`mnist example <examples/model_compress/quantization/QAT_torch_quantizer.py>` for example code.

Congratulations! You've compressed your first model via NNI. To go a bit more in depth about model compression in NNI, check out the `Tutorial <./Tutorial.rst>`__.