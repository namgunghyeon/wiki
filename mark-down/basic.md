# Basic

## OSI 7계층
OSI 모형(Open System Interconnection Refrence Model)은 국제표준화기구(ISO)에서 개발한 모델로, 네트워크 프로토콜 디자인과 통신을 계층으로 나누어 설명한 것이다.
### 1: 물리 계층
물리 계층은 네트워크의 기본 네트워크 하드웨어 전송 기술을 이룬다.
전압, 허브, 네트워크 어댑터, 리피터
### 2: 데이터 링크 계층
데이터 링크 계층은 포인트 투 포인트간 신뢰성있는 전송을 보장하기 위한 계층으로 CRC 기반의 오류 제어와 흐름 제어가 필요하다.
물리적인 네트워크 사이의 데이터 전송을 담당한다. 브리지와 스위치
### 3: 네트워크 계층
네트워크 계층은 여러개의 노드를 거칠때마다 경로를 찾아주는 역할을 하는 계층으로 다양한 길이의 데이터를 네트워크들을 통해 전달하고, 그 과정에서 전송 계층이 요구하는 서비스 품질을 제공하기 위한 기능적, 절차적 수단을 제공한다.
라우터
### 4: 전송 계층
전송 계층은 양 끝단의 사용자들이 신뢰성있는 데이터를 주고 받을 수 있도록 해주어, 상위 계층들이 데이터 전달의 유효성이나 효율성을 생각하지 않도록 해준다. 시퀀스 넘버 기반의 오류 제어 방식을 사용한다.
### 5: 세션 계층
세션 계층은 양 끝단의 프로세스가 통신을 관리하기 위한 방법을 제공한다. 동시 송수신 방식, 반이중 방식, 전이중 방식의 통신과 함께, 체크 포인팅과 유휴, 종료, 다시 시작 과정 등을 수행한다. 이 계층은 TCP/IP 세션을 만들고 없애는 책임을 진다.
***반이중: 한쪽이 송신하고 있는 동안 수신하고, 전송 방향을 전환하는 것***
***전이중: 쌍방이 동시에 송수신할 수 있는 것***
### 6: 표현 계층
표현 계층은 코드 간의 번역을 담당하여 사용자 시스템에서 데이터의 형식상 차이를 다루는 부담을 응용 계층으로부터 덜어준다. MIME 인코딩이나 암호화 등의 동작이 이 계층에서 이루어진다.
### 7: 응용 계층
응용 계층은 응용 프로세스와 직접 관계하여 일반적인 응용 서비스를 수행한다.

## 각 계층별 사용되는 일반적인 프로토콜
### 응용
HTTP, SMTP, FTP
### 표현
ASCII, MPEG, JPEG
### 세션
NetBIOS, SAP, SDP, NWLink
### 전송
TCP, UDP, SPX
### 네트워크
IP, IPX
### 데이터 링크
Ethernet, Token Ring, FDDI
### 물리
물리적인 장치

**출처:
https://ko.wikipedia.org/wiki/OSI_%EB%AA%A8%ED%98%95
http://blog.naver.com/PostView.nhn?blogId=demonicws&logNo=40117378644
http://zetawiki.com/wiki/%EC%96%91%EB%B0%A9%ED%96%A5%ED%86%B5%EC%8B%A0,_%EC%A0%84%EC%9D%B4%EC%A4%91%EB%B0%A9%EC%8B%9D,_%EB%B0%98%EC%9D%B4%EC%A4%91%EB%B0%A9%EC%8B%9D,_%EB%8B%A8%EB%B0%A9%ED%96%A5%ED%86%B5%EC%8B%A0***


## Process
현재 실행중인 프로그램

### 프로세스 상태
생성: 프로게스가 생성되었지만, 실행가능한 프로세스 집합에 소속되지 않은 상태(메모리에 적재되지 않은 상태)
준비: CPU를 할당 받기 위해 준비중인 상태
실행: CPU를 할당 받아 명령어를 실행중인 상태
대기: 어떤 사건이 발생하기를 기다리고 있는 상태
종료: 프로세스가 실행 종료된 상태(메모리에서 해제된 상태)

### 프로세스 컨드롤 블록(PCB)
PCB는 프로세스와 관련된 정보를 갖는다.

1. Process State
- Ready, Running, Waiting, Halted
2. Program Counter
- 다음에 실행될 명령어 주소
3. CPU Registers
- 여러 범용 목적의 레지스터들
4. CPU Scheduling Information
- 프로세스의 우선 순위 정보, 스케쥴링 큐
5. Memory Management Information
- 페이지 테이블
6. Accounting Information
- CPU 사용량, 프로세스의 실제 실행시간 ...
7. I/O 상태 정보
- 해당 프로세스에 할당된 디바이스 목록, Open된 파일의 목록 등등
***출처:
http://apphappy.tistory.com/115***

## Thread
스레드는 어떠한 프로그램 내에서, 특히 프로세스 내에서 시행되는 흐름이 단위, 일반적으로 한 프로그램은 하나의 스레드를 가지고 있지만, 프로그램 환경에 따라 둘 이상의 스레드를 동시에 실행 할 수 있다. 이러한 실행 방식을 멀티스레드라고 한다.

프로세스는 독립적으로 실행되며 각각 별개의 메모리를 차지하고 있고, 스레드는 프로세스 내의 메모리를 공유해 사용할 수 있다.

### 스레드의 종류
#### 사용자 레벨 스레드(User-Level Thread)
커널 영역 상위에서 지원되며 일반적으로 사용자 레벨의 라이브러리르 통해 구현, 라이브러리에서 스레드의 생성 및 스케줄링 등에 관한 관리 기능을 제공한다. 동일한 메모리 영역에서 스레드가 생성 및 관리되어 속도가 빠르다.
#### 커널 레벨 스레드(Kernel-Level Thread)
운영체제가 지원하는 스레드 기능으로 구현되며, 커널이 스레드의 생성 및 스케줄링 등을 관리한다.

***출처:
https://ko.wikipedia.org/wiki/%EC%8A%A4%EB%A0%88%EB%93%9C***
