# Git

* [Add a commit](#add-a-commit)
* [Add a signed commit](#add-a-signed-commit)
* [Change commit message](#change-commit-message)
* [Create and apply patches](#create-and-apply-patches)
* [Create a new branch](#create-a-new-branch)
* [Delete a local branch](#delete-a-local-branch)
* [Delete last commit](#delete-last-commit)
* [List all branches](#list-all-branches)
* [Overwrite an older commit](#overwrite-an-older-commit)
* [Overwrite an older commit-message](#overwrite-an-older-commit-message)
* [Overwrite the last commit](#overwrite-the-last-commit)
* [Overwrite the last commit message](#overwrite-the-last-commit-message)
* [Prune deleted branches from the remote](#prune-deleted-branches-from-the-remote)
* [Rebase a branch](#rebase-a-branch)
* [Reset to remote branch](#reset-to-remote-branch)
* [Resolve conflict](#resolve-conflicts)
* [Stash changes](#stash-changes)
* [Switch to a branch](#switch-to-a-branch)
* [Update a Git submodule](#update-a-git-submodule)

## Add a commit

```sh
# Stage all changes
git add .
# Create a commit with a message
git commit -m "commit-name"
# Push changes to the remote repository
git push
```

## Add a signed commit

```sh
# Stage all changes
git add .
# Create a signed commit with a message
git commit -s -m "commit-name"
# Push changes to the remote repository
git push
```

## Change commit message

### From the most recent commit

```sh
# Change the message of the most recent commit
git commit --amend -m "New commit message"
# Force push the changes to the remote repository
git push -f
```

### Of an older commit

```sh
# Start an interactive rebase starting from the commit before the one to change
git rebase -i <commit-hash-before-the-one-to-change>

# In the editor, replace 'pick' with 'reword' for the commit you want to modify
# Save and close the editor
# Edit the commit message as prompted, save, and close

# After successfully completing the rebase, force push the changes
git push -f
```

## Create and apply patches

### Create a patch

```sh
# Create a patch file with all changes in the working directory
git diff > patch-file.patch

# Create a patch file from the last commit
git format-patch -1 HEAD

# Create a patch file for a range of commits
git format-patch <start-commit>..<end-commit>
```

### Apply a patch

```sh
# Apply a patch file
git apply <patch-file>

# Apply a patch file and automatically commit the changes
git am <patch-file>
```

## Create a new branch

```sh
# Create a new branch
git checkout -b <branch-name>
# Push the new branch to the remote repository
git push
```

## Delete a local branch

```sh
# Delete a local branch
git branch -d <branch-name>
```

## Delete last commit

### Remove the commit from repository

```sh
# Delete the last commit
git reset --hard HEAD~1
# Force push changes to the remote repository
git push -f
```

### Keep changes in working directory

```sh
# Delete the last commit but keep the changes in the working directory
git reset --soft HEAD~1
```

## List all branches

```sh
# List all branches
git branch -a
```

## Overwrite an older commit

```sh
# Stage all changes
git add .
# Create a fixup commit for the specified commit hash
git commit --fixup <commit-hash-to-fix>
# Rebase interactively with autosquash to combine the fixup commit
git rebase -i --autosquash <prev-commit-hash-to-fix>
ctrl + x
# Force push changes to the remote repository
git push -f
```

## Overwrite an older commit message

```sh
# Rebase interactively until the selected commit
git rebase -i HEAD~<n-commits>
# Replace 'pick' with 'reword' next to the commit hash
ctrl + x
# Edit the commit message
ctrl + x
# Force push changes to the remote repository
git push -f
```

## Overwrite the last commit

```sh
# Stage all changes
git add .
# Amend the last commit without changing the commit message
git commit --amend --no-edit
# Force push changes to the remote repository
git push -f
```

## Overwrite the last commit message

```sh
# Amend the last commit and change its commit message
git commit --amend -m "New commit message"
# Force push changes to the remote repository
git push -f
```

## Prune deleted branches from the remote

```sh
# Prune deleted branches from the remote (origin)
git remote prune origin

# Dry-run to see what will be pruned without making changes
git remote prune origin --dry-run
```

## Rebase a branch

```sh
git checkout <branch-name>
# Rebase the branch with the latest changes
git rebase -i <base-branch-name>
# Force push the rebased branch to the remote repository
git push -f
```

## Reset to remote branch

```sh
# Reset the current branch to match the remote branch (origin/master)
git reset --hard origin/master
```

## Resolve conflicts

```sh
# List files with conflicts
git status
# Resolve conflicts in the conflicted files
# Stage all resolved files
git add <resolved-file>
# Continue the rebase process
git rebase --continue
# Force push the rebased branch to the remote repository
git push -f
```

## Stash changes

```sh
# Stash uncommitted changes
git stash

# List all stashed changes
git stash list

# Apply the most recent stash and keep it in the stash list
git stash apply

# Apply the most recent stash and remove it from the stash list
git stash pop

# Apply a specific stash from the list
git stash apply stash@{<index>}

# Drop a specific stash from the list
git stash drop stash@{<index>}

# Clear all stashes
git stash clear
```

## Switch to a branch

```sh
# Switch to an existing branch
git checkout <branch-name>
```

## Update a Git submodule

```sh
# Initialize and update all submodules
git submodule update --init --recursive

# Update a specific submodule
git submodule update --remote <submodule-path>

# Pull the latest changes in all submodules
git submodule foreach git pull origin main

# Sync submodule URLs to their configuration in the superproject
git submodule sync
```
