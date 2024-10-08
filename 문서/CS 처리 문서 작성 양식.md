# CS 처리 문서 작성 양식

## 개요
- CS 처리 절차를 표준화하여 처리자의 의존도를 낮추고, 누구나 쉽게 문제를 파악하고 해결할 수 있도록 돕는 실무용 작성 양식

## 양식

```
타이틀
문제를 간단하고 명확하게 나타내는 제목.

개요
문제의 전반적인 상황과 발생한 현상에 대한 간략한 설명.

중요도
문제의 심각성과 우선순위 평가 (낮음, 중간, 높음 등).

원인
문제가 발생한 원인에 대한 설명.

확인 방법
문제를 어떻게 확인하고 검증할 수 있는지 설명.

조치 방법
문제 해결을 위한 조치 사항을 설명.

단기 해결 방법(급한 불 끄는 방법)
문제를 일시적으로 해결하는 방법 제시.

근본 해결 방법(원천 제거 방법)
문제의 근본 원인을 제거하여 재발 방지하는 방법 제시.

참조 링크
문제 해결과 관련된 추가 자료나 문서의 링크.

요청자
문제 해결을 요청한 사람의 이름.

요청일자
요청이 들어온 날짜.

처리자
문제를 처리한 담당자의 이름.

처리일자
문제 해결이 완료된 날짜.
```

## 예시

```
타이틀
- 사용자 로그인 오류 발생 (Error Code: 403)

개요
- 사용자가 로그인 시 "403 Forbidden" 오류 메시지가 발생하여 시스템 접근이 불가능한 상황입니다.

중요도
- 높음 (서비스 접근 불가로 인해 사용자의 업무 지연 발생)

원인
- 사용자의 계정 권한 설정 오류 또는 세션 만료로 인한 접근 차단 발생.

확인 방법
- 사용자 계정의 권한 설정을 확인.
- 사용자 세션 로그를 확인하여 만료 여부 확인.
- 시스템 로그를 통해 오류 발생 원인 추적.

조치 방법
- 사용자 계정의 권한을 재설정하여 접근 권한을 복구.
- 세션을 초기화하고, 재로그인 안내.

단기 해결 방법 (급한 불 끄는 방법)
- 사용자 계정 권한을 일시적으로 관리자 권한으로 설정하여 접근 가능하도록 조치.

근본 해결 방법 (원천 제거 방법)
- 권한 설정 자동화 및 주기적인 세션 갱신을 통해 동일 오류 재발 방지.
- 오류 발생 시 자동 알림 및 조치 프로세스 구축.

참조 링크
- 403 오류 해결 가이드
- 사용자 권한 설정 매뉴얼

요청자
- 홍길동

요청일자
- 2024-09-01

처리자
- 김철수

처리일자
- 2024-09-01
```
