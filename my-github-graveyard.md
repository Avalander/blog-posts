---
title: My GitHub Graveyard
published: false
description: 
tags: 
cover_image: https://images.unsplash.com/photo-1464753636159-34a03ff9cb02?ixlib=rb-0.3.5&ixid=eyJhcHBfaWQiOjEyMDd9&s=9ef0e9368bf3d7e1154545aac1770fa7&auto=format&fit=crop&w=2550&q=80
---

Following [Isaac](https://dev.to/isaacandsuch/github-graveyards-ill-show-you-mine-49lh) and [K](https://dev.to/kayis/my-github-graveyards-2l95), I've decided to post some of my abandoned projects on GitHub.

# [Task List](https://github.com/Avalander/task-list)

This is a project that I actually completed. It is a simple todo list and the goal was to experiment with progressive web apps. It works entirely offline (it doesn't even have a backend) and it uses a service worker and IndexedDB to store source files and data.

# [Restventures](https://github.com/Avalander/restventures)

The idea behind Restventures was to build a classic text adventure game but instead of text commands, the player would interact with the game through REST endpoints. It never took off, but I still think that the concept is cool. I might continue it some day.

# [GoBot](https://github.com/Avalander/GoBot)

This was my first and last attempt at building something with Go. It is a discord bot and the plan was to implement several text commands that would do cool things. The only command I ever got around implementing was `uptime`, which writes how long the bot has been up. Even though I didn't get very far, I learned how to build and deploy a Go server.

# [Strawberry Alarm](https://github.com/Avalander/strawberry-alarm)

This is another completed project. It is a game that I submitted to [Game Dev's week of awesome](https://www.gamedev.net/calendar/event/95-week-of-awesome-v/) contest. The interesting part is that I built it using functional reactive programming (well, sorta, it's still hackathon code). I wrote a few posts on [my blog on Game Dev](https://www.gamedev.net/blogs/blog/2227-avalanders-journal/) about the process and a post-mortem.

# [Gandalf](https://github.com/Avalander/Gandalf)

Gandalf is a wizard (hence the name) to manage shell scripts. Basically, instead of typing `nano /usr/local/bin/my-script`, I type `gandalf create my-script -e nano` (any editor can be specified via the `-e` parameter). It allows creating templates and including templates in scripts, a template being a file or directory to which the script has access.

Originally I wanted to be able to use it to write customisable boilerplates for my projects, but I never made it sophisticated enough for that. I still use it for simpler scripts though, because it helps me distinguish the scripts that I have created from the scripts that third party packages have installed in my `/usr/local/bin` directory, which is convenient when cleaning up old stuff.

# [Elm Boilerplate](https://github.com/Avalander/elm-boilerplate)

This is a template project for Elm projects with webpack. Nothing very fancy, I just got tired of typing the same webpack config files every time I started a new project.
