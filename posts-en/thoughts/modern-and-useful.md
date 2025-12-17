---
id: fuU
title: "Thoughts on \"Advanced\" and \"Practical\""
pubDate: 2020-12-24T05:51:55.000Z
isDraft: false
tags:
  - technology
categories:
  - thought
---

Today, the system I “inherited” from my senior acted up again. After debugging it, I took the chance to re-examine the whole thing.

This system is nowhere near “cutting-edge”—from today’s perspective it even feels a bit “low-end”:

- PHP  
- Apache 2.4  
- MySQL 5.6  
- Openlayers 3  

And yet, it plays an extremely stable role in the daily operations of campus logistics, recording maintenance work and related data for the staff.

I joined this project later on to add some new features. At that time, I had just built up a bit of experience writing my own small apps, and I was obsessed with hunting for new technologies as “toys.” Saying that I looked down on PHP back then would not be an exaggeration, so when I was assigned work on this system, I was really restless and annoyed.

On top of my impatient personality, my technical skills were only half-baked. As the project progressed, the typical state of my tasks was: the basic flow worked, but any slightly more thorough testing would reveal a bunch of issues.

In the blink of an eye, my first semester of the third year of grad school is almost over. In the meantime, I’ve worked on several other projects, and the tech stacks, just as I had hoped, have all been more modern. But in terms of practical usefulness, they all fall short compared to this one. With just two or three daily active users, this system has been quietly running for years on a woefully underpowered virtual host. As a 2B system, and considering it was basically a “practice” project for non-CS majors like us, its performance is actually a bit impressive.

Technology has “new” and “old,” but it’s not so easily divided into “good” and “bad.” From a software design perspective, the code my senior wrote is quite poor in reusability and only average in readability. Yet today, even with no real PHP foundation, I can patch and tweak this old system without too much effort. In a way, its maintainability is actually excellent. The project files (\*.php) just lie quietly in their folders, faithfully carrying out their duties.

New technologies keep emerging, and excellent programmers keep pushing the envelope in terms of development efficiency, software performance, and more. For me, in my somewhat blind pursuit of new tech, I’ve gradually downplayed the fact that technology is ultimately meant to serve real applications. Using various “new technologies,” I’ve built a bunch of fun but essentially useless “toy” apps, and I still don’t have a single practical application that feels close to perfect and can run in production without complaint. Going forward, I’ll probably dial back the chase for “new tech,” focus more on strengthening fundamental skills and general principles, and aim to build a “toy” that’s actually useful.