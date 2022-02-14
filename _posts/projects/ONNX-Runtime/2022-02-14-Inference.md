---
title: ONNX Runtime C++ Inference 하기
categories: [AI]
tags: [C++, ONNX Runtime]
parent_project: ONNX Runtime C++ API
layout: project-post
order: 5
---

이번 포스트에는 드디어 Session을 이용해 model inference를 진행한다.
이전 포스트에서 얻은 정보들을 바탕으로 model을 inference한다.

Inference Step 1 - Allocate Input/Output Buffers
--------------------
이전 포스트에서 얻은 정보들은 input/output shape, input/output 개수 등이다.
이제 input/output에 해당하는 buffer를 메모리에 할당하고 ```Ort::Value```라는 객체에 저장해서 ```Ort::Session::Run``` 메소드에 전달해야 한다.  
특히 input의 경우 전처리된 이미지 등의 값을 연속적인 메모리에 저장한 값이다.  

```Ort::Value```를 생성하기 위해서는 ```Ort::MemoryInfo```를 생성해야 한다.  
```Ort::MemoryInfo```는 우리가 사용하는 버퍼 메모리에 대한 정보이다.

```c++
auto allocator_info = Ort::MemoryInfo::CreateCpu(OrtDeviceAllocator, OrtMemTypeDefault);
```
```Ort::MemoryInfo::CreateCpu```의 첫번째 인자는 allocator의 종류인데 다음의 종류가 있다.

- OrtInvalidAllocator
- OrtDeviceAllocator
- OrtArenaAllocator

두번째 인자는 우리가 사용한 allocator가 할당하는 memory의 종류인데 우리는 CPU inference하는 중이므로 
OrtMemTypeDefault를 사용하면 된다. Memory 종류는 다음의 종류가 있다.

- OrtMemTypeCPUInput  
  CPU가 아닌 Execution Provider (cudnn 등)이 사용하는 CPU Memory
- OrtMemTypeCPUOutput  
  CPU가 아닌 Execution Provider가 생성한 CPU 접근가능한 Memory (i.e. CUDA_PINNED)
- OrtMemTypeCPU  
  일시적으로 CPU가 접근가능한 Execution Provider가 할당한 Memory (i.e. CUDA_PINNED)
- OrtMemTypeDefault  
  Execution Provider가 사용하는 default 할당자
  
나중에 GPU를 사용하게 되면 CUDA_PINNED 메모리를 활용해 최적화할 수 있을 것이다.  


아래의 예시는 Resnet18 모델과 같이 단일 입력, 단일 출력인 모델의 경우 ```Ort::Value```를 생성하는 예시이다.  
```Ort::Value```는 메모리의 포인터를 빌린다.  
그러므로 ```Ort::Value```는 메모리 해제를 책임지지 않는다.
```c++
Ort::Value input = Ort::CreateValue(allocator_info, input_vec.data(), input_vec.size(),
                                    input_shape.data(), input_shape.size());

Ort::Value output = Ort::CreateValue(allocator_info, output_vec.data(), output_vec.size(),
                                    output_shape.data(), output_shape.size());
```

만약 자신이 사용하고자하는 모델의 입력이나 출력이 복수인 경우 ```Ort::Value```를 연속적인 메모리에 저장할 수 있는 방법을 사용해야 한다.
```c++
std::vector<Ort::Value> inputs;
std::vector<Ort::Value> outputs;
```

Inference Step 2 -- Inference
-------------------------------

이제 inference할 준비가 모두 끝났다. 이전 포스트에서 초기화한 session은 Run이라는 메소드를 이용해서 inference한다.
```c++
session.Run(Ort::RunOptions{nullptr}, input_name, &input, 1, output_name, &output, 1);
```

혹은 다수의 입출력인 경우
```c++
session.Run(Ort::RunOptions{nullptr}, input_names, inputs.data(), inputs.size(),
            output_names, outputs.data(), outputs.size());
```

위의 과정이 종료되면 outputs에는 모델의 inference한 결과가 담기게 된다.  
정확하게는 ```Ort::Value``` 생성시 넘겨준 buffer에 결과가 담긴다.  
이 포스트에서 다룬 내용은 CPU inference를 주제로 하기때문에 ```Ort::IoBinding```에 대해서는 다루지 않는다.
