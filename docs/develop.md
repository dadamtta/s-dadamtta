# develop

프로젝트 폴더 구조는 아래 링크에 정리된 글을 토대로 진행  
[golang-standards/project-layout](https://github.com/golang-standards/project-layout)  
[vmware-tanzu/velero](https://github.com/vmware-tanzu/velero)

---

## Http payload 암호화

웹에서 서버로 통신하는 네트워크 구간에서 **암호**와 같은 개인정보가 평문으로 전송되면 중간에 누군가가 패킷을 가로챈다면 유출될 가능성이 있다.  
공개키 암호 알고리즘인 **RSA**를 활용해서 패킷을 암호화한다.  

> [초기 설계]  
> 데이터를 전송하는 웹에서 RSA 모듈을 통해 `공개키A`와 `개인키A`를 생성한다.  
> (웹 > 서버) 생성한 `공개키A`를 전송한다.  
> (서버) 웹에서 전달받은 '공개키' 정보로 서버는 `공개키B`와 `개인키B`를 생성한다. 이 정보는 저장한다.  
> (서버 > 웹) 서버에서 생성한 `공개키B`를 전송한다.  
> (웹:1) 1. 서버에서 전달받은 `공개키B` 정보로 전송하고자 하는 데이터를 1차 암호화한다.  
> → 이 과정을 통해 데이터는 서버에서만 복호화할 수 있다.  
> → 중간에 패킷을 가로채도 `개인키B`가 없기 때문에 해독할 수 없다.  
> (웹:2) 2. 1차 암호화된 데이터를 `개인키A`로 2차 암호화한다.  
> → 데이터는 `개인키A`를 가진 웹에서만 생성 가능하다.  
> → 해커는 `개인키A`가 없기 때문에 중간에서 데이터를 조작할 수 없다.  
> → 웹의 `공개키A`를 가진 누구나 패킷을 1차 복호화를 할 수 있지만, 2차 복호화는 `개인키B`를 가진 서버만이 가능하다.  
> (웹:3 > 서버) 3. 두 번 암호화한 데이터를 전송한다.  
> (서버:1) 1. 전달받은 데이터를 웹에서 생성한 `공개키A`로 1차 복호화한다.  
> → 복호화가 된다면 약속된 웹에서 전송된 데이터임을 확신할 수 있다.  
> (서버:2) 2. 1차 복호화된 데이터를 `개인키B`로 2차 복호화한다.  
> (서버:3) 3. 저장했던 `개인키B`와 `공개키B`는 폐기한다.

> [2차 설계]  
> (웹 > 서버) 데이터 암호화에 필요한 `공개키` 정보를 요청한다.  
> (서버) 서버는 `공개키`와 `개인키`를 생성한다. `개인키`는 서버에 저장한다.  
> (서버 > 웹) 서버에서 생성한 `공개키`를 전송한다.  
> (웹) 서버에서 전달받은 `공개키` 정보로 전송하고자 하는 데이터를 암호화한다.  
> → 이 과정을 통해 데이터는 서버에서만 복호화할 수 있다.  
> → 중간에 패킷을 가로채도 `개인키`가 없기 때문에 해독할 수 없다.  
> (웹 > 서버) 암호화한 데이터를 전송한다.  
> (서버) 1. 전달받은 암호화된 데이터를 `개인키`로 복호화한다.  
> (서버) 2. 저장했던 `개인키`를 폐기한다.

[Go crypto RSA Key 인코딩/디코딩](https://www.vompressor.com/go-cypto-rsa2/)

---

## API

Http 요청, 응답 바디의 JSON 데이터 키는 **snake_case**로 표현한다.  

커스텀 `errors`로 에러 상황을 모두 대응한다.  
api error response 핸들러를 만들어서 커스텀 `errors`를 처리한다.  
등록되지 않는 errors가 발생하면 Response Code는 메시지가 없는 Http Status Code를 뱉는다.

---

## Logging

Log 라이브러리는 `zap`를 사용한다.  
Log는 나중에 찾아보기 쉽게 작성한다.  

> ** 작성 규칙  
> [Mapping] DTO 마샬, 언마샬  
> [Func] 기능  
> [Token] 토큰

---

## PostgreSQL

RDB는 **PostgreSQL**을 사용한다.  
[Docker PostgreSQL 설치 및 세팅 글(Connecting-팬도라)](https://judo0179.tistory.com/96)

---

## IDE(vscode)

[1] 테스트 함수 내 로그 출력 안되는 현상  
test 커맨드 실행 시 `-v` 옵션을 줘야 로그가 출력된다.  

```text
// 설정 위치: Visual Studio Code → Settings → User Tab → Extensions → Go → Test Flags
"go.testFlags": [ 
    "-v",
]
```

[2] 툴에서 디버깅하는 방법  
VSCode `launch.json` 파일에서 configurations를 수정한다.  

```text
{
    "name": "Launch Package",
    "type": "go",
    "request": "launch",
    "mode": "auto",
    "program": "cmd/dadamtta"
}
```
