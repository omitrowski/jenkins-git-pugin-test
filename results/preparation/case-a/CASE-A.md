### Modify file, commit and push in both repository clones. First push from `jenkins-git-pugin-test_1` clone.
![1](1-git-commit-push.jpg)

### From `jenkins-git-pugin-test_2` git pull
![2](2-git-pull.jpg)

### From `jenkins-git-pugin-test_2` git rebase
![3](3-git-rebase.jpg)

### From `jenkins-git-pugin-test_2` merge
![4](4-vim-merge.jpg)

### From `jenkins-git-pugin-test_2` git commit and push merged
![5](5-git-commit-push.jpg)

### Git log snipped
`cat results/preparation/case-a/6-git.log`
```shell
*   7d4cd7d (HEAD -> master, origin/master, origin/HEAD) Merge branch 'master' of github.com:omitrowski/jenkins-git-pugin-test
|\
| * b64f1bc manual modification from clone #1
* | 9ba7116 manual modification from clone #2
|/
```
