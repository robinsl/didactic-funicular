# Jujutsu (jj) VCS: Complete Learning Guide

## Table of Contents
1. [Quick Summary and Ecosystem Overview](#part-1-quick-summary-and-ecosystem-overview)
2. [Differences with Git](#part-2-differences-with-git)
3. [Deep Dive: Lifecycle and Features](#part-3-deep-dive-lifecycle-and-features)
4. [Complex Real-World Example](#part-4-complex-real-world-example)

---

## Part 1: Quick Summary and Ecosystem Overview

### What is Jujutsu?

Jujutsu (jj) is a modern, Git-compatible version control system designed to be both simple and powerful. It abstracts the user interface and version control algorithms from the storage systems, making it fundamentally different from traditional VCS tools.

### Key Philosophy

- **Working-copy-as-a-commit**: Changes are automatically recorded as commits
- **Conflict-first design**: Conflicts are first-class objects, not just textual diffs
- **Operation log**: Every operation is recorded and can be undone
- **Automatic rebase**: Descendants are automatically rebased when you modify commits

### Quick Start Commands

```bash
# Initialize a new repository
jj git init my-project
cd my-project

# Clone an existing Git repository
jj git clone https://github.com/user/repo.git

# Basic workflow
echo "Hello World" > file.txt
jj describe -m "Add hello world"  # Set commit message
jj new                            # Create new change
echo "More content" >> file.txt
jj describe -m "Add more content"

# View history
jj log

# Push to remote
jj git push
```

### Ecosystem and Tools

#### Core Components
- **jj CLI**: The main command-line interface
- **Git backend**: Uses Git repositories for storage (production-ready)
- **Operation store**: Records all operations for undo/redo functionality

#### Integration
- **Git compatibility**: Full interoperability with Git repositories and workflows
- **GitHub/GitLab**: Works seamlessly with Git forges
- **Co-located repos**: Can use both `jj` and `git` commands in the same repository

#### Community Tools
- **Editor integrations**: VS Code, Vim, Emacs plugins available
- **Shell completions**: Bash, Zsh, Fish support
- **CI/CD**: Works with existing Git-based CI systems

### Installation

```bash
# macOS
brew install jj

# Linux (cargo)
cargo install --git https://github.com/jj-vcs/jj.git jj-cli

# Windows
winget install jj-vcs.jj
```

---

## Part 2: Differences with Git

### Conceptual Differences

| Concept | Git | Jujutsu |
|---------|-----|---------|
| **Working Directory** | Separate from commits | Is a commit (automatically managed) |
| **Staging Area** | Explicit index/staging | No staging area - direct commits |
| **Branches** | Named references | Anonymous changes (bookmarks for naming) |
| **Conflicts** | Blocking operations | First-class objects, operations continue |
| **History Rewriting** | Complex, error-prone | Safe, automatic rebasing |
| **Undo** | Limited (reflog) | Complete operation log |

### Command Comparison

#### Basic Operations
```bash
# Git vs Jujutsu equivalents

# Initialize
git init                    # jj git init
git clone <url>            # jj git clone <url>

# Status and changes
git status                 # jj status
git diff                   # jj diff
git log                    # jj log

# Committing
git add .                  # (automatic in jj)
git commit -m "msg"        # jj describe -m "msg"

# Branching
git checkout -b feature    # jj new -m "feature work"
git checkout main          # jj edit main

# Merging
git merge feature          # jj rebase -d main (automatic)
git rebase main            # jj rebase -d main

# Remote operations
git push                   # jj git push
git pull                   # jj git fetch && jj rebase -d @-
```

#### Advanced Operations
```bash
# History editing
git rebase -i HEAD~3       # jj split / jj squash / jj describe
git commit --amend         # jj describe (works on any commit)
git cherry-pick <commit>   # jj rebase -r <commit> -d <dest>

# Conflict resolution
git merge --abort          # jj restore (conflicts don't block)
git rerere                 # (automatic conflict resolution propagation)

# Undo operations
git reflog                 # jj op log
git reset --hard HEAD~1    # jj undo
```

### Workflow Differences

#### Git Workflow
1. Make changes
2. Stage changes (`git add`)
3. Commit changes (`git commit`)
4. Push to remote (`git push`)
5. Handle conflicts manually
6. Complex history rewriting

#### Jujutsu Workflow
1. Make changes (automatically snapshotted)
2. Set commit message (`jj describe`)
3. Create new change (`jj new`)
4. Push to remote (`jj git push`)
5. Conflicts are preserved and can be resolved later
6. History rewriting is safe and automatic

### Mental Model Shift

**Git**: Think in terms of snapshots, branches, and manual staging
**Jujutsu**: Think in terms of a continuous stream of changes with automatic management

---

## Part 3: Deep Dive: Lifecycle and Features

### The Jujutsu Lifecycle

#### 1. Repository Initialization
```bash
# Create new repository
jj git init my-project

# Repository structure created:
# .jj/           - Jujutsu metadata
# .git/          - Git backend storage
```

#### 2. Working Copy Management
```bash
# Working copy is always a commit
jj status                  # Shows current working copy commit

# Make changes - automatically snapshotted
echo "content" > file.txt

# Working copy commit is automatically updated
jj log -r @                # @ represents working copy
```

#### 3. Change Creation and Management
```bash
# Create new change on top of current
jj new -m "New feature"

# Edit specific commit
jj edit <commit-id>

# Describe any commit
jj describe -r <commit-id> -m "Better message"
```

#### 4. History Manipulation
```bash
# Split a commit
jj split -r <commit-id>

# Squash changes
jj squash -r <source> -d <dest>

# Move changes between commits
jj move --from <source> --to <dest>
```

### Core Features Deep Dive

#### 1. Automatic Rebase
When you modify a commit, all descendants are automatically rebased:

```bash
# Initial state: A -> B -> C
jj describe -r B -m "Updated B"
# Result: A -> B' -> C' (C automatically rebased onto B')
```

#### 2. Conflict Management
Conflicts don't block operations:

```bash
# Create conflicting changes
jj new -m "Change 1"
echo "version 1" > file.txt
jj new -m "Change 2" 
echo "version 2" > file.txt

# Rebase creates conflict but doesn't fail
jj rebase -r @ -d @--

# Conflict is recorded in the commit
jj log  # Shows conflict markers
jj resolve  # Interactive conflict resolution
```

#### 3. Operation Log and Undo
Every operation is recorded:

```bash
# View operation history
jj op log

# Undo last operation
jj undo

# Undo specific operation
jj undo --op <operation-id>

# Restore to specific operation
jj op restore <operation-id>
```

#### 4. Revsets (Revision Selection)
Powerful query language for selecting commits:

```bash
# Select commits
jj log -r "main"           # Main bookmark
jj log -r "@"              # Working copy
jj log -r "@ | @-"         # Working copy and parent
jj log -r "ancestors(@)"   # All ancestors
jj log -r "descendants(main)" # All descendants of main
jj log -r "author(alice)"  # Commits by alice
jj log -r "description(fix)" # Commits with "fix" in message
```

#### 5. Bookmarks (Branch Management)
```bash
# Create bookmark
jj bookmark create feature

# Move bookmark
jj bookmark set main -r <commit-id>

# List bookmarks
jj bookmark list

# Delete bookmark
jj bookmark delete feature
```

#### 6. Templates (Output Formatting)
Customize output format:

```bash
# Custom log format
jj log -T 'commit_id.short() ++ " " ++ description.first_line()'

# Configure default template
jj config set --user templates.log_oneline 'commit_id.short() ++ " " ++ description.first_line()'
```

### Advanced Features

#### 1. Sparse Checkouts
```bash
# Configure sparse patterns
jj sparse set 'src/**' 'docs/**'

# List sparse patterns
jj sparse list
```

#### 2. Concurrent Operations
Jujutsu is designed for safe concurrent access:
- Multiple users can work on the same repository
- Operations are atomic and safe
- Conflicts are resolved automatically when possible

#### 3. Custom Backends
While Git is the primary backend, Jujutsu's architecture supports:
- Future cloud-based backends
- Custom storage systems
- Hybrid approaches

---

## Part 4: Complex Real-World Example

### Scenario: Feature Development with Team Collaboration

Let's walk through a complex real-world scenario involving multiple developers, feature branches, conflicts, and history rewriting.

#### Setup: Multi-Developer Project
```bash
# Project lead initializes repository
jj git init web-app
cd web-app

# Create initial structure
mkdir -p src/{components,utils,api}
echo "# Web App" > README.md
echo "console.log('Hello World');" > src/main.js
jj describe -m "Initial project structure"

# Create remote repository and push
jj git remote add origin https://github.com/team/web-app.git
jj bookmark create main
jj git push --bookmark main
```

#### Developer A: Authentication Feature
```bash
# Clone and start feature
jj git clone https://github.com/team/web-app.git
cd web-app

# Create feature branch
jj new -m "Start authentication feature"
jj bookmark create auth-feature

# Implement authentication
mkdir src/auth
cat > src/auth/login.js << 'EOF'
export function login(username, password) {
    // TODO: Implement login logic
    return fetch('/api/login', {
        method: 'POST',
        body: JSON.stringify({ username, password })
    });
}
EOF

jj describe -m "Add login function skeleton"

# Continue development
cat > src/auth/auth.js << 'EOF'
import { login } from './login.js';

export class AuthManager {
    constructor() {
        this.token = localStorage.getItem('token');
    }
    
    async authenticate(username, password) {
        const response = await login(username, password);
        if (response.ok) {
            this.token = await response.text();
            localStorage.setItem('token', this.token);
        }
        return response.ok;
    }
}
EOF

jj new -m "Add AuthManager class"

# Realize we need to split the commit
jj split -r @-
# Interactive editor opens - select login.js for first commit
# Second commit gets auth.js

jj describe -r @-- -m "Add login function"
jj describe -r @- -m "Add AuthManager class"
```

#### Developer B: UI Components (Parallel Development)
```bash
# Another developer clones
jj git clone https://github.com/team/web-app.git
cd web-app

# Start UI work
jj new -m "Start UI components"
jj bookmark create ui-components

# Create components
mkdir src/components
cat > src/components/Button.js << 'EOF'
export function Button({ children, onClick, type = 'button' }) {
    return `<button type="${type}" onclick="${onClick}">${children}</button>`;
}
EOF

jj describe -m "Add Button component"

# Add more components
cat > src/components/Form.js << 'EOF'
import { Button } from './Button.js';

export function LoginForm() {
    return `
        <form id="login-form">
            <input type="text" id="username" placeholder="Username" />
            <input type="password" id="password" placeholder="Password" />
            ${Button({ children: 'Login', type: 'submit' })}
        </form>
    `;
}
EOF

jj new -m "Add LoginForm component"
```

#### Integration and Conflict Resolution
```bash
# Developer A pushes first
jj git push --bookmark auth-feature

# Developer B fetches and integrates
jj git fetch
jj rebase -d origin/auth-feature

# Now both features are integrated, but we need to connect them
jj new -m "Integrate auth with UI"

# Create integration file
cat > src/app.js << 'EOF'
import { AuthManager } from './auth/auth.js';
import { LoginForm } from './components/Form.js';

const auth = new AuthManager();
const loginForm = LoginForm();

document.getElementById('app').innerHTML = loginForm;

document.getElementById('login-form').addEventListener('submit', async (e) => {
    e.preventDefault();
    const username = document.getElementById('username').value;
    const password = document.getElementById('password').value;
    
    const success = await auth.authenticate(username, password);
    if (success) {
        document.getElementById('app').innerHTML = '<h1>Welcome!</h1>';
    } else {
        alert('Login failed');
    }
});
EOF

jj describe -m "Integrate authentication with UI"
```

#### Code Review and Refinement
```bash
# Reviewer suggests improvements
jj new -m "Address code review feedback"

# Fix issues in previous commits without checking them out
jj diffedit -r @---  # Edit the login function commit
# Interactive editor opens showing the diff
# Make improvements directly

# Move some changes to a different commit
jj squash -i --from @ --into @-
# Interactive editor to select which changes to move

# Split a large commit that does too much
jj split -r @--
```

#### Handling Merge Conflicts
```bash
# Meanwhile, main branch has evolved
jj git fetch

# Rebase onto latest main creates conflicts
jj rebase -d origin/main
# Conflicts are recorded but operation succeeds

# View conflicts
jj log -r 'conflict()'

# Resolve conflicts interactively
jj resolve
# Opens merge tool for each conflicted file

# Or resolve manually
jj status  # Shows conflicted files
# Edit files to resolve conflicts
jj describe -m "Resolve conflicts with main"
```

#### History Cleanup Before Merge
```bash
# Clean up commit messages
jj describe -r @---- -m "feat: add user authentication system"
jj describe -r @--- -m "feat: add login UI components"
jj describe -r @-- -m "feat: integrate auth with UI"
jj describe -r @- -m "fix: resolve conflicts with main branch"

# Squash fixup commits
jj squash -r @- -d @--

# Final history looks clean
jj log --oneline -r 'ancestors(@) ~ ancestors(origin/main)'
```

#### Advanced Operations During Development

##### 1. Experimental Branch
```bash
# Try experimental approach
jj new -m "Experiment: OAuth integration"
# ... implement OAuth ...

# Decide to abandon experiment
jj abandon @
# Returns to previous commit, experiment is abandoned but preserved in op log
```

##### 2. Cherry-picking Changes
```bash
# Need a specific fix from another branch
jj rebase -r <commit-from-other-branch> -d @
```

##### 3. Partial Changes
```bash
# Move only part of changes to another commit
jj move --from @ --to @- --interactive
# Interactive selection of hunks to move
```

##### 4. Backup and Recovery
```bash
# Something went wrong, check operation log
jj op log

# Restore to before the problematic operation
jj op restore <operation-id>

# Or undo specific operations
jj undo --op <operation-id>
```

#### Final Integration
```bash
# Push final feature
jj git push --bookmark ui-components

# Create pull request (using GitHub CLI)
gh pr create --title "feat: Add authentication system with UI" \
             --body "Implements user authentication with login form"

# After approval, merge
jj git fetch
jj bookmark set main -r origin/main
jj git push --bookmark main --delete ui-components
```

### Key Takeaways from the Example

1. **Seamless Collaboration**: Multiple developers can work simultaneously without complex merge strategies
2. **Safe History Rewriting**: Commits can be modified, split, and reorganized safely
3. **Conflict Management**: Conflicts don't block progress and can be resolved incrementally
4. **Operation Safety**: Every operation is recorded and can be undone
5. **Git Compatibility**: Full integration with existing Git workflows and tools

### Best Practices Demonstrated

- Use descriptive commit messages with conventional commit format
- Split large commits into logical pieces
- Resolve conflicts incrementally rather than all at once
- Use bookmarks for feature branches
- Leverage automatic rebasing for clean history
- Use operation log for debugging and recovery

This example showcases how Jujutsu's unique features enable more flexible and safer version control workflows compared to traditional Git usage.