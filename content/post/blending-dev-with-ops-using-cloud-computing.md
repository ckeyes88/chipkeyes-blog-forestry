---
timeToRead: 4
authors: ["Chip Keyes"]
title: Blending Dev with Ops Using Cloud Computing
excerpt: 'Your job as VP of IT Operations is to ensure the fast, predictable, and uninterrupted flow of planned work that delivers value to the business while minimizing the impact...'
date: 2017-06-27
hero: ''
draft: false

---
> “Your job as VP of IT Operations is to ensure the fast, predictable, and uninterrupted flow of planned work that delivers value to the business while minimizing the impact and disruption of unplanned work, so you can provide stable, predictable, and secure IT service.”

> — [The Phoenix Project](https://www.amazon.com/Phoenix-Project-DevOps-Helping-Business/dp/0988262592) by Gene Kim, Kevin Behr, and George Spafford

I should preface this by stating that I’m new to cloud computing with both Google Cloud Platform and Amazon Web Services. I’m also not a dev-ops expert by any means, but I am actively learning about both of these areas. However, the fact that I’m even able to write this article, I think, emphasizes my point that cloud computing platforms have simultaneously lowered the barrier of entry to application production, as well as blended the developers and ops teams into a more cohesive unit that can work towards the overall business goals.

### The History

I’ve been a paid developer for almost a year now, so I can’t speak to the history of development, IT, and dev-ops departments with any real personal experience. But I have read and heard enough colloquialisms to get the impression that, in the past, these departments have largely operated independently of each other and (in some cases) at odds with each other. The set of problems the different departments faced and the goals they were trying to achieve often appeared (at best) unrelated and (at worst) at odds. This is most definitely a broad generalization and I’m not intending to open up Pandora’s Box nor dwell on this point. What I would like to convey is that during the times where companies had to purchase, provision, create, and maintain their own data centers and server racks, there was a very different relationship between the developers and the IT/Ops teams.

IT/Ops claims developers pass off broken code that crashes their servers and creates security vulnerabilities that they have to clean up. Developers claim the server infrastructure is so fragile that their locally working code just can’t run properly in the new environment. This is just an example of the kind of tension that puts the two departments in opposition of each other.

Enter cloud computing…

### The Silver Lining

Along comes Amazon and, right behind them, Google, with the idea that companies shouldn’t have to spend top dollar to create and maintain their own server architecture. Amazon has the resources to build that infrastructure and they can just rent that out to the companies at a marginal cost.

Obviously, this was a game changer in terms of how applications were built, but what I find even more interesting is the way it affected the above mentioned Dev/IT relationship. Because not only was Amazon Web Services (AWS) or Google Cloud Platform (GCP) more practical for the business goals, but it was also easier for the developer, IT professional, or data analyst to interact with. You name it and AWS/GCP has a service that simplifies and streamlines your business goals, and it’s all configurable through an easy-to-use web interface.

In relation to developers and IT, cloud platforms blend the tasks of development and operations so that, at the end of the day, it’s possible for one person to build the application, provision databases and servers, automate tests and deployment, manage the server architecture, collect and analyze application data along with all the other ever growing services that these platforms provide. And all for a pay-as-you-go pricing model that allows small companies to use it almost for free and the large companies to limit costs to the services they’re actually using.

### The Meaning

Initially, the blurring of department lines might evoke a response of, “So what?” or even, “Hey! We still need both of those departments!”

Settle down. I’m not trying to imply that companies should just merge all their departments into a cloud computing expert. We still need people that are specialized in all the areas of application management. What I am saying is that this model for application development creates an ecosystem more inline with my opening quote. Both developers and IT/Ops are now responsible for and unified by the business goals.

Developers don’t get to just hand off code to the ops team and let them clean up the mess. Ops can’t just claim bad code when the system crashes. Both teams are subject to “ensuring the fast, predictable, and uninterrupted flow of planned work that delivers value to the business”. This breaks down the wall that has developers avoiding the scary Ops world (myself included) and the Ops treating the developers like a bunch of hooligans hell-bent on breaking their infrastructure.

### The How-To

Ok, but how do we accomplish this zen camaraderie where developers and IT work together for the greater business good? Well, I can’t say from direct experience, but I do have some ideas that I think would help push things in the right direction.

#### Train Your People

Cloud computing is here to stay, and the faster your team can work proficiently in one or more of the major cloud platforms, the faster you can start taking advantage of the benefits they offer. These platforms are huge and intimidating, so putting someone unfamiliar at the helm is like putting a 16 year old driver in a Formula 1 racing car. Not to mention you will almost definitely incur unnecessary costs if they don’t know what they’re doing. Get people on board with this platform and invest the time it takes to prepare them to use it effectively.

#### Start From Day 1

When you have knowledgeable people on your team, start using it as much as possible, as early as possible. Both AWS and Google integrate with version control and continuous integration systems (and both even have their own systems built in), so as soon as your code hits a repo from day one you have your pipeline set up.

Once you figure out the pipeline, you can even have it ready to automate new projects in one easy click (an area I’m actively learning now). Trying to have every developer set up their own dev environment just wastes that much more time that could be spent accomplishing business goals.

#### Foster the Blend

Regardless of department or expertise, people should be intimately familiar with all facets of the system. The beauty of these platforms is that it’s easy for developers to go look at the different environments and Ops to go look into the code.

And the best part is you have tons of configurability over roles and permissions so that your developer doesn’t go erase your database and your financial analyst doesn’t accidentally provision all the servers.

If you move to a cloud platform, but maintain team silos, then you’ve completely defeated the point of this whole process. Encourage developers to jump in and understand and even contribute to the Ops pipeline and encourage IT/Ops to get familiar with the code so that the two teams can work together like a left and right hand.

#### Conclusion

As the book _The Phoenix Project_ pointed out over and over again, anytime a person or team is performing tasks that are not directly related to the most important business goals, they are working against the interest of the company. I think cloud computing helps bring teams together by providing better visibility among the system(s) and reminding people that they’re working on the same unified project for the success of the company. Hopefully, you come to see this as well as you explore these awesome platforms.

#### Learning Resource

I’ve learned a lot from the tutorials at [A Cloud Guru](https://acloud.guru/). It’s specific to AWS, but I find that there is quite a bit of carry over to GCP and the differences are pretty easy to pick up on.

[sc name="post-signature"]
