# This Organization
This is a semi-private GitHub Organization for our working group members to upload their own projects as git repositories or work on shared collaborations.

## Troubleshooting
When in doubt please contact Hendrik or Johnny.

## Useful git commands
```
git remote add origin git@github.com:NAME/REPO.git
git remote set-url origin git@github.com:NAME/REPO.git
git branch -M main
git branch -a
git pull origin main --allow-unrelated-histories --no-rebase
git add FILES
git status
git diff --cached --name-only
git rm -r --cached FILES
git commit -m "Commit message"
git push -u origin main
```

## Example for a git workflow routine
```
#!/bin/bash

MAX_SIZE_KB=10  # max allowed staged file size in KB (adjust as needed)

# Helper function for yes/no prompt
confirm() {
    while true; do
        read -rp "$1 (y/n)? " yn
        case $yn in
            [Yy]*) return 0 ;;
            [Nn]*) echo "Aborted."; exit 1 ;;
            *) echo "Please answer y or n." ;;
        esac
    done
}

# Get user email
user_email=$(git config user.email)
confirm "Git user.email: $user_email. Continue"

# Get repo name (assumes you are inside a git repo)
repo_name=$(basename "$(git rev-parse --show-toplevel)")
confirm "Repository: $repo_name. Continue"

# Get current branch
branch_name=$(git rev-parse --abbrev-ref HEAD)
confirm "Branch: $branch_name. Continue"

remote_url=$(git config --get branch."$branch_name".remote)
if [ -z "$remote_url" ]; then
    remote_url="origin"
fi
remote_fetch_url=$(git remote get-url "$remote_url")
confirm "Remote for branch '$branch_name': $remote_fetch_url. Continue"

# Empty cache and stage files
confirm "Emptying cache and staging files. Continue"
git rm -r --cached .
git add .

# Remove oversized staged files
echo "Checking staged files for size > ${MAX_SIZE_KB} KB..."
oversized_found=0
while IFS= read -r file; do
    if [ -f "$file" ]; then
        size_kb=$(du -k "$file" | cut -f1)
        if [ "$size_kb" -gt "$MAX_SIZE_KB" ]; then
            #echo "Removing from staging: $file (${size_kb} KB)"
            git restore --staged "$file"
            oversized_found=1
        fi
    fi
done < <(git diff --cached --name-only)

if [ $oversized_found -eq 1 ]; then
    echo "Some files were too large and removed from staging."
fi

# Show total staged size after cleanup
echo
echo "Total staged size after cleanup:"
git diff --cached --name-only | xargs du -ch 2>/dev/null | grep total$

confirm "Ready to commit these staged changes. Proceed"

# Commit (you can customize the commit message prompt here)
read -rp "Enter commit message: " commit_msg
if [ -z "$commit_msg" ]; then
    echo "Commit message cannot be empty. Aborting."
    exit 1
fi

git commit -m "$commit_msg"

echo "Commit successful."

confirm "Ready to push. Proceed"
echo "Pushing to remote..."
git push -u "$remote_url" "$branch_name"
if [ $? -eq 0 ]; then
    echo "Push successful."
else
    echo "Push failed."
    exit 1
fi
```

## Example for correct black- and whitelisting
```
# Ignore everything by default
*

# Allow git to look in directories
!*/

# Whitelist specific filetypes
!*README.md
!*.py
!*.sh
!*.top
!*.itp
!*.mdp
!*.gro
!*.xyz
!*.pdb
!*.in
!*.inp
!*.input
!*.mol2
!*.chg
!*.json

# cleanup whitelist
mdout.mdp
git_commit_routine.sh
```


