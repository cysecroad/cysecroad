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

