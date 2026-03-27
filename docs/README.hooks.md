# 🪝 Hook

Hook은 GitHub Copilot coding agent 세션 중 특정 이벤트(세션 시작, 세션 종료, 사용자 프롬프트, 도구 사용 등)에 의해 트리거되는 자동화된 워크플로우를 지원합니다.
### 기여 방법

새로운 hook을 기여하거나, 기존 hook을 개선하거나, 사용 사례를 공유하는 방법은 [CONTRIBUTING.md](../CONTRIBUTING.md#adding-hooks)를 참조하세요.

### Hook 사용 방법

**포함 내용:**
- 각 hook은 `README.md` 파일과 `hooks.json` 설정이 포함된 폴더입니다
- Hook에는 헬퍼 스크립트, 유틸리티 또는 기타 번들 에셋이 포함될 수 있습니다
- Hook은 [GitHub Copilot hooks 사양](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/use-hooks)을 따릅니다

**설치 방법:**
- Hook 폴더를 저장소의 `.github/hooks/` 디렉토리에 복사하세요
- 번들 스크립트가 실행 가능한지 확인하세요 (`chmod +x script.sh`)
- Hook을 저장소의 기본 브랜치에 commit하세요

**활성화/사용 방법:**
- Hook은 Copilot coding agent 세션 중 자동으로 실행됩니다
- `hooks.json` 파일에서 hook 이벤트를 설정하세요
- 사용 가능한 이벤트: `sessionStart`, `sessionEnd`, `userPromptSubmitted`, `preToolUse`, `postToolUse`, `errorOccurred`

**사용 시기:**
- 세션 로깅 및 감사 추적 자동화
- 세션 종료 시 변경 사항 자동 commit
- 사용량 분석 추적
- 외부 도구 및 서비스와 통합
- 커스텀 세션 워크플로우

| 이름 | 설명 | 이벤트 | 번들 에셋 |
| ---- | ----------- | ------ | -------------- |
| [Dependency License Checker](../hooks/dependency-license-checker/README.md) | Scans newly added dependencies for license compliance (GPL, AGPL, etc.) at session end | sessionEnd | `check-licenses.sh`<br />`hooks.json` |
| [Governance Audit](../hooks/governance-audit/README.md) | Scans Copilot agent prompts for threat signals and logs governance events | sessionStart, sessionEnd, userPromptSubmitted | `audit-prompt.sh`<br />`audit-session-end.sh`<br />`audit-session-start.sh`<br />`hooks.json` |
| [Secrets Scanner](../hooks/secrets-scanner/README.md) | Scans files modified during a Copilot coding agent session for leaked secrets, credentials, and sensitive data | sessionEnd | `hooks.json`<br />`scan-secrets.sh` |
| [Session Auto-Commit](../hooks/session-auto-commit/README.md) | Automatically commits and pushes changes when a Copilot coding agent session ends | sessionEnd | `auto-commit.sh`<br />`hooks.json` |
| [Session Logger](../hooks/session-logger/README.md) | Logs all Copilot coding agent session activity for audit and analysis | sessionStart, sessionEnd, userPromptSubmitted | `hooks.json`<br />`log-prompt.sh`<br />`log-session-end.sh`<br />`log-session-start.sh` |
| [Tool Guardian](../hooks/tool-guardian/README.md) | Blocks dangerous tool operations (destructive file ops, force pushes, DB drops) before the Copilot coding agent executes them | preToolUse | `guard-tool.sh`<br />`hooks.json` |
