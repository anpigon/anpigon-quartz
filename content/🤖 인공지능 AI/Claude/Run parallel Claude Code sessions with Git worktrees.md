---
title: 클로드 코드 인스턴스 간 완전한 코드 격리를 통한 동시 작업
aliases: []
related:
created: 2025-07-09 03:36:37
modified: 2025-07-09 03:43:20
share_link: https://share.note.sx/10b0hre0#ooVyetdWLSsS+Nswl7LK9QtD59kXk3mS7NQoL0+tjds
share_updated: 2025-07-09T15:43:25+09:00
summary: Git 워크트리를 활용하면 여러 클로드 코드 인스턴스를 완벽하게 격리된 환경에서 동시에 실행하여 병렬 작업을 효율적으로 수행할 수 있습니다.
---

> 원문: https://docs.anthropic.com/en/docs/claude-code/common-workflows#run-parallel-claude-code-sessions-with-git-worktrees


> [!TLDR]
> Git 워크트리를 활용하면 여러 클로드 코드 인스턴스를 완벽하게 격리된 환경에서 동시에 실행하여 병렬 작업을 효율적으로 수행할 수 있습니다.

# 클로드 코드 인스턴스 간 완전한 코드 격리를 통한 동시 작업

클로드 코드 인스턴스 간에 완전한 코드 격리를 유지하면서 여러 작업을 동시에 처리해야 한다고 가정해 봅시다.

## 1. Git 워크트리 이해하기

Git 워크트리는 동일한 저장소에서 여러 브랜치를 별도의 디렉터리로 체크아웃할 수 있도록 **허용합니다**.
각 워크트리는 격리된 파일을 가진 자체 작업 디렉터리를 **가지며**, 동시에 동일한 Git 히스토리를 **공유합니다**.
자세한 내용은 [공식 Git 워크트리 문서](https://git-scm.com/docs/git-worktree)에서 **확인할 수 있습니다**.

## 2. 새 워크트리 생성하기

```bash
# 새 브랜치로 새 워크트리 생성하기
git worktree add ../project-feature-a -b feature-a

# 또는 기존 브랜치로 워크트리 생성하기
git worktree add ../project-bugfix bugfix-123
```

이 명령은 저장소의 별도 작업 복사본을 가진 새 디렉터리를 **생성합니다**.

## 3. 각 워크트리에서 Claude Code 실행하기

```bash
# 워크트리로 이동하기
cd ../project-feature-a

# 이 격리된 환경에서 Claude Code 실행하기
claude
```

## 4. 다른 워크트리에서 Claude 실행하기

```bash
cd ../project-bugfix
claude
```

## 5. 워크트리 관리하기

```bash
# 모든 워크트리 목록 보기
git worktree list

# 작업 완료 시 워크트리 제거하기
git worktree remove ../project-feature-a
```



> [!tip]
> - 각 워크트리는 독립적인 파일 상태를 **가지므로**, 병렬 Claude Code 세션에 **완벽합니다**.
> - 한 워크트리에서 변경된 내용은 다른 워크트리에 **영향을 주지 않아**, Claude 인스턴스 간의 간섭을 **방지합니다**.
> - 모든 워크트리는 동일한 Git 히스토리와 원격 연결을 **공유합니다**.
> - 장기 실행 작업의 경우, Claude가 한 워크트리에서 작업하는 동안 다른 워크트리에서 개발을 **계속할 수 있습니다**.
> - 각 워크트리가 어떤 작업을 위한 것인지 쉽게 식별할 수 있도록 설명적인 디렉터리 이름을 **사용하세요**.
> - 새 워크트리마다 프로젝트 설정에 따라 개발 환경을 초기화해야 함을 **기억하세요**. 스택에 따라 다음을 **포함할 수 있습니다**:
>     - JavaScript 프로젝트: 의존성 설치 **실행** (`npm install`, `yarn`)
>     - Python 프로젝트: 가상 환경 **설정** 또는 패키지 관리자를 통한 **설치**
>     - 기타 언어: 프로젝트의 표준 설정 프로세스 **따르기**
