---
title: Network Simulator 3 소개
categories: [Network]
tags: [C++, Network, Simulator]
parent_project: Network Simulator 3
layout: project-post
order: 1
---

Network Simulator 3를 사용하기 전에 알아야할 정보를 나열했다.
- Network Simulator 3는 C++로 작성되었으며 실험을 위한 스크립트나 프로토콜도 C++로 작성해야한다.
- 스크립트를 Python을 이용해 작성할 수도 있다.
- 시뮬레이션은 Single Core로 동작하며 상당한 시간이 소요된다.
- OS7 레벨의 Transport layer 아래의 프로토콜도 실험할 수 있다. (wifi, 5G와 같은 경우도)
- Transport Layer 프로토콜은 UDP, TCP만 지원한다. (기본제공)
- 다양한 TCP 혼잡제어가 구현되어 있다.

Network Simulator 3 설치 및 환경설정
=================================
NS3의 주요 개발환경은 MAC이나 Linux라고 한다.
그러니 쉬운 설치를 위해 Windows는 피하자.
설사 Windows에서 설치하더라도 MinGW등을 이용할 가능성이 농후하다.
본 문서는 Ubuntu 22.04 lts에서 진행되었다.  
**(만약 Direct Code Execution을 진행하기 위해 이 문서를 읽고 있다면 바로 DCE로 넘어가라.
DCE의 경우, 원활한 설치 진행을 위해 Ubuntu 20.04 lts에서 진행된다.)**

NS3 소스 얻기
-------------

[https://github.com/nsnam/ns-3-dev-git](https://github.com/nsnam/ns-3-dev-git)  

```bash
# 코드 다운로드
git clone https://github.com/nsnam/ns-3-dev-git
cd ns-3-dev-git

# 원하는 버전으로 이동 (Github의 Release를 참고하라)
git checkout -l ns-3.37

# 원하는 버전에서 내가 개발을 시작할 branch 생성
git branch my-protocol HEAD
git switch my-protocol
```
위의 명령에서는 3.37 버전으로 checkout 했지만 자신이 원하는 버전으로 하면된다.
이제 부터는 NS3를 빌드하고 예제 스크립트들을 실행할 것이다.
이전 버전의 NS3는 waf라는 빌드도구를 이용했지만 NS3 3.37은 ns3라는 별도의 python script를 이용하게 변경되었다.
waf를 사용하는 빌드의 경우, 여기에서 특별히 다루지 않고 Direct Code Execution 부분에서 다룰것이다.

NS3 Build
-----------------
ns-3-dev 폴더 안에 구성은 아래와 같다.
```sh
(base) madplayer@madplayer-desktop:~/Public/test/ns-3-dev-git (my-protocol) 
-> ls
AUTHORS        CMakeLists.txt   examples   RELEASE_NOTES.md  utils
bindings       contrib          LICENSE    scratch           utils.py
build-support  CONTRIBUTING.md  ns3        src               VERSION
CHANGES.md     doc              README.md  test.py
```
- `ns3`는 빌드, 스크립트 실행을 하기위한 Python Script이다.
- `scratch` 폴더는 프로토콜을 개발하고 실험 스크립트를 작성하는데 사용된다.
- `doc` 폴더는 다양한 설명문서들이 있다.


`ns3`를 이용해 Configuration을 해준다.
```sh
# 프로토콜 개발시 Optimization Level은 명시하지 않는 것이 좋다.
# Optimization으로 빌드시 시뮬레이션 프린트문이 뜨지 않기 때문이다.
./ns3 configure --enable-examples --enable-tests -d optimize
```
위 명령을 실행하면 다음과 같은 출력이 나와야한다.

```sh
-- ---- Summary of optional ns-3 features:
Build profile                 : default
Build directory               : /home/madplayer/Public/test/ns-3-dev-git/build
Build version embedding       : OFF (not requested)
BRITE Integration             : OFF (missing dependency)
DES Metrics event collection  : OFF (not requested)
DPDK NetDevice                : OFF (not requested)
Emulation FdNetDevice         : ON
Examples                      : ON
File descriptor NetDevice     : ON
GNU Scientific Library (GSL)  : ON
GtkConfigStore                : ON
LibXml2 support               : ON
MPI Support                   : OFF (not requested)
ns-3 Click Integration        : OFF (missing dependency)
ns-3 OpenFlow Integration     : OFF (missing dependency)
Netmap emulation FdNetDevice  : OFF (missing dependency)
PyViz visualizer              : OFF (missing dependency)
Python Bindings               : OFF (not requested)
SQLite support                : ON
Tap Bridge                    : ON
Tap FdNetDevice               : ON
Tests                         : ON
```
`ns3`는 cmake의 wrapper이기 때문에 configure 해결법도 동일하다.
즉, 필요한 라이브러리들은 apt를 이용해 설치하면서 진행하면 된다.

마지막으로 
```sh
./ns3 build
```
을 실행하면 빌드가 시작된다.

### Documentation Build (Optional)
NS3의 `doc` 폴더에 들어가면 다음과 같이 구성되어 있다.
```sh
(base) madplayer@madplayer-desktop:~/Public/test/ns-3-dev-git/doc (my-protocol) 
-> ls
build.txt      contributing.txt            main.h  modules          ns3_html_theme
codingstd.txt  doxygen.conf                manual  namespace-2.dia  release_steps.txt
contributing   doxygen.warnings.report.sh  models  namespace-2.png  tutorial
```
manual, tutorial, models을 html로 빌드하기 위해서는

```sh
sudo apt install sphinx packagekit-gtk3-module libcanberra-gtk-module

cd manual
# 혹은
cd tutorial
# 혹은
cd models

make html
```
본 진행환경은 Ubuntu 22.03 이므로 ImageMagic이 설치되어있다고 가정한다.
만약 Security Policy `PS` 관련 문제가 발생하면 ImageMagic 문제로써 보안 설정을 수정한다.

```sh
sudo vi /etc/ImageMagick-6/policy.xml
```
이 파일을 열면 가장 아래에
```xml
  <!-- disable ghostscript format types -->
  <policy domain="coder" rights="none" pattern="PS"  />
  <policy domain="coder" rights="none" pattern="PS2" />
  <policy domain="coder" rights="none" pattern="PS3" />
  <policy domain="coder" rights="none" pattern="EPS" />
  <policy domain="coder" rights="none" pattern="PDF" />
  <policy domain="coder" rights="none" pattern="XPS" />
</policymap>
```
이전 ghostscript의 버그로 인해 막아놓은 옵션들이 있다.
이들중 "PS" 항목만 주석처리하면 된다.


빌드가 끝나면 build/html 폴더가 생성된다.
이를 보기위해서는 해당 html 폴더안에서 간단한 http 서버를 실행하면된다.
다양한 http 서버가 있지만 `ns3`를 실행시키기 위해서는 Python3가 요구되므로 Python3를 이용하겠다.

```sh
# build/html 폴더 내에서
python3 -m http.server 12345
```
이제 webbrowser로 localhost:12345로 접속하면 문서를 볼 수 있다.
http 말고 latex등 다른 옵션도 있지만 여기서 빌드하지는 않겠다.


NS3 Example 실행
-----------------
이제 빌드가 완료되었으니 간단한 스크립트들을 돌려보자.  
NS3를 빌드한 폴더에서 다음을 실행해보자
```sh
./ns3 run scratch/scratch-simulator.cc

# 실행결과는 다음과 같이 나와야...
Scratch Simulator
```

기존에 제공되는 예제도 살펴보자
```sh
./ns3 run examples/tcp/tcp-bbr-example.cc
```
위 예제는 TCP의 BBR Congestion Control을 실험하는 스크립트이다.
실행 후에는 `bbr-results`라는 폴더에 다양한 시뮬레이션 결과들이 저장되어 있는것을 확인할 수 있다.


NS3로 실험 개발
---------------
이제 자신의 프로토콜을 NS3에 어떻게 개발하는지를 간단하게 설명하겠다.
자신이 개발한 프로토콜을 NS3에 Contribute할 것이 아니라면 모든 코드는 `scratch` 폴더 안에서 이루어진다.
그러므로 코드관리를 NS3의 git tree를 이용해도 되고 scratch 안에 자신만의 git tree를 생성해도 된다.
매우 친절하게도 이미 `scratch` 폴더 안에 `subdir` 라는 별도의 예제 프로젝트가 생성되어 있다.  
이를 실행해보자.
```sh
./ns3 run scratch/subdir/scratch-subdir.cc

#출력은
Scratch Subdir
```
즉 자신만의 폴더를 생성한 후 스크립트를 작성하면되는 것이다.


맺음
----
이후 포스트에서 NS3를 이용해 스크립트 작성하는 법을 간단히 설명하려고 한다.
대부분 문서나 넷상에서 검색하면 알수있는 정보들이 많이 있으므로 자세하게 포스팅하지는 않을것 같다.
