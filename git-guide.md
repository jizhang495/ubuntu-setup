# Git Setup and Workflow (Ubuntu)

This guide assumes Ubuntu Linux. After `git` is installed, [generate an SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent), [add it to your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account), then [configure your user name and email](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup).

```bash
# Install (if needed)
sudo apt update
sudo apt install -y git openssh-client

# Generate a key (use your own email)
ssh-keygen -t ed25519 -C "your.email@example.com"
ls ~/.ssh
cat ~/.ssh/id_ed25519.pub # paste whole file content to GitHub Settings - New SSH key
```

Start an SSH agent and add the key (needed on a fresh shell session):

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

```bash
ssh -T git@github.com
# The authenticity of host 'github.com (20.26.156.215)' can't be established.
# ED25519 key fingerprint is SHA256:+DiY3....
# This key is not known by any other names.
# Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
# Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
# Enter passphrase for key '/home/<you>/.ssh/id_ed25519':
# Hi <username>! You've successfully authenticated, but GitHub does not provide shell access.
cat ~/.ssh/known_hosts
# login-knl.hpc.cam.ac.uk,128.232.224.49 ssh-ed25519 AAAAC....
# github.com ssh-ed25519 AAAAC....
# github.com ssh-rsa AAAAB3...
# github.com ecdsa-sha2-nistp256 AAAA....
rm -f ~/.ssh/known_hosts.old
```

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
cat ~/.gitconfig
vim ~/.gitconfig
```

```bash
git config --list --show-origin
git config --global merge.conflictstyle diff3
```
On Ubuntu/Linux, prefer `core.autocrlf=input` (or leave it unset). Avoid `core.autocrlf=true` unless you specifically need Windows-style line endings.

```bash
git config --global core.autocrlf input
```

## Branching Strategy

We use the following branch types: 

- `main` -- common stable code
- `dev` -- main development branch, changes are tested before merging into main
- `feat/<name>` -- new features deployed
- `fix/<name>` -- fixing bugs

## Commit Messages

Use clear, consistent messages. Follow this [format](https://gist.github.com/qoomon/5dfcdf8eec66a051ecd85625518cfd13):

- `feat`: new feature
- `fix`: bug fix
- `refactor`: code refactoring (no behavior change)
- `style`: formatting, white-space, missing semicolons
- `test`: add or fix tests
- `docs`: documentation changes, update README
- `chore`: miscellaneous

## Basic Commands

### Clone repo:
```bash
cd ~/code/
git clone git@github.com:jizhang495/ubuntu-setup.git
```
### Create repo:
```bash
git init -b main
# git branch -m master main # if used git init
# uv init
git add .
git commit -m "chore: init"
# git branch -M main
git remote add origin git@github.com:jizhang495/ubuntu-setup.git # make sure remote repo is empty
git push -u origin main
```

### Working in repo:

To normalise line endings on existing files, add .gitattributes,
```bash
git add .gitattributes
git add --renormalize .
git commit -m "add .gitattributes, normalise line endings"
```

Pull latest changes from remote:
```bash
git checkout main # if not in main
git pull origin main # or git pull
```
Create new branch and switch to the branch:
```bash
# git branch my-feature
# git checkout my-feature
git checkout -b my-feature
```
Make changes and commit with a [message](https://gist.github.com/qoomon/5dfcdf8eec66a051ecd85625518cfd13):
```bash
git diff # show what's still unstaged
git add .
git add -A # if changes include deletion
# git diff --cached # show what's staged
git status
git commit -m "<commit message>"
```
Show what was committed:
```bash
git show # commit message + diff
git log -1 # commit summary
git show --name-only # files changed in last commit
git diff --name-only HEAD~1 HEAD # files changed in last commit
git show --stat # lines added/removed
```

Push changes to remote:
```bash
git push -u origin my-feature
```
Create Pull Request on GitHub and wait for admin to merge it. 

### Squashing, Resetting, and Reverting

Squash the last N (e.g., 3) commits:
```bash
git add .
git commit -m "WIP: save before rebase"
git rebase -i HEAD~3
```
This opens editor. Change something like:
```sql
pick a1b2c3 First commit
pick d4e5f6 Second commit
pick 789abc Third commit
```
to
```sql
pick a1b2c3 First commit
squash d4e5f6 Second commit
squash 789abc Third commit
```
This tells git: "Keep the first commit message, and squash the others into it."
```bash
git log --oneline
git reflog
git push --force-with-lease origin my-feature
```

Reset all unstaged changes:
```bash
git checkout -- .
```

Undo the last commit but keep the changes in working directory and staging area:
```bash
git reset --soft HEAD~1
```
Uncommit and unstage changes:
```bash
git reset --mixed HEAD~1
```
Uncommit and discard all changes:
```bash
git reset --hard HEAD~1
```

Resetting to a previous commit:
```bash
git checkout my-feature
# git reflog
# git reset --hard HEAD@{3}
git log --oneline --graph # --decorate
git reset --hard abc1234
# git push -f origin my-feature
git push --force-with-lease origin my-feature
```

Resetting to match remote:
```bash
# Optional: backup your current main just in case
git checkout main
git checkout -b backup-main
# Then reset
git checkout main
git reset --hard origin/main
```

Reverting a previous commit:
```bash
git checkout my-feature
git log --oneline
git revert -m 1 abc1234
git push origin my-feature
```

### Resolving Conflicts

If cannot fastforward merge:
```bash
git checkout main
git pull origin main
git checkout my-feature
git rebase main # starts rebase
# solve conflicts if conflicts. 
# current = the version of the file in the commit you're replaying (i.e. one of your earlier commits from your branch being rebased)
# incoming = the version from the branch you're rebasing onto (e.g. main)
# base = the common ancestor of both
git add . # if conflicts
git rebase --continue # if conflicts
# git rebase --abort
git status # On branch my-feature, Your branch is ahead of 'origin/my-feature' by X commits.
# git log
# git push -f origin my-feature
git push --force-with-lease origin my-feature
```

After admin merges it (rebase and merge), delete branch on GitHub, and then delete the local branch:
```bash
git checkout main
git pull
git branch
git branch -d my-feature # git branch -D my-feature
git branch -r
git fetch -p # optional, to clean up remote refs
```
