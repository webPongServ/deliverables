# API 설계

[TOC]



### Figma

https://www.figma.com/file/6Sid9eXwWlhJDzxtpyucAS/Untitled?node-id=0-1&t=zlpPaCfVpKiEbTiy-0



### 화면 구성

| 1차메뉴 | 2차메뉴            | 구분       | 화면 ID | comment                                 |
| ------- | ------------------ | ---------- | ------- | --------------------------------------- |
| 로그인  | 42seoul 로그인     |            | LG001   | 닉네임 초기 : intraid (내정보수정 가능) |
|         | 2차 인증           | 팝업       | LG002   |                                         |
|         |                    |            |         |                                         |
| 대기실  | 메인화면           |            | MN001   |                                         |
|         | 친구목록           | 네비게이션 | MN002   |                                         |
|         | 유저 프로필 - 전적 | 팝업       | MN003   |                                         |
|         | 유저 프로필 - 업적 | 팝업       | MN004   |                                         |
|         | 유저 검색          | 팝업       | MN005   |                                         |
|         | 로그아웃           | 팝업       | MN006   |                                         |
|         |                    |            |         |                                         |
| 프로필  | 내 프로필 - 전적   |            | PF001   |                                         |
|         | 내 프로필 - 업적   |            | PF002   |                                         |
|         | 프로필 수정        | 팝업       | PF003   |                                         |
|         | 2차 인증 설정      | 팝업       | PF004   |                                         |
|         |                    |            |         |                                         |
| 채팅    | 채팅목록           |            | CH001   |                                         |
|         | 채팅방 생성        | 팝업       | CH002   |                                         |
|         | 채팅방             | (팝업)     | CH003   |                                         |
|         | 유저 프로필 - 전적 | 팝업       | CH004   |                                         |
|         | 유저 프로필 - 업적 | 팝업       | CH005   |                                         |
|         | 채팅방 정보수정    | 팝업       | CH006   |                                         |
|         | 유저목록           | 팝업       | CH007   |                                         |
|         | 차단목록           | 팝업       | CH008   |                                         |
|         |                    |            |         |                                         |
| 게임    | 게임목록           |            | GM001   |                                         |
|         | 일반 게임 입장     | 팝업       | GM002   |                                         |
|         | 일반 게임 생성     | 팝업       | GM003   |                                         |
|         | 일반 게임 대기     | (팝업)     | GM004   |                                         |
|         | 일반 게임 수정     | 팝업       | GM005   |                                         |
|         | 레더 게임 시작     | 팝업       | GM006   |                                         |
|         | 게임               |            | GM007   |                                         |
|         | 게임결과           |            | GM008   |                                         |
|         |                    |            |         |                                         |



# 로그인



## 42seoul 로그인(LG001)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-LG001.png" style="zoom:150%;" />



| No   | request - HTTP method, Endpoint | request - parameter | description                           | response - status code | response - data                                          | Table-Column-Variable          |
| ---- | ------------------------------- | ------------------- | ------------------------------------- | ---------------------- | -------------------------------------------------------- | ------------------------------ |
| 1    | POST /users/login/42oauth       |                     | 42 인트라 로그인 페이지(/main)로 이동 | 201                    | login_case, session_id, (is_twofactor, is_already_login) | TB_UA01M-, TB_UA01L-SESSION_ID |

- request - parameters의 내용은 42OAuth 로그인 페이지를 프론트에서 바로 보낼지 아니면 백엔드를 거쳐서 redirection을 통해서 갈지 여부로 결정됨. 전자의 경우 code 항목이 추가되고 후자의 경우는 그대로 없이 두면 됨.
- 목표: (jwt), session_id를 받아서 MN001로 넘어가야함. 즉, MN001로 넘어가기 직전에 jwt, session_id를 받도록 만들면 됨. 여기서는 MN001로 가는 경우인 case 1에 대해서만 jwt, session_id 값이 넘어감.
- TB_UA01L 에 '로그인 진행중'과 Case별 단계를 추적하는 column을 추가해야할 듯
  - 혹은 메모리 상으로 로그인 진행 중인 session_id를 key로 이용해 잠시동안 관리해야할 것 같다. (stateful)


- 즉, 어느 세션이든 42OAuth를 통해 로그인 시도를 한다면 session_id를 이용해서 stateful하게 관리해야함.



- 2차 인증, 기존 로그인 세션 유무 Case 분류표

|        | 2차 인증 | 기존 로그인 세션 (이미 로그인) | 페이지 라우팅 순서                                |
| ------ | -------- | ------------------------------ | ------------------------------------------------- |
| Case 1 | X        | X                              | MN001                                             |
| Case 2 | X        | O                              | LG003(기존 로그인 세션 종료 유무 선택창) -> MN001 |
| Case 3 | O        | X                              | LG002 -> MN001                                    |
| Case 4 | O        | O                              | LG002 -> LG003 -> MN001                           |



## 2차 인증(LG002)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-LG002.png" style="zoom:150%;" />



| No   | request - HTTP method, Endpoint | request - parameter | description                           | response - status code | response - data | Table-Column-Variable |
| ---- | ------------------------------- | ------------------- | ------------------------------------- | ---------------------- | --------------- | --------------------- |
| 1    | POST /users/login/twofactor     | two factor_pw       | 42 인트라 로그인 페이지(/main)로 이동 | 201                    | case            |                       |

- response - data에서 case에 대한 정보를 넘겨준 이유: 백엔드에서 case를 넘겨주지 않으려면 is_twofactor와 is_already_login 상태 정보를 프론트에서 계속 들고 있어야 한다. 하지만 페이지가 새로고침될 경우, local storage 등에 저장하지 않는 이상 휘발된다. 그러므로 페이지가 새로고침되어도 2차 인증 



#### # 3차 인증(LG003)





# 대기실

## 메인화면(MN001), 친구목록(MN002)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-MN001.png" style="zoom:150%;" />

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-MN002.png" style="zoom:150%;" />



| No   | request - HTTP method, Endpoint | request - parameter | description              | response - status code | response - data                    | Table-Column-Variable |
| ---- | ------------------------------- | ------------------- | ------------------------ | ---------------------- | ---------------------------------- | --------------------- |
| 1    | GET /users/friends-list         |                     | 친구목록 리스트 요청     | 200                    | user_name, ...                     | TB_UA02L              |
| 2    | GET /games/rooms-list           |                     | 게임방(일반) 리스트 요청 | 200                    | {gameroom_name, owner}, ...        | TB_GM01L              |
| 3    | GET /chats/rooms-list           |                     | 채팅방 리스트 요청       | 200                    | {chatroom_name, owner, count}, ... | TB_CH01L              |
|      |                                 |                     |                          |                        |                                    |                       |

- 5초마다 api 새로고침(네이버 스포츠 댓글 새로고침)

- Example
  - {chanhyle, susong, seongtki, mgo, ...}






## 유저 프로필 - 전적(MN003), 업적(MN004)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-MN003.png" style="zoom:150%;" />



| No   | request - HTTP method, Endpoint | request - parameter | description      | response - status code | response - data                              | Table-Column-Variable |
| ---- | ------------------------------- | ------------------- | ---------------- | ---------------------- | -------------------------------------------- | --------------------- |
| 1    | GET /users/profile/record-count | user_id             | 전적, 래더 점수  | 200                    | win: number, lose: number, ladder: number    | TB_GM02S              |
| 2    | GET /users/profile/records      | user_id             | 게임 전적 리스트 | 200                    | {victory: boolean, opposite: string}, ...    | TB_GM01D              |
| 3    | GET /users/profile/achievements | user_id             | 업적 리스트      | 200                    | {achiv_name: string, achiv_cmt: string}, ... | TB_UA03M              |
| 4    | POST /chats/dm                  | user_id             | DM 요청          | 201                    |                                              |                       |
| 5    | POST /users/add-friend          | user_id             | 친구 추가        | 201                    |                                              |                       |
| 6    | POST /users/block               | user_id             | 차단             | 201                    |                                              |                       |

- 위에 1~3번 API는 아래의 프로필 영역의 API와 중복되기 때문에 여기서는 삭제할 예정 (유저간의 상호작용 API만 여기에 명세)



## 유저 검색(MN005)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-MN005.png" style="zoom:150%;" />



| No   | request - HTTP method, Endpoint | request - parameter | description  | response - status code | response - data            | Table-Column-Variable |
| ---- | ------------------------------- | ------------------- | ------------ | ---------------------- | -------------------------- | --------------------- |
| 1    | GET /users/search               | Input               | user id 검색 | 200                    | {user1, user2, user3, ...} |                       |





## 로그아웃(MN006)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-MN006.png" style="zoom:150%;" />



| No   | request - HTTP method, Endpoint | request - parameter | description | response - status code | response - data | Table-Column-Variable |
| ---- | ------------------------------- | ------------------- | ----------- | ---------------------- | --------------- | --------------------- |
| 1    | PUT /users/logout               |                     | 로그아웃    | 201                    |                 |                       |





# 프로필

## 내 프로필 - 전적(PF001), 업적(PF002)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-PF001.png" style="zoom:150%;" />

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-PF002.png" style="zoom:150%;" />



| No   | request - HTTP method, Endpoint | request - parameter | description      | response - status code | response - data                              | Table-Column-Variable |
| ---- | ------------------------------- | ------------------- | ---------------- | ---------------------- | -------------------------------------------- | --------------------- |
| 1    | GET /users/profile/record-count | user_id             | 전적, 래더 점수  | 200                    | win: number, lose: number, ladder: number    | TB_GM02S              |
| 2    | GET /users/profile/records      | user_id             | 게임 전적 리스트 | 200                    | {victory: boolean, opposite: string}, ...    | TB_GM01D              |
| 3    | GET /users/profile/achievements | user_id             | 업적 리스트      | 200                    | {achiv_name: string, achiv_cmt: string}, ... | TB_UA03M              |





## 프로필 수정(PF003)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-PF003.png" style="zoom:150%;" />



| No   | request - HTTP method, Endpoint | request - parameter     | description             | response - status code              | response - data | Table-Column-Variable |
| ---- | ------------------------------- | ----------------------- | ----------------------- | ----------------------------------- | --------------- | --------------------- |
| 1    | POST /users/profile/updating    | nickname, profile_image | 프로필 수정 데이터 전송 | 201, 403(닉네임 중복, 이미지 허용x) |                 |                       |

- websocket으로 닉네임 중복 검증 할 수 있겠지만 아직 반영하지 않음.



## 2차 인증 설정(PF004)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-PF004.png" style="zoom:150%;" />



| No   | request - HTTP method, Endpoint | request - parameter | description   | response - status code | response - data | Table-Column-Variable |
| ---- | ------------------------------- | ------------------- | ------------- | ---------------------- | --------------- | --------------------- |
| 1    | PUT /users/auth/twofactor       |                     | 2차 인증 설정 | 201                    |                 |                       |

- 2차 인증 설정 부분은 아직 구체적으로 나오지 않아서 간략히 작성함



# 채팅

## 채팅목록(CH001)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-CH001.png" style="zoom:150%;" />

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-CH001-1.png" style="zoom:150%;" />



| No   | request - HTTP method, Endpoint | request - parameter | description                                       | response - status code | response - data                                              | Table-Column-Variable |
| ---- | ------------------------------- | ------------------- | ------------------------------------------------- | ---------------------- | ------------------------------------------------------------ | --------------------- |
| 1    | GET /chats/rooms-list           |                     | 채팅방 목록                                       | 200                    | {chatroom_id: string, chatroom_name: string, owner: string, type: string, curr_user_count: number, max_user_count: number}, ... |                       |
| 2    | POST /chats/entrance            | password: string    | 채팅방 입장 (public일 경우에 password 검증 안 함) | 201, 403(입장 불가)    |                                                              |                       |

- 채팅방 입장 승인 후에 채팅방 안의 데이터들은 websocket을 이용



## 채팅방 생성(CH002)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-CH002.png" style="zoom:150%;" />



| No   | request - HTTP method, Endpoint | request - parameter                                          | description | response - status code | response - data     | Table-Column-Variable |
| ---- | ------------------------------- | ------------------------------------------------------------ | ----------- | ---------------------- | ------------------- | --------------------- |
| 1    | POST /chats/creating-room       | chatroom_name: string, chatroom_type: string, password: string | 채팅방 생성 | 201, 403(형식 제한)    | chatroom_id: string |                       |

- 채팅방 입장 후에 채팅방 안의 데이터들은 websocket을 이용 (유저 목록, 유저 권한, 메시지 등)
- 채팅방 제목은 최대 20자로 제한한다.
- 채팅방 비밀번호는 최대 10자로 제한한다.



## 채팅방 상세(CH003)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-CH003.png" style="zoom:150%;" />



| No   | request - HTTP method, Endpoint | request - parameter | description | response - status code | response - data | Table-Column-Variable |
| ---- | ------------------------------- | ------------------- | ----------- | ---------------------- | --------------- | --------------------- |
|      |                                 |                     |             |                        |                 |                       |

- 채팅방 입장 후에 채팅방 안의 데이터들은 websocket을 이용



## 채팅방 정보 수정(CH006)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-CH006.png" style="zoom:150%;" />



| No   | request - HTTP method, Endpoint | request - parameter                                          | description      | response - status code | response - data | Table-Column-Variable |
| ---- | ------------------------------- | ------------------------------------------------------------ | ---------------- | ---------------------- | --------------- | --------------------- |
| 1    | PUT /chats/changing-room        | chatroom_id: number, chatroom_name: string, chatroom_type: string, password: string | 채팅방 정보 수정 | 201, 403(권한 없음)    |                 |                       |



## 채팅방 유저 목록(CH007)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-CH007.png" style="zoom:150%;" />

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-CH007-1.png" style="zoom:150%;" />

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-CH007-2.png" style="zoom:150%;" />



| No   | request - HTTP method, Endpoint | request - parameter                             | description      | response - status code | response - data                           | Table-Column-Variable |
| ---- | ------------------------------- | ----------------------------------------------- | ---------------- | ---------------------- | ----------------------------------------- | --------------------- |
| 1    | GET /chats/room/users-list      | chatroom_id: string                             | 채팅방 유저목록  | 201, 403(권한 없음)    | {user_id: string, user_auth: string}, ... |                       |
| 2    | PUT /chats/room/kick            | chatroom_id: string, user_id_to_kick: string    | 채팅방 내보내기  | 201, 403(권한 없음)    |                                           |                       |
| 3    | POST /chats/room/ban            | chatroom_id: string, user_id_to_ban: string     | 채팅방 차단      | 201, 403(권한 없음)    |                                           |                       |
| 4    | POST /chats/room/mute           | chatroom_id: string, user_id_to_mute: string    | 벙어리 적용      | 201, 403(권한 없음)    |                                           |                       |
| 5    | POST /chats/room/empowerment    | chatroom_id: string, user_id_to_empower: string | 관리자 권한 부여 | 201, 403(권한 없음)    |                                           |                       |
| 6    | POST /chats/room/game-request   | chatroom_id: string, user_id_to_game: string    | 대결 신청        | 201                    |                                           |                       |

- No1에서 현재 벙어리 여부 데이터를 보내지 않은 이유: 현재 벙어리 상태여도 다시 벙어리 시키면 시간 리셋 가능하게 하도록

- 대다수가 websocket을 통한 ws 프로토콜을 이용하는 것으로 바뀔 가능성 높음



## 채팅방 차단 목록(CH008)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-CH008.png" style="zoom:150%;" />

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-CH008-1.png" style="zoom:150%;" />



| No   | request - HTTP method, Endpoint | request - parameter                         | description          | response - status code | response - data                      | Table-Column-Variable |
| ---- | ------------------------------- | ------------------------------------------- | -------------------- | ---------------------- | ------------------------------------ | --------------------- |
| 1    | GET /chats/room/bans-list       | chatroom_id: string                         | 채팅방 차단 유저목록 | 200                    | {user_id: string, auth: string}, ... |                       |
| 2    | PUT /chats/room/ban-removal     | chatroom_id: string, userid_to_free: string | 채팅방 차단 해제     | 201                    |                                      |                       |



# 게임

## 게임목록(GM001)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-GM001.png" style="zoom:150%;" />

| No   | attribute           | event                                  | Request   | Response                                                     | route |
| ---- | ------------------- | -------------------------------------- | --------- | ------------------------------------------------------------ | ----- |
| 1    | 게임방 목록         | 참여할 수 있는 게임 목록을 로드        | -         | <게임방> 목록<br />- <게임방> : 게임방 이름, 게임방 owner, 채팅방 유형 생성 시간, 목표 점수, 난이도 | -     |
| 2    | 정보 보기 버튼      | 해당 유저 프로필 레이어팝업을 호출     | -         | -                                                            | MN003 |
| 3    | 일반 게임 생성 버튼 | 일반 게임을 생성하는 레이어팝업을 호출 | -         | -                                                            | GM003 |
| 4    | 래더 게임 시작 버튼 | 레더 게임을 시작하는 레이어팝업을 호출 | 내 닉네임 | -                                                            | GM006 |
| 5    | 입장 버튼           | 일반 게임 입장 레이어팝업을 호출       | -         | -                                                            | GM002 |
|      |                     |                                        |           |                                                              |       |

- 래더 게임은 시작 버튼을 누르면 큐에 추가가 되기 때문에 닉네임을 포함하여 서버에 전달한다.

  



## 일반 게임 입장(GM002)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-GM002.png" style="zoom:150%;" />

| No   | attribute | event                              | Request | Response | route |
| ---- | --------- | ---------------------------------- | ------- | -------- | ----- |
| 1    | 확인 버튼 | 참여할 수 있는 게임 목록을 로드    | -       | -        | GM007 |
| 2    | 취소 버튼 | 해당 유저 프로필 레이어팝업을 호출 | -       | -        | GM001 |
|      |           |                                    |         |          |       |

- 일반 게임방에 입장하기 전에 레이어팝업을 호출 : 바로 게임이 시작되면 당황스럽기 때문에 설정하였다.
- 연결이 완료된 시점에서 게임 화면(GM007)로 바뀌기 전에 3초 로딩 시간을 설정한다.





## 일반 게임 생성(GM003)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-GM003.png" style="zoom:150%;" />

| No   | attribute           | event            | Request                                  | Response    | route |
| ---- | ------------------- | ---------------- | ---------------------------------------- | ----------- | ----- |
| 1    | 게임방 제목 폼      | 게임방 제목 입력 | -                                        | -           | -     |
| 2    | 목표 점수 콤보 박스 | 공목표 점수 입력 | -                                        | -           | -     |
| 3    | 난이도 선택 폼      | 난이도 입력      | -                                        | -           | -     |
| 4    | 확인 버튼           | 게임방 생성      | 게임방 제목, 게임 목표 점수, 게임 난이도 | 게임방 번호 | GM004 |
|      |                     |                  |                                          |             |       |

- 게임방 제목은 최대 20자이다.
- 목표 점수는 3점에서 10점까지 1점 단위로 고를 수 있다.
- 난이도는 게임 내 바의 길이를 의미힌다.
  - ? 아이콘에서 간략한 설명을 달아둘 것!





## 일반 게임 대기(GM004)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-GM004.png" style="zoom:150%;" />

| No   | attribute                   | event                              | Request                | Response    | route |
| ---- | --------------------------- | ---------------------------------- | ---------------------- | ----------- | ----- |
| 1    | 게임방 정보                 | 게임방 정보 로드                   | 게임방 번호            | 게임방 제목 | -     |
| 2    | 게임방 정보 수정 레이어팝업 | 게임방 정보 수정 레이어팝업을 호출 | -                      | -           | GM005 |
| 3    | 닫기 버튼                   | 게임방 목록 대기실로 복귀          | 내 닉네임, 게임방 번호 | -           | GM001 |
|      |                             |                                    |                        |             |       |





## 일반 게임 정보 수정(GM005)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-GM005.png" style="zoom:150%;" />

| No   | attribute           | event            | Request                                               | Response    | route |
| ---- | ------------------- | ---------------- | ----------------------------------------------------- | ----------- | ----- |
| 1    | 게임방 제목 폼      | 게임방 제목 입력 | -                                                     | -           | -     |
| 2    | 목표 점수 콤보 박스 | 공목표 점수 입력 | -                                                     | -           | -     |
| 3    | 난이도 선택 폼      | 난이도 입력      | -                                                     | -           | -     |
| 4    | 수정 버튼           | 게임방 정보 수정 | 게임방 번호, 게임방 제목, 게임 목표 점수, 게임 난이도 | 게임방 번호 | GM004 |
|      |                     |                  |                                                       |             |       |





## 레더 게임 시작(GM006)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-GM006.png" style="zoom:150%;" />

| No   | attribute | event                 | Request   | Response | route |
| ---- | --------- | --------------------- | --------- | -------- | ----- |
| 1    | 취소 버튼 | 레더 게임 큐에서 제거 | 내 닉네임 | -        | GM004 |
|      |           |                       |           |          |       |





## 게임 화면(GM007)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-GM007.png" style="zoom:150%;" />

| No   | attribute | event                                                        | Request     | Response                                                  | route |
| ---- | --------- | ------------------------------------------------------------ | ----------- | --------------------------------------------------------- | ----- |
| 1    | 경기 매칭 | 소켓을 통해? 경기 매칭                                       | -           | 게임방 번호                                               | GM007 |
| 2    | 경기 정보 | 경기 초기 정보를 로드                                        | 게임방 번호 | 내 닉네임, 상대 닉네임, 경기 점수 정보, 난이도, 목표 점수 | -     |
| 3    | 공        | 공의 위치가 양 끝이 되면 점수가 올라감<br />- 경기 정보를 업데이트 | -           | -                                                         | -     |
| 4    | 막대      | 막대 위치를 주기적으로 계속 업데이트                         | -           | -                                                         | -     |
| 5    | 점수판    | 경기 정보를 표시<br />- 점수가 났을 때, 점수판을 업데이트    | -           | -                                                         | -     |
| 6    | 경기 종료 | 경기 결과 화면을 띄움                                        | 게임방 번호 |                                                           | GM008 |
|      |           |                                                              |             |                                                           |       |





## 게임 결과 화면(GM008)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-GM008.png" style="zoom:150%;" />

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-GM008-1.png" style="zoom:150%;" />

| No   | attribute       | event                          | Request     | Response                                                     | route |
| ---- | --------------- | ------------------------------ | ----------- | ------------------------------------------------------------ | ----- |
| 1    | 게임 결과       | 게임 결과를 로드               | 게임방 번호 | 내 닉네임, 내 이미지, 상대 닉네임, 상대 이미지, 경기 점수 정보, 전체 전적(승, 패), 래더 점수, 레더 점수 증감분 | -     |
| 2    | 되돌아가기 버튼 | 게임 메인 화면으로 되돌아간다. | -           | -                                                            | MN001 |
|      |                 |                                |             |                                                              |       |

































