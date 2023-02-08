# 🛒 Ngether
>실시간 위치기반 공동구매 플랫폼  
>
>유저는 원하는 지역을 지정하여 공동구매를 희망하는 사람들을 모집할 수 있습니다.  
>모집글을 생성함과 동시에 채팅방이 개설되며, 유저들은 공동구매와 관련한 대화를 나눌 수 있습니다.  
>Geolocation API와 Kakao Map API를 통해 지도 및 게시판에 모집글을 렌더링합니다.   
>별도의 검색기능으로 위치, 제목, 내용, 작성자 등으로 검색할 수 있습니다.  
>
>[Ngether 서비스](https:/ngether.xyz)
>[레포지토리](http://github.com/codestates-seb/seb41_main_024/issues?page=3&q=assignee%3ASHWsgithub+is%3Aclosed)

</br>

## 1. 제작 기간 & 참여 인원
- 제작 기간: 23.1.3 ~ 2023.1.31
- 팀 구성: FE - 4명, BE - 3명
- 담당 기능
  -  실시간 채팅 기능 (WebSocket)
  -  관리자 페이지 UI 및 기능 구현
- 추가 구현
  -  엑세스 토큰 유효성 관리를 위한 함수화
  -  회원가입 폼 제작
  -  회원가입 페이지 이메일 인증

</br>

## 2. 사용 스택
- Next.js 13.1
- TypeScript 4.9
- WebSocket(Sockjs, stompJs)
- tanstack/react-query 4.22
- MUI
- tailwind CSS
- js-cookie

</br>

## 3. 컴포넌트
- 재사용성을 위해 [아토믹 디자인](https://fe-developers.kakaoent.com/2022/220505-how-page-part-use-atomic-design-system/) 패턴 적용을 시도했습니다.
- 재사용될 컴포넌트를 파악하기 위해 피그마에 UI에 대한 설계를 미리 그렸습니다. → [피그마 링크](https://www.figma.com/file/c5ndHFggdYDwI79WPIBJQp/Ngether?node-id=0%3A1&t=dZwcfnnKq3tK5ELi-1) 
- `atoms`, `morcules`, `organisms` 폴더에 각 컴포넌트를 위치시켰습니다.

</br>

## 4. hooks & utility function
### 1. useForm
- 회원가입 또는 회원 정보 수정 폼 등에 사용할 수 있습니다.
- useForm 내부에서 useState로 인풋의 상태를 관리합니다.
- 보일러 플레이트를 줄일 수 있습니다.
### 2. useRegexText
- Regex test가 필요한 곳에서 사용할 수 있습니다.
- Regex 정규표현식을 선언해놓고 useRegexText에 각 상태에 맞는 문구를 설정할 수 있습니다.
- 인풋의 라벨을 조건부 선언 필요없이 관리할 수 있습니다.
### 3. transDateFormat
- localeDate를 통해 생성된 날짜 문자열을 형식에 맞게 변환시켜줍니다.

</br>

## 4. 핵심 구현 기능
>실시간 채팅은 유저가 모집글을 작성하거나, 모집글에서 참여할 때 입장됩니다.  
>채팅방 내부에서 모집글 완료, 강퇴, 신고 등의 처리를 할 수 있습니다.  
>채팅방 내부에서 볼 수 있는 UI들은 각 이벤트에 맞춰 최신화 되기 때문에 퇴장, 강퇴, 신고 등에 따라 실시간으로 업데이트 됩니다.

<details>
<summary><b>실시간 채팅 핵심 기능</b></summary>
<div markdown="1">

## 1. 전체 흐름
![image](https://user-images.githubusercontent.com/111102006/216899026-b85e5c99-144e-4a66-a4e6-740c37684d92.png)

## 2. 채팅방 입장 → [코드](https://github.com/codestates-seb/seb41_main_024/blob/main/client/hooks/useWebSocketClient.ts)
>채팅에 관련한 대화내용, 멤버 등의 상태를 하나의 hook으로 만들어 관리했습니다.  
>해당 훅은 useEffect를 통해 채팅페이지에 접근시 실행됩니다.  
>채팅방 인원이 아니라면 잘못된 접근이라는 문구를 보여주거나 로딩 써클을 보여줍니다.
  
### 2.1 채팅 참여자 여부 확인
![image](https://user-images.githubusercontent.com/111102006/216907374-b876fe42-aab0-4782-ab5a-9e20057690d8.png)
- 채팅 페이지로 이동하게 되면 채팅 참여자 중 이동한 유저가 있는지 확인합니다.
- 존재하지 않는다면 해당 유저는 채팅방에 접근할 수 없습니다.

### 2.2 채팅 기록 최신화
![image](https://user-images.githubusercontent.com/111102006/216906912-3d431057-7697-4f0f-aa6e-ec6b2a0c1b81.png)
- 서버에 채팅방 입장 요청을 보내고, 채팅방 내역을 요청해 채팅 기록의 상태를 최신으로 갱신합니다.

### 2.3 웹소켓 pub-sub
![image](https://user-images.githubusercontent.com/111102006/216905908-cf0d1aed-029f-4b13-8a40-e15a0e2c6828.png)
- 유저가 입장할 경우 웹소켓 클라이언트를 생성합니다.
  
  - 웹소켓 클라이언트를 통해 서버의 웹소켓에 연결하고 pub-sub을 맺습니다.
  - 구독 후 전송되는 메세지의 타입에 따라 상태를 최신으로 유지하기 위한 요청을 합니다.
  
</div>
</details>

</br>

## 5. 트러블 슈팅
<details>
<summary>MUI에서 테일윈드 css가 제대로 작동하지 않던 문제</summary>
<div markdown="1">
  
  - MUI의 컴포넌트를 주어진 property나 sx가 아닌 더 효율성있게 수정하길 원했습니다.  
>        - MUI 컴포넌트에 className으로 테일윈드의 방식을 적용했지만 일부 css들이 작동하지 않아 지체됐습니다.  
>        - 이는 별도의 config 설정(preflight) 및 MUI의 StyledEngineProvider를 통해 해결했습니다.    
        - https://github.com/vercel/next.js/discussions/32565  
        - https://levelup.gitconnected.com/using-material-ui-with-next-js-13-and-tailwind-css-41c201855dcf  
        
</div>
</details>


<details>
<summary>Client에서 채팅 페이지를 나갈시 서버의 세션ID가 null로 치환되지 않던 문제</summary>
<div markdown="1">
  
  - Websocket의 subscribe를 끊어주는 코드가 필요했습니다.  
>        - 세션ID가 null로 변해야 전송된 채팅을 읽은 유저와 안 읽은 유저로 나눠 읽음처리를 할 수 있었습니다.  
>        - useEffet에 return으로 익명함수를 넣어 웹소켓 클라이언트를 disconnect 시켜 해결했습니다.
</div>
</details>
<details>
<summary>채팅 송수신 시 UI</summary>
<div markdown="1">
  
  - 새로운 메세지로 상태가 변경될 때 채팅 풍선 element가 생성은 되지만 직접 스크롤을 내려야 했습니다.  
>        - 채팅 아래에 별도의 div를 만들어 useRef로 참조했습니다.  
>        - 메세지 상태가 변경될 때마다 scrollIntoView 메소드로 가장 아래로 이동하게 해 더 나은 유저 경험을 만들었습니다.        
</div>
</details> 
<details>
<summary>채팅 알람 구현</summary>
<div markdown="1">
  
  - 채팅 알람을 구현하기 위해 고안한 첫 번째 방법은 롱 폴링이었습니다.  
    (로그인 시 롱 폴링 재귀함수 실행, 서버에서 connection을 5초간 유지 -> 안읽은 메세지 없을 시 time out 에러, 있을 시 true 리턴)  
>        - 처음엔 롱폴링을 _app.tsx에서 호출하여 Context로 메세지 탭을 담은 컴포넌트로 보냈습니다.  
>        - 롱폴링을 통한 알람은 성공적이었으나 서버의 db connection에 부하가 왔고, 팀원들과 회의한 결과 db connection을 늘리는 것은 근본적인 해결이 아니라고 결정했습니다.  
>        - 따라서 매 페이지 이동 시 롱폴링에서 보냈던 요청을 실행하고 서버에서는 connection을 없앤 후 즉시 리턴을 받는 방식으로 전환했습니다.        
</div>
</details> 


## 회고 / 느낀점
>프로젝트 개발 회고 글: 
