---
layout: post
title:  "How I Found a Secret Menu"
date:   2017-01-04
---

There is this cool restaurant that my girlfriend and I like to visit called  [Gordos](https://www.facebook.com/gordosrestaurant/). This place serves some awesome good looking burgers and milkshakes.  
Recently I heard some rumors about a secret menu they have. I googled a bit, and found out that there is actually a secret menu and it’s hidden in their mobile app. According to what people say on the internet, in order to find the secret menu you have to click an item in the regular menu, and the secret menu would just pop.  
I tried clicking items on the menu for a couple of minutes, but with no success. So I decided to go deeper.

![abdd928ceb8c9e10268b31c3c26be47c2fe1edb8037feae792a1fcc40a05e8ca](/public/images/abdd928ceb8c9e10268b31c3c26be47c2fe1edb8037feae792a1fcc40a05e8ca.jpg?w=246&h=160)

I started by  [downloading](https://apkpure.com/)  and  [decompiling](http://www.decompileandroid.com/)  the apk file. I had no idea where to start looking in this big project, so I tried searching for some keywords like ‘סודי’ (secret) and ‘תפריט’ (menu), but I found nothing. That was weird. The word ‘תפריט’ for sure should exist in the app, it’s right there, on the main screen. There is only one solution for this problem I could think of – it downloads the content of the app from the interwebz.

First I wanted to check my theory. I put the phone on airplane mode and opened the app – nothing. The app doesn’t work without internet connection, five points to team ‘it loads the content from the interwebz’.

Now I just need to see what it loads from the internet – in order to intercept the traffic of the app I downloaded  [Chales](http://charlesproxy.com/), set it as a proxy server on my phone and opened the app. I was expecting for some javascript code to be loaded, but no, it just loaded insane amount of json files (~20MB of json files). This time, when I searched for ‘סודי’, it found several results. Lots of these results seemed not relevant, but then I found this:

![screen-shot-2017-01-04-at-10-58-15](/public/images/screen-shot-2017-01-04-at-10-58-15.png?w=312&h=344)

The value in the ‘title’ field is ‘Gordos secret menu’ – jackpot! But, I expected that the ‘items’ field would contain the actual items in the secret menu, but no – it just contained null value. Luckily, the second object in the ‘catalog’ array was another object where the value of ‘title’ was just ‘secret menu’, and there the ‘items’ array actually contained the items of the secret menu:

![Screen Shot 2017-01-04 at 11.01.35.png](/public/images/screen-shot-2017-01-04-at-11-01-35.png?w=291&h=374)

Another jackpot! But that wasn’t good enough, I wanted to see this menu inside the actual app – I needed to know which item in the menu I had to click in order for the secret menu to appear. I thought to myself, something in the app should know how to display the secret menu, so somewhere inside this json or the code of the app I decompiled there had to be a reference to this secret menu. I took the id of the secret menu and searched it in the json file, and I found it 🙂

![screen-shot-2017-01-04-at-17-17-05](/public/images/screen-shot-2017-01-04-at-17-17-05.png?w=340&h=352)

As you can see, it looks like there is an action related with this item which is some sort of baked potatoes and the id of the secret menu.. Looks like there should be a button on the element which says ‘חזור’ (go back) and it should perform a ‘openCatalogItems’ on this secret menu catalog. I opened the app, found this baked potato item, clicked on it, it showed me a picture of the meal, with two buttons on the bottom – ‘go back’ and ‘share on facebook’.. I clicked the ‘go back’ button.. and voila:

![whatsapp-image-2017-01-03-at-16-34-54](/public/images/whatsapp-image-2017-01-03-at-16-34-54.jpeg?w=200&h=356)

_**“Shhhhhhhhhhhh!”  
**“Dont tell anyone! You found our secret menu 🙂 No doubt_  
_you deserve a prize for finding the treasure”_

And here’s the menu, in case you ever visit the restaurant:

![whatsapp-image-2017-01-04-at-17-34-14](/public/images/whatsapp-image-2017-01-04-at-17-34-14.jpeg?w=262&h=466)