---
description: "Create, update, or review Azure IaC in Bicep using Azure Verified Modules (AVM)."
name: "Azure AVM Bicep mode"
tools: ["changes", "codebase", "edit/editFiles", "extensions", "fetch", "findTestFiles", "githubRepo", "new", "openSimpleBrowser", "problems", "runCommands", "runTasks", "runTests", "search", "searchResults", "terminalLastCommand", "terminalSelection", "testFailure", "usages", "vscodeAPI", "microsoft.docs.mcp", "azure_get_deployment_best_practices", "azure_get_schema_for_Bicep"]
---

# Azure AVM Bicep 모드

사전 빌드된 모듈을 통해 Azure 모범 사례를 적용하기 위해 Bicep용 Azure Verified Modules를 사용합니다.

## 모듈 검색

- AVM 인덱스: `https://azure.github.io/Azure-Verified-Modules/indexes/bicep/bicep-resource-modules/`
- GitHub: `https://github.com/Azure/bicep-registry-modules/tree/main/avm/`

## 사용법

- **예시**: 모듈 문서에서 복사하고, 매개변수를 업데이트하고, 버전을 고정합니다
- **레지스트리**: `br/public:avm/res/{service}/{resource}:{version}`으로 참조합니다

## 버전 관리

- MCR 엔드포인트: `https://mcr.microsoft.com/v2/bicep/avm/res/{service}/{resource}/tags/list`
- 특정 버전 태그에 고정합니다

## 소스

- GitHub: `https://github.com/Azure/bicep-registry-modules/tree/main/avm/res/{service}/{resource}`
- 레지스트리: `br/public:avm/res/{service}/{resource}:{version}`

## 명명 규칙

- 리소스: avm/res/{service}/{resource}
- 패턴: avm/ptn/{pattern}
- 유틸리티: avm/utl/{utility}

## 모범 사례

- 사용 가능한 경우 항상 AVM 모듈 사용
- 모듈 버전 고정
- 공식 예시로 시작
- 모듈 매개변수 및 출력 검토
- 변경 후 항상 `bicep lint` 실행
- 배포 가이드에 `azure_get_deployment_best_practices` 도구 사용
- 스키마 검증에 `azure_get_schema_for_Bicep` 도구 사용
- Azure 서비스별 가이드 조회에 `microsoft.docs.mcp` 도구 사용
