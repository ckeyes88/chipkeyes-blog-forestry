---
timeToRead: 3
authors: ["Chip Keyes"]
title: Slackin'
excerpt: 'A story on finding and solving real problems There’s a lot that goes into the thought, design, creation and execution of an application. While the road from an inkling of...'
date: 2017-11-14
hero: ''
draft: false

---
#### A story on finding and solving real problems

There’s a lot that goes into the thought, design, creation and execution of an application. While the road from an inkling of an idea to a fully functioning useable “thing” may seem like a simple easy path from the outset, we often round the first corner on this trek only to find a windy, rocky, road.

This was the case for my internal Slackin’ tool, built to provide a simple, visual report who was out of the office on any given day. It’s data was based on our company’s time off tracker, [KinHR](https://kinhr.com/).

Get it? Slack+Kin=Slackin’

I’d like to walk through some of the lessons I’ve learned while building a so-called ‘simple app’ designed to do only one thing. It should’ve been easy, right? We’ll see…

### Easy as 1, 2, 3…Or Not.

> “Sure, we can build that in almost no time and no money and it’ll be better than everyone else’s.”

> — Every Software Company

Everything starts with a little itch, a little problem that needs to be solved. This is the mentality of a developer — or at least it should be. Development is an industry based on building solutions to problems.

Well, my itch started with time off. Team members in our company would have to manually post scheduled time-off to our Slack channel, even though they had already logged their scheduled time off in Kin. As you can imagine, it wasn’t always clear who was going to be out of office, especially if they had posted regarding their time off well in advance of the actual days they’d be away.

So, I naturally thought to myself, “There’s a Slack integration for everything…I’ll just go download the Kin-Slack app and we’ll be good to go”.

Much to my surprise, I couldn’t find a single integration or even an article about anyone trying to make any use of Kin from within Slack.

My next thought was that perhaps they don’t have an API that can handle this. Surely, if they did, someone would have built it by now. Right?

Wrong. It turns out they actually have a pretty well documented API that can do a lot more than I would’ve initially thought. How hard could it be…just get a token, hit the API, display the results in Slack. Boom…done!

Except not done. No boom. The API is protected by a pretty robust OAuth2.0 set up (understandably so, since they handle sensitive information like SSNs). Overcoming this newest hurdle was definitely not impossible, but for someone who hasn’t dealt with that type of integration it seemed like a brick wall. Not only was I trying to navigate through the consent flow and obtain an authorization token, I was trying to do it in a semi-headless system — Slack. I couldn’t have users being redirected from a Slack conversation every time they check who’s in the office that day.

A problem is almost never as simple as it appears to be. What initially seemed like a basic data fetch and display, turned into a circus act of jumping through authentication hoops.

This is almost always the case with projects. Those projects that appear to be simple and straightforward are often riddled with unforeseen complexities. While not impossible to overcome, if you don’t plan for them, they’ll derail your project and frustrate those that are depending on you.

### **Version 1.0 (More Like 0.0.1)**

> “This system doesn’t work for my use case. I’ll just build a new BETTER one from scratch!”

> — Every developer…ever

Since the flow I originally wanted didn’t _seem_ possible using their OAuth2 authentication system, I decided it might just be easier to build a totally separate system.

Why not? I’m a developer. I can just make a little app bigger — and I can cover more use cases with that anyway.

So, on I went. I chose to write Slackin’ in Elixir because it’s a language I’m passionate about and I wanted to dive a little deeper with a real project. (I certainly think Elixir is a great tool for this project, but my beginner/intermediate knowledge added to the project complexity.)

I put together a standalone internal Slack bot for our team that captured messages and responded if it was mentioned. Based on the message the user sent, it did some kind of manipulation on the data for that user, like adding Work From Home days. With a simple command like _/slackin 10/13/2017 WFH,_ the bot would parse the message and store a representation in a user’s ‘calendar’ that could be fetched later. Then, when someone said _/slackin who’s in today?_ the bot would look at all the user’s calendars and respond with anyone that had time off logged that day.

Again, pretty simple, right? Again, not so much.

At every level there were added challenges. For example, string parsing, data storage, user id mapping, handling errors, handling crashes… the list goes on. I put it up on Heroku and it took one of our employees only 3 tries to crash the system and wipe everyone’s data. Back to the drawing board.

While I do think there are reasons to build something that exists from scratch, keep in mind that you’re going to have to solve all of the same problems that someone else has already worked through plus any new ones that you create yourself. In this case, I didn’t necessarily think I could build it better than Kin, but I had no idea that I would create at least as many problems as I solved. So, while I might not have had authentication to deal with, I now had to manage the entirety of the architecture and every little bug that it comes with it.

Free tip. Think hard before you decide to build a new _anything_ from scratch. It might end up costing you more than it’s worth.

### Version 2.0

> “This is how I decided to set it up and I’m going to stick with it or die trying!”

> — Stubborn/All Developers

After some time away from the project, I decided to revisit the KinAPI and do a little more digging into the auth system to see if I had dismissed it too quickly. I had avoided the project for a while, telling myself that some distance was the best thing to do before I went back to begin fixing bugs. (To be honest it just didn’t sound like a lot of fun to me—especially for something that was just a little side project meant to scratch an itch.)

After some time away and some new found resolve to finish this, the OAuth2.0 stuff didn’t seem like quite as big a hurdle as it was initially. Thanks to the support team at KinHR, I was provided with step-by-step instructions, complete with a live demo by one of their developers. I wrapped my head around obtaining an auth token not only from Kin, but also from the Slack OAuth system, which would potentially allow other companies to add this integration to their Slack in the future. I even decided to change the language to Golang, which I was using regularly at the time.

After a few days of playing around in Postman and looking at what data I had to work with, I had a 3-endpoint solution that could be added to any Slack team and used by anyone with a Kin account. Best of all, I had the knowledge to integrate with _almost_ any OAuth2.0 API in the future. I tied it to a slash command and with a few tweaks got back the time off info for our company.

The lesson here is that you shouldn’t be too afraid of revisiting your previous solution. While time and other resources may not allow it, don’t make the mistake of assuming you’re tied to the original architecture, or even the language.

I certainly don’t recommend changing things over and over, but captains that go down with their ship are, in the end, just dead captains.

### Iteration Makes Perfect.

> “Let’s build the simplest feature set that gets the job done and then make it better over time.”

> — Every successful project…ever

The biggest takeaway from this experience is that if you go after production version one from day one, you’re going to end up in a ‘_lose lose_ scenario. Slackin’ has slowly evolved since I first set out to complete this ‘simple’ project.

Turns out, Kin’s API allows requesting, approving, and viewing time off for individual employees. If I had set out from the beginning to have a complete integration with all of Kin’s API, I would probably have nothing to share with you today. Instead, I have a single functionality from their API that can be called from inside Slack. I would like to add more in time, but you have to be able to tackle individual issues one at a time. It’s ok to have 10 micro versions with incremental improvements… because the guy or girl trying to do it all is still at version 0, which is the least useful version of all.

The book “Zero to One” by Peter Theil was a fantastic book about how much harder it is to go from nothing to something (0 to n) than to go from something to something else (n to n+1). How many projects have become stuck in the zero phase when all they needed to do was put something out there that could be iterated on? It might have made all the difference.

Find your itch, come up with a simple solution, and execute it. That’s how projects are completed.

### Credits

Huge thanks to the staff at KinHR for putting up with all my confused emails and for all their fantastic [documentation](https://kinhr.com/welcome-to-the-kin-api/).

Also, the Slack API [documentation](https://api.slack.com) is incredibly thorough and helped a ton.

I’m working on getting the code of my integration up on github and will hopefully be able to distribute the integration soon enough for anyone using Kin and Slack.

[sc name="post-signature"]
