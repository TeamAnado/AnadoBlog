## 🪓 브랜치 전략 (Branch Strategy)

### Git Flow Vs. Github Flow
#### Git Flow
- 오랫동안 생존하는 복잡한 가지치기 모델
- 주요 브랜치로 master, develop 사용
- 서브 브랜치로 feature/\*, release/\*, hotfix/\* 사용
- 정기적으로 배포하는 복잡한 프로덕트에 적합한 방식

#### Github Flow
- 하나의 메인 브랜치를 활용하는 간단한 브랜칭 모델
- 주요 브랜치로 언제든 deploy 가능한 단일 main 사용
- 서브 브랜치로 짧은 라이프타임을 가지는 feature 브랜치 사용
- 짧은 간격으로 지속적인 배포가 이루어지는 프로덕트에 적합한 방식

#### 그래서 우리의 선택은...
- 이 프로젝트는 기술 블로그 작성을 통해 지식을 정제하여 구성원들의 역량을 끌어올리고자 하는 목적입니다.
- 따라서 작성된 아티클을 빠르게 공유하고 손쉽게 교정하기 위해 Github Flow 전략이 가장 적당하다고 판단했습니다.

```markdown
- main: 최종 결과물(리뷰까지 완료된 포스트)만 존재하는 가장 안정적인 브랜치입니다. 보호된 브랜치 기능을 통해 직접 커밋이 금지되며, 과반수의 리뷰를 통해 승인된 pull request를 통해서만 병합됩니다.
- feat/week{number}/{name}: 각자의 글을 작성하는 브랜치입니다. 자유롭게 커밋하며 작업해도 좋습니다.
```

## 💬 커밋 컨벤션 (Commit Convention)

커밋 메시지만 봐도 어떤 작업을 했는지 알 수 있도록 팀 규칙을 정했습니다.

```shell
- feat: 새로운 글 작성 
    ex) JVM 데이터 영역 포스팅 초안 작성
- fix: 내용 오류 수정 
    ex) '메소드 영역'을 '메타스페이스'로 용어 수정
- style: 코드 형식 변경
    ex) 마크다운 문법 정리 및 가독성 개선 
- docs: 문서 수정 
    ex) 스터디 규칙에 PR 템플릿 사용법 추가
- refactor: 파일 구조 변경
    ex) 주차별 폴더 구조 변경 
```
## 👥 팀 문화 (Team Culture)

- PR 템플릿에 맞게 작성하기
- 코드 리뷰할 때 말투는
    - 개인적인 견해보다는 객관적인 견해를 포함해주세요.
    - `-요`체 사용하고, 최소 1개 이모지를 포함해주세요.

## 👋 온보딩 플로우 (Onboarding Flow)

### 1. 프로젝트 클론

```bash
$ git clone ~.git
```

### 2. 문서 추가
#### `feat/[주차]/[이름]`으로 브랜치를 생성해주세요.
```bash
$ git checkout -b feat/weekX/이름(닉네임)
```

#### 생성한 브랜치를 최신화 해주세요.
```bash
$ git pull origin main 
```

#### 주차별 폴더에 블로그에 올릴 주제를 `{name}.md` 파일로 작성해주세요.

### 3. Github Pull Request
위에서부터 차례대로 진행해주세요.

```bash
$ git add {name}.md
$ git commit -m "feat: ~~"
$ git push origin feat/week{number}/{이름}
```

## 📁 프로젝트 구조 (Project Structure)

```markdown
root/
└── week{number}/
    └── 📄 {name}.md
```