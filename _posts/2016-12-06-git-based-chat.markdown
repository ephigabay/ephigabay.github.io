---
layout: post
title:  "GIT Based Chat"
date:   2016-12-06
---

A while ago I noticed the similarity between how a git repository is structured to a chat. So I decided to write the completely not useful yet funny version of a chat client that uses git server as the backend server.

This chat client uses the branches in a git repository as the channels in the chat, and commit messages as actual messages.

You can clone the client from here: https://github.com/ephigabay/GIC

In order to run it you should run  `npm install` in the directory you cloned the client to, edit the  `config.js` file and enter a git repository you have access to and then run the client using  `npm start`.

**Be aware that this client will create branches and add commits to the repository you entered there, so be careful.**