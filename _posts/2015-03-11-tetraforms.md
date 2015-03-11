---
layout:     post
title:      Creating a Windows Phone 8 game in 24 hours
date:       2015-01-28 12:31:19
categories: retrospective
summary:    "I’ve been planning to write this article for months, but it’s better late then never. Last November, local Imagine Cup took place here in Kazan and I’d like to share the experience. For those of you who’ve never heard of the event before, it’s a student hackathon organized by Microsoft annually around the world.
<br><br>
As the theme states, participants are expected to “solve tough problems by technology” — on practice, this means building an app or a game for Windows platform in 24-hour format. Obviously, making use of things like Microsoft Azure or Kinect is highly encouraged by the organizers.
<br><br>
Rules of the event allow to take part individually or in a small team, so I thought it’d be a cool idea to invite Kostya and Dima, my friends from the university, to join me. We have never been to hackathons before and had absolutely zero experience in developing software for Windows (though collectively we had good background in iOS and web development). Nevertheless, we were quite excited to play around with something new."
---

I’ve been planning to write this article for months, but it’s better late then never. Last November, local [Imagine Cup](http://en.wikipedia.org/wiki/Imagine_Cup) took place here in Kazan and I’d like to share the experience. For those of you who’ve never heard of the event before, it’s a student hackathon organized by Microsoft annually around the world.

As the theme states, participants are expected to “solve tough problems by technology” — on practice, this means building an app or a game for Windows platform in 24-hour format. Obviously, making use of things like Microsoft Azure or Kinect is highly encouraged by the organizers.

Rules of the event allow to take part individually or in a small team, so I thought it’d be a cool idea to invite Kostya and Dima, my friends from the university, to join me. We have never been to hackathons before and had absolutely zero experience in developing software for Windows (though collectively we had good background in iOS and web development). Nevertheless, we were quite excited to play around with something new.

### The Idea

A few days ahead of the Imagine Cup, we started thinking about possible projects we would like to build during the hackathon. Initially, I was arguing that we should try to make a simple app, something that many people would find useful. However, a quick research on past Imagine Cup winners made it clear that we should develop a mobile game instead.

We had a couple arcade game ideas and decided to pick the easiest one to implement. I drew a quick prototype in Photoshop on the eve of the event — it looked pretty cool.

<img src="{{ site.media_url }}/tetraforms/prototype.jpg" alt="prototype" style="width: 250px; "/>

The idea is simple enough to describe in a paragraph — there are squares and circles, colored differently, falling down from the upper edge of the screen and four buckets in the bottom. Player’s goal is to land the falling object in a bucket of correct shape/color by swiping right and left. Buckets randomly change their position on each pass, the speed increases, and you loose if you miss three objects in a row.

### Getting Started

Two of us rely on MacBooks on a daily basis and we had just one testing device (Nokia Lumia) in a team, so running Windows Phone simulator inside Windows virtual machine via Parallels seemed like the most pragmatic solution to us. It went generally okay, though sometime around midnight during the hackathon my WP emulator suddenly stopped working. (I didn’t have much time to dig into the problem, so it made testing a little bit harder).

We also had to decide which game engine to use — either [MonoGame](https://github.com/mono/MonoGame) or [Unity](http://unity3d.com). Having absolutely zero clue and no time to do a proper research, we just picked MonoGame judging by download size (something like ~100mb vs ~2gb). Looking back, I wish someone had told us to think more carefully when making this choice.

Sitting in the hall of Kazan IT Park (kind of like startup incubator / co-working space), really nice venue where Imagine Cup was held in, we were trying to come up with a name for our game. We had a couple catchy phrases floating around, and chose the one that was slightly less terrible comparing to the others — TetraForms (inspired by Tetris shapes). 

We created a new repository on BitBucket and everything was ready to rock’n’roll.

<img src="{{ site.media_url }}/tetraforms/pano.jpg" alt="pano"/>

### The Night of Code

The event began by a short introduction and an interactive “team forming” game. After warming up the audience, organizers had a bunch of presentations to explain some aspects of Windows software development. The tech talks would be more useful if they were more to-the-point, though we weren’t paying much attention to them anyway, since we were learning on our own by actually coding.

The start was kind of frustrating and disappointing though. We had to deal with unexpected geometry logic handling in MonoGame — shapes like circle and square looked skewed on the device, even though they were correctly described in code. It took us about three hours of experimenting and at that point we started to question if MonoGame was the right choice after all. Only by applying some weird aspect ratio coefficients to every on-screen calculation, Kostya was able to work-around the issue to some degree. 

Having this problem solved, things started speeding up a little bit as we were getting used to the new programming language (C#) and Visual Studio IDE. I have worked on falling objects logic and Dima drew cool background stripes & gradients in code (not a trivial thing to do, he had to deal with strange stuff like triangle coordinates).

Swipe gesture recognition would be a pretty easy thing to implement on iOS, but it took me a few hours during the late evening to get it *somehow* working on Windows Phone. It was still terrible and unnatural though, luckily Kostya was able to make it a bit more predictable later in the night.

Another simple thing which took us much longer than expected — displaying a text label on the screen with current player’s score. Apparently, you need to manually compile a font into a resource file (wat) in order to show a label with MonoGame. It was midnight and it was surprisingly frustrating.

Maybe tired of dealing with all these bugs and dirty MonoGame workarounds, Dima fired up GarageBand and started creating some tunes for our game. It was really fun to step back from programming for half an hour (we didn’t go to sleep) and spend some time to choose a soundtrack. Lia, a friend of ours, suggested a really dopey Japanese anime song, and we thought it was a perfect match, of course. She also drew a game icon for us… in a Microsoft Paint.

At about 4am, different pieces of the game finally started coming up together. I’ve had high hopes since the very beginning (why bother participating otherwise), but around that time we actually started believing that we are going to make it.

We did some code refactoring (on a hackathon, ha-ha) and I made a simple game menu at the dawn. A little secret: we didn’t have time to implement statistics, so “Max Score” label is hardcoded to always display 97  — please, just don’t tell anyone.

### Morning

We weren’t happy with most of the code, there were lots of bugs and dirty hacks, but over 60 commits later, you could actually play TetraForms by the early morning ([here’s a short video](https://dl.dropboxusercontent.com/u/10859705/tetraforms.mov)).

After a long night, hanging out with other folks was a great way to cool down and rest for a bit. Walking around the venue was really fun at 7am — some students were sleeping on the floor, others drinking Red Bull and yelling at the compiler while trying to code.

As more and more projects got finished to some degree, it was interesting to walk up to random desks to say hi because people were always excited to tell what they were working on last night. Most of the projects were 2D games written with Unity (it seems we were the only ones to use MonoGame).

Sometime around noon, we got hungry and went out to the city to have a quick lunch. On our way to local BK, we shared our thoughts about other projects, guessed our chances of winning and discussed how we should present the game to organizers.

### Win-win

Imagine Cup ended by a series of project presentations by participants. In total, there were about 50 projects announced, half of them got actually implemented and sort of finished (in a hackathon sense of that word). 

Apparently, when you mirror your Windows Phone screen to your laptop via cable, you only get a fraction of color signal, so we had to replace our beautiful gradients with plain gray background. And like for many other folks, organizers’ VGA adaptor failed when we tried to connect to the big display, so we had to awkwardly show the game on a laptop screen.

Anyway, TetraForms was chosen be the third and we were even awarded Nokia Lumia. Tired and sleep deprived, it still felt really incredible to walk up on the stage.

We made it. A Widnows Phone 8 game in 24 hours, with zero prior experience developing for the platform. Wow, what an experience.

See you on the next hackathon.