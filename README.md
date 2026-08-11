# Git 협업 실습 - Fetch, Merge 및 충돌 해결

## 실습 개요

하나의 GitHub 원격 저장소를 두 개의 로컬 저장소에 Clone하여  
**작업자 A와 작업자 B가 협업하는 상황**을 재현한다.

이번 실습에서는 다음 과정을 통해 Git 협업의 전체적인 흐름을 이해한다.

- 서로 다른 파일을 수정하며 충돌 없는 협업 진행
- `fetch`와 `merge`를 이용한 원격 변경사항 동기화
- 동일한 파일의 같은 부분을 수정하여 Merge Conflict 발생
- 충돌 해결 후 Merge Commit 생성 및 Push
- 최종적으로 두 작업자와 GitHub의 상태 동기화

---

## 실습 흐름

```mermaid
sequenceDiagram
    autonumber

    participant A as 작업자 A 로컬
    participant GH as GitHub 원격 저장소
    participant B as 작업자 B 로컬

    Note over A,B: 같은 GitHub 저장소를 각각 독립된 로컬 저장소로 사용

    %% =========================
    %% 1. 충돌 없는 협업
    %% =========================
    rect rgb(235, 245, 255)
        Note over A,B: ① 충돌 없는 협업

        A->>A: worker-a.md 생성
        A->>A: git add / commit
        A->>GH: git push

        Note over GH,B: B의 로컬에는 A의 변경이 아직 없음

        B->>GH: git fetch origin
        GH-->>B: origin/main 최신 정보 전달
        B->>B: git merge origin/main

        Note over B: A의 변경사항 로컬에 반영

        B->>B: worker-b.md 생성
        B->>B: git add / commit
        B->>GH: git push

        A->>GH: git fetch origin
        GH-->>A: origin/main 최신 정보 전달
        A->>A: git merge origin/main

        Note over A: B의 변경사항 로컬에 반영
    end

    %% =========================
    %% 2. 충돌 발생
    %% =========================
    rect rgb(255, 245, 230)
        Note over A,B: ② 동일한 README.md 수정 → 충돌 상황 생성

        A->>A: README.md 같은 문장 수정
        A->>A: git add / commit
        A->>GH: git push

        Note over GH,B: B는 A의 변경을 아직 가져오지 않음

        B->>B: README.md 같은 문장을 다르게 수정
        B->>B: git add / commit
        B->>GH: git push

        GH-->>B: Push 거절<br/>fetch first

        Note over B,GH: local main과 origin/main이<br/>서로 다른 커밋을 가진 상태
    end

    %% =========================
    %% 3. 충돌 해결
    %% =========================
    rect rgb(255, 235, 235)
        Note over B,GH: ③ Fetch → Merge → Conflict 해결

        B->>GH: git fetch origin
        GH-->>B: A의 최신 커밋 전달

        B->>B: git merge origin/main

        Note over B: README.md Merge Conflict 발생<br/>main|MERGING

        B->>B: 충돌 내용 확인
        Note over B: HEAD = 작업자 B 내용<br/>origin/main = 작업자 A 내용

        B->>B: 최종 내용 결정
        B->>B: 충돌 표시 제거
        B->>B: git add README.md
        B->>B: git commit

        Note over B: Merge Commit 생성

        B->>GH: git push

        Note over GH: 충돌 해결 결과가<br/>원격 main에 반영
    end

    %% =========================
    %% 4. 최종 동기화
    %% =========================
    rect rgb(235, 255, 235)
        Note over A,GH: ④ 작업자 A 최종 동기화

        A->>GH: git fetch origin
        GH-->>A: B의 Merge Commit 전달

        A->>A: git merge origin/main

        Note over A,B: 작업자 A / 작업자 B / GitHub<br/>모두 동일한 최신 상태
    end
```

---

## 핵심 흐름

```text
[① 충돌 없는 협업]
