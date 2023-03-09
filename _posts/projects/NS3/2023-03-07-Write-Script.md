---
title: Network Simulator 3 사용법
categories: [Network]
tags: [C++, Network, Simulator]
parent_project: Network Simulator 3
layout: project-post
order: 2
---

스크립트 작성
===============
`example` 폴더안에 가장 간단한 예제인 `udp-echo.cc` 를 살펴보자
`udp-echo.cc` 는 `example/udp` 폴더에 위치하고 있다.

사실 주어진 예제들만 잘 살펴봐도 많은 것을 알 수 있는데 `udp-echo.cc`는 서버 & 클라이언트 간에 udp로 
메시지를 주고받는 간단한 시뮬레이션이다.
NS3의 스크립트는 다음의 단계로 작성한다.
1. [네트워크 시뮬레이션 설계](#네트워크-시뮬레이션-설계) 
2. [엔드 포인트 할당](#엔드-포인트-할당)
3. [네트워크 하드웨어 설정](#네트워크-하드웨어-설정)
4. [IP 할당](#ip-할당)
5. [각 엔드 포인트에 응용 할당](#각-엔드-포인트에-응용-할당)
6. [테스트 통계 측정 설정](#테스트-통계-측정-설정)
7. [시뮬레이터 시작](#시뮬레이터-시작)

네트워크 시뮬레이션 설계
-------------------
시뮬레이션 스크립트를 작성하기 전에 시나리오, 네트워크 구성을 미리 구상하는 것이 필요하다.
`udp-echo.cc`를 보면 각 항목을 다음과 같이 분석할 수 있다.

시나리오는 UDP를 이용해 Echo Server를 Client가 사용하는 것이다.
Echo Server는 그 이름에서 알 수 있듯이 Client가 보넨 메시지와 동일한 메시지를 다시 Client에게 전송한다.

```c++
// Network topology
//
//       n0    n1   n2   n3
//       |     |    |    |
//       =================
//              LAN
//
// - UDP flows from n0 to n1 and back
// - DropTail queues
// - Tracing of queues and packet receptions to file "udp-echo.tr"
```
설명을 읽어보면 n0 (Client)가 n1 (Echo Server)에게 메시지를 전송하고 다시 n1이 n0에게 동일한 메시지를 돌려주는 시나리오이다.
아스키 아트를 보면 LAN에 n2, n3도 연결되어 있는데 왜 있는지는 모르겠다.

위의 시나리오를 구현하기 위해서 필요한 헤더들은 다음과 같다.
```c++
#include "ns3/applications-module.h" // 노드에 Application 설치 위한것
#include "ns3/core-module.h"         // 어떤 스크립트든 포함해야하는것
#include "ns3/csma-module.h"         // 유선 하드웨어를 위한 것
#include "ns3/internet-module.h"     // IP Address 설정을 위한 것
```

main 함수 이전에 몇줄이 더 있는데 살펴보면
```c++
#include <fstream>

using namespace ns3;

NS_LOG_COMPONENT_DEFINE("UdpEchoExample");
```
`fstream`은 C++의 표준 라이브러리의 일부로 데이터를 파일로 출력하기 위한 것이다.
`NS_LOG_COMPONENT_DEFINE`는 현재 파일을 Logging 즉, 시뮬레이션 동안 화면에 메시지를 출력할때 등록하는 메크로이다. 시뮬레이션 중에 로그가 출력되면 "UdpEchoExample"이라는 이름과 함께 출력되어 어느 파일의 메시지인지 구분할 수 있다.

엔드 포인트 할당
----------------
```c++
//
// Explicitly create the nodes required by the topology (shown above).
//
NS_LOG_INFO("Create nodes.");
NodeContainer n;
n.Create(4);
```
아주 친절하게도 노드를 생성한다고 적어놓았다.
이렇게 생성된 노드는 시뮬레이션에서 엔드포인트 역할을 한다.

위의 코드는 사실 조금 보기 불편한데, `NodeContainer`를 초기화하면서 노드를 생성할 수 있기 때문이다.
다음과 같이 수정하여 사용할 수 있다.

```c++
NodeContainer n(4);
```
이제 노드들에 TCP/UDP/IP 기능을 부여한다.
```c++
InternetStackHelper internet;
internet.Install(n);
```

네트워크 하드웨어 설정
--------------------
UDP Echo 예제에서 CSMA로 유선랜을 설정해줄것이다.
CSMA는 유선이랑 상관없는데? 이럴수도 있는데 그렇게 못미더우면 
[https://www.nsnam.org/docs](https://www.nsnam.org/docs/release/3.10/manual/html/csma.html) 
여기에 들어가보면 된다.
```c++
NS_LOG_INFO("Create channels.");
//
// Explicitly create the channels required by the topology (shown above).
//
CsmaHelper csma;
csma.SetChannelAttribute("DataRate", DataRateValue(DataRate(5000000)));
csma.SetChannelAttribute("Delay", TimeValue(MilliSeconds(2)));
csma.SetDeviceAttribute("Mtu", UintegerValue(1400));
NetDeviceContainer d = csma.Install(n);
```
하드웨어를 설정하기 위해서도 Helper를 제공한다.
NS3는 일반적인 C++ 객체를 초기화하는 것과 다른 방식을 요구한다.
`SetChannelAttribute`라는 메소드를 이용해 LAN을 설정한다.
이는 단순히 맴버변수를 변경하는 것보다 Log 출력이나 정보 Tracing시 풍부한 정보를 제공할 수 있기 때문으로 생각된다.
NS3의 클래스가 어떤 Attribute를 가지고 어떻게 설정하는지는 NS3 문서를 찾아보면 자세히 설명되어 있다.
또 눈여겨볼 지점은 NS3는 암시적 타입변환(Implicit Casting)을 경계하는 것이다.
모든 값을 명시적으로 캐스팅해줘야하므로 `DataRateValue(DataRate(5000000))`과 같은 표현이 자주 등장한다. 사실 대부분 명시적 캐스팅이 코드를 읽기도 좋고 오류를 줄일 수 있는 방법임은 자명하다.


역시나 마지막으로 설정이 완료된 `CsmaHelper`를 이용해 노드들에 설치해준다.
이전과 다른점은 설치 후 Network Device Container 즉, Network Device들을 리턴한다는 것이다.
이후 Network Device들에 IP를 할당하게 된다.
노드가 두개 이상의 Network Device를 가진 경우에 IP 할당에 모호함을 방지한다.

IP 할당
----------------------
```c++
//
// We've got the "hardware" in place.  Now we need to add IP addresses.
//
NS_LOG_INFO("Assign IP Addresses.");
if (useV6 == false)
  {
    Ipv4AddressHelper ipv4;
    ipv4.SetBase("10.1.1.0", "255.255.255.0");
    Ipv4InterfaceContainer i = ipv4.Assign(d);
    serverAddress = Address(i.GetAddress(1));
  }
 else
   {
     Ipv6AddressHelper ipv6;
     ipv6.SetBase("2001:0000:f00d:cafe::", Ipv6Prefix(64));
     Ipv6InterfaceContainer i6 = ipv6.Assign(d);
     serverAddress = Address(i6.GetAddress(1, 1));
   }
```
코드를 읽어보면 알 수 있듯이 IPv6와 IPv4를 설정하는 방법이 구현되어 있다.
Internet을 노드에 설치한 후 얻은 디바이스들에 IP를 할당하는 것이므로
`Ipv4AddressHelper`의 Install 메소드에 네트워크 디바이스들을 넘겨준다.
할당된 IP는 다시 `Ipv4InterfaceContainer`를 이용해 특정 노드의 IP를 얻을 수 있다. 
여기서 서버(1번 노드)의 IP를 얻은 이유는 echo 클라이언트가 서버에 UDP packet을 전송해야하기 때문이다.

각 엔드 포인트에 응용 할당
--------------------
드디어 노드들에 응용프로그램을 할당한다.  
우선 Echo 서버부터 설정해보자.
Echo 서버 - 클라이언트는 NS3가 제공하는 기본응용프로그램이다.
그러므로 여기는 이미 잘 작성된 `UdpEchoServerHelper`를 이용해 노드에 Echo 서버를 설치할 수 있다.
port는 9번으로 설정했고 생성한 노드들 중 두번째 노드에 설치했다.
두번째 노드 즉 1번 노드는 생성한 네개의 노드들 중 서버로 사용하기 위해 계획된 것으로 스크립트
구상부분에서부터 계획된 것이다.
NS3의 응용 설치 Helper가 응용을 노드 혹은 노드들에 설치한 후에는 설치한 응용들을
`ApplicationContainer`로 반환한다.
`ApplicationContainer`를 이용해 시뮬레이션 시간으로 언제 시작하고 종료할지 설정해준다.
```c++
NS_LOG_INFO("Create Applications.");
//
// Create a UdpEchoServer application on node one.
//
uint16_t port = 9; // well-known echo port number
UdpEchoServerHelper server(port);
ApplicationContainer apps = server.Install(n.Get(1));
apps.Start(Seconds(1.0));
apps.Stop(Seconds(10.0));
```


Echo 클라이언트 설치도 동일하게 진행된다.
응용 설치 Helper를 생성할때 서버의 IP와 포트번호를 인자로 전달하는 것을 확인할 수 있다.
클라이언트가 서버에게 메시지를 전송하는 간격, packet 용량, packet 갯수 등을 설정한다.
시나리오에 맞게 첫번째 노드인 0번 노드에 응용을 설치하고 응용이 시뮬레이션 상에 언제 시작하고 종료할지를
지정해준다. 서버 시작시간이 1초이고 클라이언서 시작시간이 2초이므로 시뮬레이션 중에 서버가 초기화되지 않아 발생하는 오류를 방지한다.
```c++
//
// Create a UdpEchoClient application to send UDP datagrams from node zero to
// node one.
//
uint32_t packetSize = 1024;
uint32_t maxPacketCount = 1;
Time interPacketInterval = Seconds(1.);
UdpEchoClientHelper client(serverAddress, port);
client.SetAttribute("MaxPackets", UintegerValue(maxPacketCount));
client.SetAttribute("Interval", TimeValue(interPacketInterval));
client.SetAttribute("PacketSize", UintegerValue(packetSize));
apps = client.Install(n.Get(0));
apps.Start(Seconds(2.0));
apps.Stop(Seconds(10.0));
```



메크로로 제외된 부분은 이후에 pcap 파일등을 분석할시 packet의 내용을 원하는 값으로 채우는 부분으로
필수적인 것은 아니다.
```c++
#if 0
//
// Users may find it convenient to initialize echo packets with actual data;
// the below lines suggest how to do this
//
  client.SetFill (apps.Get (0), "Hello World");

  client.SetFill (apps.Get (0), 0xa5, 1024);

  uint8_t fill[] = { 0, 1, 2, 3, 4, 5, 6};
  client.SetFill (apps.Get (0), fill, sizeof(fill), 1024);
#endif
```


테스트 통계 측정 설정
--------------------
NS3는 다양한 방법으로 네트워크 통계를 작성할 수 있게 도와준다.
직접 본인이 원하는 통계를 작성하기 위해 probe 코드를 삽입한 후 NS3가 제공하는 gnuplot tool을 사용할 
수도 있지만, 본 예제에서는 NS3가 제공하는 방법을 사용하고 있다.
```c++
AsciiTraceHelper ascii;
csma.EnableAsciiAll(ascii.CreateFileStream("udp-echo.tr"));
csma.EnablePcapAll("udp-echo", false);
```
보다시피 csma 즉 하드웨어 객체는 자신을 통해 전송되는 packet을 기록하는 기능을 가지고 있다.
위 두줄은 Pcap 파일과 text 파일에 정보를 저장할 것이다.

시뮬레이터 시작
--------------------
드디어 시뮬레이터를 시작한다.
```c++
//
// Now, do the actual simulation.
//
NS_LOG_INFO("Run Simulation.");
Simulator::Run();
Simulator::Destroy();
NS_LOG_INFO("Done.");
```
시뮬레이터가 종료된 후 생성된 Pcap 파일은 wireshark로 읽으면 시뮬레이션을 이해하는데 도움이 될 것이다.
