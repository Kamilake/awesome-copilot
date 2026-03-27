# AGENTS.md

## 프로젝트 개요

Awesome GitHub Copilot 저장소는 다양한 도메인, 언어 및 사용 사례에 걸쳐 GitHub Copilot 경험을 향상시키기 위해 설계된 커스텀 에이전트와 인스트럭션의 커뮤니티 주도 컬렉션입니다. 이 프로젝트에는 다음이 포함됩니다:

- **Agents** - MCP 서버와 통합되는 특화된 GitHub Copilot 에이전트
- **Instructions** - 특정 파일 패턴에 적용되는 코딩 표준 및 모범 사례
- **Skills** - 특화된 작업을 위한 인스트럭션과 번들 리소스가 포함된 독립형 폴더
- **Hooks** - 개발 중 특정 이벤트에 의해 트리거되는 자동화된 워크플로우
- **Workflows** - GitHub Actions에서 AI 기반 저장소 자동화를 위한 [Agentic Workflows](https://github.github.com/gh-aw)
- **Plugins** - 관련 에이전트, 명령 및 스킬을 특정 테마별로 그룹화한 설치 가능한 패키지

## 저장소 구조

```
.
├── agents/           # 커스텀 GitHub Copilot 에이전트 정의 (.agent.md 파일)
├── instructions/     # 코딩 표준 및 가이드라인 (.instructions.md 파일)
├── skills/           # Agent Skills 폴더 (각각 SKILL.md 및 선택적 번들 에셋 포함)
├── hooks/            # 자동화된 워크플로우 훅 (README.md + hooks.json이 포함된 폴더)
├── workflows/        # Agentic Workflows (GitHub Actions 자동화를 위한 .md 파일)
├── plugins/          # 설치 가능한 플러그인 패키지 (plugin.json이 포함된 폴더)
├── docs/             # 다양한 리소스 유형에 대한 문서
├── eng/              # 빌드 및 자동화 스크립트
└── scripts/          # 유틸리티 스크립트
```

## 설정 명령어

```bash
# 의존성 설치
npm ci

# 프로젝트 빌드 (README.md 및 marketplace.json 생성)
npm run build

# 플러그인 매니페스트 검증
npm run plugin:validate

# marketplace.json만 생성
npm run plugin:generate-marketplace

# 새 플러그인 생성
npm run plugin:create -- --name <plugin-name>

# Agent Skills 검증
npm run skill:validate

# 새 스킬 생성
npm run skill:create -- --name <skill-name>
```

## 개발 워크플로우

### Agents, Instructions, Skills 및 Hooks 작업

모든 에이전트 파일 (`*.agent.md`)과 인스트럭션 파일 (`*.instructions.md`)에는 적절한 마크다운 front matter가 포함되어야 합니다. Agent Skills는 frontmatter가 포함된 `SKILL.md` 파일과 선택적 번들 에셋이 있는 폴더입니다. Hooks는 frontmatter가 포함된 `README.md`와 `hooks.json` 설정 파일이 있는 폴더입니다:

#### Agent 파일 (*.agent.md)
- `description` 필드가 있어야 합니다 (작은따옴표로 감싸기)
- 파일 이름은 소문자로, 단어는 하이픈으로 구분
- `tools` 필드 포함 권장
- `model` 필드 지정 강력 권장

#### Instruction 파일 (*.instructions.md)
- `description` 필드가 있어야 합니다 (작은따옴표로 감싸기, 비어있지 않아야 함)
- 파일 패턴을 지정하는 `applyTo` 필드가 있어야 합니다 (예: `'**.js, **.ts'`)
- 파일 이름은 소문자로, 단어는 하이픈으로 구분

#### Agent Skills (skills/*/SKILL.md)
- 각 스킬은 `SKILL.md` 파일이 포함된 폴더입니다
- SKILL.md에는 `name` 필드가 있어야 합니다 (소문자 하이픈 구분, 폴더 이름과 일치, 최대 64자)
- SKILL.md에는 `description` 필드가 있어야 합니다 (작은따옴표로 감싸기, 10-1024자)
- 폴더 이름은 소문자로, 단어는 하이픈으로 구분
- 스킬에는 번들 에셋(스크립트, 템플릿, 데이터 파일)을 포함할 수 있습니다
- 번들 에셋은 SKILL.md 인스트럭션에서 참조되어야 합니다
- 에셋 파일은 적절한 크기여야 합니다 (파일당 5MB 미만)
- 스킬은 [Agent Skills 사양](https://agentskills.io/specification)을 따릅니다

#### Hook 폴더 (hooks/*/README.md)
- 각 훅은 frontmatter가 포함된 `README.md` 파일이 있는 폴더입니다
- README.md에는 `name` 필드가 있어야 합니다 (사람이 읽을 수 있는 이름)
- README.md에는 `description` 필드가 있어야 합니다 (작은따옴표로 감싸기, 비어있지 않아야 함)
- 훅 설정이 포함된 `hooks.json` 파일이 있어야 합니다 (이 파일에서 훅 이벤트 추출)
- 폴더 이름은 소문자로, 단어는 하이픈으로 구분
- 번들 에셋(스크립트, 유틸리티, 설정 파일)을 포함할 수 있습니다
- 번들 스크립트는 README.md와 hooks.json에서 참조되어야 합니다
- [GitHub Copilot hooks 사양](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/use-hooks)을 따릅니다
- 선택적으로 분류를 위한 `tags` 필드를 포함합니다

#### Workflow 파일 (workflows/*.md)
- 각 워크플로우는 `workflows/` 디렉토리의 독립형 `.md` 파일입니다
- `name` 필드가 있어야 합니다 (사람이 읽을 수 있는 이름)
- `description` 필드가 있어야 합니다 (작은따옴표로 감싸기, 비어있지 않아야 함)
- Agentic workflow frontmatter (`on`, `permissions`, `safe-outputs`)와 자연어 인스트럭션을 포함합니다
- 파일 이름은 소문자로, 단어는 하이픈으로 구분
- `.md` 파일만 허용됩니다 — `.yml`, `.yaml`, `.lock.yml` 파일은 CI에서 차단됩니다
- [GitHub Agentic Workflows 사양](https://github.github.com/gh-aw/reference/workflow-structure/)을 따릅니다

#### Plugin 폴더 (plugins/*)
- 각 플러그인은 메타데이터가 포함된 `.github/plugin/plugin.json` 파일이 있는 폴더입니다
- plugin.json에는 `name` 필드가 있어야 합니다 (폴더 이름과 일치)
- plugin.json에는 `description` 필드가 있어야 합니다 (플러그인의 목적 설명)
- plugin.json에는 `version` 필드가 있어야 합니다 (시맨틱 버전, 예: "1.0.0")
- 플러그인 콘텐츠는 Claude Code 사양 필드 (`agents`, `commands`, `skills`)를 사용하여 plugin.json에 선언적으로 정의됩니다. 소스 파일은 최상위 디렉토리에 있으며 CI에 의해 플러그인으로 구체화됩니다.
- `marketplace.json` 파일은 빌드 시 모든 플러그인에서 자동으로 생성됩니다
- 플러그인은 GitHub Copilot CLI를 통해 검색 및 설치할 수 있습니다

### 새 리소스 추가

새 에이전트, 인스트럭션, 스킬, 훅, 워크플로우 또는 플러그인을 추가할 때:

**Agents 및 Instructions의 경우:**
1. 적절한 front matter가 포함된 파일을 생성합니다
2. 해당 디렉토리에 파일을 추가합니다
3. `npm run build`를 실행하여 README.md를 업데이트합니다
4. 생성된 README에 리소스가 표시되는지 확인합니다

**Hooks의 경우:**
1. `hooks/`에 설명적인 이름으로 새 폴더를 생성합니다
2. 적절한 frontmatter (name, description, hooks, tags)가 포함된 `README.md`를 생성합니다
3. GitHub Copilot hooks 사양에 따라 훅 설정이 포함된 `hooks.json`을 생성합니다
4. 폴더에 번들 스크립트나 에셋을 추가합니다
5. 스크립트를 실행 가능하게 만듭니다: `chmod +x script.sh`
6. `npm run build`를 실행하여 README.md를 업데이트합니다
7. 생성된 README에 훅이 표시되는지 확인합니다


**Workflows의 경우:**
1. `workflows/`에 설명적인 이름으로 새 `.md` 파일을 생성합니다 (예: `daily-issues-report.md`)
2. `name`과 `description`, 그리고 agentic workflow 필드 (`on`, `permissions`, `safe-outputs`)가 포함된 frontmatter를 포함합니다
3. `gh aw compile --validate`로 컴파일하여 유효한지 확인합니다
4. `npm run build`를 실행하여 README.md를 업데이트합니다
5. 생성된 README에 워크플로우가 표시되는지 확인합니다


**Skills의 경우:**
1. `npm run skill:create`를 실행하여 새 스킬 폴더를 스캐폴딩합니다
2. 생성된 SKILL.md 파일을 인스트럭션으로 편집합니다
3. 스킬 폴더에 번들 에셋(스크립트, 템플릿, 데이터)을 추가합니다
4. `npm run skill:validate`를 실행하여 스킬 구조를 검증합니다
5. `npm run build`를 실행하여 README.md를 업데이트합니다
6. 생성된 README에 스킬이 표시되는지 확인합니다

**Plugins의 경우:**
1. `npm run plugin:create -- --name <plugin-name>`을 실행하여 새 플러그인을 스캐폴딩합니다
2. Claude Code 사양 필드를 사용하여 `plugin.json`에 에이전트, 명령 및 스킬을 정의합니다
3. 생성된 `plugin.json`을 메타데이터로 편집합니다
4. `npm run plugin:validate`를 실행하여 플러그인 구조를 검증합니다
5. `npm run build`를 실행하여 README.md와 marketplace.json을 업데이트합니다
6. `.github/plugin/marketplace.json`에 플러그인이 표시되는지 확인합니다

**외부 Plugins의 경우:**
1. `plugins/external.json`을 편집하고 `name`, `source`, `description`, `version`이 포함된 항목을 추가합니다
2. `source` 필드는 GitHub 저장소, git URL, npm 패키지 또는 pip 패키지를 지정하는 객체여야 합니다 ([CONTRIBUTING.md](CONTRIBUTING.md#adding-external-plugins) 참조)
3. `npm run build`를 실행하여 marketplace.json을 재생성합니다
4. `.github/plugin/marketplace.json`에 외부 플러그인이 표시되는지 확인합니다

### 테스트 지침

```bash
# 모든 검증 확인 실행
npm run plugin:validate
npm run skill:validate

# README 생성 빌드 및 확인
npm run build

# 줄 끝 수정 (commit 전 필수)
bash scripts/fix-line-endings.sh
```

commit 전:
- 모든 마크다운 front matter가 올바르게 포맷되었는지 확인합니다
- 파일 이름이 소문자-하이픈 규칙을 따르는지 확인합니다
- `npm run build`를 실행하여 README를 업데이트합니다
- **항상 `bash scripts/fix-line-endings.sh`를 실행**하여 줄 끝을 정규화합니다 (CRLF → LF)
- 새 리소스가 README에 올바르게 표시되는지 확인합니다

## 코드 스타일 가이드라인

### 마크다운 파일
- 필수 필드가 포함된 적절한 front matter를 사용합니다
- 설명은 간결하고 유익하게 작성합니다
- description 필드 값은 작은따옴표로 감쌉니다
- 소문자 파일 이름에 하이픈을 구분자로 사용합니다

### JavaScript/Node.js 스크립트
- `eng/` 및 `scripts/` 디렉토리에 위치합니다
- Node.js ES 모듈 규칙을 따릅니다 (`.mjs` 확장자)
- 명확하고 설명적인 함수 및 변수 이름을 사용합니다

## Pull Request 가이드라인

Pull request를 생성할 때:

> **중요:** 모든 pull request는 `main`이 아닌 **`staged`** 브랜치를 대상으로 해야 합니다.

1. **README 업데이트**: `npm run build`를 실행하면 새 파일이 자동으로 README에 추가됩니다
2. **Front matter 검증**: 모든 마크다운 파일에 필수 front matter 필드가 있는지 확인합니다
3. **파일 명명**: 모든 새 파일이 소문자-하이픈 명명 규칙을 따르는지 확인합니다
4. **빌드 확인**: commit 전에 `npm run build`를 실행하여 README 생성을 확인합니다
5. **줄 끝**: **항상 `bash scripts/fix-line-endings.sh`를 실행**하여 줄 끝을 LF (Unix 스타일)로 정규화합니다
6. **설명**: 에이전트/인스트럭션이 무엇을 하는지 명확한 설명을 제공합니다
7. **테스트**: 플러그인을 추가하는 경우 `npm run plugin:validate`를 실행하여 유효성을 확인합니다

### Pre-commit 체크리스트

PR을 제출하기 전에 다음을 확인하십시오:
- [ ] `npm install` (또는 `npm ci`)을 실행하여 의존성을 설치했습니다
- [ ] `npm run build`를 실행하여 업데이트된 README.md를 생성했습니다
- [ ] `bash scripts/fix-line-endings.sh`를 실행하여 줄 끝을 정규화했습니다
- [ ] 모든 새 파일에 적절한 front matter가 있는지 확인했습니다
- [ ] 기여한 내용이 GitHub Copilot에서 작동하는지 테스트했습니다
- [ ] 파일 이름이 명명 규칙을 따르는지 확인했습니다

### 코드 리뷰 체크리스트

Instruction 파일 (*.instructions.md)의 경우:
- [ ] 마크다운 front matter가 있습니다
- [ ] 작은따옴표로 감싼 비어있지 않은 `description` 필드가 있습니다
- [ ] 파일 패턴이 포함된 `applyTo` 필드가 있습니다
- [ ] 파일 이름이 소문자 하이픈 형식입니다

Agent 파일 (*.agent.md)의 경우:
- [ ] 마크다운 front matter가 있습니다
- [ ] 작은따옴표로 감싼 비어있지 않은 `description` 필드가 있습니다
- [ ] 사람이 읽을 수 있는 이름의 `name` 필드가 있습니다 (예: "Address Comments"이지 "address-comments"가 아님)
- [ ] 파일 이름이 소문자 하이픈 형식입니다
- [ ] `model` 필드가 포함되어 있습니다 (강력 권장)
- [ ] `tools` 필드 사용을 고려합니다

Skills (skills/*/)의 경우:
- [ ] 폴더에 SKILL.md 파일이 포함되어 있습니다
- [ ] SKILL.md에 마크다운 front matter가 있습니다
- [ ] 폴더 이름과 일치하는 `name` 필드가 있습니다 (소문자 하이픈 형식, 최대 64자)
- [ ] 작은따옴표로 감싼 비어있지 않은 `description` 필드가 있습니다 (10-1024자)
- [ ] 폴더 이름이 소문자 하이픈 형식입니다
- [ ] 모든 번들 에셋이 SKILL.md에서 참조됩니다
- [ ] 번들 에셋이 파일당 5MB 미만입니다

Hook 폴더 (hooks/*/)의 경우:
- [ ] 폴더에 마크다운 front matter가 포함된 README.md 파일이 있습니다
- [ ] 사람이 읽을 수 있는 이름의 `name` 필드가 있습니다
- [ ] 작은따옴표로 감싼 비어있지 않은 `description` 필드가 있습니다
- [ ] 유효한 훅 설정이 포함된 `hooks.json` 파일이 있습니다 (이 파일에서 훅 이벤트 추출)
- [ ] 폴더 이름이 소문자 하이픈 형식입니다
- [ ] 모든 번들 스크립트가 실행 가능하고 README.md에서 참조됩니다
- [ ] [GitHub Copilot hooks 사양](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/use-hooks)을 따릅니다
- [ ] 선택적으로 분류를 위한 `tags` 배열 필드를 포함합니다

Workflow 파일 (workflows/*.md)의 경우:
- [ ] 파일에 마크다운 front matter가 있습니다
- [ ] 사람이 읽을 수 있는 이름의 `name` 필드가 있습니다
- [ ] 작은따옴표로 감싼 비어있지 않은 `description` 필드가 있습니다
- [ ] 파일 이름이 소문자 하이픈 형식입니다
- [ ] frontmatter에 `on`과 `permissions`가 포함되어 있습니다
- [ ] 워크플로우가 최소 권한 원칙과 safe outputs를 사용합니다
- [ ] `.yml`, `.yaml`, `.lock.yml` 파일이 포함되지 않았습니다
- [ ] [GitHub Agentic Workflows 사양](https://github.github.com/gh-aw/reference/workflow-structure/)을 따릅니다

Plugins (plugins/*/)의 경우:
- [ ] 디렉토리에 `.github/plugin/plugin.json` 파일이 포함되어 있습니다
- [ ] 디렉토리에 `README.md` 파일이 포함되어 있습니다
- [ ] `plugin.json`에 디렉토리 이름과 일치하는 `name` 필드가 있습니다 (소문자 하이픈 형식)
- [ ] `plugin.json`에 비어있지 않은 `description` 필드가 있습니다
- [ ] `plugin.json`에 `version` 필드가 있습니다 (시맨틱 버전, 예: "1.0.0")
- [ ] 디렉토리 이름이 소문자 하이픈 형식입니다
- [ ] `keywords`가 있는 경우 소문자 하이픈 문자열의 배열입니다
- [ ] `agents`, `commands`, `skills` 배열이 있는 경우 각 항목이 유효한 상대 경로입니다
- [ ] 플러그인이 존재하지 않는 파일을 참조하지 않습니다
- [ ] `npm run build`를 실행하여 marketplace.json이 올바르게 업데이트되었는지 확인합니다

## 기여

이것은 커뮤니티 주도 프로젝트입니다. 기여를 환영합니다! 다음을 참조하십시오:
- [CONTRIBUTING.md](CONTRIBUTING.md) - 기여 가이드라인
- [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) - 커뮤니티 표준
- [SECURITY.md](SECURITY.md) - 보안 정책

## MCP 서버

이 저장소에는 이 저장소에서 직접 리소스를 검색하고 설치하기 위한 MCP (Model Context Protocol) 서버가 포함되어 있습니다. 서버를 실행하려면 Docker가 필요합니다.

## 라이선스

MIT License - 자세한 내용은 [LICENSE](LICENSE)를 참조하십시오.
