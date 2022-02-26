---
title: 'Run tests from networked location'
date: 2012-07-25 00:13
draft: false
tags: ['Visual Studio', 'Testing']
summary: 'Running Visual Studio tests from a network location'
---

I use Parallels on OSX to run Visual Studio inside a VM.

With a test project I had written, I was getting the following exception thrown

> Unit Test Adapter threw exception: URI formats are not supported..
> Which was highly annoying!

After some googling, I found the <a href="http://www.seleniumwiki.com/visual-studio-2010/unit-test-adapter-threw-exceptionuri-formats-are-not-supported/">solution</a>:

- Double click on Local.testsettings which is under the Solution Items of Project ![1]
- Test Settings window is displayed. Click on the Deployment link.
- You will see a checkbox Enable Deployment. Select the checkbox and click Apply. ![2]
- The Test Settings can also be found under TraceAndTestImpact.testsettings and just follow the same steps.

[1]: http://i.imgur.com/Ip6YP.png
[2]: http://i.imgur.com/c1lUG.png
