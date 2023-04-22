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



| No   | request - HTTP method, Endpoint | request - parameter | description      | response - status code | response - data                           | Table-Column-Variable |
| ---- | ------------------------------- | ------------------- | ---------------- | ---------------------- | ----------------------------------------- | --------------------- |
| 1    | GET /users/profile/record-count | user_id             | 전적, 래더 점수  | 200                    | win: number, lose: number, ladder: number | TB_GM02S              |
| 2    | GET /users/profile/records      | user_id             | 게임 전적 리스트 | 200                    | {victory: boolean, opposite: string}      | TB_GM01D              |
| 3    | GET /users/profile/achievements | user_id             | 업적 리스트      | 200                    | {achiv_name: string, achiv_cmt: string}   | TB_UA03M              |
| 4    | POST /chats/dm                  | user_id             | DM 요청          | 201                    |                                           |                       |
| 5    | POST /users/add-friend          | user_id             | 친구 추가        | 201                    |                                           |                       |
| 6    | POST /users/block               | user_id             | 차단             | 201                    |                                           |                       |





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





| No   | attribute          | event                       | Request   | Response                                                     | route |
| ---- | ------------------ | --------------------------- | --------- | ------------------------------------------------------------ | ----- |
| 1    | 내 프로필          | 내 프로필 상세 내용을 로드  | 내 닉네임 | 내 닉네임, 내 이미지, 전체 전적(승, 패), 래더 점수           | -     |
| 2    | 전적 버튼          | 내 전적 로드                | 내 닉네임 | <전적> 목록<br />- <전적> : 내 닉네임, 내 이미지, 내 점수, 내 승패 여부, 상대 닉네임, 상대 이미지, 상대 점수 | PF001 |
| 3    | 업적 버튼          | 내 업적 로드                | 내 닉네임 | <업적> 목록<br />- <업적> : 업적 이름, 업적 내용, 업적 이미지 | PF002 |
| 4    | 프로필 수정 버튼   | 프로필 수정 레이어팝업 호출 | -         | -                                                            | PF003 |
| 5    | 2차 인증 설정 버튼 | 2차 인증 레이어팝업 호출    | -         | -                                                            | PF004 |
|      |                    |                             |           |                                                              |       |

- 2차 인증 세부 사항 미구현





| No   | request - HTTP method, Endpoint | request - parameter | description              | response - status code | response - data                           | Table-Column-Variable |
| ---- | ------------------------------- | ------------------- | ------------------------ | ---------------------- | ----------------------------------------- | --------------------- |
| 1    | GET /users/profile/record-count | user_id(intra id)   | 친구목록 리스트 요청     | 200                    | win: number, lose: number, ladder: number | TB_GM02S              |
| 2    | GET /users/profile/record       | user_id(intra id)   | 게임방(일반) 리스트 요청 | 200                    | {victory: boolean, opposite: string}      | TB_GM01D              |
| 3    | GET /users/profile/achievement  | user_id(intra id)   | 업적 리스트 요청         | 200                    | {achiv_name: string, achiv_cmt: string}   | TB_UA03M              |





## 프로필 수정(PF003)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-PF003.png" style="zoom:150%;" />

| No   | attribute              | event                              | Request             | Response               | route |
| ---- | ---------------------- | ---------------------------------- | ------------------- | ---------------------- | ----- |
| 1    | 닉네임 입력 폼         | 수정할 닉네임을 입력               | 닉네임 입력         | 닉네임 중복 여부       | -     |
| 2    | 이미지 업로드 버튼     | 탐색기/finder 호출                 | -                   | -                      | -     |
| 3    | 기본 프로필 이미지 7종 | 기본 프로필 이미지를 로드          | -                   | 기본 프로필 이미지 7종 | -     |
| 4    | 수정 버튼              | 수정사항을 적용할 것을 서버에 요청 | 닉네임, 이미지 파일 | -                      | PF002 |
|      |                        |                                    |                     |                        |       |

- 이미지 크기는 200KB로 제한, png/jpg/jpeg/gif 파일로 제한한다.
- 닉네임 검증이 안 되었거나, 위의 조건에 부합하지 않으면 수정할 수 없도록 설정한다.



## 2차 인증 설정(PF004)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-PF004.png" style="zoom:150%;" />

| No   | attribute      | event                | Request     | Response         | route |
| ---- | -------------- | -------------------- | ----------- | ---------------- | ----- |
| 1    | 닉네임 입력 폼 | 수정할 닉네임을 입력 | 닉네임 입력 | 닉네임 중복 여부 | -     |
|      |                |                      |             |                  |       |

- 미구현

  

# 채팅

## 채팅목록(CH001)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-CH001.png" style="zoom:150%;" />

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-CH001-1.png" style="zoom:150%;" />

| No   | attribute        | event                           | Request             | Response                                                     | route |
| ---- | ---------------- | ------------------------------- | ------------------- | ------------------------------------------------------------ | ----- |
| 1    | 채팅 목록        | 참여할 수 있는 채팅 목록을 로드 | 내 닉네임           | <채팅방> 목록<br />- <채팅방> : 채팅방 이름, 채팅방 owner, 채팅방 유형, 채팅방 현재 인원/최대 인원, 생성 시간, (채팅방 입장 가능 여부?) | -     |
| 2    | 공개 채팅방      | 공개 채팅방에 입장              | 내 닉네임           | 채팅방 번호, 채팅방 입장 가능 여부                           | CH003 |
| 3    | 비공개 채팅방    | 비밀번호 입력 레이어팝업을 호출 | -                   | -                                                            | CH001 |
| 4    | 비밀번호 입력 폼 | 비밀번호를 입력하고 폼을 제출   | 내 닉네임, 비밀번호 | 채팅방 번호, 채팅방 입장 가능 여부                           | CH003 |
| 5    | 채팅방 생성 버튼 | 채팅방 생성 레이어팝업 호출     | -                   | -                                                            | CH002 |
|      |                  |                                 |                     |                                                              |       |

- DM(private) 채팅방을 제외하기 위해서 내 닉네임을 서버로 전송합니다.
- 생성 시간 등 전체적인 목록 정렬은 신경쓰지 않아도 되는가?
- 채팅방에서 차단 목록에 추가된 유저는 채팅방에 입장하려고 하면 입장 가능 여부를 확인하고 방에 입장한다.
  - 다른 방법이 있는가? 서버로 보내기 전에 프론트에서 채팅방 목록을 받을 때 입장 가능 여부를 같이 보내오는 방법



## 채팅방 생성(CH002)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-CH002.png" style="zoom:150%;" />

| No   | attribute           | event                                   | Request                                            | Response    | route |
| ---- | ------------------- | --------------------------------------- | -------------------------------------------------- | ----------- | ----- |
| 1    | 채팅방 제목 폼      | 채팅방 제목 입력                        | -                                                  | -           | -     |
| 2    | 공개 여부 콤보 박스 | 공개 여부 입력                          | -                                                  | -           | -     |
| 3    | 비밀번호 입력 폼    | 채팅방 비밀번호 입력(공개일때만 활성화) | -                                                  | -           | -     |
| 4    | 확인 버튼           | 채팅방 생성                             | 채팅방 제목, 채팅방 공개 여부, 비밀번호, 내 닉네임 | 채팅방 번호 | CH003 |
|      |                     |                                         |                                                    |             |       |

- 채팅방 제목은 최대 20자로 제한한다.
- 채팅방 비밀번호는 최대 10자로 제한한다.



## 채팅방 상세(CH003)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-CH003.png" style="zoom:150%;" />

| No   | attribute             | event                            | Request                | Response                  | route |
| ---- | --------------------- | -------------------------------- | ---------------------- | ------------------------- | ----- |
| 1    | 채팅방 정보           | 채팅방 정보 로드                 | 채팅방 번호            | 채팅방 제목, 채팅방 owner | -     |
| 2    | 채팅                  | 소켓을 통해 채팅을 업데이트      | -                      | -                         | -     |
| 3    | 메시지 입력 폼        | 채팅 메시지를 입력하고 전송      | 메시지 입력 문자열     | -                         | -     |
| 4    | 유저 프로필 버튼      | 유저 프로필 레이어팝업을 호출    | -                      | -                         | MN003 |
| 5    | 유저 목록 버튼        | 유저 목록 레이어팝업 호출        | -                      | -                         | CH007 |
| 6    | 채팅방 정보 수정 버튼 | 채팅방 정보 수정 레이어팝업 호출 | -                      | -                         | CH006 |
| 7    | 닫기 버튼             | 채팅방에서 퇴장 및 대기실로 복귀 | 내 닉네임, 채팅방 번호 | -                         | CH001 |
|      |                       |                                  |                        |                           |       |

- 채팅방 정보 수정 버튼은 채팅방 admin 이상의 권한이 필요하다.
- 채팅 owner가 채팅방에서 퇴장할 시, 1) 가장 오래된 admin 2) 가장 오래된 일반 유저 우선순위로 넘긴다.
  - 채팅방을 나갈 때, api를 호출해야 하는가?



## 채팅방 정보 수정(CH006)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-CH006.png" style="zoom:150%;" />

| No   | attribute           | event                                   | Request                                                      | Response    | route |
| ---- | ------------------- | --------------------------------------- | ------------------------------------------------------------ | ----------- | ----- |
| 1    | 채팅방 제목 폼      | 채팅방 제목 입력                        | -                                                            | -           | -     |
| 2    | 공개 여부 콤보 박스 | 공개 여부 입력                          | -                                                            | -           | -     |
| 3    | 비밀번호 입력 폼    | 채팅방 비밀번호 입력(공개일때만 활성화) | -                                                            | -           | -     |
| 4    | 확인 버튼           | 채팅방 정보 수정                        | 채팅방 번호, 채팅방 제목, 채팅방 공개 여부, 비밀번호, 내 닉네임 | 채팅방 번호 | CH003 |
|      |                     |                                         |                                                              |             |       |

- 채팅방 제목은 최대 20자로 제한한다.
- 채팅방 비밀번호는 최대 10자로 제한한다.



## 채팅방 유저 목록(CH007)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-CH007.png" style="zoom:150%;" />

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-CH007-1.png" style="zoom:150%;" />

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-CH007-2.png" style="zoom:150%;" />

| No   | attribute             | event                                                        | Request                  | Response                                                     | route |
| ---- | --------------------- | ------------------------------------------------------------ | ------------------------ | ------------------------------------------------------------ | ----- |
| 1    | 유저 목록 콤보 박스   | 유저 목록/차단 목록 중 선택                                  | 채팅방 번호              | <유저> 목록<br />- <유저> : 닉네임, 이미지, 유저 정보(owner, admin, normal), 입장 시간 | -     |
| 2    | 채팅방 내보내기 버튼  | 특정 유저를 채팅방에서 추방. 현재 유저 목록에서는 제거       | 채팅방 번호, 유저 닉네임 | -                                                            | -     |
| 3    | 채팅방 차단 버튼      | 특정 유저를 채팅방 차단 목록에 추가. 현재 유저 목록에서는 제거 | 채팅방 번호, 유저 닉네임 | -                                                            | -     |
| 4    | 채팅방 벙어리 버튼    | 특정 유저를 30초 동안 채팅을 하지 못하게 함                  | 채팅방 번호, 유저 닉네임 | -                                                            | -     |
| 5    | 권리자 권한 부여 버튼 | 특정 유저에게 관리자 권한을 부여                             | 채팅방 번호, 유저 닉네임 | -                                                            | -     |
| 6    | 대결 신청 버튼        | 일반 게임 화면으로 이동                                      | -                        | -                                                            | GM000 |
| 7    | 정보 보기 버튼        | 해당 유저 프로필 레이어팝업을 호출                           | -                        | -                                                            | MN003 |
| 8    | 닫기 버튼             | 채팅방으로 되돌아감                                          | -                        | -                                                            | CH006 |
|      |                       |                                                              |                          |                                                              |       |

- 채팅방 유저 관리 상세
  - 채팅방 내보내기 : 일시적으로 내보내기, 추방된 유저는 다시 접속할 수 있음
  - 채팅방 차단  : 채팅방 차단 목록에 추가. 차단된 유저는 다시 접속하지 못함
  - 채팅방 벙어리 : 30초 동안 채팅을 쳐도 다른 사람들에게 채팅이 전달되지 않음.
- 권한 별 메뉴 목록
  - **owner/admin -> normal** : **채팅창 내보내기/차단/벙어리/관리자 권한 부여**, 정보 보기, 대결 신청
  - owner/admin -> owner/admin : 정보 보기, 대결 신청
  - normal -> owner/admin : 정보 보기, 대결 신청
  - normal -> normal : 정보 보기, 대결 신청

- 대결 신청 버튼
  - 신청한 유저는 일반 게임 화면으로 이동하고, 신청받은 유저는 신청 대결 메시지를 받음





## 채팅방 차단 목록(CH008)

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-CH008.png" style="zoom:150%;" />

<img src="/Users/refigo/github-repository/webPongServ/deliverables/img/03-CH008-1.png" style="zoom:150%;" />

| No   | attribute        | event                                                        | Request                  | Response | route |
| ---- | ---------------- | ------------------------------------------------------------ | ------------------------ | -------- | ----- |
| 1    | 채팅방 차단 해제 | 특정 유저를 채팅방 차단 목록에서 제거. 현재 유저 목록에서에 추가 | 채팅방 번호, 유저 닉네임 | -        | -     |
| 2    | 정보 보기 버튼   | 해당 유저 프로필 레이어팝업을 호출                           | -                        | -        | MN003 |
| 3    | 닫기 버튼        | 채팅방으로 되돌아감                                          | -                        | -        | CH006 |
|      |                  |                                                              |                          |          |       |

- 권한 별 메뉴 목록
  - **owner/admin -> normal** : **채팅방 차단 해제**, 정보 보기
  - owner/admin -> owner/admin : 정보 보기
  - normal -> owner/admin : 정보 보기
  - normal -> normal : 정보 보기



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

































