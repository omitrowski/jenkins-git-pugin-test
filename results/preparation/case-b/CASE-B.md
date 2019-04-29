### Ensure clean status on both repo clones
![1](0-git-status.jpg)

### Modify different files in both repo clones and commit
![2](1-git-commit.jpg)

### Git push from both repo clones
![3](2-git-push.jpg)

### From `jenkins-git-pugin-test_2` git pull and git rebase
![4](3-git-pull-rebase.jpg)

### From `jenkins-git-pugin-test_2` git push after successful git rebase
![5](4-git-push-after-rebase.jpg)

### Git log, last two commits
`results/preparation/case-b/5-git.log`
```bash
commit 665263c9398eec8a4ccee1d01d267b8b4667893f
Author: Oskar <o.mitrowski@gmail.com>
Date:   Sun Apr 28 15:02:40 2019 +0200

    Test preparation case b, manual modification from clone #2

commit 8380bf6cf2d91f3f0b50eacd5fef26b821cb5b64
Author: Oskar <o.mitrowski@gmail.com>
Date:   Sun Apr 28 15:02:35 2019 +0200

    Test preparation case b, manual modification from clone #1
```

### Git log showing history after rebase
![5](5-git-log.jpg)
