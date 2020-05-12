---
title: Small task for iggy 
date: "2020-05-12T01:07:02Z"
description: Quickly generate .gitignore files
---

Often when I create projects I end up copying `.gitignore` files from past 
projects or finding one on GitHub.  Even though this task may take a matter of 
minutes, it can add up to a sizable amount of time as I create more projects.  
Similarly, I often find myself buried within my terminal, so I desired a small 
application to this accomplish for me.  After a quick google search I did not 
find an application that looked overly appealing to me, so I tossed together 
iggy quickly.  Even though simple, this application will allow me to build an 
inventory of `.gitignore` files by language and generate them into my current 
directory.

Listed below is an example usage of iggy to generate a `.gitignore` for a 
JavaScript or TypeScript project.

```bash
~/sandbox/sample-project
➜ exa -la

~/sandbox/sample-project
➜ iggy -g js

~/sandbox/sample-project
➜ exa -la
.rw-rw-r-- 217 matt 11 May 21:19 .gitignore

~/sandbox/sample-project
➜ cat .gitignore
# Coverage reports
coverage

# API keys and secrets
.env

# Dependency directory
node_modules

# Editors
.idea
*.iml

# OS metadata
.DS_Store
Thumbs.db

# Ignore built ts files
dist/**/*

# ignore yarn.lock
yarn.lock
```

I only included a few `.gitignore` files for languages I use most frequently 
and plan to build up the inventory as necessary over time.

Listed below shows the currently available `.gitignore` files in iggy.

```bash
➜ iggy -l
java
js
kt
```

You can find iggy at: [https://github.com/kubejm/iggy](https://github.com/kubejm/iggy)
