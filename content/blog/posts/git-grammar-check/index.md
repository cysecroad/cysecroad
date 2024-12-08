---
title: "When Git Becomes Your Blog Editor: Automated Grammar Checking for Tech Writers"
date: 2024-12-09T09:00:00+01:00
draft: false
ShowToc: true

section: "blog"
url: "/blog/grammar-pre-push"

tags: ["git", "automation", "blogging", "documentation", "spell-check", "github-actions", "git-hooks"]
categories: ["tutorials", "automation", "git"]
keywords: ["git grammar check", "github actions tutorial", "blog automation", "spell check automation", "git hooks tutorial", "documentation tools", "markdown spell check", "technical writing", "blog tools", "cspell configuration", "write-good usage", "git pre-merge hooks"]

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

# When Git Becomes Your Blog Editor: Automated Grammar Checking for Tech Writers

¡Hola amigos! Ever sent an important email to notice that embarrassing typo right after hitting send? Now imagine doing that with your technical documentation that thousands of developers might read. Or submitting a PR with working code, only to have someone comment "typo in line 47" and nothing else? **Or starting your own blog and…** *¡Qué horror!*

I know this is still not cybersecurity, but *despacio*, we will get there. We have to set up our blog first.

## Why we want to do this

You know what's funny about starting a tech blog? You spend hours perfecting your code examples, making sure every technical detail is spot-on, and then someone comments "you wrote 'recieve' instead of 'receive' in the first paragraph." *¡Ay, caramba!* **We don't want this to happen**\
After writing my first couple of blog posts about my journey from verification to cybersecurity, I noticed I was spending almost as much time proofreading as writing. And still, those sneaky typos kept slipping through!\
I decided to do what any reasonable developer would: automate the problem away.

Since I'm planning to write more about my cybersecurity adventures (*aventuras*, as we say in Spanish), I needed something better than just staring at my screen hoping to catch all typos. And being the tech person I am, I thought - why not make Git do the work?

Here's what we're building today:

- A GitHub Action that catches typos before they go public
- A local pre-merge hook for immediate feedback
- A bilingual setup because, well, you've seen my Spanish learning attempts in these posts

## How are we going to do this

Let's break this down into manageable pieces. And no, I won't make a cooking analogy here - I'm actually writing this while hungry, and that would just be cruel. 🥘

### Setting Up GitHub Actions

First, let's create our workflow structure. Remember, good organization is like good code - it makes life easier later:
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

      # Install dependencies including the Spanish dictionary
      - name: Install dependencies
        run: |
          npm install
          npm install --save-dev @cspell/dict-es-es  
          npm install --save-dev textlint textlint-rule-write-good textlint-rule-no-dead-link \
            textlint-rule-terminology textlint-rule-period-in-list-item \
            textlint-rule-no-start-duplicated-conjunction @textlint/textlint-plugin-markdown
          pip install proselint

      # Step 5: Run the spell checker
      - name: Run spell check
        run: |
          echo "Running spell check..."
          # Check all markdown, mdx, and txt files
          # Exit with error (exit 1) if spell check fails
          cspell "**/*.{md,mdx,txt}" || exit 1

      # Step 6: Run the grammar checker
      - name: Run textlint
        run: |
          echo "Running textlint check..."
          find . -type f -name "*.md" -o -name "*.mdx" -o -name "*.txt" | while read file; do
            echo "Checking $file"
            npx textlint "$file" || true
          done

      - name: Run proselint
        run: |
          echo "Running proselint check..."
          find . -type f -name "*.md" -o -name "*.mdx" -o -name "*.txt" | while read file; do
            echo "Checking $file"
            proselint "$file" || true
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

### Testing if this works (As we use to say it *trust but verify*)

As any verification engineer would tell you (and boy, do we love telling people this), if you haven't tested it, it doesn't work! Let's intentionally break stuff - it's the only way to be sure our checks are working.

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

While GitHub Actions are great, waiting for the cloud to tell you about typos is like waiting for your code to compile - ain't nobody got time for that! Let's set up local checks.

We want to set this spell check in a way that when we do merge locally, it's invoked and merging won't be executed until this check passes.\
For this, we will need to use Git hooks, as Git Actions works only for remote/cloud checks.

1. First, install all the required packages
    ```bash
    # Install cspell
    npm install --save-dev cspell @cspell/dict-es-es 

    # Install textlint and common rules
    npm install --save-dev \
      textlint \
      textlint-rule-write-good \
      textlint-rule-no-dead-link \
      textlint-rule-terminology \
      textlint-rule-period-in-list-item \
      textlint-rule-no-start-duplicated-conjunction \
      @textlint/textlint-plugin-markdown

    # Install proselint
    pip install proselint
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
4. Let's create a `.textlintrc` file
    ```json
    {
      "plugins": [
        "@textlint/markdown"
      ],
      "rules": {
        "write-good": {
          "passive": true,
          "tooWordy": true,
          "weasel": true,
          "adverb": true
        },
        "terminology": {
          "defaultTerms": true
        },
        "period-in-list-item": true,
        "no-start-duplicated-conjunction": {
          "interval": 2
        },
        "no-dead-link": {
          "ignore": [
            "mailto:",
            "http://localhost"
          ]
        }
      }
    }
    ```
5. When you try to merge locally
    ```bash
    git checkout develop
    git merge test-spellcheck
    ```

## Updates: Making it Más Interesante

### Updating cspell configuration file: Because We Speak Spanglish

Remember how I mentioned I'm learning Spanish after that trip to Madrid? Well, might as well make our spell checker bilingual too. Who knows, maybe it'll help with my Spanish blog posts in the future (yes, that's coming - *próximamente*).

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

## Running spell check locally and overruling the merge checks

### Local check: For the Impatient Developers

Sometimes you just want to check things quickly without going through the whole merge process. Here's how:

```bash
# Check a single file
npx cspell path/to/your/file.md

# Check multiple specific files
npx cspell file1.md file2.md

# Check all markdown files in a directory
npx cspell "**/*.md"
```

To check now
```bash
# Run textlint
npx textlint your-file.md

# Run proselint
proselint --json your-file.md
proselint your-file.md

# Run both
npx textlint your-file.md && proselint your-file.md
```


### Overruling the merge checks: The Nuclear Option

Sometimes you need to merge something urgently, even with typos. Like when it's Friday at 4:55 PM and you really want to start your weekend. (I'm not judging, we've all been there!) Here's how to do it:
```bash
# --no-verify flag skips all pre-merge hooks
git merge --no-verify branch-name
```

Remember, with great power comes great responsibility. Use this like you'd use sudo - only when you really need to!

## Pro Tips from a Reformed Typo Offender

- Add your technical terms to cspell.json immediately - trust me, you don't want to fix "cybersecurity" 50 times
- Run checks locally before pushing - it's faster than waiting for GitHub Actions to tell you about that typo you already knew about
- Don't blindly trust the grammar checker - sometimes "incorrect" grammar is more readable
- Keep your custom word list somewhere safe - rebuilding it after a fresh clone isn't fun

**¡Y eso es todo, mis amigos!**

Remember, perfect grammar won't fix broken code, but it might make your readers focus on what you're actually trying to say. And isn't that the whole point of writing?\
Happy coding, *y que tus pull requests sean siempre aprobados!*[^1] 🚀\
P.S. If you found any typos in this post... well, clearly I need to update my `cspell.json`! 😉
[^1]: And let your pull requests be always approved