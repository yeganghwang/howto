# 리베이스 사용하기
이전 장의 3-way 병합을 하면 병합 커밋이 생성된다.
```
yegang@yegangs:~/hello-git-cli$ git log --oneline --all --graph -n4
*   65352c8 (HEAD -> feature1) Merge branch 'master' into feature1
|\
| * ef34ef4 (origin/master, master, hotfix) hotfix 실습
* | 247cb34 새로운 기능 1 추가
|/
* db8ebff (tag: v0.1) mybranch1의 두 번째 커밋
```
병합 커밋이 생성되면 지저분해질 수 있으므로, 트리를 깔끔하게 하고 싶다면 Rebase(재배치) 할 수 있다.

Rebase의 원리를 살펴보면 아래와 같다.
- HEAD와 대상 브랜치의 공통 조상을 찾는다
- 공통 조상 이후에 생성한 커밋들을 대상 브랜치 뒤로 재배치한다.

예를 들어 위 git log를 확인해보면, `ef34ef4 (hotfix)`의 조상은 `db8ebff (tag: v0.1)`이다. 그렇다면 병합 커밋을 만드는 대신에 `247cb34` 뒤로 재배치하는 것도 가능했을 것이다.

다만 이렇게 리베이스를 하는 것에 장점만이 있는 것은 아니다. **재비치된 커밋은 커밋 체크섬이 바뀌기 때문이다.** rebase 명령어는 로컬 브랜치를 깔끔하게 정리하기 위함이고, 원격에 Push한 브랜치를 rebase하는 경우 다른 사용자가 혼란을 겪을 수 있으니 적절하지 않다.

우선 이전의 병합 커밋ㅇ르 되돌리고 rebase 해 보겠다. `feature1` 브랜치를 한 단계 되돌리기 위해 git reset --hard 명령어를 사용해 보겠다.

우선 `feature`로 체크아웃한다.
```
yegang@yegangs:~/hello-git-cli$ git checkout feature1
Switched to branch 'feature1'
```

다음으로 하드리셋을 실행한다. 여기서 `HEAD~`는 HEAD에서 한 칸 전이라는 뜻이다.(HEAD~1과 동일)
```
yegang@yegangs:~/hello-git-cli$ git reset --hard HEAD~
HEAD is now at 247cb34 새로운 기능 1 추가
```

로그를 확인해보면 한 단계 하드리셋 된 것을 확인할 수 있다. 
```
yegang@yegangs:~/hello-git-cli$ git log --oneline --graph --decorate --all -n3
* ef34ef4 (origin/master, master, hotfix) hotfix 실습
| * 247cb34 (HEAD -> feature1) 새로운 기능 1 추가
|/
* db8ebff (tag: v0.1) mybranch1의 두 번째 커밋
```

이제 rebase해 보겠다.
```
yegang@yegangs:~/hello-git-cli$ git rebase master
Auto-merging file1.txt
CONFLICT (content): Merge conflict in file1.txt
error: could not apply 247cb34... 새로운 기능 1 추가
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
```
안 되잖아? 뭐가 문제인지 확인해 보겠다.
우선 `247cb34` 커밋을 rebase 실패했다. 내용을 보자면, 충돌을 해결(Resolve all conflicts)한 다음 스테이지에 추가할 것을 알려준다. 그리고 git rebase --continue 명령어를 사용하라고 한다.

우선 어디에서 충돌이 발생했는지 확인해 보겠다.
```
yegang@yegangs:~/hello-git-cli$ git status
interactive rebase in progress; onto ef34ef4
Last command done (1 command done):
   pick 247cb34 새로운 기능 1 추가
No commands remaining.
You are currently rebasing branch 'feature1' on 'ef34ef4'.
  (fix conflicts and then run "git rebase --continue")
  (use "git rebase --skip" to skip this patch)
  (use "git rebase --abort" to check out the original branch)

Unmerged paths:
  (use "git restore --staged <file>..." to unstage)
  (use "git add <file>..." to mark resolution)
        both modified:   file1.txt

no changes added to commit (use "git add" and/or "git commit -a")
```
`file1.txt`를 수정하면 되겠다.
`vi file1.txt` 명령어를 이용하여 수정한다.
```
hello test
second
third - mybranch1
fourth - mybranch1
<<<<<<< HEAD
핫픽스
=======
기능 1 추가
>>>>>>> 247cb34 (새로운 기능 1 추가)
```
수정을 완료했다면 스테이징한다.
```
git add file1.txt
```

이제 status를 확인해 보겠다.
```
yegang@yegangs:~/hello-git-cli$ git status
interactive rebase in progress; onto ef34ef4
Last command done (1 command done):
   pick 247cb34 새로운 기능 1 추가
No commands remaining.
You are currently rebasing branch 'feature1' on 'ef34ef4'.
  (all conflicts fixed: run "git rebase --continue")

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   file1.txt
```
모든 충돌이 수정(all conflicts fixed)라고 한다.
이제 위 설명에 따라 "git rebase --continue" 명령을 사용하겠다.
```
yegang@yegangs:~/hello-git-cli$ git rebase --continue
[detached HEAD b30409f] 새로운 기능 1 추가
 1 file changed, 1 insertion(+)
Successfully rebased and updated refs/heads/feature1.
```
다음으로 로그를 확인해 보겠다.
```
yegang@yegangs:~/hello-git-cli$ git log --oneline --decorate --all --graph -n4
* b30409f (HEAD -> feature1) 새로운 기능 1 추가
* ef34ef4 (origin/master, master, hotfix) hotfix 실습
* db8ebff (tag: v0.1) mybranch1의 두 번째 커밋
* 0e4f33b mybranch1의 첫 번째 커밋
```
merge와는 달리 병합 커밋도 없고 히스토리도 한 줄기로 깔끔해졌다. 다만 `feature1` 브랜치가 가리키는 커밋의 체크섬 값이 `b30409f`로 바뀐 것을 볼 수 있다. 이렇기 때문에 여러 작업자가 동시에 작업할 때 원격저장소에 리베이스 하는 것은 권장되지 않다.

### rebase와 merge의 차이?
3-way 병합은 기존 커밋의 변경 없이 새로운 병합 커밋을 하나 생성한다. 따라서 충돌도 한 번만 발생한다. 충돌 수정 완료 후 "git commit" 명령어를 사용하면 merge 작업이 완료된다.

그러나 rebase는 **재배치 대상이 여러 개일 경우 여러 번 충돌이 발생할 수 있다.** 또한 기존의 커밋을 하나씩 단계별로 수정하기 때문에 "git rebase --continue" 명령을 사용하면 충돌로 중지된 리베이스를 **재개**하게 되는 것이다. 여러 커밋에 충돌이 여러 개 발생했다면 충돌을 해결할 때마다 매번 continue 해야 한다. 이러한 경우에는 리베이스를 하는 것보다 그냥 3-way 병합을 하는 것이 더 나을 수 있다.
- `3-way 병합`:
    - 머지 커밋 생성
    - 한 번만 충돌 발생
    - 트리가 지저분해짐
- `rebase`:
    - 현재 커밋들을 수정하면서 대상 브랜치 위로 재배치함
    - 깔끔한 히스토리
    - 여러 번 충돌이 발생할 수 있음
