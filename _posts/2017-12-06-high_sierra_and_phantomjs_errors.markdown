---
layout: post
title:      "High Sierra and phantomjs errors"
date:       2017-12-07 01:20:56 +0000
permalink:  high_sierra_and_phantomjs_errors
---


I was running a test that used Capybara/Poltergeist and was getting an error that Cliver couldn't find the phantomjs version required. 


```
Cliver::Dependency::VersionMismatch:
            Could not find an executable 'phantomjs' that matched the requirements '>= 1.8.1', '< 3.0'. Found versions were {"/usr/local/bin/phantomjs"=>"325"}.
```

What kind of version is 325 for phantomjs? 

It's not. I tried getting the version of phantomjs with a simple `phantomjs -v` but that gave me a spawn ENOTDIR error, and 325 was the line of code that was throwing the error. 

After a bit of reasearch it turns out that after the High Sierra OSX update the phantomjs binary wasn't globally available. 

The easy fix was a new global install of phantomjs:
```
npm install -g phantomjs
```

Hope this helps if anyone is bumping into the same problem.
