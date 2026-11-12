# pawse ERD (초안)

관련 문서: [개요](overview.md) · [API](api.md)

모든 테이블은 `id`(bigint, PK), `created_at`, `updated_at`을 가진다. 아래에는 생략한다.
IntelliJ의 Markdown 미리보기에서 Mermaid 플러그인을 켜면 그림으로 보인다.

## MVP (1~6단계)

```mermaid
erDiagram
    MEMBER ||--o{ PET : "기른다"
    SPECIES ||--o{ PET : "분류"
    SPECIES ||--|| WORLD : "종마다 하나"
    PET ||--o{ PHOTO : "앨범"
    PET ||--o{ PET_TRAIT : "습관·성격"
    PET ||--o| AVATAR : "아바타"
    PHOTO ||--o{ AVATAR : "기준 사진"
    WORLD ||--o{ AVATAR : "거주"
    PET ||--o{ DAILY_MESSAGE : "보낸다"
    AVATAR ||--o{ DAILY_MESSAGE : "함께 나온 친구"
    PET ||--o{ LETTER : "받는다"
    LETTER ||--o| DAILY_MESSAGE : "답장"

    MEMBER {
        string email UK
        string password_hash
        string nickname
    }
    SPECIES {
        string code UK "DOG, CAT, HAMSTER"
        string name "강아지, 고양이"
    }
    PET {
        bigint member_id FK
        bigint species_id FK
        string name
        string breed
        date birth_date
        date adopted_date
        date passed_date "null이면 함께 지내는 중"
        string status "TOGETHER, PASSED"
        boolean message_enabled "하루 메시지 수신"
        date message_paused_until "쉬어가기"
    }
    PHOTO {
        bigint pet_id FK
        string file_key "저장소 경로"
        date taken_date "타임라인 정렬 기준"
        string caption
    }
    PET_TRAIT {
        bigint pet_id FK
        string category "PERSONALITY, HABIT, FAVORITE, SPEECH"
        string content
    }
    WORLD {
        bigint species_id FK
        string name "강아지 월드"
        string background_key
    }
    AVATAR {
        bigint pet_id FK
        bigint world_id FK
        bigint source_photo_id FK
        string image_key "고정 저장된 픽셀 아바타"
        string status "PENDING, READY, FAILED"
        boolean allow_appearance "다른 사람 사진에 등장 동의"
    }
    DAILY_MESSAGE {
        bigint pet_id FK
        date message_date "pet_id와 함께 UK"
        string activity "PLAY, SLEEP, EAT, WALK"
        bigint friend_avatar_id FK "혼자면 null"
        bigint reply_to_letter_id FK "편지 답장이면"
        string content
        string image_key
        timestamp read_at
    }
    LETTER {
        bigint pet_id FK
        bigint member_id FK
        string content
    }
```

### 설계 메모

- **PET.status**: "이별 준비"와 "이별 후"를 한 테이블에서 구분한다. 화면과 메시지 말투가 이 값으로 갈린다.
- **SPECIES와 WORLD 분리**: 지금은 1:1이지만 계절 이벤트 월드(F17)가 생기면 한 종에 월드가 여러 개가 된다. 그때 `WORLD`에 기간 컬럼만 더하면 된다.
- **AVATAR는 PET당 하나**: 다시 만들면 기존 행의 이미지를 교체한다. 이력이 필요해지면 그때 분리한다.
- **DAILY_MESSAGE (pet_id, message_date) 유니크**: 스케줄러가 두 번 돌아도 하루 한 통만 생긴다.
- **이미지는 DB에 넣지 않는다**: 파일 저장소 경로(`*_key`)만 저장한다.

## 이후 단계

```mermaid
erDiagram
    PET ||--o{ CHAT_MESSAGE : "대화"
    AVATAR ||--o{ AVATAR_OUTFIT : "보유·착용"
    OUTFIT ||--o{ AVATAR_OUTFIT : ""
    PET ||--o{ BUCKET_ITEM : "버킷리스트"
    PET ||--o{ MEDIA_CLIP : "목소리·영상"
    PET ||--o{ ANNIVERSARY : "기념일"
    AVATAR ||--o{ FLOWER : "받은 꽃"
    MEMBER ||--o{ FLOWER : "놓은 사람"
    PET ||--o{ PET_GUARDIAN : "가족 공유"
    MEMBER ||--o{ PET_GUARDIAN : ""
    AVATAR ||--o{ FRIENDSHIP : "단짝"

    CHAT_MESSAGE {
        bigint pet_id FK
        bigint member_id FK
        string role "USER, PET"
        string content
    }
    OUTFIT {
        string name
        string slot "HEAD, BODY, NECK"
        string image_key
        date available_from "한정 의상"
        date available_until
    }
    AVATAR_OUTFIT {
        bigint avatar_id FK
        bigint outfit_id FK
        boolean equipped
    }
    BUCKET_ITEM {
        bigint pet_id FK
        string title
        date done_date
        bigint photo_id FK
    }
    MEDIA_CLIP {
        bigint pet_id FK
        string type "AUDIO, VIDEO"
        string file_key
    }
    ANNIVERSARY {
        bigint pet_id FK
        string type "BIRTHDAY, ADOPTED, PASSED, CUSTOM"
        date date
    }
    FLOWER {
        bigint avatar_id FK
        bigint member_id FK
    }
    PET_GUARDIAN {
        bigint pet_id FK
        bigint member_id FK
        string role "OWNER, FAMILY"
    }
    FRIENDSHIP {
        bigint avatar_id FK
        bigint friend_avatar_id FK
        int appear_count "함께 나온 횟수"
    }
```

가족 공유(F15)를 넣을 때 `PET.member_id`를 `PET_GUARDIAN`으로 옮긴다. MVP에서는 소유자 한 명으로 단순하게 둔다.
