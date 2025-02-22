---
title: Github Action Templates
date: 2024-05-03 16:10:31
tags:
---

---

```bash
act -W .\.github\workflows\pipeline.yml --secret-file .\.secrets -e .\event.json
```

## event

```yaml
name: print github context

on:
  push:
    branches:
      - main

jobs:
  a_test_job:
    runs-on: ubuntu-20.04
    steps:
      - uses: actions/checkout@v4

      - name: github context
        env:
          GITHUB_CONTEXT: ${{ toJson(github) }}
        run: echo "$GITHUB_CONTEXT"

      - name: commits
        env:
          COMMITS: ${{ toJson(github.event.commits) }}
        run: echo "$COMMITS"

      - name: commit messages
        env:
          COMMIT_MESSAGES: ${{ toJson(github.event.commits.*.message) }}
        run: echo "$COMMIT_MESSAGES"
```

## skip commit

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: build
        run: echo 'run build'

      - name: deploy
        if: ${{ !contains(github.event.commits[0].message, '#skip') }}
        run: echo 'skip this'
```

- `commit[0].message` is the oldest commit

- `head_commit.message` is the youngest commit

## test

```yaml
- name: Generate 0 or 1
  id: generate_number
  run: echo "random_number=$(($RANDOM % 2))" >> $GITHUB_OUTPUT
- name: Pass or fail
  run: |
    if [[ ${{ steps.generate_number.outputs.random_number }} == 0 ]]; then exit 0; else exit 1; fi
```

---

[Github action test if a commit containing a specific word was previously made](https://stackoverflow.com/a/71364384/6575354)
