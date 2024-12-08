---
title: "Git Grammar Check"
date: 2024-12-01T22:41:12+01:00
draft: true
ShowToc: true

section: "blog"
url: "/blog/grammar-pre-push"

tags: []
categories: []
keywords: []

params:
  author: "El Viajero"

summary: " "
  
cover:
    image: "cover_grammar.png"
    alt: "Cover image"
    relative: false
    hidden: false
    linkFullImages: true
    responsiveImages: false
editPost:
  URL: "mailto:cysecroad@gmail.com"
  Text: "Suggest Changes" # edit text
  appendFilePath: true # to append file path to Edit link
---

# 

## Why we want to do this

## How are we going to do this

### 

First, locally create the workflow folders inside your project directory
```bash
# cd to you project direcrory
mkdir -p .github/workflows
# Create and edit the workflow file
vim .github/workflows/quality-check.yml
```
Then, edit the workflow configuration
```yaml
# Name of the workflow - this will appear in GitHub Actions tab
name: Spell and Grammar Check

# Define when this workflow should run
on:
  pull_request:
    # Only run on PRs targeting the develop branch
    branches: [ develop ]
    # Specify which pull request actions should trigger this
    # opened: when PR is created
    # synchronize: when new commits are pushed to the PR
    # reopened: when a closed PR is reopened
    types: [opened, synchronize, reopened]

# Define the jobs that this workflow will run
jobs:
  # Job name is 'spellcheck'
  spellcheck:
    # Use the latest Ubuntu runner provided by GitHub
    runs-on: ubuntu-latest
    
    # List of steps to execute in this job
    steps:
      # Step 1: Check out the repository code
      - uses: actions/checkout@v3

      # Step 2: Set up Node.js environment
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'  # Specify Node.js version

      # Step 3: Install cspell globally
      - name: Install cspell
        run: npm install -g cspell

      # Step 4: Install write-good globally
      - name: Install write-good
        run: npm install -g write-good

      # Step 5: Run the spell checker
      - name: Run spell check
        run: |
          echo "Running spell check..."
          # Check all markdown, mdx, and txt files
          # Exit with error (exit 1) if spell check fails
          cspell "**/*.{md,mdx,txt}" || exit 1

      # Step 6: Run the grammar checker
      - name: Run grammar check
        run: |
          echo "Running grammar check..."
          # Find all markdown, mdx, and txt files and check each one
          # The '|| true' ensures the workflow continues even if write-good finds issues
          # because we want grammar issues to be warnings, not errors
          find . -type f -name "*.md" -o -name "*.mdx" -o -name "*.txt" | while read file; do
            echo "Checking $file"
            write-good "$file" || true
          done
```

Then, create a customized *cspell* configuration file to customize spell checker
```bash
npm init -y
npm install --save-dev cspell
```
Then, create *cspell.json*
```json
{
  "version": "0.2",
  "language": "en",
  "words": [],
  "flagWords": [],
  "ignorePaths": [
    "node_modules/**"
  ]
}
```

### Testing if this works

What should we do after completing every little part of making changes? **Verify** it works, of course.

1. First, make sure to push the workflow file to repository (it's okay to do this on develop branch)
    ```bash
    git add .github/workflows/quality-check.yml
    git commit -m "Add spell check workflow"
    git push origin your-current-branch
    ```
2. Create a test branch with some intentional spelling mistakes:
    ```bash
    # Create and checkout a new branch
    git checkout -b test-spellcheck

    # Create a test markdown file with some spelling mistakes
    echo "This is a testt documment with misspelled wurds." > test-post.md
    ```
3. Then push this branch
    ```bash
    git add test-post.md
    git commit -m "Add test post with spelling mistakes"
    git push origin test-spellcheck
    ```
4. Now, on GitHub repository
   1. Click on "Pull requests"
   2. Click "New pull request"
   3. Set "base" to develop
   4. Set "compare" to test-spellcheck
   5. Create the pull request
5. After creating the PR, you can check the workflow results
   - Go to the PR page
   - Look for the "Checks" tab or scroll down to see the GitHub Actions status
   - Click on the "Details" link next to the spell check workflow to see the full output


## Local merging spell check

We want to set this spell check in a way that when we do merge locally, it's invoked and mergin won't be available until this check passes.\
For this, we will need to use Git hooks, as Git Actions only works for remote/cloud checks.

1. First, install all the required packages
    ```bash
    npm install --save-dev cspell write-good
    ```
2. Create the hooks directory if it doesn't exist
    ```bash
    mkdir -p .git/hooks
    ```
3. Create pre-merge-commit hook and make it executable
    ```bash 
    vim .git/hooks/pre-merge-commit
    ```

    Now , copy the following into the hook
    ```bash
    #!/bin/bash

    # This script runs automatically when you try to merge branches

    # Get the name of the current branch we're on
    # git rev-parse --abbrev-ref HEAD returns the short name of the current branch
    CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)

    # The branch we want to protect with spell checking
    # Change this if you want to protect a different branch
    TARGET_BRANCH="develop"

    # Only run the checks when we're merging into the target branch (develop)
    if [ "$CURRENT_BRANCH" = "$TARGET_BRANCH" ]; then
        echo "Running spell and grammar checks before merge..."
        
        # Get a list of all markdown files that are staged for commit
        # git diff --name-only : List changed files
        # --cached : Look at staged files
        # --diff-filter=ACM : Only look at Added, Changed, or Modified files
        # grep -E '\.(md|mdx|txt)$' : Only look at markdown and text files
        FILES=$(git diff --name-only --cached --diff-filter=ACM | grep -E '\.(md|mdx|txt)$')
        
        # If we found any markdown files to check
        if [ -n "$FILES" ]; then
            # Run spell check using cspell
            # The '!' means "if the command fails"
            # npx ensures we use the project's installed version of cspell
            if ! npx cspell $FILES; then
                echo "❌ Spell check failed! Please fix the spelling errors before merging."
                # Exit with error code 1 to prevent the merge
                exit 1
            fi
            
            # Run grammar check using write-good
            # We run this even if spell check fails
            # '|| true' means the script continues even if write-good finds issues
            echo "Grammar check (warnings):"
            for file in $FILES; do
                npx write-good "$file" || true
            done
            
            # If we got here, spell check passed
            echo "✅ Spell check passed!"
            echo "⚠️  Review any grammar warnings above"
        else
            # No markdown files were changed
            echo "No markdown files to check."
        fi
    fi

    # Exit successfully - allow the merge to proceed
    exit 0
    ```

    ```bash
    chmod +x .git/hooks/pre-merge-commit
    ```
4. Now, when you try to merge locally
    ```bash
    git checkout develop
    git merge test-spellcheck
    ```

## Updates

### Updating cspell configuration file

We can edit the configuration's a bit, so it won't be jump simple english checking
```json
{
    // Version of the config file format
    "version": "0.2",
    
    // List of languages to use for spell checking
    "language": "en,es",
    

    // Enable mixing languages within the same file
    "enableFiletypes": ["*"],
    "enabledLanguageIds": ["*"],
    "allowCompoundWords": true,

    // Import Spanish dictionary
    "import": [
        "@cspell/dict-es-es/cspell-ext.json"
    ],

    // Words to always consider correct
    "words": [
        "CySecRoad"  // Your project name
        // Add your custom words here
        "ciberseguridad",
        "cybersecurity",
        "pentesting",
        "hacking"
    ],

    // Dictionaries to load
    "dictionaries": [
        "en_US",      // American English
        "es",         // Spanish
        "es-ES",      // Spanish (Spain)
        "companies",  // Company names
        "softwareTerms", // Programming terms
        "misc"        // Miscellaneous terms
    ],

    // Words to always consider incorrect
    "flagWords": [
        "irregardless",
        "alot"
    ],

    // Ignore files and folders
    "ignorePaths": [
        "node_modules/**",
        ".git/**",
        "public/**",
        "dist/**"
    ],

    // File patterns to check
    "files": [
        "**/*.md",
        "**/*.mdx",
        "**/*.txt"
    ],

    // Allow acronyms that are uppercase
    "allowCompoundWords": true,

    // Maximum number of problems to report
    "maxNumberOfProblems": 1000,

    // Maximum word length
    "maxDuplicateProblems": 100,

    // Minimum word length to check
    "minWordLength": 3,

    // No more language-specific overrides since we want mixed language support
    "languageSettings": [
        {
            "languageId": "*",
            "locale": "*",
            "allowCompoundWords": true,
            "enabledLanguageIds": ["en", "es"]
        }
    ]
}
```

## Making this all make sense

We wan't to do this, so we won't have any weird spelling errors on our blog.\
Because of this, we should also make it possible to merge only develop with main branch\
To do this locally, add following code to the `.git/hooks/pre-merge-commit`
```bash
# Protection for main branch
SOURCE_BRANCH=$(git rev-parse --abbrev-ref MERGE_HEAD)

# If we're on main branch
if [ "$CURRENT_BRANCH" = "main" ]; then
    # Check if the merge is coming from develop
    if [ "$SOURCE_BRANCH" != "develop" ]; then
        echo "❌ ERROR: You can only merge into main from develop branch"
        echo "Current merge attempt: $SOURCE_BRANCH -> main"
        exit 1
    fi
fi
```

Remotely, we will set up pull request checking:
1. Go to your GitHub repository
2. Click on "Settings"
3. In the left sidebar, click "Branches"
4. Under "Branch protection rules" click "Add rule"
5. Set up the rule:
   1. First, give your ruleset a name
   2. In "Target branches" section, add target annd add `main` as target branch pattern
   3. In the "Rules" section, you should enable:
      - "Require a pull request before merging"
      - "Block force pushes" 
      - "Restrict deletions" 
   4. Under "Require a pull request before merging" settings:
      - Set "Required approvals" to at least 1 if you want reviews
      - Set "Dismiss stale pull request approvals when new commits are pushed" 
      - If you have code owners, keep "Require review from Code Owners" checked

## Running spellcheck locally and overruling the merge 

### Local check

### Overruling the merge

To make a merge, even with misspelled word, you can do the following\
`--no-verify` flag skips all pre-merge hooks
```bash
git merge --no-verify branch-name
```
