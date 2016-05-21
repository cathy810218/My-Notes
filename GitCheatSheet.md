##Inspect & Compare

1. `git log` show the commit history for the currently active branch
2. `git log` show the commits on branchA that are not on branchB
3. `git diff branchB...branchA` show the diff of that is in branchA that is not in branchB


##Tracking Path Change
1. `git rm [file]`
2. `git mv [existing-path] [new-path]`


##Share & Update
1. `git remote add [alias] [url]` add a gir URL as an alias
2. `git fetch [alias]` fetch down all the branches from that Git remote
2. `git merge [alias]/[branch]` merge a remote branch into your current branch to bring it up to date
4. `git push [alias] [branch]`
5. `git pull`

##Rewrite History
1. `git rebase [branch]` apply any commits of current branch ahead of specified one
2. `git reset --hard [commit]` clear staging area, rewrite working tree from specified commit

##Temporary Commits
1. `git stash` Save modified and staged changes
2. `git stash list` list stack-order of stashed file changes
3. `git stash pop` write working from top of stash stack
4. `git stash drop` discard the changes from top of stash stack


