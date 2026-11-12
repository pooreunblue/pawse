# pawse API 목록 (초안)

관련 문서: [개요](overview.md) · [ERD](erd.md)

- 기본 경로: `/api`
- 인증: `Authorization: Bearer {JWT}`. 🔓 표시가 없는 API는 모두 로그인이 필요하다.
- 반려동물 하위 자원은 본인 소유가 아니면 `403`을 돌려준다.
- 오류 응답 형식: `{ "code": "PET_NOT_FOUND", "message": "..." }`

## MVP (2~6단계)

### 회원 (2단계)

| 메서드 | 경로 | 설명 |
| --- | --- | --- |
| POST | `/api/auth/signup` 🔓 | 가입 (이메일, 비밀번호, 닉네임) |
| POST | `/api/auth/login` 🔓 | 로그인, JWT 발급 |
| GET | `/api/members/me` | 내 정보 |
| DELETE | `/api/members/me` | 탈퇴. 사진·아바타·메시지까지 모두 삭제 |

### 반려동물·앨범 (3단계)

| 메서드 | 경로 | 설명 |
| --- | --- | --- |
| GET | `/api/species` 🔓 | 종 목록 |
| POST | `/api/pets` | 반려동물 등록 |
| GET | `/api/pets` | 내 반려동물 목록 |
| GET | `/api/pets/{petId}` | 상세 |
| PATCH | `/api/pets/{petId}` | 수정 (이별 날짜 입력 포함) |
| DELETE | `/api/pets/{petId}` | 삭제 |
| POST | `/api/pets/{petId}/photos` | 사진 여러 장 업로드 (multipart) |
| GET | `/api/pets/{petId}/photos` | 앨범 (페이지) |
| GET | `/api/pets/{petId}/timeline` | 날짜순 연표 |
| PATCH | `/api/photos/{photoId}` | 설명·촬영일 수정 |
| DELETE | `/api/photos/{photoId}` | 삭제 |
| PUT | `/api/pets/{petId}/traits` | 습관·성격 기록 전체 교체 |
| GET | `/api/pets/{petId}/traits` | 조회 |

### 아바타 (4단계)

| 메서드 | 경로 | 설명 |
| --- | --- | --- |
| POST | `/api/pets/{petId}/avatar` | 기준 사진으로 생성 요청. 즉시 `202`와 `PENDING` 반환 |
| GET | `/api/pets/{petId}/avatar` | 상태와 이미지 조회 (앱이 완료될 때까지 주기적으로 조회) |
| PATCH | `/api/pets/{petId}/avatar` | 등장 동의 변경 |
| DELETE | `/api/pets/{petId}/avatar` | 월드에서 내리기 |

### 월드 (5단계)

| 메서드 | 경로 | 설명 |
| --- | --- | --- |
| GET | `/api/worlds` | 월드 목록 (거주 아바타 수 포함) |
| GET | `/api/worlds/{worldId}/avatars` | 화면에 그릴 아바타 무작위 N개. 내 아바타는 항상 포함 |

### 하루 메시지·편지 (6단계)

| 메서드 | 경로 | 설명 |
| --- | --- | --- |
| GET | `/api/pets/{petId}/messages` | 받은 메시지 목록 (페이지) |
| GET | `/api/pets/{petId}/messages/today` | 오늘 메시지. 없으면 `204` |
| POST | `/api/messages/{messageId}/read` | 읽음 처리 |
| PATCH | `/api/pets/{petId}/message-settings` | 수신 켜기/끄기, 쉬어가기 날짜 |
| POST | `/api/pets/{petId}/letters` | 편지 쓰기 |
| GET | `/api/pets/{petId}/letters` | 내가 쓴 편지 목록 |

## 이후 단계

| 단계 | 메서드 | 경로 | 설명 |
| --- | --- | --- | --- |
| 7 | POST | `/api/pets/{petId}/chat` | 대화 한 턴 (응답은 스트리밍 검토) |
| 7 | GET | `/api/pets/{petId}/chat` | 대화 기록 |
| 8 | GET | `/api/outfits` | 의상 목록 |
| 8 | PUT | `/api/pets/{petId}/avatar/outfits` | 착용 의상 교체 |
| 9 | POST/GET/PATCH | `/api/pets/{petId}/bucket-items` | 버킷리스트 |
| 9 | POST/GET | `/api/pets/{petId}/clips` | 목소리·영상 |
| 9 | POST/GET | `/api/pets/{petId}/anniversaries` | 기념일 |
| 9 | POST | `/api/avatars/{avatarId}/flowers` | 꽃 놓기 |
| 10 | POST | `/api/pets/{petId}/guardians/invite` | 가족 초대 |
| 10 | GET | `/api/pets/{petId}/friends` | 단짝 목록 |
