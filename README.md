# Label-free-Network-Compression
Caffe implementation of "Learning Compression from Limited Unlabeled Data" (ECCV2018).

### How to use?
##### Part I. Create Quantized Model and Prototxt
```shell
# Python2.7
vim config.py # edit pycaffe_path / model_name / train_dataset path / val_dataset path
python quantize.py # quantize weights to 4-bit
python renorm.py # BN re-normalization in CPU mode
python act_quan.py # quantize activations to 8-bit
```
##### Part II. Test on validation set
1. Add `quantize.cpp` and `quantize.cu` to `your_caffe_root/src/caffe/layers/`.
2. Add `quantize.hpp` to `your_caffe_root/include/caffe/layers/`.
3. ```make -j2```
4. ```./build/tools/caffe test --weights /your/BN_quantized_caffemodel/in/config.py --model /your/val_prototxt/in/config.py --gpu XX --iterations 1000 # val_batch_size = 50 in default (Line 10 in config.py)``` 

##### WARNING:

`renorm.py` will use 1K images to update BN parameters in default. The memory consumption can be pretty large for deep networks (>12G).
You may edit Line 8 in `config.py` to alleviate this problem.

### Results
| Models | Weights | Activations | Top-1 (%) | Top-5 (%) 
| ------ | -----| ------ | ---------- | -----------
| [AlexNet-BN](https://github.com/HolmesShuan/AlexNet-BN-Caffemodel-on-ImageNet) | 32-bit | 32-bit | 60.43 | 82.47
| ReNorm |  4-bit | 8-bit
| [ResNet-18](https://github.com/HolmesShuan/ResNet-18-Caffemodel-on-ImageNet) | 32-bit | 32-bit | 69.08 | 89.03
| ReNorm |  4-bit | 8-bit
| [ResNet-50](https://github.com/KaimingHe/deep-residual-networks) | 32-bit | 32-bit | 75.30 | 92.11
| ReNorm |  4-bit | 8-bit

##### Details: 

1. We report the 224x224 single-crop (cropped from `256xN/Nx256` images) validation accuracy on the ImageNet validation set. BN parameters are updated using 1K randomly selected unlabeled training images.

2. We quantize the first and last layer to 8-bit using fixed-point quantizer.
