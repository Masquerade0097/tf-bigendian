# tf-bigendian
This repository contains scripts that fix the pretrained TF models (SavedModel/Frozen Model) and make them usable on big-endian systems.

## Problem
As commented in the TF (codebase)[https://github.com/tensorflow/tensorflow/blob/f5fb417ebc18485b7e2493e766d658da539f007c/tensorflow/core/framework/tensor.proto#L35], `tensor_content` is "a serialized raw tensor content from either Tensor::AsProtoTensorContent or
// memcpy in tensorflow::grpc::EncodeTensorToByteBuffer". The byte order for tensor_content is same as the byte order of the machine in which the model is created. The models available at https://github.com/tensorflow/models were created and trained on machines with little-endian byte order which makes the byte order for tensor_content little-endian.

`tensor_content` is a part of the TensorProto message which is embedded inside AttrValue messages embedded inside NodeDef messages embedded inside FunctionDef messages embedded inside FunctionDefLibrary messages embedded inside GraphDef messages embedded inside MetaGraphDef messages embedded inside SavedModel messages embedded inside the saved_model.pb file. This is the reason SavedModel fails to load on big-endian systems.


## Fix
This repository contains 2 python notebooks -

1. tf-saved-model.ipynb
2. tf-frozen-model.ipynb

These scripts traverse the model graph in the TF SavedModel/Frozen model, unwraps the Proto messages and identify the tensors involved. Then byte-swap the tensors having the `tensor_content` field as per the the dataype of the tensors. Then we rewrite the new graph definition of the tf model to a protobuf file which will ready to be use on big-endian systems.

Both of the scripts are tested to work on IBM Z (s390x architecture) machine.
