# CLAUDE.md

이 문서는 Claude Code가 본 프로젝트(pawse)에서 작업할 때 따라야 할 규칙을 정의한다.

## 0. 작업 시작 전 모델 / Effort 제안 규칙 (필수)

- 사용자가 작업을 지시하면 곧바로 실행하지 않고, 먼저 작업에 적합한 모델과 Effort를 제안한다.
  - 모델: `Opus 5`(복잡한 설계/아키텍처 변경/난이도 높은 디버깅) 또는 `Sonnet 5`(일반 기능 구현, 단순 수정, 반복 작업) 중 하나를 추천하고 이유를 간단히 밝힌다.
  - Effort: 작업 범위에 맞는 수준(low / medium / high)을 함께 제안한다.
- 사용자의 동의를 받은 뒤에만 모델/Effort 설정을 변경한다. 동의 없이 임의로 바꾸지 않는다.
- 모델/Effort가 확정되어도 "시작하자"라는 명시적 입력이 있을 때만 실제 작업(코드 수정, 브랜치 생성 등)을 시작한다. 제안과 동의만으로는 시작하지 않는다.
- 사용자가 곧바로 "시작하자"라고 말해도 제안 단계를 생략하지 않는다. 제안 → 동의 확인 → 시작 순서를 지킨다.

```
사용자: "로그인 API 만들어줘"
Claude: "Sonnet 5 + medium effort를 제안합니다. 진행할까요?"
사용자: "좋아"
Claude: "설정을 변경했습니다. '시작하자'라고 말씀하시면 시작하겠습니다."
사용자: "시작하자"
Claude: (작업 시작)
```

### 0-1. Commit / PR은 별도 지시가 있을 때만 실행

- `git commit`과 Pull Request 생성은 사용자가 명시적으로 요청하기 전까지 수행하지 않는다.
- "시작하자"만으로 커밋/PR까지 자동 진행하지 않는다. 코드/문서 변경을 마치면 결과만 보고하고 커밋 여부를 확인받는다.
- "커밋해줘", "PR 올려줘"처럼 명시 지시가 있을 때만 아래 Commit/PR 규칙에 따라 실행한다.

## 1. GitHub Branch 규칙

- 형식: `feature/{작업내용}`, `fix/{작업내용}`, `refactor/{작업내용}`, `setup/{작업내용}`(저장소 공통 기초 설정)
- 예: `feature/login-api`, `fix/login-error`, `refactor/auth-service`, `setup/coderabbit-config`
- 작업 내용은 영어 소문자와 하이픈(-)을 사용한다.
- 작업 기준 Branch는 `develop`이다. 새 Branch는 최신 `develop`에서 생성한다.

```bash
git checkout develop
git pull origin develop
git checkout -b feature/login-api
```

- Pull Request의 base도 `develop`으로 한다. 저장소 기본 Branch는 `main`이므로 PR 생성 시 base가 `develop`인지 반드시 확인한다. `main`에는 `develop → main` PR로만 반영한다.
- 선행 작업의 결과물이 필요하면 그 PR이 `develop`에 Merge된 뒤에 분기한다.

## 2. Commit 규칙

- 형식: `영어카테고리 : 한글 설명 메시지`
- 예: `feat : 로그인 API 구현`, `fix : 로그인 예외 처리 수정`, `refactor : 인증 서비스 리팩터링`, `test : 로그인 테스트 추가`

| Type | 설명 |
| --- | --- |
| feat | 새로운 기능 |
| fix | 버그 수정 |
| refactor | 기능 변경 없는 코드 개선 |
| test | 테스트 추가/수정 |
| docs | 문서 수정 |
| chore | 설정 및 기타 |
| style | 코드 포맷팅/스타일 변경 |

- 카테고리는 영어, 설명은 한글로 작성한다.
- `Co-Authored-By` 등 AI 공동작성자 표기는 Commit 메시지와 PR 본문 어디에도 넣지 않는다.

## 3. Pull Request 규칙

- 제목 형식: `[작업 내용]` (예: `[로그인 API 구현]`). Conventional Commit 형식을 쓰려면 `feat: 작업 내용`.
- Description에는 최소한 아래 항목을 포함한다.

```markdown
## 작업 내용
- 구현한 항목 1
- 구현한 항목 2

## 의사결정 기록
- docs/decisions/{작업명}.md
- (생략 시) 생략 — 단순 작업

## 테스트
- [ ] 테스트 항목 1
- [ ] 테스트 항목 2

## 확인 사항
- API 정상 동작 확인
- 예외 처리 확인
```

## 4. 의사결정 기록 (Decision Log)

- "무엇을 구현했는가"뿐 아니라 "왜 이렇게 풀었는가"를 남긴다. 코드나 PR만 보고도 "왜 이렇게 구현했지?", "다른 방법은 검토 안 했나?", "이 제약 때문인가?"에 답할 수 있어야 한다.
- 작업 하나의 개발이 끝나면(PR 올리기 전) 작업자가 의사결정 기록 문서를 작성한다.
- 위치: `docs/decisions/{작업명}.md` (예: `docs/decisions/login-api.md`)
- 작성 기준: 선택의 여지가 거의 없는 단순 CRUD/반복 작업은 생략 가능. 설계·구조·라이브러리·자료구조·에러 처리 방식 등에서 대안을 검토했거나 제약 때문에 특정 방식을 택했다면 반드시 작성한다. 애매하면 작성한다.
- 보안: `docs/decisions/`는 공개 저장소에 포함될 수 있으므로 비밀값(API Key·비밀번호 등), 서버 접속 정보(IP·인스턴스 ID 등), 실제 사건 정보는 적지 않는다.
- 근거 문서가 이미 있으면 내용을 복사하지 말고 링크한다(내용이 여러 곳에서 어긋나는 것을 막기 위함).
- 한 작업이 여러 PR로 나뉘면 같은 파일에 이어서 쓰고 `## 2차 (PR #xxx)`처럼 PR 단위로 구분한다.
- 형식:

```markdown
# {작업명} 의사결정 기록

## 문제 / 배경

## 검토한 대안

## 선택한 방법과 이유

## 고려한 제약 사항

## 추후 재검토가 필요하면 (선택)
```

- 이 문서는 Commit/PR 생성 여부와 무관하게 구현이 실질적으로 끝난 시점에 바로 작성한다.

## 5. Code Review

검토자는 아래 항목을 확인한다.

- [ ] 요구사항 구현 여부
- [ ] 기존 기능 영향
- [ ] 예외 처리
- [ ] 코드 구조
- [ ] 네이밍
- [ ] 불필요한 코드
- [ ] 테스트
- [ ] 보안

수정 요청이 오면 새 PR을 만들지 않고 같은 Branch에서 추가 Commit으로 반영한 뒤 다시 Review를 요청한다.

```bash
git add .
git commit -m "fix : 리뷰 반영 - 예외 처리 보완"
git push origin feature/login-api
```

## 6. PR Merge

Merge 전 아래 항목을 확인한다.

- Code Review 완료
- CI/CD 통과
- 충돌 없음
- 테스트 통과
- 요구사항 충족

## 7. 전체 흐름 요약 (Cheat Sheet)

```
모델/Effort 제안
      ↓
사용자 동의
      ↓
"시작하자"
      ↓
Branch 생성 (develop에서 분기)
      ↓
개발
      ↓
Commit (명시 지시 시)
      ↓
의사결정 기록 작성 (docs/decisions/)
      ↓
PR 생성 (명시 지시 시, base: develop)
      ↓
Code Review
      ↓
PR Merge
```

## 8. 반드시 지킬 것

1. 모델/Effort를 먼저 제안하고 동의를 받은 뒤, "시작하자" 입력이 있어야 작업을 시작한다.
   1-1. Commit과 PR 생성은 "시작하자"와 별개로 사용자가 명시 지시할 때만 수행한다.
2. Branch 이름은 규칙(타입/영어 소문자-하이픈)을 따른다.
3. Commit 메시지는 `영어카테고리 : 한글 설명` 형식을 따른다.
4. PR은 `develop`을 base로 생성한다.
5. 리뷰 요청을 받으면 가능한 한 빠르게 확인한다.
6. 진행이 막히면 임의로 판단하지 말고 사용자에게 상황을 공유한다.
7. 대안을 검토했거나 제약 때문에 특정 방식을 택한 작업은 PR 전에 `docs/decisions/`에 의사결정 기록을 남긴다.
