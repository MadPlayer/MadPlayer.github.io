---
title: ONNX Runtime C++ Ort::Env 사용하기
categories: [AI]
tags: [C++, ONNX Runtime]
parent_project: ONNX Runtime C++ API
layout: project-post
order: 2
---

ONNX Runtime은 모델을 inference 하기 위해 Session을 생성해야 하는데 그 전에 몇가지 선행사항이 있다.
그 선행사항 중 하나가 바로 ```Ort::Env```를 설정하는 일이다.

```Ort::Env```의 목적
-----------------------

객체 생성자는 다음의 것을 사용했다.
다른 생성자에 대한 정보는 이 포스트 멘 아래에 제시된 링크에서 찾아볼 수 있다.
하지만 대부분의 경우 함수형 말고는 어떠한 정보도 얻을 수 없을 것이다.
대신 C++ document는 아니지만 공식 문서는 github에서 찾을 수 있다.
[onnx runtime github document](https://github.com/microsoft/onnxruntime-openenclave/blob/openenclave-public/docs/ONNX_Runtime_Perf_Tuning.md) 

```c++
Ort::Env (OrtLoggingLevel logging_level=ORT_LOGGING_LEVEL_WARNING, const char *logid="");
```

생성자를 통해 유추하면 ```Ort::Env```는 logging level을 정하는 용도로 사용할 수 있는 것 같다.
Ort::Env는 Session이 유지되는 동안 해제되면 안되므로 같은 스코프에 생성해 유지하자.
생성은 다음과 같이 이루어진다.
```c++
Ort::Env env{ORT_LOGGING_LEVEL_WARNING, "test env"};
```

이전에 언급한바 API doc이 없는 것과 같다.
전혀 어떠한 설명도 없다.
다만 성능을 위해서 logging level 정도는 조절해야 하지 않겠는가.

공식 API doc에 몇 안되는 설명을 가져와봤다.

- ORT_LOGGING_LEVEL_VERBOSE  
  Verbose informational messages (least severe)

- ORT_LOGGING_LEVEL_INFO  
  Informational messages.

- ORT_LOGGING_LEVEL_WARNING  
  Warning messages.

- ORT_LOGGING_LEVEL_ERROR  
  Error messages.

- ORT_LOGGING_LEVEL_FATAL  
  Fatal error messages (most severe)


위에서 설명한 생성자 이외에도 몇가지가 더 있고 또 메소드도 있지만 그것은 차후에 아래에 적도록 하겠다.
