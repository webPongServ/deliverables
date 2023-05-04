[TOC]

## 테이블명 format

- prefix
  - TB : 테이블 (object 구분)

- middle
  - "소기능" + "sequence 번호"
    - UA01 : user agent table
    - CH01 : channel table
    - GM01 : game table
    - CM01 : common table

- postfix
  - M : 기본 (master)
  - L : 내역 (list)
  - D : 상세 (detail)
  - S : 통계 (statistics)
  - H : 이력 (history)



tip: systimestamp's FF : the foramt string to represent fractional seconds



## Table list

| No   | table id | table name         | comment |
| ---- | -------- | ------------------ | ------- |
| 1    | TB_UA01M | user agent master |         |
| 2 | TB_UA01L | user agent state list |         |
| 3 | TB_UA02L | user agent friend list |         |
| 4 | TB_UA03M | user agent acheivement master | |
| 5 | TB_UA03D | user agent acheivement detail |         |
| 6 | TB_GM01L | game list |         |
| 7 | TB_GM01D | game detail |         |
| 8 | TB_GM02S | game statistics by user |         |
| 9 | TB_GM03D | game option detail |         |
| 10 | TB_GM04L | ladder ready list |         |
| 11 | TB_CH01L | chatroom list |         |
| 12 | TB_CH02L | chatroom user list |         |
| 13 | TB_CH02D | chatroom restriction detail |         |
| 14 | TB_CH03L | chatroom message list |         |
| 15 | TB_CH04L | chatroom block list |         |
| 16 | TB_CM01D | commond code detail | |





## Table Detail

- TB_UA01M

|      | key  | colunm id      | column name            | null     | data type    | domain                 | comment          | default         |
| ---- | ---- | -------------- | ---------------------- | -------- | ------------ | ---------------------- | ---------------- | --------------- |
| 1    | k    | USER_ID        | user id                | not null | varchar(8)   | 넘으면 가입불가 로직   |                  |                 |
| 2    |      | NICKNAME       | 닉네임                 | not null | varchar(8)   | 1~8 글자(숫자영문한글) |     UNIQUE             | USER_ID         |
| 3    |      | TWOFACTOR      | 투펙터여부             | not null | boolean      |                        | 42intra 검증컬럼 |                 |
| 4    |      | TWOFACTOR_DATA | 투벡터데이터           |          | varchar(50)  |                        | 42intra 검증컬럼 |                 |
| 5    |      | IMG_PATH       | 이미지경로             | not null | varchar(200) |                        |                  | img/default.png |
| 6    |      | DEL_TF         | 삭제여부               | not null | boolean      | 삭제:true              |                  | false           |
| 7    |      | FRST_DTTM      | 최초 입력 연월일시분초 | not null | timestamp(6) |                        |                  | now()           |
| 8    |      | LAST_DTTM      | 최종 입력 연월일시분초 | not null | timestamp(6) |                        |                  | now()           |

- 회원가입에 의해서 데이터가 추가된다.
- 최초 회원가입 시 닉네임은 intra_id 로 대체한다.
  - 새로운 닉네임으로 변경하려면 1~8글자 내에서만 가능하다.
- 이미지는 경로가 저장되고, 기본이미지가 존재한다.
  - 이미지를 입력하려면 기존파일이름이 아닌,새로운 파일이름으로 unique한 데이터를 생성해서 저장해야 한다.



- TB_UA01L 유저상태 내역

|      | key  | colunm id   | column name            | null     | data type    | domain             | comment | default              |
| ---- | ---- | ----------- | ---------------------- | -------- | ------------ | ------------------ | ------- | -------------------- |
| 1    | k, F | USER_ID     | user id                | not null | varchar(8)   |                    |         |                      |
| 2    | k    | LOGIN_SEQ   | 로그인 시퀀스          | not null | number       | increment sequence |         |                      |
| 3    |      | LOGIN_DTTM  | 로그인시간             | not null | timestamp(0) |                    |         | current_timestamp(0) |
| 4    |      | LOGOUT_DTTM | 로그아웃시간           |          | date         |                    |         |                      |
| 5    |      | CHT_TF      | 채팅여부               | not null | boolean      |                    |         | false                |
| 6    |      | GM_TF       | 게임여부               | not null | boolean      |                    |         | false                |
| 7    |      | SESSION_ID  | 세션ID                 | not null |              |                    |         |                      |
| 8    |      | LOGIN_TF    | 로그인여부             | not null | boolean      |                    |         | true                 |
| 9    |      | DEL_TF      | 삭제여부               | not null | boolean      | 삭제:true          |         | false                |
| 10   |      | FRST_DTTM   | 최초 입력 연월일시분초 | not null | timestamp(6) |                    |         | now()                |
| 11   |      | LAST_DTTM   | 최종 입력 연월일시분초 | not null | timestamp(6) |                    |         | now()                |

- 로그인 시 데이터가 추가된다. (로그인여부Y, 채팅여부N, 게임여부N)
- 친구목록의 상태는 해당 테이블을 참고한다.
- 로그아웃되는 순간 로그인여부,채팅여부,게임여부는 N이 되어야 한다.
- 로그아웃 : 1) 웹사이트 이벤트가 10분이상 없는 경우 서버가 로그아웃 처리. 2) 사용자가 로그아웃 버튼 클릭
- 채팅여부 :  Y -  채팅입장, N - 채팅나가기, 게임참가, 로그아웃, 채팅소켓의 헬스체크 미응답
- 게임여부 : Y - 게임참가, N - 게임종료, 게임나가기, 채팅참가, 로그아웃 , 채팅소켓의 헬스체크 미응답
- 세션ID : 추가할지 말지 고민중. (중복로그인 체크)



- TB_UA02L 유저친구내역

|      | key  | colunm id  | column name            | null     | data type    | domain          | comment | default |
| ---- | ---- | ---------- | ---------------------- | -------- | ------------ | --------------- | ------- | ------- |
| 1    | K, F | USER_ID    | user id                | not null |              |                 |         |         |
| 2    | K    | FR_USER_ID | 친구 user id           | not null |              |                 |         |         |
| 3    |      | ST_CD      | 상태코드               | not null | varcahr(2)   | 01:등록,02:해제 |         |         |
| 4    |      | RSST_DTTM  | 등록일시               |          | timestamp(0) |                 |         |         |
| 5    |      | RELE_DTTM  | 해제일시               |          | timestamp(0) |                 |         |         |
| 6    |      | DEL_TF     | 삭제여부               | not null | boolean      | 삭제:true       |         | false   |
| 7    |      | FRST_DTTM  | 최초 입력 연월일시분초 | not null | timestamp(6) |                 |         | now()   |
| 8    |      | LAST_DTTM  | 최종 입력 연월일시분초 | not null | timestamp(6) |                 |         | now()   |

- 사용자가 웹사이트에서 직접 등록 혹은 해제한다.
- 같은 친구 등록/해제를 반복할 경우 데이터추가가 아닌 변경이 이루어진다.



- TB_UA03M 유저업적기본


|      | key  | colunm id     | column name            | null     | data type     | domain    | comment | default |
| ---- | ---- | ------------- | ---------------------- | -------- | ------------- | --------- | ------- | ------- |
| 1    | K    | ACHV_CD       | 업적코드               | not null | varchar(200)  |           |         |         |
| 2    |      | ACHV_NM       | 업적명                 | not null | varchar(200)  |           |         |         |
| 3    |      | ACHV_CMT      | 업적설명               | not null | varchar(1000) |           |         |         |
| 4    |      | ACHV_IMG_PATH | 업적이미지위치         | not null | varchar(200)  |           |         |         |
| 5    |      | DEL_TF        | 삭제여부               | not null | boolean       | 삭제:true |         | false   |
| 6    |      | FRST_DTTM     | 최초 입력 연월일시분초 | not null | timestamp(6)  |           |         | now()   |
| 7    |      | LAST_DTTM     | 최종 입력 연월일시분초 | not null | timestamp(6)  |           |         | now()   |



- TB_UA03D  유저별업적상세


|      | key  | colunm id | column name            | null     | data type    | domain    | comment | default |
| ---- | ---- | --------- | ---------------------- | -------- | ------------ | --------- | ------- | ------- |
| 1    | K    | USER_ID   | user id                | not null | varchar(200) |           |         |         |
| 2    | F    | ACHV_CD   | 업적코드               | not null | varchar(200) |           |         |         |
| 3    |      | ACHV_DTTM | 업적일시               |          | timestamp(0) |           |         |         |
| 4    |      | DEL_TF    | 삭제여부               | not null | boolean      | 삭제:true |         | false   |
| 5    |      | FRST_DTTM | 최초 입력 연월일시분초 | not null | timestamp(6) |           |         | now()   |
| 6    |      | LAST_DTTM | 최종 입력 연월일시분초 | not null | timestamp(6) |           |         | now()   |

- 업적은 일단 한번 달성하면 이후 충족되지 않더라도 유지됨을 가정한다.
- 업적달성 시 검증 후 데이터를 추가하거나, 배치를 통해 추가할 수 있다.
  - 검증로직은 게임룰 문서만 명시하고 DB로 관리하지는 않도록 한다. (**회의필요**)
- 프로필에서 조회할 때에는 게임업적기본 테이블과 outer join해서 사용한다.





- TB_GM01L  게임내역
  - 게임내역을 관리하는 테이블이다.


|      | key  | colunm id     | column name                  | null     | data type    | domain                                 | comment | default     |
| ---- | ---- | ------------- | ---------------------------- | -------- | ------------ | -------------------------------------- | ------- | ----------- |
| 1    | K    | GM_SRNO       | 게임일련번호                 | not null | vachar(12)   | YYYYMMDDNNNN, N : seq                  |         |             |
| 2    |      | GM_STRT_DTTM  | 게임시작시간                 |          | timestamp(0) |                                        |         |             |
| 3    |      | GM_END_DTTM/  | 게임종료시간                 |          | timestamp(0) |                                        |         |             |
| 4    |      | GM_TYPE       | 게임유형                     | not null | varchar(2)   | 01:1v1, 02:ladder                      |         |             |
| 5    |      | END_TYPE      | 종료유형                     | not null | varchar(2)   | 01:게임중 02:점수, 03:닷지, 04: 미실행 |         | 01          |
| 6    |      | TRGT_SCR      | 목표점수                     | not null | number       |                                        |         | 5           |
| 7    |      | LV_DFCT       | 난이도 (level of difficulty) | not null | number       |                                        |         | 100         |
| 8    |      | BGRD_IMG_PATH | 배경이미지경로               | not null | vachar(200)  |                                        |         | default.png |
| 9    |      | DEL_TF        | 삭제여부                     | not null | boolean      | 삭제:true                              |         | false       |
| 10   |      | FRST_DTTM     | 최초 입력 연월일시분초       | not null | timestamp(6) |                                        |         | now()       |
| 11   |      | LAST_DTTM     | 최종 입력 연월일시분초       | not null | timestamp(6) |                                        |         | now()       |

- 게임시작시간 : 플레이어2명 입장 시 now() 로 업데이트 된다.
- 게임종료시간 : 게임종료 or 플레이어임의탈주일 경우 now()로 세팅된다.
- 요구점수,바길이,배경은 게임옵션상세 테이블을 참고하여 세팅해야 한다.



- TB_GM01D  게임상세
  - 게임 1판당 플레이어의 속성을 관리하는 테이블이다.


|      | key  | colunm id  | column name            | null     | data type    | domain                       | comment | default |
| ---- | ---- | ---------- | ---------------------- | -------- | ------------ | ---------------------------- | ------- | ------- |
| 1    | K, F | GM_SRNO    | 게임일련번호           | not null | vachar(12)   | YYYYMMDDNNNN, N : seq        |         |         |
| 2    | K    | USER_ID    | user ID                | not null | varchar(200) | TB_UA01M.USER_ID             |         |         |
| 3    |      | GET_SCR    | 획득점수               | not null | number       |                              |         | 0       |
| 4    |      | GM_RSLT_CD | 게임결과코드           | not null | varchar(2)   | 미결과: 00, 승리:01, 패배:02 |         | 00      |
| 5    |      | RSLT_LLVL  | 결과레더레벨           | not null | number       | ELO룰에 따라 생성됨          |         | 0       |
| 6    |      | ENTRY_DTTM | 입장시간               | not null | timestamp(0) |                              |         |         |
| 7    |      | EXIT_DTTM  | 퇴장시간               |          | timestamp(0) |                              |         |         |
| 8    |      | DEL_TF     | 삭제여부               | not null | boolean      | 삭제:true                    |         | false   |
| 9    |      | FRST_DTTM  | 최초 입력 연월일시분초 | not null | timestamp(6) |                              |         | now()   |
| 10   |      | LAST_DTTM  | 최종 입력 연월일시분초 | not null | timestamp(6) |                              |         | now()   |

- 게임일련번호 : 202304160001 ~ 9999
  - 하루 최대 1만판 미만 가능.

- 게임 입장 시 데이터가 추가된다.





- TB_GM02S  유저별게임통계
  - 게임 종료 후 갱신된다.


|      | key  | colunm id  | column name            | null     | data type    | domain    | comment | default |
| ---- | ---- | ---------- | ---------------------- | -------- | ------------ | --------- | ------- | ------- |
| 1    | K    | USER_ID    | user ID                | not null | varchar(200) |           |         |         |
| 2    |      | TOT_RCD    | 총전적                 | not null | number       |           |         | 0       |
| 3    |      | TOT_VCT    | 총승리                 | not null | number       |           |         | 0       |
| 4    |      | TOT_DFT    | 총패배                 | not null | number       |           |         | 0       |
| 5    |      | LLVL       | 레더레벨               | not null | number       |           |         | 1000    |
| 6    |      | LDDR_PCENT | 레더승률               | not null | number       | 60(%)     |         | 0       |
| 7    |      | DEL_TF     | 삭제여부               | not null | boolean      | 삭제:true |         | false   |
| 8    |      | FRST_DTTM  | 최초 입력 연월일시분초 | not null | timestamp(6) |           |         | now()   |
| 9    |      | LAST_DTTM  | 최종 입력 연월일시분초 | not null | timestamp(6) |           |         | now()   |

- 일별,월별 통계 생성 고려
  - 데이터량이 많지 않을것으로 예상되어 고려안함.



-   TB_GM05D 게임옵션상세
    - 게임환경을 정의하는 테이블이다.
    - 게임 시작 시 해당테이블을 참고해야 한다.


|      | key  | colunm id  | column name            | null     | data type    | domain    | comment | default |
| ---- | ---- | ---------- | ---------------------- | -------- | ------------ | --------- | ------- | ------- |
| 1    | k    | OPT_CD     | 옵션코드               | not null | varchar(100) |           |         |         |
| 2    | k    | OPT_DTL_CD | 옵션상세코드           | not null | varchar(100) |           |         |         |
| 3    |      | OPT_NM     | 옵션명                 | not null | varchar(200) |           |         |         |
| 4    |      | OPT_VAL    | 옵션값                 | not null | varchar(200) |           |         |         |
| 5    |      | DEL_TF     | 삭제여부               | not null | boolean      | 삭제:true |         | false   |
| 6    |      | FRST_DTTM  | 최초 입력 연월일시분초 | not null | timestamp(6) |           |         | now()   |
| 7    |      | LAST_DTTM  | 최종 입력 연월일시분초 | not null | timestamp(6) |           |         | now()   |

- 업무가 복잡하지 않으므로 옵선기본,상세 테이블로 나누지 않는다.



- **SAMPLE**

| 옵션코드    | 옵션상세코드 | 옵션명       | 옵션값       |
| ----------- | ------------ | ------------ | ------------ |
| BAR         | 01           | 바 길이      | 10           |
| BAR         | 02           | 바 길이      | 50           |
| BAR         | 03           | 바 길이      | 100          |
| SCORE       | 01           | 점수         | 3            |
| SCORE       | 02           | 점수         | 5            |
| SCORE       | 03           | 점수         | 7            |
| BACKBROUND  | 01           | 배경         | default.png  |
| BACKBROUND  | 02           | 배경         | default2.png |
| BACKBROUND  | 03           | 배경         | default3.png |
| LADDER_WAIT | 01           | 레더대기시간 | 60           |





- TB_GM04L 레더대기내역
  - 서버 스케쥴러에서 해당테이블을 참조하면서 게임매칭을 진행한다.

|      | key  | colunm id      | column name            | null     | data type    | domain       | comment | default |
| ---- | ---- | -------------- | ---------------------- | -------- | ------------ | ------------ | ------- | ------- |
| 1    | k, F | USER_ID        | user ID                | not null | varchar(8)   |              |         |         |
| 2    | k    | LDDR_RDY_SRNO  | 레더대기일련번호       | not null | varchar(12)  | YYYYMMDDNNNN |         |         |
| 3    |      | MTCH_APLY_DTTM | 매칭신청일시           | not null | timestamp(0) |              |         |         |
| 4    |      | MTCH_TF        | 매칭여부               | not null | boolean      |              |         | false   |
| 5    |      | RDDR_RDY_TF    | 레더대기여부           | not null | boolean      |              |         | true    |
| 6    | F    | MTCH_USER_ID   | 매칭 user ID           |          | varchar(8)   |              |         |         |
| 7    | F    | GM_SRNO        | 게임일련번호           |          | varchar(12)  |              |         |         |
| 8    |      | DEL_TF         | 삭제여부               | not null | boolean      | 삭제:true    |         | false   |
| 9    |      | FRST_DTTM      | 최초 입력 연월일시분초 | not null | timestamp(6) |              |         | now()   |
| 10   |      | LAST_DTTM      | 최종 입력 연월일시분초 | not null | timestamp(6) |              |         | now()   |

- 임의사용자가 레더신청 시  데이터가 추가된다.

- 매칭이후 게임일련번호가 업데이트 되어야 한다. 
- 매칭제한시간이 지나면 레더대기여부가 N 으로 세팅되고 더이상 추적되지 않는다.



- TB_CH01L

|      | key  | colunm id    | column name            | null     | data type    | domain                               | comment          | default                         |
| ---- | ---- | ------------ | ---------------------- | -------- | ------------ | ------------------------------------ | ---------------- | ------------------------------- |
| 1    | p    | CHT_RM_ID    | chat room  id          | not null | varchar(12)  | YYYYMMDDNNNN                         |                  |                                 |
| 2    |      | CHT_RM_NM    | channel name           | not null | varchar(50)  |                                      |                  | TB_UA01M.USER_ID + ' 의 채팅방' |
| 3    |      | CHT_RM_TYPE  | channel type           | not null | varchar(2)   | public: 01, protected:02, private:03 |                  | 01                              |
| 4    |      | MAX_USER_CNT | Maximum people         | not null | Number       |                                      |                  | 5                               |
| 5    |      | CHT_RM_PWD   | channel password       |          | varchar(64)  | sha256                               |                  |                                 |
| 6    |      | CHT_RM_TF    | existence              | not null | Boolean      | 생성:true, 종료: false               | 채팅방 존재 유무 | true                            |
| 7    |      | DEL_TF       | 삭제여부               | not null | boolean      | 삭제:true                            |                  | false                           |
| 8    |      | FRST_DTTM    | 최초 입력 연월일시분초 | not null | timestamp(6) |                                      |                  | now()                           |
| 9    |      | LAST_DTTM    | 최종 입력 연월일시분초 | not null | timestamp(6) |                                      |                  | now()                           |

- NOTICE: current user count는 TB_CH02L과 join을 이용



- TB_CH02L

|      | key  | colunm id      | column name            | null     | data type    | domain                                | comment | default           |
| ---- | ---- | -------------- | ---------------------- | -------- | ------------ | ------------------------------------- | ------- | ----------------- |
| 1    | f, k | CHT_RM_ID      | channel id             | not null | varchar(12)  | YYYYMMDDNNNN                          |         |                   |
| 2    | f, k | USER_ID        | user id                | not null | varchar(8)   |                                       |         |                   |
| 3    |      | CHT_RM_AUTH    | authority              | not null | varchar(2)   | Owner:01, Administrator:02, Normal:03 |         |                   |
| 4    |      | CHT_RM_JOIN_TF | existence              | not null | Boolean      | 채팅참여:true, 채팅미참여:false       |         | true              |
| 5    |      | ENTRY_DTTM     | entry time             | not null | timestamp(0) |                                       |         | curr_timestamp(0) |
| 6    |      | AUTH_CHG_DTTM  | change time            | not null | timestamp(0) |                                       |         | curr_timestamp(0) |
| 7    |      | DEL_TF         | 삭제여부               | not null | boolean      | 삭제:true/                            |         | false             |
| 8    |      | FRST_DTTM      | 최초 입력 연월일시분초 | not null | timestamp(6) |                                       |         | now()             |
| 9    |      | LAST_DTTM      | 최종 입력 연월일시분초 | not null | timestamp(6) |                                       |         | now()             |



- TB_CH02D

|      | key  | colunm id      | column name            | null     | data type    | domain                   | comment | default           |
| ---- | ---- | -------------- | ---------------------- | -------- | ------------ | ------------------------ | ------- | ----------------- |
| 1    | k    | CHT_RM_ID      | channel id             | not null | varchar(12)  | YYYYMMDDNNNN             |         |                   |
| 2    | k    | USER_ID        | user id                | not null | varchar(8)   |                          |         |                   |
| 3    | k    | CHT_RM_RSTR_CD | restriction            | not null | varchar(2)   | mute:01, ban:02, kick:03 |         |                   |
| 4    |      | RSTR_CRTN_DTTM | 제약생성시간           | not null | timestamp(0) |                          |         | curr_timestamp(0) |
| 5    |      | RSTR_TM        | 제약시간               | not null | number       | 42초, 무한:-1            |         |                   |
| 6    |      | VLD_TF         | Valid                  | not null | Boolean      | 지정:true, 해제:false    |         | true              |
| 7    |      | DEL_TF         | 삭제여부               | not null | boolean      | 삭제:true                |         | false             |
| 8    |      | FRST_DTTM      | 최초 입력 연월일시분초 | not null | timestamp(6) |                          |         | now()             |
| 9    |      | LAST_DTTM      | 최종 입력 연월일시분초 | not null | timestamp(6) |                          |         | now()             |




- TB_CH03L

|      | key  | colunm id | column name            | null     | data type     | domain       | comment                                            | default |
| ---- | ---- | --------- | ---------------------- | -------- | ------------- | ------------ | -------------------------------------------------- | ------- |
| 1    | k    | CHT_RM_ID | channel id             | not null | vachar(12)    | YYYYMMDDNNNN |                                                    |         |
| 2    | k    | USER_ID   | user id                | not null | number        |              |                                                    |         |
| 3    | k    | MSG_SEQ   | message sequence       | not null | number        | seq          |                                                    | 0       |
| 4    |      | MSG       | Message                | not null | varchar(1000) |              | 메세지길이 넘으면 오류표출 적절히 할 수 있어야 함. |         |
| 5    |      | DEL_TF    | 삭제여부               | not null | boolean       | 삭제:true    |                                                    | false   |
| 6    |      | FRST_DTTM | 최초 입력 연월일시분초 | not null | timestamp(6)  |              |                                                    | now()   |
| 7    |      | LAST_DTTM | 최종 입력 연월일시분초 | not null | timestamp(6)  |              |                                                    | now()   |



- TB_CH04L  메세지차단유저내역

|      | key  | colunm id     | column name            | null     | data type    | domain          | comment | default |
| ---- | ---- | ------------- | ---------------------- | -------- | ------------ | --------------- | ------- | ------- |
| 1    | K, F | USER_ID       | user id                | not null | varrcher(8)  |                 |         |         |
| 2    | K    | BLOCK_USER_ID | block user  id         | not null | varrcher(8)  |                 |         |         |
| 3    |      | ST_CD         | 상태코드               | not null | varchar(2)   | 01:등록,02:해제 |         |         |
| 4    |      | RSST_DTTM     | 등록일시               |          | timestamp(0) |                 |         |         |
| 5    |      | RELE_DTTM     | 해제일시               |          | timestamp(0) |                 |         |         |
| 6    |      | DEL_TF        | 삭제여부               | not null | boolean      | 삭제:true       |         | false   |
| 7    |      | FRST_DTTM     | 최초 입력 연월일시분초 | not null | timestamp(6) |                 |         | now()   |
| 8    |      | LAST_DTTM     | 최종 입력 연월일시분초 | not null | timestamp(6) |                 |         | now()   |

- 처음 블락지정 시 데이터가 생성되고, 이후 상태가 변경되면 update 만 진행하고 히스토리는 관리하지 않음.





- TB_CM01D 공통코드상세
  - WPS 에서 관리하는 각종 코드들을 정의하는 테이블이다.

|      | key  | colunm id      | column name            | null     | data type    | domain    | comment | default |
| ---- | ---- | -------------- | ---------------------- | -------- | ------------ | --------- | ------- | ------- |
| 1    | K    | CMMN_CD        | 공통코드               | not null |              |           |         |         |
| 2    | K    | CMMN_DTL_CD    | 공통코드상세코드       | not null |              |           |         |         |
| 3    |      | CMMN_CD_NM     | 공통코드명             | not null |              |           |         |         |
| 4    |      | CMMN_CD_DTL_NM | 공통코드상세명         | not null |              |           |         |         |
| 5    |      | DEL_TF         | 삭제여부               | not null | boolean      | 삭제:true |         | false   |
| 6    |      | FRST_DTTM      | 최초 입력 연월일시분초 | not null | timestamp(6) |           |         | now()   |
| 7    |      | LAST_DTTM      | 최종 입력 연월일시분초 | not null | timestamp(6) |           |         | now()   |

- 업무가 복잡하지 않으므로 공통코드 기본,상세 테이블로 나누지 않는다.



- **sample**

| 공통코드     | 공통코드상세코드 | 상세내용         |      |
| ------------ | ---------------- | ---------------- | ---- |
| 게임유형     | 01               | 1v1              |      |
|              | 02               | ladder           |      |
| GM_EXIT_TYPE | 01               | 게임중           |      |
| GM_EXIT_TYPE | 02               | 점수             |      |
| GM_EXIT_TYPE | 03               | 닷지             |      |
| GM_EXIT_TYPE | 04               | 미실행           |      |
| IMG_PATH     | 01               | imt/default1.png |      |
| IMG_PATH     | 02               | imt/default2.png |      |
| IMG_PATH     | 03               | imt/default3.png |      |
| IMG_PATH     | 04               | imt/default4.png |      |
|              |                  | ,,,              |      |
|              |                  |                  |      |

