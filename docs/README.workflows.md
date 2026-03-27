# ⚡ Agentic Workflow

[Agentic Workflow](https://github.github.com/gh-aw)는 GitHub Actions에서 coding agent를 실행하는 AI 기반 저장소 자동화입니다. 마크다운과 자연어 instruction으로 정의되며, 내장된 가드레일과 보안 우선 설계를 통해 이벤트 트리거 및 예약 자동화를 지원합니다.
### 기여 방법

새로운 workflow를 기여하거나, 기존 workflow를 개선하거나, 사용 사례를 공유하는 방법은 [CONTRIBUTING.md](../CONTRIBUTING.md#adding-agentic-workflows)를 참조하세요.

### Agentic Workflow 사용 방법

**포함 내용:**
- 각 workflow는 YAML frontmatter와 자연어 instruction이 포함된 단일 `.md` 파일입니다
- Workflow는 `gh aw compile`을 통해 `.lock.yml` GitHub Actions 파일로 컴파일됩니다
- Workflow는 [GitHub Agentic Workflows 사양](https://github.github.com/gh-aw)을 따릅니다

**설치 방법:**
- `gh aw` CLI 확장을 설치하세요: `gh extension install github/gh-aw`
- Workflow `.md` 파일을 저장소의 `.github/workflows/` 디렉토리에 복사하세요
- `gh aw compile`로 컴파일하여 `.lock.yml` 파일을 생성하세요
- `.md`와 `.lock.yml` 파일을 모두 commit하세요

**활성화/사용 방법:**
- Workflow는 설정된 트리거(스케줄, 이벤트, 슬래시 명령)에 따라 자동으로 실행됩니다
- `gh aw run <workflow>`로 수동 실행을 트리거하세요
- `gh aw status`와 `gh aw logs`로 실행을 모니터링하세요

**사용 시기:**
- 이슈 분류 및 라벨링 자동화
- 일일 상태 보고서 생성
- 문서 자동 유지 관리
- 예약된 코드 품질 검사 실행
- 이슈 및 PR의 슬래시 명령에 응답
- 다단계 저장소 자동화 오케스트레이션

| 이름 | 설명 | 트리거 |
| ---- | ----------- | -------- |
| [Daily Issues Report](../workflows/daily-issues-report.md) | Generates a daily summary of open issues and recent activity as a GitHub issue | schedule |
| [OSPO Contributors Report](../workflows/ospo-contributors-report.md) | Monthly contributor activity metrics across an organization's repositories. | schedule, workflow_dispatch |
| [OSPO Organization Health Report](../workflows/ospo-org-health.md) | Comprehensive weekly health report for a GitHub organization. Surfaces stale issues/PRs, merge time analysis, contributor leaderboards, and actionable items needing human attention. | schedule, workflow_dispatch |
| [OSPO Stale Repository Report](../workflows/ospo-stale-repos.md) | Identifies inactive repositories in your organization and generates an archival recommendation report. | schedule, workflow_dispatch |
| [OSS Release Compliance Checker](../workflows/ospo-release-compliance-checker.md) | Analyzes a target repository against open source release requirements and posts a detailed compliance report as an issue comment. | issues, workflow_dispatch |
| [Relevance Check](../workflows/relevance-check.md) | Slash command to evaluate whether an issue or pull request is still relevant to the project | slash_command, roles |
| [Relevance Summary](../workflows/relevance-summary.md) | Manually triggered workflow that summarizes all open issues and PRs with a /relevance-check response into a single issue | workflow_dispatch |
