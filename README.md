# TASO: A Tensor Algebra SuperOptimizer for Deep Learning

TASO optimizes the computation graphs of DNN models using automatically generated and verified graph transformations.
For a given DNN model, the transformations build a large search space of potential computation graphs.
TASO employs a cost-based search algorithm to explore the space, and automatically discovers an optimized computation graph.

## Installation

### Prerequisties

* Recent C++ compiler supporting C++11
* CMAKE 3.2 or higher
* ProtocolBuffer 3.6.1 or higher
* Cython 0.28 or higher
* ONNX 1.5 or higher
* CUDA 9.0 or higher and CUDNN 7.0 or higher

### Install from Source

* To get started, clone the TASO source code from github.
```
git clone https://www.github.com/jiazhihao/taso
cd taso
```

* Build the TASO runtime library. The configuration of the TASO runtime can be modified by `config.cmake`. The default configuration builds the CUDA backend and automatically finds the CUDA libraries (e.g., cuDNN, cuBLAS). You can manually choose a CUDA path by changing `set(USE_CUDA ON)` to `set(USE_CUDA /path/to/cuda/library`). MKL support is coming soon.
```
mkdir build; cd build; cmake ..
sudo make install -j 4
```

* Install the TASO python package.
```
cd python
python setup.py install
```

## Using TASO 

### Optimize Pre-trained ONNX Models

TASO can be used to optimize pre-trained DNN models in the [ONNX](https://onnx.ai/) format, and this can be done in just a few lines of Python code.
The following code snippet shows how to load a pre-trained DNN model from ONNX, optimize the model, and save the optimized model into a ONNX file.
```python
import taso
import onnx

old_model = taso.load_onnx("/path/to/load/onnx/model")
taso_graph = taso.optimize(old_model)
new_model = taso.export_onnx(taso_graph)
onnx.save(new_model, "/path/to/save/new/onnx/model")
```
The optimized model has the same accuracy as the original and can be directly used by existing deep learning frameworks.
The following figure shows the end-to-end inference performance comparison on a NVIDIA V100 GPU.
The original and TASO-optimized ONNX files are available in the `onnx` folder.
<div align="center">
  <img src="https://github.com/jiazhihao/TASO/blob/master/figures/inference.png">
</div>

### Optimizing DNN Models using the Python Interface

TASO can also optimize arbitrary DNN models using the Python interface. 
The following code snippet builds the left-most DNN graph depicted in the figure. TASO automatically performs a series of non-trivial transformations, and eventually discovers the right-most DNN graph, which is 1.3x faster on a V100 GPU. More DNN examples are available in the `examples` folder.

<div align="center">
  <img src="https://github.com/jiazhihao/TASO/blob/master/figures/graph_subst.png">
</div>

```python
import taso
import onnx

#Build DNN model
graph = taso.new_graph()
input = graph.new_input(dims=(1,128,56,56))
w1 = graph.new_weight(dims=(128,128,3,3))
w2 = graph.new_weight(dims=(128,128,1,1))
w3 = graph.new_weight(dims=(128,128,3,3))
left = graph.conv2d(input=input, weight=w1, strides=(1,1), padding="SAME", activation="RELU")
left = graph.conv2d(input=input, weight=w3, strides=(1,1), padding="SAME")
right = graph.conv2d(input=input, weight=w2, strides=(1,1), padding="SAME", activation="RELU")
output = graph.add(left, right)
output = graph.relu(output)

#Optimize DNN model
new_graph = taso.optimize(graph)
onnx_model = taso.export_onnx(new_graph)
onnx.save(onnx_model, "/path/to/save/new/onnx/model")
```

## Publication
* Zhihao Jia, Oded Padon, James Thomas, Todd Warszawski, Matei Zaharia, and Alex Aiken. [TASO: Optimizing Deep Learning Computation with Automated Generation of Graph Substitutions](http://theory.stanford.edu/~aiken/publications/papers/sosp19.pdf). In Proceedings of the Symposium on Operating Systems Principles (SOSP), Ontario, Canada, October 2019.

* Zhihao Jia, James Thomas, Todd Warszawski, Mingyu Gao, Matei Zaharia, and Alex Aiken. [Optimizing DNN Computation with Relaxed Graph Substitutions](https://theory.stanford.edu/~aiken/publications/papers/sysml19b.pdf). In Proceedings of the Conference on Systems and Machine Learning (SysML), Palo Alto, CA, April 2019.

