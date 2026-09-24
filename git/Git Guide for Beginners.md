
# Git Command Reference & Workflow Guide

A comprehensive, step-by-step technical handbook covering foundational configuration, local history management, branching strategies, conflict resolution, interactive rebasing, and remote GitHub workflows.

Tags: #git #github 

## Concept 1: Environment & SSH Configuration

1. Configure Your Identity
Set your name and email globally across your system (only needs to be run once per computer):

```shell
git config --global user.name "Your First and Last Name"
git config --global user.email "your.email@example.com"
````

2. Generate SSH Key Pair  
  
Generate a modern ed25519 key pair:

```
ssh-keygen -t ed25519 -C "your.email@example.com"
```

- **Save Location**: Press `Enter` to accept the default file path.
- **Passphrase**: Press `Enter` twice to leave empty (suitable for sandboxed environments).

3. Start SSH Agent & Add Key  
  
Ensure the background SSH agent service is running and register your key:

```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

4. Copy Public Key  
  
Display the public key in your terminal:

```
cat ~/.ssh/id_ed25519.pub
```

- Copy the entire printed line (starts with `ssh-ed25519` and ends with your email).

5. Add Key to GitHub

6. Sign into GitHub in a web browser.
7. Navigate to **Profile Picture** -> **Settings**.
8. Under the left sidebar, click **SSH and GPG keys**.
9. Click **New SSH key**.
10. Set a descriptive title (e.g., "My Laptop"), paste your public key into the **Key** field, and click **Add SSH key**.

11. Test Authentication  
  
Verify connection to GitHub:

```
ssh -T git@github.com
```

- If prompted with _"The authenticity of host 'github.com' can't be established"_, type `yes` and hit `Enter`.
- Expected response: `"Hi [your-username]! You've successfully authenticated..."`

## Concept 2: Repository Setup & Perimeter Security
1. Create Empty Repository on GitHub

2. Go to GitHub, click **+** in the upper-right corner -> **New repository**.
3. Name the repository: `git-sandbox`.
4. Select **Public**.

> \[\!warning\] Critical Requirement  
> Leave all options unchecked (do **not** add a README, .gitignore, or license).

5. Click **Create repository**.

6. Clone the Empty Repository  
  
Copy the SSH URL (`git@github.com:YOUR-USERNAME/git-sandbox.git`) and execute:

```
git clone git@github.com:YOUR-USERNAME/git-sandbox.git
cd git-sandbox
```

_(A warning about cloning an empty repository is expected.)_3. Download Official .gitignore  
  
Fetch GitHub's standard Python `.gitignore` file using `curl`:

```
curl -o .gitignore [https://raw.githubusercontent.com/github/gitignore/main/Python.gitignore](https://raw.githubusercontent.com/github/gitignore/main/Python.gitignore)
```

4. Commit .gitignore  
  
Stage and commit the file to lock repository configuration into history:

```
git add .gitignore
git commit -m "Add Python .gitignore configuration"
```

## Concept 3: Building History with Commits
> Mechanics Overview  
>  
>   - `echo "text"`: Prints text to stdout.  
>   - `>`: Overwrites file content or creates a new file.  
>   - `>>`: Appends text to the end of an existing file.  
>   - `-m`: Specifies inline commit message.  
>   - `-am`: Stages tracked changes and commits simultaneously.Commits Execution

**Commit 1: Create File**

```
echo "# Git Sandbox" > README.md
git add README.md
git commit -m "Create initial README file"
```

>  `git add` is required for newly created, untracked files.

**Commit 2: Add Project Description**

```
echo "This repository is for practicing Git commands." >> README.md
git commit -am "Add project description to README"
```

**Commit 3: Add Subheading**

```
echo "## Setup Instructions" >> README.md
git commit -am "Add setup section header to README"
```

**Commit 4: Add Final Instructions**

```
echo "Follow the tasks in order." >> README.md
git commit -am "Add instructions to README"
```

History Verification

- Check contents: `cat README.md`
- Inspect commit graph: `git log --oneline`

## Concept 4: Branching and Merging
1. Create and Switch to Branch

```
git checkout -b my-new-feature
```

- The `-b` flag creates `my-new-feature` and immediately switches active context to it.

2. Make Commit on Feature Branch

```
echo "This is my experimental feature." > feature.txt
git add feature.txt
git commit -m "Add experimental feature file"
```

3. Switch Back to Main

```
git checkout main
```

- `feature.txt` is removed from your working tree because it only exists in the `my-new-feature` branch context.

4. Merge Branch into Main

```
git merge my-new-feature -m "merge feature branch into main"
```

- Merges snapshot histories and restores `feature.txt` onto `main`.
- View timeline: `git log --graph --oneline`

## Concept 5: Resolving Merge Conflicts
1. Set Up Base File on Main

```
echo "The best fruit is an Apple." > fruit.txt
git add fruit.txt
git commit -m "Add base fruit file"
```

2. Modify File on Feature Branch

```
git checkout -b feature/change-fruit
echo "The best fruit is a Mango." > fruit.txt
git commit -am "Change fruit to Mango on branch"
```

3. Modify File on Main

```
git checkout main
echo "The best fruit is a Banana." > fruit.txt
git commit -am "Change fruit to Banana on main"
```

4. Trigger Conflict

```
git merge feature/change-fruit
```

- Git halts execution: `CONFLICT (content): Merge conflict in fruit.txt`

5. Resolve Conflict Manually  
  
Open file in terminal editor:

```
nano fruit.txt
```

Review conflict markers:

```
<<<<<<< HEAD
The best fruit is a Banana.
=======
The best fruit is a Mango.
>>>>>>> feature/change-fruit
```

1. Delete conflict indicators (`<<<<<<<`, `=======`, `>>>>>>>`).
2. Format desired text (e.g., `The best fruit is a Mango AND a Banana.`).
3. Save (`Ctrl+O`, `Enter`) and exit (`Ctrl+X`).

4. Finalize Merge

```
git add fruit.txt
git commit -m "Resolve fruit merge conflict"
```

## Concept 6: Rebasing
>Overview   
>   - **Merge:** Retains independent timelines and links them via a merge commit (diamond history graph).  
>   - **Rebase:** Rewinds working branch, updates base commit to current main tip, and replays feature commits sequentially (linear history).Steps

1. **Create Base File on Main**

```
echo "PORT=8080" > server.txt
git add server.txt
git commit -m "Add base server config"
```

1. **Modify File on Feature Branch**

```
git checkout -b feature/update-port
echo "PORT=3000" > server.txt
git commit -am "Change port to 3000 on branch"
```

1. **Modify File on Main**

```
git checkout main
echo "PORT=5000" > server.txt
git commit -am "Change port to 5000 on main"
```

1. **Initiate Rebase**

```
git checkout feature/update-port
git rebase main
```

1. **Resolve Conflict**

```
nano server.txt
```

Remove markers, finalize config value, save (`Ctrl+O`, `Enter`), and exit (`Ctrl+X`).

1. **Continue Rebase Process**

```
git add server.txt
git rebase --continue
```

> Do **not** run `git commit` during a rebase continuation.

## Concept 7: Interactive Rebase (Squashing Commits)
1. Set System Editor to Nano  
  
Set system editor to Nano for ease of use:

```
git config core.editor "nano"
```

2. Create Unsquashed Commits

```
echo "Line 1" > poem.txt
git add poem.txt
git commit -m "Start poem"

echo "Line 2" >> poem.txt
git commit -am "Add second line"

echo "Line 3" >> poem.txt
git commit -am "Finish poem"
```

3. Launch Interactive Rebase  
  
Rebase last 3 commits relative to HEAD:

```
git rebase -i HEAD~3
```

4. Edit Rebase Instructions  
  
In nano, modify lines 2 and 3 from `pick` to `squash` (or `s`):

```
pick a1b2c3d Start poem
squash x8y9z0a Add second line
squash q1w2e3r Finish poem
```

Save (`Ctrl+O`, `Enter`) and exit (`Ctrl+X`).5. Finalize Combined Commit Message  
  
Replace previous commit log text with a clean message:

```
Write complete 3-line poem
```

Save and exit nano. Verify linear timeline: `git log --oneline`.
## Concept 8: Pull Requests (PRs) & Remote Workflows
1. Push Local Main to Remote

```
git checkout main
git push -u origin main
```

2. Create & Push Feature Branch

```
git checkout -b feature/t8-pull-request
echo "This file was created for a PR." > pr-test.txt
git commit -m "Add file to test pull request workflow"
git push -u origin feature/t8-pull-request
```

3. Create Pull Request

4. Open repository page on GitHub.
5. Click **Compare & pull request** on the action banner.
6. Add PR description comment: _"Here is the new file, please review!"_
7. Click **Create pull request**.

8. Squash and Merge on GitHub

9. At bottom of PR view, click dropdown arrow next to **Merge pull request**.
10. Select **Squash and merge**.
11. Confirm merge.

12. Sync Local Repository  
  
Pull merged updates down to local main:

```
git checkout main
git pull origin main
```

## Concept 9: Forking and Upstream Remotes
1. Fork Public Repository

2. Navigate to upstream repo: `https://github.com/octocat/Spoon-Knife`
3. Click **Fork** -> **Create fork**.
4. Confirm repo exists under your user profile (e.g., `username/Spoon-Knife`).

5. Clone Fork Locally  
  
Navigate outside `git-sandbox` folder and clone fork:

```
cd ..
git clone git@github.com:username/Spoon-Knife.git
cd Spoon-Knife
```

3. Configure Upstream Remote  
  
Add original repository as upstream source:

```
git remote add upstream [https://github.com/octocat/Spoon-Knife.git](https://github.com/octocat/Spoon-Knife.git)
```

Verify remotes (`origin` and `upstream`):

```
git remote -v
```

4. Fetch Upstream History  
  
Download latest commits from original upstream repository:

```
git fetch upstream
```