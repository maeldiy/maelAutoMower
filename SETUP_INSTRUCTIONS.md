# GitHub Setup Instructions - maelAutoMower Repository

Complete step-by-step guide to set up your GitHub repository and push all documentation.

---

## Prerequisites

1. **GitHub Account** - Already have this
2. **Git Installation** - Need to install
3. **Email Configuration** - maeldiy@gmail.com

---

## Step 1: Install Git

### Windows Installation

**Option A: Git for Windows (Recommended)**

1. Download from: https://git-scm.com/download/win
2. Run the installer (GitSetup-*.exe)
3. Accept all defaults (they're good!)
4. Restart your computer
5. Open PowerShell and verify:
   ```powershell
   git --version
   ```
   You should see: `git version 2.x.x.windows.1`

**Option B: GitHub Desktop (Easier alternative)**
- Download: https://desktop.github.com/
- Already includes Git
- Has visual interface instead of command line

---

## Step 2: Configure Git Locally

Open PowerShell as Administrator and run:

```powershell
git config --global user.email "maeldiy@gmail.com"
git config --global user.name "Mael"
git config --global core.autocrlf true
```

**Verify configuration:**
```powershell
git config --global --list
```

You should see:
```
user.email=maeldiy@gmail.com
user.name=Mael
core.autocrlf=true
```

---

## Step 3: Organize Your Repository Structure

**On your computer**, organize files like this:

```
C:\Users\mael\Documents\maelAutoMower\
├── README.md                          (moved from README_ROOT.md)
├── SETUP_INSTRUCTIONS.md              (this file)
├── documents/
│   ├── QUICK_START.md
│   ├── ROVER_SYSTEM_OVERVIEW.md
│   ├── WIRING_GUIDE.md
│   ├── ARDUROVER_PARAMETERS.md
│   ├── RTK_SETUP_GUIDE.md
│   ├── TROUBLESHOOTING_GUIDE.md
│   ├── SYSTEM_ARCHITECTURE.md
│   ├── INDEX.md
│   └── SYSTEM_ARCHITECTURE.drawio
```

### Instructions to Create Structure:

1. **Create new folder:**
   ```powershell
   mkdir C:\Users\mael\Documents\maelAutoMower
   mkdir C:\Users\mael\Documents\maelAutoMower\documents
   ```

2. **Copy files:**
   - Copy `README_ROOT.md` → `C:\Users\mael\Documents\maelAutoMower\README.md`
   - Copy all `QUICK_START.md`, `ROVER_SYSTEM_OVERVIEW.md`, etc. → `documents/` folder
   - Copy `SYSTEM_ARCHITECTURE.drawio` → `documents/` folder

3. **OR use PowerShell commands:**

```powershell
# Navigate to documents folder
cd C:\Users\mael\Documents\mael

# Create new repo structure
mkdir ..\maelAutoMower\documents
cp README_ROOT.md ..\maelAutoMower\README.md
cp QUICK_START.md ..\maelAutoMower\documents\
cp ROVER_SYSTEM_OVERVIEW.md ..\maelAutoMower\documents\
cp WIRING_GUIDE.md ..\maelAutoMower\documents\
cp ARDUROVER_PARAMETERS.md ..\maelAutoMower\documents\
cp RTK_SETUP_GUIDE.md ..\maelAutoMower\documents\
cp TROUBLESHOOTING_GUIDE.md ..\maelAutoMower\documents\
cp SYSTEM_ARCHITECTURE.md ..\maelAutoMower\documents\
cp SYSTEM_ARCHITECTURE.drawio ..\maelAutoMower\documents\
cp INDEX.md ..\maelAutoMower\documents\

# Verify structure
ls ..\maelAutoMower\documents
```

---

## Step 4: Create GitHub Repository (Online)

1. **Go to GitHub**: https://github.com/new
2. **Create new repository:**
   - **Repository name**: `maelAutoMower`
   - **Description**: `Autonomous lawn mower rover - Complete setup guides and documentation`
   - **Visibility**: Public (so others can find and learn from it)
   - **DO NOT** check "Initialize with README" (you already have one)
   - Click **"Create repository"**

3. **Copy the repository URL** shown after creation
   - Will look like: `https://github.com/YourUsername/maelAutoMower.git`

---

## Step 5: Initialize Local Git Repository

Open PowerShell and navigate to your repository folder:

```powershell
cd C:\Users\mael\Documents\maelAutoMower
```

**Initialize git:**
```powershell
git init
```

**Add remote (replace with YOUR username):**
```powershell
git remote add origin https://github.com/YourUsername/maelAutoMower.git
```

**Verify remote:**
```powershell
git remote -v
```

You should see:
```
origin  https://github.com/YourUsername/maelAutoMower.git (fetch)
origin  https://github.com/YourUsername/maelAutoMower.git (push)
```

---

## Step 6: Add and Commit Files

**Add all files:**
```powershell
git add .
```

**Verify files are staged:**
```powershell
git status
```

You should see all your files listed in green (to be committed).

**Create first commit:**
```powershell
git commit -m "Initial commit: Complete autonomous rover documentation

- System architecture diagrams (Mermaid)
- Wiring guide with detailed instructions
- ArduRover parameter optimization
- RTK setup guide for CentipedeRTK
- Comprehensive troubleshooting guide
- Quick start checklist for assembly
- 91+ pages of custom documentation"
```

---

## Step 7: Push to GitHub

**First push (set upstream branch):**
```powershell
git push -u origin main
```

**If that fails with "main" not found:**
```powershell
git branch -M main
git push -u origin main
```

**GitHub Authentication:**
- If prompted, choose one:
  - **Option 1**: Personal Access Token (PAT) - More secure
    1. Go to GitHub Settings → Developer settings → Personal access tokens
    2. Create new token (select `repo` scope)
    3. Copy and paste token when prompted
  
  - **Option 2**: GitHub CLI
    ```powershell
    gh auth login
    ```

---

## Step 8: Verify Upload

Go to https://github.com/YourUsername/maelAutoMower

You should see:
- ✅ README.md displayed on main page
- ✅ `documents/` folder with all guides
- ✅ All markdown files properly rendered
- ✅ Commit history showing your initial commit

---

## Step 9: Create .gitignore (Optional but Recommended)

Create a `.gitignore` file in your repo root to exclude unnecessary files:

```powershell
# Create .gitignore in PowerShell
@"
# OS Files
.DS_Store
Thumbs.db
*.tmp

# IDEs
.vscode/
.idea/
*.swp

# Backup files
*.backup
*.bak

# Build artifacts
*.log
dist/
build/

# Keep documentation
!*.md
!*.drawio
"@ | Out-File -FilePath C:\Users\mael\Documents\maelAutoMower\.gitignore -Encoding UTF8

git add .gitignore
git commit -m "Add gitignore file"
git push
```

---

## Step 10: Add More Content (Future)

When you have updates:

```powershell
cd C:\Users\mael\Documents\maelAutoMower

# Make your changes / add new files

# Stage changes
git add .

# Commit
git commit -m "Update documentation with [specific change]"

# Push
git push
```

---

## Troubleshooting

### "git is not recognized"
- Git not installed (see Step 1)
- Restart PowerShell/computer after installation

### "fatal: not a git repository"
```powershell
# Make sure you're in the right directory
cd C:\Users\mael\Documents\maelAutoMower
git status
```

### "Permission denied" on push
- Verify credentials (authentication token or SSH key)
- Try: `git push --set-upstream origin main`

### Files not appearing on GitHub
1. Verify `git push` showed no errors
2. Check GitHub website (refresh browser)
3. Verify files were staged: `git status` should show "nothing to commit"

### "Please tell me who you are" error
- Run: `git config --global user.email "maeldiy@gmail.com"`
- Run: `git config --global user.name "Mael"`

---

## Command Reference

**Useful one-liners:**

```powershell
# Check status
git status

# See all commits
git log --oneline

# See differences
git diff

# Undo last commit (before push)
git reset --soft HEAD~1

# Undo last commit (after push) - creates new commit
git revert HEAD

# Update local from remote
git pull

# Check remote
git remote -v
```

---

## Next Steps After GitHub Setup

1. **Share your repository** with others
   - GitHub URL: `https://github.com/YourUsername/maelAutoMower`
   - Perfect for sharing with community!

2. **Create GitHub Issues** for documentation improvements
   - Suggestions and feedback

3. **Create GitHub Discussions** for community questions

4. **Add GitHub Pages** (optional)
   - Turn your README into a website
   - Settings → Pages → Select main branch

5. **Start building your rover!**
   - Use the documentation
   - Come back to update as you progress

---

## Final Verification Checklist

- [ ] Git installed and working
- [ ] Git configured with maeldiy@gmail.com
- [ ] GitHub repo created (maelAutoMower)
- [ ] Local files organized in proper structure
- [ ] Repository initialized locally
- [ ] All files staged (git add .)
- [ ] Initial commit created
- [ ] Files pushed to GitHub
- [ ] README.md appears on GitHub homepage
- [ ] All documents/ files visible on GitHub
- [ ] README has working hyperlinks to documents

---

## Example GitHub Repository Structure

Your finished repository should look like:

```
maelAutoMower
├── README.md                                    [Master index & links]
├── SETUP_INSTRUCTIONS.md                       [This file]
├── documents/
│   ├── QUICK_START.md                         [Assembly checklist]
│   ├── ROVER_SYSTEM_OVERVIEW.md               [System architecture]
│   ├── WIRING_GUIDE.md                        [Wiring instructions]
│   ├── ARDUROVER_PARAMETERS.md                [Firmware config]
│   ├── RTK_SETUP_GUIDE.md                     [RTK integration]
│   ├── TROUBLESHOOTING_GUIDE.md               [Diagnostics]
│   ├── SYSTEM_ARCHITECTURE.md                 [Mermaid diagrams]
│   ├── SYSTEM_ARCHITECTURE.drawio             [DrawIO version]
│   └── INDEX.md                               [Doc navigation]
```

---

## Success! 🎉

Once complete, your maelAutoMower repository will be:

✅ **Public** - Shareable with others
✅ **Organized** - Professional structure
✅ **Well-documented** - 91+ pages of guides
✅ **Version controlled** - Track all changes
✅ **GitHub-hosted** - Available worldwide
✅ **Linkable** - Easy to share via URL

---

## Questions?

If you get stuck:
1. Check this file again (Troubleshooting section)
2. Visit: https://docs.github.com/
3. Search: "git [your issue]" on Google
4. GitHub Community Forums: https://github.community/

---

**Created**: 2026-03-22
**For**: maelAutoMower autonomous rover project
**Email**: maeldiy@gmail.com
