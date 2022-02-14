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

### Provider
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

### Max Thread Number
```c++
SessionOptions & SetIntraOpNumThreads(int intra_op_num_threads);
SessionOptions & SetInterOpNumThreads(int inter_op_num_threads);
```
일단 이름에서 유추할 수 있듯이 위의 두 메소드는 연산을 수행하는 최대 thread 수를 정하는데 연관있다.
중요한 것은 intra operation thread number와 inter operation thread number의 관계이다.
intra operation thread number는 하나의 operation 내부에서 수행하는 thread 수를 말하고
inter operation thread number는 서로 다른 operation을 수행하는 thread 수를 말한다.


[ONNX Runtime github doc](https://github.com/microsoft/onnxruntime-openenclave/blob/openenclave-public/docs/ONNX_Runtime_Perf_Tuning.md)  

위의 페이지에는 다음과 같은 설명이 나온다.
>- 사용하는 ONNX Runtime이 OpenMP를 사용해 빌드된 경우, intra op thread number를 지정하지 마라
>- 만약 ONNX Runtime이 OpenMP를 사용해 빌드되지 않은 경우, 적절한 intra op thread number를 지정해라
>- inter op num threads (parallel execution이 사용 설정된 경우에만 의미를 가진다.)는 OpenMP에 영향받지 않으므로 항상 적절한 값을 필요로 한다.

즉 사용하고 있는 ONNX Runtime이 OpenMP를 사용하여 빌드된 경우 intra operation thread number는 건드리지 말고 inter operation thread number만 설정해 주면 된다는 말이다.
inter operation thread number도 직접 돌려보면서 최적의 값을 찾는 것이 좋아보인다.

ONNX Runtime 홈페이지에서 OpenMP 사용시 설정할 수 있는 옵션 중에 2가지를 추천해서 가져와 봤다.
```bash
export OMP_NUM_THREADS=n
export OMP_WAIT_POLICY=PASSIVE/ACTIVE
```
보다시피 bash에서 환경변수로 설정해줘야 한다.

*OpenMP는  ONNXRuntime 1.8.0 버전 부터 deprecate 되었다.*


### Graph Optimization Level
```c++
SessionOptions &
SetGraphOptimizationLevel(GraphOptimizationLevel graph_optimization_model);
```
이 메소드는 inference시 사용할 최적화의 정도를 지정하는데 사용된다.
아래의 레벨 중에 하나를 골라 사용하면 된다.

- ORT_DISABLE_ALL 	
- ORT_ENABLE_BASIC 	
- ORT_ENABLE_EXTENDED 	
- ORT_ENABLE_ALL 

개인적인 생각으로 사용가능한 모든 최적화를 사용하고 싶어서 ORT_ENABLE_ALL을 사용했다.
(아마 이 값이 default인 것으로 알고 있다.)

### Save Optimized Model
모델을 로드하는 시간을 줄이는 것이 중요하다면 SessionOption에 Optimize를 설정하고 다음의 메소드를 호출하자.
```c++
SessionOptions &
SetOptimizedModelFilePath(const char *optimized_model_file);
```
위 메소드를 호출하면 ONNX Runtime이 최적화한 모델을 전달한 경로에 저장한다.
이 과정을 거친 후에 다시 로드하면 최적화하는데 소모되는 CPU 자원과 시간을 아낄 수 있다.

### Profiling on / off
```c++
SessionOptions & EnableProfiling(const char *profile_file_prefix);
SessionOptions & DisableProfiling();
```
세션의 성능 파악을 위해 데이터를 기록한다.
ONNX Runtime으로 세션을 profiling하기 위해서는 위의 ```EnableProfiling``` 옵션을 켜고 inference한다.
Session이 종료된 이후에 인자로 전달한 profile_file_prefix 폴더에 json 파일이 생성되는데 이를 이용해 inference 과정을
분석할 수 있다.
MS Edge나 Google Chrome을 켜고 edge://tracing 혹은 chrome://tracing으로 들어가서 생성된 json 파일을 로드하면
일반적인 웹브라우져의 프로파일 기능을 사용해 분석할 수 있다.

### Execution Mode
```c++
SessionOptions & SetExecutionMode(ExecutionMode execution_mode);
```
이 옵션은 inter operation 간에 threading을 사용할 것인지 설정하는 것이다.
다음의 옵션 중 하나를 전달하면 된다.
- ORT_SEQUENTIAL 	
- ORT_PARALLEL


여기까지 대략적인 세션 옵션에 대해 적었다.
원래 ONNX Runtime C++ API는 C API의 랩퍼인 관계로 여기에 명시되지 않은 옵션들 중에는 C API를 직접사용
해야하는 경우도 있다.
그런 경우는 C API 특유의 문제를 정확히 파악하고 사용하기 바란다.
