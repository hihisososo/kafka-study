# Chapter 10. 스키마 레지스트리

## 스키마의 개념과 유용성
* 카프카에서 스키마는 RDB의 스키마와 같이 정보를 구성하고 해석하기 위한 개념이다.
* 카프카는 수많은 애플리케이션이 접근하는데, 스키마는 각 정의된 형태의 데이터를 사용할 수 있도록하여 타 시스템 간에 호환성 이슈를 방지한다.
* 스키마를 통해 타 부서간의 커뮤니케이션 비용을 줄일 수 있다.

## 카프카와 스키마 레지스트리
* 스키마를 카프카에서 활용하면 관리자는 데이터 처리 시 유연성을 확보할 수 있으며 불필요한 커뮤니케이션 비용을 줄일 수 있다.

### 스키마 레지스트리 개요
* 카프카에서 스키마를 활용하는 방법은 스키마 레지스트리라는 별도의 애플리케이션을 이용하는 것이다.
  * 스키마 레지스트리는 스키마를 등록하고 관리하는 애플리케이션이다.
* 스키마 레지스트리는 카프카와 별도로 구성된 독립적인 애플리케이션이며, 카프카로 전송하는 프로듀서와 직접 통신하며, 카프카로부터 메시지를 컨슈밍하는 컨슈머와도 직접 통신한다.
* 클라이언트들이 스키마 정보를 사용하기 위해서는 프로듀서와 컨슈머, 스키마 레지스트리 간에 직접 통신이 이뤄져야 한다.
  * 프로듀서는 스키마 레지스트리에 스키마를 등록하고, 스키마 레지스트리는 프로듀서에 의해 등록된 스키마 정보를 카프카의 내부 토픽에 저장한다.
  * 프로듀서는 스키마 레지스트리에 등록된 스키마의 ID와 메시지를 카프카로 전송하고, 컨슈머는 스키마 ID를 스키마 레지스트리로부터 읽어온 후 프로듀서가 전송한 스키마 ID와 메시지를 조합해 읽을 수 있다.
* 스키마 레지스트리가 지원하는 데이터 포맷을 사용해야 하는데, 가장 대표적인 포맷은 "에이브로"다.

### 스키마 레지스트리의 에이브로 지원
* 에이브로는 시스템, 프로그래밍 언어, 프로세싱 프레임워크 사이에서 데이터 교환을 도와주는 오픈소스 직렬화 시스템이다.
* 빠른 바이너리 포맷을 지원하며 JSON 형태의 스키마를 정의할 수 있는 간결한 데이터 포맷이다.
* 에이브로는 바이너리 형태이므로 매우 빠르다.

## 스키마 레지스트리 호환성
* 스키마 레지스트리는 스키마에 대해 고유한 ID와 버전 정보를 관리한다.
* 스키마가 진화함에 따라 호환성 레벨을 검사해야 하는데, BACKWARD, FORWARD, FULL 등의 호환성 레벨이 있다.

### BACKWARD 호환성
* 진화된 스키마를 적용한 컨슈머가 진화 전의 스키마가 적용된 프로듀서가 보낸 메시지를 읽을 수 있도록 허용하는 호환성을 말한다.
* 스키마의 버전 업데이트가 필요하다면 프로듀서와 컨슈머의 스키마도 업데이트해줘야 하는데, BACKWARD 호환성에서는 먼저 상위 버전의 스키마를 컨슈머에게 적용하고 난 뒤에 프로듀서에게 상위 버전의 스키마를 적용해야 한다.
* 모든 하위 버전의 스키마를 호환하고자 한다면 BACKWARD_TRANSITIVE로 설정해야 한다.

### FORWARD 호환성
* 진화된 스키마가 적용된 프로듀서가 보낸 메시지를 진화 전의 스키마가 적용된 컨슈머가 읽을 수 있게 하는 호환성이다.
* 스키마 진화가 일어나는 경우 상위 버전의 스키마를 프로듀서에 먼저 적용한 다음, 컨슈머에 적용한다.
* 모든 버전의 스키마를 호환하고자 한다면 FORWARD_TRANSITIVE로 설정해야 한다.

### FULL 호환성
* BACKWARD와 FORWARD 호환성 모두를 지원한다.
* 스키마가 진화함에 따라 프로듀서와 컨슈머 양쪽에서 모두 호환이 되므로 제약 없이 더 편리하게 사용할 수 있다.
* 모든 버전의 스키마를 지원하고자 한다면 FULL_TRANSITIVE로 설정해야 한다.