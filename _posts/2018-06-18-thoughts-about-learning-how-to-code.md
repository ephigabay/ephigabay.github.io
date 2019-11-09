---
layout: post
title:  "Thoughts About Learning how to Code"
date:   2018-06-18
---

A few months ago a friend of mine asked me to teach him how to code in JavaScript. He learned how to code in C# when he was in high school, so I thought it should be fairly easy. I had never thought how enlighting this experience would be for me.

My first suggestion was that we code something together in react-native, this way we could both learn. We started coding, and most of the times, he could lay the general flow of the code, but using the “magic” of react-native and understanding how everything works as an addition to learning the language was too confusing. It was not possible to understand which “magic” is part of the language and which is from react-native.

Once I understood that learning a language and a framework can be very confusing, I recommended him to solve katas on  [CodeWars](https://codewars.com/). It has an isolated environment he can execute code, and to really understand the language without much stuff that could confuse you.

I was watching him solving the katas, and it went very well. One of my observations, though, was that he had some recurring pitfalls. For example, understanding which functions he can execute on which variables. The concept of variable types in JavaScript, as in most other scripting languages, is very vague. As a novice developer, it is really hard to understand this concept in a not strongly typed language.

It reminded me of the time when I decided to go to college and took a CS 101 class after coding for almost 10 years. They taught us C, which really surprised me because this language doesn’t have many of the “magical” things we have in modern languages, like memory management, strings, and array sizes. But after finishing the course, I could understand how arrays worked memory-wise, why the size in bytes of a string is its actual size plus 1, and much more. These understandings helped me, later on, write much better code and understand some of the limitations there are on different variable types. For example, before the course, I had to remember that arrays are passed as a reference, but integers are not. After the course, I could understand that arrays are actually pointers to the memory, and it makes absolute sense that they are passed as a reference. I didn’t have to remember anymore – I understood.

If I had to teach anyone else how to code, after this experience, I would absolutely go with as less “magic” in the language as possible. It might slow the process a bit but would save a lot of time and frustration in the future.