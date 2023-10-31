# 기본 명령어
---

### 목차
- git

## git init - 저장소 생성
`` git init ``
- **로컬** Git 저장소로 설정

#### 명령어
```bash
mkdir sample
cd sample
touch red orange
echo "빨강" >> red
echo "주황" >> orange
git init
```

- sample 디렉토리에 Git 저장소 생성 <br>
- 디렉토리 하위에 .git 디렉토리 생성 - Git과 관련된 정보 저장 <br>
- 쉘 프롬프트가 ➜ sample에서 ➜ sample git:(main) ✗로 변경

---

## git status - 현재 상태 확인
`` git status ``
- 현재 작업 중인 파일 상태 확인

#### 명령어
```bash
git status
git status # gst
```
> ### # gst 는 뭔가요?
> 명령어 뒤에 주석으로 써있는 부분은 alias로 *oh-my-zsh*을 설치하면 사용할 수 있는 별칭입니다.<br>
> git status대신 gst만 입력해도 동일하게 동작합니다. alias를 적극적으로 써보세요! <br>

#### 결과
```bash
On branch main

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	orange
	red

nothing added to commit but untracked files present (use "git add" to track)
```
- 현재 브랜치(main)와 커밋 상태, 작업 중인 파일의 상태 확인
- untracked files(추적하지 않는 파일)이 존재하는 것을 확인

---

## git add - 현재 상태 추적
`` git add [-A] [<pathspecs>..] ``

- 파일의 변경사항을 인덱스(index)에 추가합니다. Git은 커밋하기 전, 인덱스에 먼저 커밋할 파일을 추가합니다.

1. -A 옵션을 이용하여 전체 파일(orange, red)을 인덱스에 추가
2. 상태 확인

#### 명령어
```bash
git add -A # gaa
git status # gst
```

#### 결과
```bash
On branch main

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   orange
	new file:   red
```

- untracked files에 있던 orange와 red의 상태가 변경된 것을 확인

---

## git commit - 현재 상태 저장
`` git commit [-m <msg>] ``

#### 명령어
```bash
git commit -m "v1 commit" # gc -m "v1 commit"
```

#### 결과
```bash
[main (root-commit) 60e448b] Initial Commit
 2 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 orange
 create mode 100644 red
```

---
