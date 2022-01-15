---
title: ONNX Runtime C++ 개발환경 설정
categories: [AI]
tags: [C++, ONNX Runtime]
parent_project: ONNX Runtime C++ API
layout: project-post
---

ONNX Runtime C++ 개발을 위한 개발환경 설정 (Linux)
==============================================================
ONNX Runtime을 설치하기 위해서는 ONNX Runtime 공식 github 저장소를 찾아가야한다.\\
[ONNX Runtime Release](https://github.com/microsoft/onnxruntime/releases) \\
현재 글을 쓰는 시점에 가장 최신 버전은 1.10.0이다. 당장은 CPU만 사용할 예정이므로 
onnxruntime-linux-x64-1.10.0.tgz를 받아 사용한다. 압축을 풀면 다음과 같이 구성되어 있다.
```console
$ tree .
.
├── GIT_COMMIT_ID
├── include
│   ├── cpu_provider_factory.h
│   ├── onnxruntime_c_api.h
│   ├── onnxruntime_cxx_api.h
│   ├── onnxruntime_cxx_inline.h
│   ├── onnxruntime_run_options_config_keys.h
│   ├── onnxruntime_session_options_config_keys.h
│   └── provider_options.h
├── lib
│   ├── libonnxruntime.so -> libonnxruntime.so.1.10.0
│   └── libonnxruntime.so.1.10.0
├── LICENSE
├── Privacy.md
├── README.md
├── ThirdPartyNotices.txt
└── VERSION_NUMBER

2 directories, 15 files
```
사용할 헤더파일은 ```onnxruntime_cxx_api.h```가 전부다.


이제 링킹만 잘해주면 바로 사용할 수 있다. Makefile의 예제는 다음과 같다.
```make
ONNX_LIB = $(HOME)/path/to/lib/
ONNX_INCLUDE = $(HOME)/path/to/include/

a.out: main.cpp
	g++ main.cpp -I$(ONNX_INCLUDE) -L$(ONNX_LIB) -lonnxruntime
```

