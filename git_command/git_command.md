1. git add . + git commit -m

    - `git commit -am`

2. Force push

    - `git push -f` 

3. Create branch

    - `git checkout -b <branch_name>`

4. Cancel add & commit operation, no change to files

    - `git reset <commit_sha>`

5. Only cancel commit operation, no change to files  

    - `git reset --soft <commit_sha>`

6. Cancel certain commit record and changes, change files
    - `git reset --hard <commit_sha>`

7. Cancel retain commit changes, but keep commit record
    - `git revert <commit-sha>`

8. Merge commits from start to end
    - `git rebase -i <start commit-sha> <end commit-sha>`

9. Merge certain commit to current branch
    - `git cherry-pick <commit-sha1>`

10. Merge commits in range to current branch
    - `git cherry-pick commit-sha1..commit-sha5`

11. List stash
    - `git stash list`
