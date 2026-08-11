# Git 협업 실습 - Fetch, Merge 및 충돌 해결

## 실습 개요

하나의 GitHub 원격 저장소를 두 개의 로컬 저장소에 Clone하여  
작업자 A와 작업자 B가 협업하는 상황을 재현한다.

이번 실습에서는 다음 내용을 확인한다.

- 각 로컬 저장소는 서로 독립적으로 동작한다.
- 다른 작업자가 Push한 내용은 내 로컬에 자동으로 반영되지 않는다.
- `git fetch`로 원격 변경을 확인하고, `git merge origin/main`으로 로컬 브랜치에 반영한다.
- 같은 파일의 같은 부분을 수정하면 Merge Conflict가 발생할 수 있다.
- 충돌이 발생하면 직접 내용을 수정한 뒤 `add → commit → push` 순서로 해결한다. fileciteturn0file0L456-L470

---

## 실습 흐름

```mermaid
sequenceDiagram
    autonumber

    participant A as 작업자 A 로컬
    participant GH as GitHub 원격 저장소
    participant B as 작업자 B 로컬

    Note over A,B: 같은 GitHub 저장소를 각각 독립된 로컬 저장소로 사용

    %% 1. 충돌 없는 협업
    A->>A: worker-a.md 생성
    A->>A: git add / commit
    A->>GH: git push

    Note over GH,B: B의 로컬에는 A의 변경이 아직 없음

    B->>GH: git fetch origin
    GH-->>B: origin/main 최신 정보 전달
    B->>B: git merge origin/main

    B->>B: worker-b.md 생성
    B->>B: git add / commit
    B->>GH: git push

    A->>GH: git fetch origin
    GH-->>A: origin/main 최신 정보 전달
    A->>A: git merge origin/main

    %% 2. 충돌 실습
    Note over A,B: 같은 README.md 문장을 서로 다르게 수정

    A->>A: README.md 수정
    A->>A: git add / commit
    A->>GH: git push

    Note over GH,B: B는 A의 변경을 아직 가져오지 않음

    B->>B: README.md 같은 문장 수정
    B->>B: git add / commit
    B->>GH: git push

    GH-->>B: Push 거절<br/>fetch first

    %% 3. 충돌 해결
    B->>GH: git fetch origin
    GH-->>B: A의 최신 커밋 전달

    B->>B: git merge origin/main
    Note over B: README.md Merge Conflict 발생

    B->>B: 충돌 내용 확인
    B->>B: 최종 내용 결정 및 충돌 표시 제거
    B->>B: git add README.md
    B->>B: git commit

    Note over B: Merge Commit 생성

    B->>GH: git push

    %% 4. 최종 동기화
    A->>GH: git fetch origin
    GH-->>A: Merge Commit 전달
    A->>A: git merge origin/main

    Note over A,B: 작업자 A / 작업자 B / GitHub 최종 동기화
```

---

## 핵심 흐름

```text
작업자 A Push
    ↓
작업자 B Fetch
    ↓
Merge
    ↓
동일 파일 수정 시 충돌 발생
    ↓
충돌 해결
    ↓
git add
    ↓
git commit
    ↓
git push
    ↓
작업자 A 최종 동기화
```

> 핵심은 각 로컬 저장소가 서로 독립적이기 때문에  
> 다른 작업자의 변경사항을 직접 `fetch`하고 `merge`해야 한다는 점이다. fileciteturn0file0L506-L520
