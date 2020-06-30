---
title: Jumping into TypeScript 
date: '2020-06-29T23:36:50Z'
description: Rewriting koa with TypeScript 
---

I have spent a significant amount of my career writing JavaScript, but have not 
put the investment into learning TypeScript in depth.  So, I sought after a 
reasonably sized project to gain more exposure and hands-on experience with the 
language.

After some research I settled on attempting a rewrite of [KoaJS](https://koajs.com/).
The 3.0 roadmap of the project indicated some interest of potentially rewriting 
the project with TypeScript from even a couple years back.  This interested me 
as it could provide an opportunity to give back to a project I have used rather 
extensively and allowed for me to focus purely on learning TypeScript due to my
familiarity with Koa.

After chipping away at this effort on and off for a few weeks I reached a 
reasonable stopping point.  I rewrote the implementation in TypeScript and 
ported the tests from mocha to jest.

Overall, I found the experience rather pleasant when writing TypeScript.  The 
use of types greatly added to the clarity of the code, eliminated ambiguity, 
and created safer code with guarded type checks.  This resulted in a Koa 
implementation that I would argue is easier to maintain, is more resilient to 
misuse of consumers by inappropriately utilizing the API, and I even believe I 
caught a few bugs along the way.

Unfortunately, after presenting my side project to the Koa community via a 
GitHub issue, the maintainer of the project did not find a TypeScript rewrite 
as something beneficial and closed my issue.  So, I will shelve this for now, 
but at least it served as great learning experience for working with TypeScript 
on a reasonably sized project from start to finish.

You can find koa-ts at: [https://github.com/kubejm/koa-ts](https://github.com/kubejm/koa-ts)
