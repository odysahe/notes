# My Exprience about GIT
## Update Local Repo, But Keep Your Another File
- `git pull --rebase --autostash`
## ....
- `git add .`
- `git pull origin main --rebase --autostash`
## ....
## Switch Branch
For example you have 2 branch `main` and `develop`, in your local git exist a `main`.
- check exist branch
  `git branch`
- get all branch from cloud
  `git fetch origin`
- show all branch from cloud `git branch -a`
- if you can get the `develop` branch use the command `git checkout -b develop origin/develop` this command can copy from cloud to your local repository
