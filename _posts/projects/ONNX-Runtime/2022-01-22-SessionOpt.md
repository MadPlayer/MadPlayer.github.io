---
title: ONNX Runtime C++ Ort::SessionOption 사용하기
categories: [AI]
tags: [C++, ONNX Runtime]
parent_project: ONNX Runtime C++ API
layout: project-post
order: 3
---

```Ort::SessionOptions```는 세션에 대한 다양한 설정을 제공한다.

```Ort::SessionOptions``` 설정
-------------------------------

생성자는 인자가 없다. 그러므로 생성은 다음과 같다.
```c++
Ort::SessionOptions session_options;
```
```Ort::Session_Options```의 메소드는 모두 자신의 reference를 리턴하기 때문에 버전 체인 형태로 사용
할 수 있다.

가장 관심 가는 부분은 아래의 Provider 부분이다.
```c++
SessionOptions &
AppendExecutionProvider_CUDA(const OrtCUDAProviderOptions &provider_options);
 
SessionOptions &
AppendExecutionProvider_ROCM(const OrtROCMProviderOptions &provider_options);
 
SessionOptions &
AppendExecutionProvider_OpenVINO(const OrtOpenVINOProviderOptions &provider_options);
 
SessionOptions &
AppendExecutionProvider_TensorRT(const OrtTensorRTProviderOptions &provider_options);
```
하지만 위의 부분은 이 포스트에서 다루지 않는다.
왜냐하면 우리는 지금 CPU로도 inference하지 못하는 중이기 때문이다.
GPU를 이용하는 부분은 차후에 다룬다.

많은 설정이 있지만 다음의 것만 간략하게 알아보자.

```c++
SessionOptions & SetIntraOpNumThreads(int intra_op_num_threads);
SessionOptions & SetInterOpNumThreads(int inter_op_num_threads);
```
일단 이름에서 유추할 수 있듯이 위의 두 메소드는 연산을 수행하는 최대 thread 수를 정하는데 연관있다.
중요한 것은 intra operation과 inter operation의 관계이다.



```c++
SessionOptions &
SetGraphOptimizationLevel(GraphOptimizationLevel graph_optimization_model);
```

```c++
SessionOptions & EnableProfiling();
SessionOptions & DisableProfiling();
```

```c++
SessionOptions & SetExecutionMode(ExecutionMode execution_mode);
```
