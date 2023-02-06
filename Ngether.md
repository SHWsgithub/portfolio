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
  -  채팅 알람
  -  관리자 페이지 UI 및 기능 구현
- 추가 구현
  -  엑세스 토큰 유효성 관리를 위한 함수화
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

## 4. 구현 기능
>프로젝트에서 실시간 채팅과 관리자 페이지 구현을 담당했습니다.  
>
>실시간 채팅은 유저가 모집글을 작성하거나, 모집글에서 참여할 때 입장됩니다.  
>채팅방 내부에서 모집글 완료, 강퇴, 신고 등의 처리를 할 수 있습니다.  
>채팅방 내부에서 볼 수 있는 UI들은 각 이벤트에 맞춰 최신화 되기 때문에 퇴장, 강퇴, 신고 등에 따라 실시간으로 업데이트 됩니다.
>
>관리자는 접수된 신고, 1:1 문의 등을 관리자 페이지에서 처리할 수 있습니다.  
>신고 열람시 한 페이지 내에서 유저 정지처리, 신고 반려등을 할 수 있습니다.

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
<details>
<summary><b>관리자 페이지 기능</b></summary>
<div markdown="1">

## 1. 전체 흐름

  
</div>
</details>



</br>

## 회고 / 느낀점
>프로젝트 개발 회고 글: https://zuminternet.github.io/ZUM-Pilot-integer/
