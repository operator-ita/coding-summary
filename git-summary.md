# Github Basic Conceps 

## Create a new repository 
```
git clone /path/to/repository 
git clone username@host:/path/to/repository
```
## Workflow 
- Working Dictory: holds your actual files  
- Index: acts as a staging area 
- HEAD: points to the last commit you've made. 

## Add & Commit 
Add changes to the Index. 
``` 
git add <filename> 
git add * 
```
Commit these changes to the HEAD of your local working copy (But no to the remote repository yet)
```
git commit -m "Commit message"
```

## Pushing changes
Send changes in the head to the remote repository.  
```
git push oring <branch>
```
The main branch is call `master`. 
If you have not cloned an existing repository and want to connect your repository to a remote server, you need to add it with
`git remote add origin <server>`

## Branching
Braches are ysed to develop features isolated from each other. The mastaer is the main brach. 
Develop in other branches and merge them bach to the master branch upon completion. 
- Create a new branch 
```
git checkout -b <branch name>
```
- switch back to anoter brach 
```
git checkout <another brach name>
```
- delete a brach 
```
git branch -d <brach name>
```
- Push a brach to your remote respositoy 
```
git push origin <branch>
```
## Update & merge
- Update your local repository to the newest commit, open the working directory and execute  
```
git pull
```
To merge another brach into your active branch (e.g. master), use
```
git merge <brach>
```
Mark them as merged 
```
git add <filename>
```
Preview differences before merging 
```
git diff <source_brach><target_brach>

## Tagging 
```
Create a new tag 
```
git tag <tag name><commit id>
```
- e.g. git tag 1.0.0 1b2e1d63ff

## log 
Study a repository history 
```
git log 
```
See commits of certain autor: 
```
git log --author=bob
```
To see a very compressed log where each commit is one line 
```
git log --pretty=oneline
```
To see an ASCII art tree of all the branches, decorated with the names of tags and branches: 
```
git log --graph --oneline --decorate --all
```
See only which files have changed: 
```
git log --name-status
```
For more info 
```
git log --help
```

## Replace local changes 
Replace local changes using the command shown below. This replaces the changes in your working tree with the last content in HEAD. Changes already added to the index, as well new files, will kept. 
```
git checkout -- <filename> 
```
 If you instead want to drop all your local changes and commits, fetch the latest history from the server and point your local master branch at it like this
 ```
git fetch origin
git reset --hard origin/master
 ```





