
# Monoliths, Done Right.  Why be Wary of Micro Services.

I write this article mainly as a stern warning to site and business owners to be wary of going the micro services route, as it could potentially become far more costly than you currently realize.  I'll preface this article with a quick analogy -- the infamous SQL vs. NoSQL battle royale, which is strikingly similar to what is currently happening with monolith vs. micro services.  Back in about 2011 the noSQL hype had hit fever pitch, and developer communities across the internet were embracing it as the only way forward.

Developers from every walk and skill level would adamantly denounce SQL as an old relic of technology that had its day, while gleefully embracing the wonderous magic of NoSQL, and most importantly, that feeling of being ahead of the technological curve.  Many developers thought finally, they had a huge advantage by mastering NoSQL probably years before their competition caught up, giving them the huge advantage they had been waiting for.  NoSQL was the future, and everyone who thought differently was living in the past and will soon be forgotten they thought.

What actually happened though?  Naturally faults and limitations within NoSQL began appearing.  Fast forward a handful of years, and SQL relational databases such as mySQL, PostgreSQL and SQL Server among others are back to being the undisputed dominant data storage solutions without hesitation amongst developers, while NoSQL solutions have taken their rightful place within special use cases throughout the internet where it makes technical and logical sense to use them.  This is as it should be.


## Monolith vs. Micro Services

Make no mistake, the current heated debate of monolith vs. micro services is the exact same, and will end in the same fashion with one major difference.  If you make the wrong decision in this debate, it could potentially cost you a whole lot more, and cause you far more pain than choosing the wrong database engine.  Simply put, you can't unscramble a scrambled egg, and let me explain.

If you initially chose NoSQL only to later realize it didn't live up to its expectations and needed to switch back to a SQL relational database, your developers would grumble and most likely flip you the bird behind your back, but nonetheless, it was a possible feat.  Although grueling and an all around pain, it is possible to transition an entire system from NoSQL to SQL while keeping the system itself for the most part intact.  You would pay for the bad decision with time, money and morale loss, but at the end of the day would probably recover with a hard lesson learned.

However, I can promise you that will not happen if you choose micro services only to later experience problems and decide a monolith would be better.  There is no reverting from micro services to a monolith, same as there is no unscrambling a scrambled egg.  For all intents if you find yourself in a position where reverting to monolith is required, you can basically expect to throw everything you have in the garbage, and start again with virtually nothing more than a blank file in Notepad.

I am sure you can envision how costly this could become, even to the point of business failure or worse, bankruptcy.  The good news is, it is very possible to move from a monolith to micro services, and that process is currently underway in business offices all throughout the world.  Again, the reason of the article is to urge caution before jumping into micro services too quickly, only to find yourself in a world of hurt.


## Concept of Micro Services

I want to touch on this point, as I don't see it hammered home enough in conversation.  The whole advent of modern day micro services came from companies the likes of Netflix, Amazon and Google, as they had a need for better managability across their teams of developers, "teams" being the keyword here.  Aside from certain cases where a different stack is required, modern day micro services are mainly grounded in personnel reasons, not technical.  This means unless you have a minimum of 20+ developers working on your project and a need to separate them into teams, you most likely won't realize the intended benefits of micro services, but will take on considerable added tech debt to implement them.  Very simple point, but doesn't get mentioned enough.


## Monolits, Done Right

A lightweight base framework, proper modularization of your code base, and a good, configurable message / job routing library.  It's really that simple, and unless you have a massive system with 20+ developers, there is a 99.8% chance this is all you need.  It will keep development simple, organized, cohesive, well maintained and tested, scalable, and allow you to quickly become operational with no last minute surprises.

Realistically, for 98% of people reading this article, if you strip away all of the third party dependencies and extranneous assets (images, CSS, from your system, the resulting code will most likely be less than 100MB, and most times even far less than that.  A codebase of this size is no problem to maintain, scale, deploy across a plethora of server instances, and keep all instances updated with the latest version.

Here's some guidelines for a well structured monolith:

- Lightweight framework to provide uniformity, and efficiently route all incoming requests.  Ensure an efficient router that knows exactly where in the system to route each request, and only loads the minimum amount of code needed to complete each request.
- Framework that allows for well organized modularization of the code.  This can be simple Symphony packages, Apex packages, or a wide array of other options, allowing developers to work in isolation on different aspects of the system, but in a uniformed environment, providing proper separation of duties.
- Efficient events / jobs messaging router.  Basically all actions within the software should be handled via a messaging router allowing for scalability with a clear separation between front-end and back-end processes.
- Ensure the messaging router is configurable, meaning the ability to define which subsets of messages get handled by which back-end server instances allowing the system to scale with precision.  For example, you should be able to setup multiple back-end server instances, then configure the router so those instances only handle a certain set of messages allowing you to isolate and scale specific operations within your system as desired.  
- Maintain a separate change log for all modifications to interfaces, and ensure all developers on the team are immediately notified upon modifications to interfaces.
- This doesn't get mentioned enough, but spend time on your database schema.  A clean and well structured database architecture is insturmental to the success of your technical operation, so please don't discount its importance.
- Version control and a proper CI pipeline.  Once code is developed and tested, you need a way to deploy it across all server instances.  This can even be as quick and dirty as some rsync commands in a shell script, but there are better solutions out there.

If you are reading this article, more than likely a well structured monolith that ticks the above points is all you need.  Afterall, the end goal here is not to be trendy, but instead to have an efficient and robust software system that runs like a well oiled machine, correct?  Don't unnecessarily over-complicate things by going micro services, maintain a "simple is good" mantra, and reap the rewards of a well structured and organized monolith.


## When to use Micro Services?

The long answer is, it depends.  The short answer is once you have a minimum of 10+ developers working on the project, because generally it's only then the benefits of a micro services architecture will outweigh the added tech debt required to properly implement them.

General rule is you should always start with a monolith, and if and when the system becomes so large that micro services are required, you will know it and can begin the process of slowly splitting off chunks of the monolith into their own services.  I would urge caution here as well, as many developers will emphatically state micro services are required when that simply isn't the case.  Simply put, stick with a monolith for as long as possible, and 95% of the time this is all you will need for the entirety of your operation.

The one exception to this rule is when a different tech stack is required for a certain process.  For example, good chance your system will be developed in PHP, because whether people are willing to admit it or not, PHP is simply by far the best language for web applications.  However, maybe part of the system does video processing or consists of an AI algorithm with lots of data crunching.  In this instance, it makes clear sense to split that off into its own service with a Python stack, and build out an internal API that allows for communication between the system and the Python service.  


## What about Agility?

Good point, and without question micro services will generally be far more agile, but a counter argument can also be made.  Think of your system as an orchestra, with each micro service being a different type of insturment.  Each set of instruments practices together in isolation daily and absolutely perfect their portion of the performance.  However, unless the full orchestra practices in full enough together, there's a good chance performance night will turn into an out of tune train wreck.

The same goes for a micro services architecture, and is one of the main reasons to be wary.  If each micro service gets developed and perfected in isolation, unless everything is constantly developed and tested together in a uniformed environment, you run the risk of costly issues arising when it comes time to bring everything online, and have all micro services working together in tandem as a large cohesive system.  If this happens, then any previously preceived additional agility obviously goes out the window while you try to salvage the system due to all the unexpected failures that arose.


## In Conclusion, I Like Micro Services

Although this article will probably lead you to believe otherwise, I do actually really like micro services, and believe they are hugely beneficial when warranted and implemented properly.  The reason for painting them so negatively in this article is because anyone who is currently developing out a proper and warranted micro services architecture already knows what they are doing, will not be affected by this article, and most likely will agree with my assessment.

This article is geared towards folks who are currently debating whether or not to go the micro services route, and if you are having that debate, the answer more than likely is no.  When the time comes where a micro services architecture will be of benefit to your operation, it will be glaringly obvious without much of a debate.

I know the current hype and trend is to go micro services, but I assure you just as the hype from NoSQL died down, so will the hype of micro services in the coming years.  You can already search and read many stories about micro service failures, and you can expect those stories to appear more frequently over the coming years until the hype dies down.  I write this article to urge caution with hopes you do not become one of those failures, because it could potentially cost you, big time.


## About the Author

Matt Dizak is a software entrepreneur with 19 years of high paced experience within the online software industry, and is the 
creator of Apex, an open source PHP based software platform at (https://apex-platform.org/).  Please feel free to 
follow me on [Twitter](https://twitter.com/ApexPlatform), [Youtube](https://www.youtube.com/channel/UCa-gT-JbroF1EIbBB7nxHHg), or feel free to e-mail 
directly anytime at matt.dizak@gmail.com.




