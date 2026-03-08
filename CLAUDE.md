# Skills Repository

Claude Code 커스텀 스킬을 보관하고 배포하는 플러그인 마켓플레이스 저장소입니다.

## 설치 방법

```
/plugin marketplace add kekmodel/skills
/plugin install python-dev@kekmodel-skills
```

## 새 스킬 추가

1. `skills/<skill-name>/SKILL.md` 작성 (참조 문서는 `references/`에)
2. `.claude-plugin/marketplace.json`의 해당 plugin에 경로 추가
3. 커밋 & 푸시
