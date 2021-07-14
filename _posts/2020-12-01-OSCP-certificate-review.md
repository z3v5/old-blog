---
layout: default
title: "OSCP review"
tags: OSCP
description: "My thoughts and takeaway from Offensive Security PWK/OSCP certificate"
---

## Table of Content

* Introduction
* About the PWK/OSCP course
* Expectations
* Before taking the course
* Lab time
* Preparations for the exam
* The exam
* Closing thoughts
* Useful resources

---

## Introduction
The OSCP (Offensive Security Certified Professional) also, knows as PWK (Penetration Testing with Kali Linux) certificate is something that I was afraid of, but at the same time wanted so badly for a very long time.  
When I just started my journey as a penetration tester and shyly poked my first targets at [HackTheBox](https://www.hackthebox.eu/), that was something that cool kids talk about. This milestone seems unreachable the people who took it were way more professional and prepared than me.  
Soon enough, I learned that as with everything in life:

> Nothing is impossible if you are ready to invest enough time and effort into it.

So, that is how this journey has begun. A passionate about hacking Padawan who found a goal and decided to make this happen. No matter what.  
Allow me to introduce you to the most interesting professional experience I ever had. Shall we?

---

## About the PWK/OSCP course

OSCP (or PWK) is the course from Offensive Security. A quote from the official site:

> This online ethical hacking course is self-paced. It introduces penetration testing tools and techniques via hands-on experience. PWK trains not only the skills but also the mindset required to be a successful penetration tester.

The mindset? Don't worry, we will come back to it later.

The course itself is being around for quite some time (the [Pentesting with BackTrack PWB](https://www.offensive-security.com/backtrack/penetration-testing-with-backtrack-v-3-0-alive/), the predecessor of the PWK, was there a decade ago), but it was updated in 2020. This update brings a few new modules, updated learning materials, and, what is more important, a lab environment. Cybersecurity as a field is growing rapidly, and as a penetration tester, you should be well aware of new trends and attack vectors, right?

As this certification exists for a while already, and everyone who took it knows what it takes, the OSCP becomes a sort of an industrial standard for the junior level pentest positions. When I'm saying `junior level pentest positions` I mean that after it you can apply for your first job as a pentester, and it will massively boost your CV. The OSCP is a certificate with a high level of recognition among both HR and cybersecurity communities. But a VERY common misunderstanding here is that if it's a `junior level` certification it should be easy. It shouldn't. To be successful as a penetration tester, you should have a solid background in many areas (networks, programming, etc.). I'm not saying that you should have 197IQ to pass it, but it could take more time to prepare that background if you don't have it already. Keep that in mind if you are thinking of taking that exam. There is no magic book/course/pillow to take that will put all of that knowledge and skills into your head overnight.

The OSCP is very different from the more "classic" certificates that you might be used to. It's not a multiple-choice questions exam. The OSCP is a hands-on certificate. On the exam, you will get five vulnerably boxes that you will need to pentest.  
To be able to do that, you need a lot, no, let me put it like that, A LOT of experience.  
The learning materials cover all subjects that you will need, but you can't just read the pdf (or watch the video) and expect yourself to pass the exam. To pass the exam, you MUST go for the extra mile in your learning.

You can find the syllabus and more info about this course [here](https://www.offensive-security.com/pwk-oscp/).

To pass the exam, you should earn at least 70 points.  
Each machine has its value:
* Buffer Overflow [`+25`]
* Hard machine [`+25`]
* Two Medium machines [`+20`]
* Easy Machine [`+10`]

By using the magic spell called `math`, you might find out that in the best-case scenario, you can earn your certificate by doing just three machines, `25`+`25`+`20`=`70`.  
Medium and Hard machines have two stages, user and root. If you are familiar with the HackTheBox machine, you will feel at home.  
By submitting the user flag, you will score **some** points, Offensive Security didn't specify how much exactly in the exam guide, but according to other students, it's something about half of the points.

The PWK exam takes two days. The first one - proctored hands-on assessment, the second one is to write and submit the report.

In a few days, you will receive the PASS/FAIL email. I've spoken to the people who scored enough points but failed the exam anyway.  
The report is equally important as the fact that you know how to do a pentest. After all, the report is the thing that your client is paying for. Make sure that you have read the exam guide several times and triple-checked your report before submitting it.

Pricing.  
You can check the full list [here](https://www.offensive-security.com/pwk-oscp/), but prices start from 999$ for the 30 days of labs + the exam. I strongly recommend you to go for 90 days of lab time if you can afford it. 90 days one will cost you 1349$. In case if you will fail the exam, you will have to pay an extra 150$ for each retake.

> Keep in mind that you might fail the first time! It's completely fine. I will cover that later in the text.

I'm not going to discuss pricing here, but in my opinion, it's quite decent, comparing to the other companies that providing similar services. The overall price for the benefits that you get is not that high, as you get an education and the cert itself have quite a recognition. Again, my goal here is not to mock other certs or sell you the OSCP I'm just trying to share my experience and thoughts, so let's stop here.

---

## Expectations

As a self-thought student, I knew that I've potentially lacked some fundamental knowledge in many areas. I expected to fill those gaps with a consistent understanding of the basics to build my methodology upon it. Probably the methodology and the mindset will be your main takeaway from this course. I also was afraid of approaching anything that "touching" memory in a low-level way. Assembly, memory corruptions, binary exploitation, all of that immediately gave me creeps, so I expected to get rid of that fear as well.  

---

## Before taking the course

Before taking this course, I wasn't sure that I'm ready. I had five years of work in IT (which gave me so needed troubleshooting skills) as a background but didn't work in security, especially on its offensive side. I said didn't work, but I was heavily interested! At the time of taking the OSCP course, I got the `Pro Hacker` rank on [HackTheBox](https://www.hackthebox.eu/), did most of the machines from [TJNull's OSCP-like list of machines](https://docs.google.com/spreadsheets/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/edit#gid=1839402159), played a ton of CTFs (Capture the Flag) and consumed an enormous amount of hacking-related content. Eventually, that helped me to get the job as a pentester without a certificate, which gave me some background on the current security posture of a production system, some report writing skills, and a good base for the methodology and mindset.

---

## Lab time

The important lesson that I learned very quickly is that preparation for the OSCP exam is not only about pentesting. I would say that it's more about time management, resource control, and mental health. If you are not doing things smart, you are risking to burn yourself out rapidly. It's hard to keep your motivation up to the needed level if the only thing that you do is study.

A few things that helped me to manage this:
* Sport. Regular physical activities will boost your cognitive functions too.  
* Feel alone in this battle? No worries! Join other OSCP (and many others) students and professionals in `#RedTeamFit` \ `#DFIRFit` \ `#BlueTeamFit` almost everywhere (`Twitter`, `Strava`, `Garmin Connect`, you name it). The community vibes could help a lot.
* Discord servers with other students. Shout out to the fantastic people at [InfoSec Prep](https://discord.gg/rSkrNqW7EB) Discord server! You can find there others, who struggle with the same lab machine as you, ask questions, share your knowledge, isn't that great?
* Time management. After one week of hard work, I noticed that I'm completely exhausted and don't want to look at the terminal anymore. Your brain is a muscle in some way, and it needs some time to rest. It was a hard decision to make, but I decided to not touch my computer during weekends. It could take some time to get used to feeling guilty for the fact that you are wasting your paid lab time, but it's needed.

Alright, we figured out the time management part. Let's talk about learning itself.  
You will receive course materials in both video and pdf. They are covering the same topics, but sometimes pdf is a bit more detailed than the video course and vise versa.

> Make sure that you will **download** both pdf and the video materials in 48 hours after Offensive Security will send it to you. Otherwise, they will ask you to pay for the new download link. Not cool.

Choose the preferred way to obtain knowledge (video or pdf) and do that first. Don't touch labs at first, you will get there.  
I recommend going after the labs if you already have a solid background or prior experience as a penetration tester.  

Exercises. Each chapter contains some exercises that you can do to make sure that you understood the subject and ready to move on. Some of them are easy, and some of them quite challenging.  
> If you will do **all** of them and document your progress for ten lab machines, you can submit it after the exam as a lab report.  

If you will do so (and don't have mistakes in **all** of them), Offensive Security will add the bonus `+5` points to your final score, which might or might not change the result of the game for you.  
I was in the situation when I stuck at `65`/`70`, and I wish to have those `5` extra points. Likely, I was able to find the problem, and rooted the box.   

However, I think that it's too much work for just 5 points, but that is a controversial opinion. Decide that for yourself.  

My goal was to finish as many labs as I can instead. PWK is an exposure exam. Well, the whole security is based on exposure.  
To find something that stood off you should know both how things are looking normal, and how they are looking if misconfigured.  

Another common misunderstanding is that you must finish all the lab machines to pass the exam. Again, this will very much depend on the methodology and mindset that you will develop, not the number of lab machines that you will do. Plus, there are too many of them! Some of them too similar to others up to the point that even the same exploits will work there.  

And that's a tricky thing, here you should force yourself to not choose the easy way. You truly should `TRY HARDER`. You can straight forward pwn 5 boxes in 10 minutes with a kernel exploit and `Metasploit` but what you will learn from it? On the exam, you can use the `Metasploit` only once, and it's probably the worst idea to use it for a kernel exploit there (as most likely it will not work, and you will waste time). I should probably mention that if your first idea, when it comes to the exploitation, is to go for the kernel exploit - you didn't learn your homework right. Kernel exploits are usually unstable, and randomly shoot them as the first thing in a production environment of your client is in general very bad practice.

Cracked the box with `Metasploit`? Good, now you know that it is possible to exploit that vulnerability on that machine. Go and repeat that manually.  

> If there is a Metasploit module for something, that means you can do the same without it.

The downside of that could be that many fresh OSCP holders would rather go for the random exploit from the `GitHub` over the `Metasploit` module, which is generally more stable and well tested, but that is a story for another blog post.  

My goal was to pwn at least 30 machines, but not because it's a round number (but partially that's too). I had a list of machines that circulating among students. Machines there were selected to make sure that you will not miss the important aspects of the machines in labs.

I also had a goal set up for myself - do all machines manually, and summarize each machine and find the answer to two fundamental questions:
* Why this machine is on the list? What is so different about it?
* What this machine is trying to teach me?

The same approach I used to the mentioned early TJNull's list, and later on for the [Offensive Pentesting](https://tryhackme.com/path/outline/pentesting) path on [TryHackMe](https://tryhackme.com/). That approach meant not only to make sure that I understood the vulnerability and know how to exploit it but also started to build my understanding of how machines on the exam might look like.

p.s. The `Try Harder` phrase becomes a meme for all people who does the offensive security type of works mainly because of PWK/PWB. But it's not just a meme, and there is an idea behind it. I strongly recommend reading [Try Harder: From Mantra to Mindset](https://www.offensive-security.com/offsec/what-it-means-to-try-harder/) article on that matter. That's the main thing that you will learn along the way: what to do if you are stuck. You frequently will find yourself in a situation where nothing works according to your plan, or you starting suspect that you falling into a rabbit hole. And if for the labs you can go and ask for help on the forum/Discord, on the exam, you are on your own. This ability to "unstuck" yourself is a game-changer, and you have to develop it by yourself.

---

## Preparations for the exam

After finishing your lab time, you will have a pretty big time window to schedule your exam.  
Lab time should highlight your gaps in knowledge, take some time to analyze them, and make some actions to level up your game.  
From experience, I know that many people struggling more with privilege escalation on Windows than on Linux. Feel the same? Read up on that or even take a course to fill out this gap. There are a few awesome courses out there for quite a little money:
* [Tib3rius](https://twitter.com/TibSec)'s [Linux](https://www.udemy.com/course/linux-privilege-escalation/) and [Windows](https://www.udemy.com/course/windows-privilege-escalation/). $20 for each, very often on a discount
* [TCM](https://twitter.com/thecybermentor)'s [Linux](https://www.udemy.com/course/linux-privilege-escalation-for-beginners/) and [Windows](https://www.udemy.com/course/windows-privilege-escalation-for-beginners/). $30 for each, often on a discount too

I also didn't feel confident about my technical writing skills, so I fired up this blog again and decided to document here my progress for the [TryHackMe](https://tryhackme.com/)'s [Offensive Pentesting](https://tryhackme.com/path/outline/pentesting) path.

During the course, I also followed the instructions on how to do a stack Buffer Overflow attack but wanted to understand it better to be able to do it faster on the exam. The time factor is the key when you have only 24 hours for everything.  

I started with [Justin](https://twitter.com/justinsteven)'s [DoStackBufferOverflowGood](https://github.com/justinsteven/dostackbufferoverflowgood) and ended up doing every single vulnerable input on TibSec's room on TryHackMe - [Buffer Overflow Prep](https://tryhackme.com/room/bufferoverflowprep). On average I was able to do the BOF with all steps of the exploit development in less than 20 minutes.   

To learn it better, I tried to write it less technically (if that was possible) and avoided screenshots to focus more on the exploitation itself. For example, here you can find my write-up for [Buffer Overflow Prep](https://hackish.space/Basic-Stack-Buffer-Overflow).

I had my exam scheduled in a month after expirations of my lab time, which was enough to fill those gaps that I identified but wasn't too long to forget something that I learned.  
Again, think for yourself, but if you are looking for advice, there you go:

> Fail faster!

Remember that even if you will fail the exam, it's good! Fail is not the opposite of success it's part of it.  
You will learn from your mistakes, you will know what to expect from your next exam attempt. Otherwise, you automatically miss all shots that you didn't take.

The exam day is not the best day to make yourself familiar with the exam report template. Make sure that you will read it in advance and pre-write as much stuff there as you could do. Keep in mind that most of your findings on the exam will be with high or critical severity, as they will lead you to spawn the shell or escalate your privileges.

---

## The exam

Because of the obvious reasons, I will not discuss any details about the machines on the exam.  
You will get the list of them right before the exam, together with the list of requirements for each of them.  
Make sure that you will carefully read those requirements! Especially the part about what exactly you should put into your report, which screenshots you should take, etc.  

Each machine crafted in a way to make you suffer. Nothing will work as expected, and that is where your methodology and mindset should kick in. Long story short, the whole exam will be like `Alright, that didn't work, what else I can do?`.  

Please keep in mind that the exam shouldn't be easy, but it's designed to be solved in those given 24 hours.  
I know it's a hard pill to swallow, but if those 24 hours are not enough for you, that means that you not ready yet. Which is completely fine! You will score better next time when you will improve your methodology.  

Breaks. Breaks are important. Make sure that you will have proper breaks for meals. I used a timer on my watches (a kitchen timer will do the trick as well) to make sure that I'll not fall into the flow and stuck with one box for several hours, so each one hour I had a short break.  

It will help you to stretch yourself, both physically and mentally. I used that time to refill my water bottle, walk around, do some exercises.  

Keep yourself hydrated so you won't need that much caffeine. Overdose of caffeine is the last thing that you want to do when you are in a situation where you need to have a sharp mind.  

When you will feel that you are done and not making any progress - go and try to sleep for a few hours, and then come back to work. No matter what, before the end of the first day, take your time to make sure that you captured all needed screenshots of flags, exploitation steps, etc. You will need all of them to write a report the next day when you will don't have the access to the VPN network with the machines themselves. Everything should be done in advance.  

The second day only seems simpler than the first one, but it's not. Don't forget about the breaks too! It's equally important as in day one. Triple-check your report before sending it out. Make sure that it's covering all checkboxes that you received right before the exam.  

When it's done - it's done. Get some rest and try not to think much about the exam. In 3-5 days, you will receive your PASS/FAIL email.  
Together with the pass email, you will get a digital badge, that you surely want to claim and share on LinkedIn or whenever you are pleased. Sharing and celebrating success is important. Don't be shy to share it and tell others what you achieved, it well deserved.

---

## Closing thoughts

Please keep in mind that sharing your fail experience is even more important! OSCP is hard enough by itself, but you also have to have decent time management and problem-solving skills, which might take a while to develop.   
Many fantastic professionals whom I was honored to know or work with didn't pass their OSCP for the first time. Some of them needed multiple attempts to make it happen.  
Don't be afraid to share this experience, you will be surprised how supportive the community is!

If you already failed your attempt and reading this, don't worry, you will be there! If I managed, you will surely do as well.

A bit of advice if you failed:
* Try to analyze what took you down on the exam. Maybe your enumeration wasn't good enough, and you want to look at tools like [Autorecon](https://github.com/Tib3rius/AutoRecon). Maybe you missed some common PrivEsc techniques, and need to refresh on that. Maybe you didn't have enough experience with BOF and need to practice a bit.
* Don't be ashamed of sharing and talk about your failure
* Believe in yourself! You will get this!

Can I recommend this course to take? If you are interested in a penetration test and want to discover the field better - hell yes!  

Will it be easy? Hell no!  

Does it worth it? Absolutely!

---

## Useful resources

* [Official PWK/OSCP page](https://www.offensive-security.com/pwk-oscp/)
* [InfoSec Prep Discord server](https://discord.gg/rSkrNqW7EB)
* [Tib3rius PrivEsc course for Linux](https://www.udemy.com/course/linux-privilege-escalation/), and [Windows](https://www.udemy.com/course/windows-privilege-escalation/)
* [TCM  PrivEsc course for Linux](https://www.udemy.com/course/linux-privilege-escalation-for-beginners/), and [Windows](https://www.udemy.com/course/windows-privilege-escalation-for-beginners/)
* [PrivEsc workshop from sagishahar](https://github.com/sagishahar/lpeworkshop)
* [DoStackBufferOverflowGood from justinsteven](https://github.com/justinsteven/dostackbufferoverflowgood)
* [Buffer Overflow Prep](https://tryhackme.com/room/bufferoverflowprep)
* [Payload All The Things repo](https://github.com/swisskyrepo/PayloadsAllTheThings)
* [Autorecon](https://github.com/Tib3rius/AutoRecon)
* [List of OSCP-like machines on VulnHub and HackTheBox from TJNull](https://docs.google.com/spreadsheets/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/edit#gid=1839402159)
* [Offensive Pentesting path on TryHackMe](https://tryhackme.com/path/outline/pentesting)
