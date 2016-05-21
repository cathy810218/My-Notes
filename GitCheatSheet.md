##Inspect & Compare

`git log` show the commit history for the currently active branch
`git log` show the commits on branchA that are not on branchB
`git diff branchB...branchA` show the diff of that is in branchA that is not in branchB


##Tracking Path Change
`git rm [file]`
`git mv [existing-path] [new-path]`


##Share & Update
`git remote add [alias] [url]` add a gir URL as an alias
`git fetch [alias]` fetch down all the branches from that Git remote
`git merge [alias]/[branch]` merge a remote branch into your current branch to bring it up to date
`git push [alias] [branch]`
`git pull`

##Rewrite History
`git rebase [branch]` apply any commits of current branch ahead of specified one
`git reset --hard [commit]` clear staging area, rewrite working tree from specified commit

##Temporary Commits
`git stash` Save modified and staged changes
`git stash list` list stack-order of stashed file changes
`git stash pop` write working from top of stash stack
`git stash drop` discard the changes from top of stash stack


