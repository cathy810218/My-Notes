##Inspect & Compare
1. `git log` show the commit history for the currently active branch
2. `git diff branchB...branchA` show the diff of that is in branchA that is not in branchB


##Tracking Path Change
1. `git rm [file]`
2. `git mv [existing-path] [new-path]`


##Share & Update
1. `git remote add [alias] [url]` add a git URL as an alias
2. `git fetch [alias]` fetch down all the branches from that Git remote
2. `git merge [alias]/[branch]` merge a remote branch into your current branch to bring it up to date
4. `git push [alias] [branch]`
5. `git pull`

##Rewrite History
1. `git rebase [branch]` apply any commits of current branch ahead of specified one
2. `git reset --hard [commit]` clear staging area, discard all uncommited changes
3. `git commit --amend` amend previous commit
4. `git push -f origin xxxx` force push to a branch (with ^ amend)

##Temporary Commits
1. `git stash` Save modified and staged changes
2. `git stash list` list stack-order of stashed file changes
3. `git stash pop` write working from top of stash stack
4. `git stash drop` discard the changes from top of stash stack


##Update Changes
1. `git cherry-pick 99daed2` get a certain commit out and add to another branch
2. `git checkout .` discard all unstaged changes
3. `git chechout -- <file>` discard this unstaged change

##Clean Up
1. `git clean -n` check what will be deleted
2. `git clean -fdx` remove untracked files
3. `git reset --hard <SOME-COMMIT>` make your current branch (typically master) back to point at <SOME-COMMIT> .

##Undo a Commit and Redo
1. `git commit -m xxx` OH NO! I shouldn't have commit this file!
2. `git reset HEAD~` Now everything is unstaged
3. `git add ...`
4. `git commit -c ORIG_HEAD` redo the commit by starting with its log message

##Add More Changes to the Previous Commit
1. `git commit -m xxx` OH NO I forgot to add some file
2. `git reset --soft HEAD~` this leaves the existing changes staged or git reset HEAD~1
3. Edit your file
4. `git commit -a -c ORIG_HEAD` redo the commit by starting with its log message

##Get Back the Deleted Commit
1. `git reflog` to get a list of commit shas
2. `git checkout -b someNewBranchName shaYouDestroyed`
