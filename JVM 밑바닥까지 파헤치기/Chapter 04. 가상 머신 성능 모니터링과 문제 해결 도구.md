# Chapter 04. 가상 머신 성능 모니터링과 문제 해결 도구

## 4.2 기본적인 문제 해결 도구

> bin 디렉터리의 다양한 도구들의 세 범주
> - 상용 인증 도구: JMC와 JFR이 여기 속한다.
> - 공식 지원 도구
> - 실험적 도구

- 실제로 동작하는 코드는 자바로 구현되어 JDK 도구 라이브러리에 담겨 있다.

### 4.2.1 jps: 가상 머신 프로세스 상태 도구

- 동작 중인 가상 머신 프로세스 목록을 보여주며, 각 프로세스에서 가상 머신이 실행한 메인 클래스(main() 메서드를 포함하는 클래스)의 이름과 로컬 가상 머신 식별자를 알려 준다.
- 로컬 가상 머신 프로세스의 LVMID 값은 운영 체제의 프로세스 아이디(PID)와 같다.

> 옵션
> 1. -q: LVMID만 출력
> 2. -m: 가상 머신 프로세스 시작 시 main() 메서드에 전달된 매개 변수 출력
> 3. -l: 메인 클래스의 완전한 이름 출력
> 4. -v: 가상 머신 프로세스 시작 시 주어진 가상 머신 매개 변수 출력

### 4.2.2 jstat: 가상 머신 통계 정보 모니터링 도구

- 가상 머신의 다양한 작동 상태 정보를 모니터링하는 데 사용한다.
- 로컬 또는 원격 가상 머신 프로세스의 클래스 로딩, 메모리, 가비지 컬렉션, JIT 컴파일과 같은 런타임 데이터를 보여 준다.

> 옵션
> 1. -class
> 2. -gc
> 3. -gccapacity
> 4. -gcutil
> 5. -gccause
> 6. -gcnew
> 7. -gcnewcapacity
> 8. -gcold
> 9. -gcoldcapacity
> 10. -compiler
> 11. -printcompilation

### 4.2.3 jinfo: 자바 설정 정보 도구

- 가상 머신의 다양한 매개 변수를 실시간으로 확인하고 변경하는 도구다.

> ex
> jinfo -flag ConcGCThreads 1444
> 결과: -XX:ConcGCThreads=3

### 4.2.4 jmap: 자바 메모리 매핑 도구

- 힙 스냅숏을 파일로 덤프해 주는 자바용 메모리 맵 명령어다.

> 옵션
> 1. -dump
> 2. -finalizerinfo
> 3. -heap
> 4. -histo

### 4.2.5 jhat: 가상 머신 힙 덤프 스냅숏 분석 도구

- JDK 8까지 제공되었었다.
- jmap으로 덤프한 힙 스냅숏을 분석할 수 있다.
- 작은 HTTP-웹 서버를 내장하고 있어서 분석이 완료되면 웹 브라우저로 결과를 살펴볼 수 있다.

### 4.2.6 jstack: 자바 스택 추적 도구

- 현재 가상 머신의 스레드 스냅숏을 생성하는 데 쓰인다.
- 스레드가 장시간 멈춰 있을 때 원인을 찾기 위해 생성한다.

> 옵션
> 1. -l
> 2. -e

- JDK 5부터는 java.lang.Thread 클래스에 추가된 getAllStackTraces() 메서드로 가상 머신이 실행하는 모든 스레드의 StackTraceElement 객체를 얻을 수 있다. 이 메서드를 이용하면 코드 단 몇 줄로 jstack의 기능 대부분을 구현할 수 있다.

### 4.2.7 기본 도구 요약

> 기본 도구
> 1. appletviewer
> 2. extcheck
> 3. jar
> 4. java
> 5. javac
> 6. javadoc
> 7. javah
> 8. javap
> 9. jlink
> 10. jdb
> 11. jdeps
> 12. jdeprscan
> 13. jwebserver

> 보안 도구
> 1. keytool
> 2. jarsigner
> 3. policytool

> 국제화 도구
> 1. native2ascii

> 원격 메서드 호출 도구
> 1. rmic
> 2. rmiregistry
> 3. rmid
> 4. serialver

> 자바 IDL과 RMI-IIOP
> 1. tnameserv
> 2. idlj
> 3. orbd
> 4. servertool

> 배포 도구
> 1. javapackager
> 2. jpackage
> 3. pack200
> 4. unpack200

> 자바 웹 스타트
> 1. javaws

> JVM 성능 모니터링 도구 및 문제 해결 도구
> 1. jps
> 2. jstat
> 3. jstatd
> 4. jinfo
> 5. jmap
> 6. jfr
> 7. jhat
> 8. jstack
> 9. jhsdb
> 10. jsadebugd
> 11. jcmd
> 12. jconsole
> 13. jmc
> 14. jvisualvm

> 웹 서비스 도구
> 1. schemagen
> 2. wsgen
> 3. wsimport
> 4. xjc

> REPL과 스크립트 도구
> 1. jshell
> 2. jjs
> 3. jrunscript
