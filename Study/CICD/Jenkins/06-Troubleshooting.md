---
title: Jenkins Troubleshooting
description: Jenkins GitHub GitOps 자동화 문제 확인
published: true
tags: jenkins,troubleshooting,github,gitops
editor: markdown
---

# Jenkins Troubleshooting

## GitOps PR이 자동 Merge되지 않음

### 확인 순서

1. Jenkins Credential 확인
2. GitHub App 설치 대상 Repository 확인
3. GitHub App Repository 권한 확인
4. Branch Ruleset Bypass 확인
5. PR 생성 성공 여부 확인
6. Auto Merge 요청 성공 여부 확인
7. PR 작업 주체 확인

<br>

## 사용자 PAT 사용 문제

기존 Credential이 사용자 PAT를 사용하면 GitHub App에 설정한 Ruleset Bypass가 적용되지 않는다.

```text
Jenkins
  ↓
gitops-repo-push
  ↓
User PAT
  ↓
GitHub에서 사용자 작업으로 인식
  ↓
GitHub App Bypass 적용 X
```

GitHub App Credential로 변경한다.

```text
credentialsId: moongcheap-jenkins-ci
```

<br>

## GitHub App을 설치했는데 자동 Merge되지 않음

GitHub App 설치만으로 Auto Merge가 실행되는 것은 아니다.

필요한 구성은 세 가지이다.

```text
1. Jenkins가 GitHub App Credential 사용
2. GitHub App이 Ruleset Bypass 대상
3. Jenkins가 PR 생성 후 Auto Merge 요청
```

세 조건을 각각 확인한다.

<br>

## Private Key

GitHub App 설정 화면의 `SHA256:...` 값은 Private Key가 아니다.

Jenkins에는 GitHub App에서 발급한 `.pem` 파일의 실제 Private Key를 등록한다.

Private Key는 노출하거나 Repository에 Commit하지 않는다.

<br>

## Usage가 비어 있음

Jenkins Credential을 생성한 직후 `Usage`가 비어 있을 수 있다.

Jenkinsfile에서 해당 Credential ID를 실제로 사용한 Pipeline이 실행된 뒤 사용 기록을 확인한다.

<br>

## 최종 확인

정상 동작 시 다음 흐름을 확인한다.

```text
Jenkins 실행
  ↓
GitOps 임시 Branch 생성
  ↓
GitHub App 인증으로 Push
  ↓
PR 생성
  ↓
Auto Merge 활성화
  ↓
Ruleset Bypass
  ↓
Merge
  ↓
임시 Branch 삭제
  ↓
ArgoCD Sync
```
