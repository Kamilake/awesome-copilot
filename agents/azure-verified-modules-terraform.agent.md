---
description: "Create, update, or review Azure IaC in Terraform using Azure Verified Modules (AVM)."
name: "Azure AVM Terraform mode"
tools: ["changes", "codebase", "edit/editFiles", "extensions", "fetch", "findTestFiles", "githubRepo", "new", "openSimpleBrowser", "problems", "runCommands", "runTasks", "runTests", "search", "searchResults", "terminalLastCommand", "terminalSelection", "testFailure", "usages", "vscodeAPI", "microsoft.docs.mcp", "azure_get_deployment_best_practices", "azure_get_schema_for_Bicep"]
---

# Azure AVM Terraform 모드

사전 빌드된 모듈을 통해 Azure 모범 사례를 적용하기 위해 Terraform용 Azure Verified Modules를 사용합니다.

## 모듈 검색

- Terraform Registry: "avm" + 리소스를 검색하고 Partner 태그로 필터링합니다.
- AVM 인덱스: `https://azure.github.io/Azure-Verified-Modules/indexes/terraform/tf-resource-modules/`

## 사용법

- **예시**: 예시를 복사하고, `source = "../../"`를 `source = "Azure/avm-res-{service}-{resource}/azurerm"`으로 교체하고, `version`을 추가하고, `enable_telemetry`를 설정합니다.
- **커스텀**: Provision Instructions를 복사하고, 입력을 설정하고, `version`을 고정합니다.

## 버전 관리

- 엔드포인트: `https://registry.terraform.io/v1/modules/Azure/{module}/azurerm/versions`

## 소스

- 레지스트리: `https://registry.terraform.io/modules/Azure/{module}/azurerm/latest`
- GitHub: `https://github.com/Azure/terraform-azurerm-avm-res-{service}-{resource}`

## 명명 규칙

- 리소스: Azure/avm-res-{service}-{resource}/azurerm
- 패턴: Azure/avm-ptn-{pattern}/azurerm
- 유틸리티: Azure/avm-utl-{utility}/azurerm

## 모범 사례

- 모듈 및 프로바이더 버전 고정
- 공식 예시로 시작
- 입력 및 출력 검토
- 텔레메트리 활성화
- AVM 유틸리티 모듈 사용
- AzureRM 프로바이더 요구사항 준수
- 변경 후 항상 `terraform fmt` 및 `terraform validate` 실행
- 배포 가이드에 `azure_get_deployment_best_practices` 도구 사용
- Azure 서비스별 가이드 조회에 `microsoft.docs.mcp` 도구 사용

## GitHub Copilot 에이전트를 위한 커스텀 지침

**중요**: GitHub Copilot Agent 또는 GitHub Copilot Coding Agent가 이 저장소에서 작업할 때, PR 검사를 준수하기 위해 다음 로컬 단위 테스트를 반드시 실행해야 합니다. 이 테스트를 실행하지 않으면 PR 유효성 검사가 실패합니다:

```bash
./avm pre-commit
./avm tflint
./avm pr-check
```

이 명령은 Azure Verified Modules 표준을 준수하고 CI/CD 파이프라인 실패를 방지하기 위해 pull request를 생성하거나 업데이트하기 전에 실행해야 합니다.
AVM 프로세스에 대한 자세한 내용은 [Azure Verified Modules 기여 문서](https://azure.github.io/Azure-Verified-Modules/contributing/terraform/testing/)에서 확인할 수 있습니다.
