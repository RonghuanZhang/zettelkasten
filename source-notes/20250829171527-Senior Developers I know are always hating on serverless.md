---
type: source-note
title: Senior Developers I know are always hating on serverless
id: 20250829170827
created: 2025-08-29T17:15:27
source:
  - web
url: https://www.reddit.com/r/serverless/comments/1b8zc51/senior_developers_i_know_are_always_hating_on/
tags:
  - source-note
  - cloud-native/serverless
processed: false
archived: false
---
I had a conversation recently with some Senior developers on my team about serverless and the responses I got were alarming to me. Everyone I talked to said things like "serverless ends up being more expensive", "serverless is to limiting", "serverless cold starts are a non starter" "serverless isn't viable", etc. It appeared to me that they were just spitting out talking points against serverless. All the conversations came off as red flags to me, that i'm working with developers that are way behind the times. Note that I'm not saying severless is the only tool or should be adopted by everyone, I'm just saying that serverless clearly has a bright future and many of the problems that existed early on have been solved and serveless is very viable for a large number of applications and companies. What are your thoughts on developers hating on serverless?

---

## Comments

> **kondro** • [12 points](https://reddit.com/r/serverless/comments/1b8zc51/comment/ktu1ycq/) •
> 
> As a dev with more than 20 years experience, it’s all a matter of perspective.
> 
> I just spent 3 days building a serverless solution that reliably and idempotently collects and aggregates data usage metrics from multiple AWS regions into a single region/database.
> 
> The serverless solution will cost sub-$10/month to run, even at scale across 6 regions. A non-serverless solution would cost thousands per month and would’ve taken weeks to build to get to the same level of reliability.
> 
> But this service is collecting metrics from a non-serverless system that’s designed to scale to millions of requests per second at running costs of sub $0.01/million requests (yes in AWS and yes, that includes bandwidth costs 😅). That’s impossible to implement with any kind of serverless implementation and critical to its viability.
> 
> So like everything, it’s a matter of using the right tool for the right job. Nothing needs to be completely serverless or non-serverless.
> 
> track me

> **Ceigey** • [7 points](https://reddit.com/r/serverless/comments/1b8zc51/comment/ktwb7tg/) •
> 
> Not red flags, just different point of views. There’s a lot of marketing around serverless but a lot of healthy skepticism to be had as well. It can have a bright future but companies and cloud providers have a motivation to make money now, with you as the beta tester ;-)
> 
> Speaking from my own experience in both worlds (and I’m still in both) and playing the devil/senior’s advocate:
> 
> > “Serverless is limiting”
> 
> It can be. AWS Lambdas have trouble running things asynchronously in the background unless you use Lambda on Edge. As a developer, you are encouraged to use AWS services in your architecture to get around that, eg EventBridge. But a standard Laravel, Django, ASP application might not face the same limitation and can run hooks after a request or use middleware without involving a whole event bus oriented architecture.
> 
> You also have to start worrying about queues and backpressure from events, etc. So normal auto scaling architecture issues, but serverless-native developers have to account for those problems earlier, because they’re immediately in the auto scaling world (see later comments about price, data egress, queries etc)
> 
> (It’s also easier to debug a service that can run locally, Serverless Stack is a good exception, as is Miniflare from Cloudflare, where they just ship the worker runtime to your computer 😂)
> 
> > “Serverless ends up being more expensive”
> 
> If you’re constantly under load, yes it can. Per unit of compute above free tier, it’s generally more expensive. For ephemeral services under free tier, yes it’s often cheap or free.
> 
> But your cloud provider doesn’t want you to stay under free tier forever, just remember that. They want you to architect your entire company around their services, then find it too hard to leave their cloud.
> 
> By the way, data egress costs are also relevant for serverless. It’s not always compute that’s the issue. Vercel and Netlify have some of the most expensive data egress fees. AWS, Azure, GCP are pretty expensive too.
> 
> > Serverless cold starts are a non starter
> 
> They can be. You should take this one pretty seriously. Only some configurations, runtimes, or application designs are immune to this. The more libraries your code brings in, the longer your cold start. There are ways to improve cold starts for the Java runtime on AWS, but it’s not 100% effective. Typically Go has one of the fastest cold starts, but it’s not immune. Especially since things have to be dynamically provisioned.
> 
> Compare to an ECS instance (via App Runner or Elastic Beanstalk) and the responsiveness can be day or night.
> 
> One historic way to avoid the cold starts was to keep a bunch of lambdas warmed up all the time. Well, that costs $$$ eventually, turns out it’s not very price effective ;-)
> 
> Serverless databases are also not always strictly speaking “serverless”, but rather have some sort of horizontal and vertical scaling processes in place. This scaling is not always perfect. They can still run out of concurrent connections. And queries can get expensive if your indices are not done correctly. These represent operational risks.
> 
> (Not all serverless DB solutions are equal or similar, eg Aurora Serverless v1 and v2 are very different to one another, and both are very different to eg Turso)
> 
> Sometimes it’s better to have performance or scaling issues, than to have everything scale perfectly… along with your monthly bill ;-)
> 
> > “Serverless is very viable for a large number of applications and companies”
> 
> Perhaps, if you don’t have any existing software or processes.
> 
> Regardless, Serverless Stack (SST) on AWS is a work of art… but I feel that ASP will be more likely to stick around longer (though I don’t use it… heh)
> 
> In the JavaScript world, there are developments like WinterCG that are promising for the standardisation of serverless request handlers. But we need to see this more consistently for the whole domain of serverless. Once things mature and standardise and the migration pathways become simpler, you’ll see more adoption.

> **heavy-minium** • [13 points](https://reddit.com/r/serverless/comments/1b8zc51/comment/ktshufw/) •
> 
> Might be specific to your situation. Most seniors I know are happy with serverless.
> 
> > **\[deleted\]** • [2 points](https://reddit.com/r/serverless/comments/1b8zc51/comment/ktsursg/) •
> > 
> > Whats the quote "if you're the smartest person in the room, you're in the wrong room"? NOTE: I'm not the smartest person in the room, far from it, but I want to be in the right room for me. I guess my question a two part question, one is my concerns that my circle is set in there ways and to hey each there own and two, why our some a many people I've met in the anti serverless camp. Maybe, I'm fully missing the serverless time bomb, but I don't think so. When you want to maximize personal development you need to be around the right people.

> **rdlpd** • [4 points](https://reddit.com/r/serverless/comments/1b8zc51/comment/kttohy8/) •
> 
> I dont think you have explained what services ur company has or their problem space or even the size of the company. But serverless is not the right thing to use 100% of time.
> 
> Also for me its not clear what most people call serverless these days as its becoming an overloaded term for too many things, eg: if you are in aws most people consider serverless lambdas, But aws calls (at least used to call) ecs, serverless also. After all alot underpinnings are shared.
> 
> In gcp google, calls cloud runs serverless, and so are cloud functions. But cloud runs are a similar product to ecs.
> 
> Now, If you are referring to the old debate about lambdas vs containers (like ecs).
> 
> Having done lambdas since 2017 and argued like you in the past, i think now every time tool has a time and place.
> 
> I think its more important to define a problem and focus on a solution, than to focus on the technology.
> 
> Looking at ur cost reference: If u get millions calls with each call at 100ms+ per day. Then maybe running an ecs containers is indeed cheaper.
> 
> There is always a threshold where aws lambdas become more expensive than ecs.
> 
> Regarding cold starts it depends how they are used and cloud do refer to, eg:
> 
> In lambdas java and .net were always super slow i remember waiting for my java lambdas to wait 7secs (when vpcs were slow at creating enis) but this was never really a problem with nodejs lambdas. I used to keep scheduler ticking a couple lambdas to keep them "warm" until aws added the ability to keep lambdas always on.
> 
> Gcp offers this with cloud runs. But to my knowledge it doesn't with cloud functions.
> 
> So again depending on what u do yes it indeed might be a problem.
> 
> Coordinating lots of lambdas also comes at a cost, vs small monolith with less moving pieces as its easier to deploy, test and prove if the product/feature you are building actually works.
> 
> All i am trying to say is that without context it is hard to say if you are indeed the cleverest person in the room or not, or your seniors are just old fashioned and lazy.
> 
> It all depends on ur constraints, the stage of the company you work for (eg a startup with a handful of employees most likely will fail to create a magical distributed system, where most things are event based, etc, as everything has a trade off),
> 
> Or if you work on a big old corp with a java monolith old bursting at the seams.
> 
> Maybe you could provide some more context. Or maybe you can think to yourself why is serverless so important, is it because you want to use the tech regardless of how it fits your problem space or is it because it solves whatever problem you want to solve brilliantly.
> 
> Using tech for tech sake usually ends up being replaced faster than using the best tech for the problem ahead and your problem space.

> **franchise-csgo** • [4 points](https://reddit.com/r/serverless/comments/1b8zc51/comment/ktsytg8/) •
> 
> Serverless is good when you know how to use it. Also it’s not a solution to every problem and it’s possible in their use case it is more expensive. Not everything should be solved with serverless. I also think there’s unjust hate towards serverless as well on top of that, mainly stemming from ignorance

> **uNki23** • [2 points](https://reddit.com/r/serverless/comments/1b8zc51/comment/ktw1neg/) •
> 
> If you start thinking about serverless technology / services being just tools for specific jobs and not a silver bullet for everything, it gets easier…

> **creminology** • [9 points](https://reddit.com/r/serverless/comments/1b8zc51/comment/ktslsu1/) •
> 
> Sounds like you have a theoretical perspective on serverless and they have an empirical one.

> **InfiniteMonorail** • [5 points](https://reddit.com/r/serverless/comments/1b8zc51/comment/kttet4b/) •
> 
> There's just so much baggage they don't tell you when you first get into serverless. It usually adds to latency, like API Gateway and Cloudfront both add +125ms each. Then you realize how easy it would be for someone to denial of wallet you and just hammer your endpoints unless you start adding CloudFront plus a WAF, which adds the previously mentioned +250ms. This is in addition to the already slow cold starts.
> 
> Okay, so you decide this isn't a problem for you. You've been sold on how great serverless is for scaling. But when it finally does scale, it costs 10x more.[https://www.primevideotech.com/video-streaming/scaling-up-the-prime-video-audio-video-monitoring-service-and-reducing-costs-by-90](https://www.primevideotech.com/video-streaming/scaling-up-the-prime-video-audio-video-monitoring-service-and-reducing-costs-by-90)
> 
> Amazon themselves can't figure out how to make serverless cheap. They had to scrap their entire infrastructure and rewrite it with servers to save a whopping 90%. That is the real non-starter.
> 
> So to use serverless, you need to be limited to an app that gets no traffic at all and you don't care about latency either. Basically it would be good for a prototype. Now consider that in my experience it takes about 10x longer to write a serverless app than a server app. So I guess it's useless for prototyping too.
> 
> Then there are other gotchas. Aurora Serverless V1 is abandonware. Aurora Serverless V2 doesn't shut down all the way and doesn't have a Data API. HTTP API is missing I think WAF and auth stuff, and Lambda URLs have nothing at all. All this stuff was difficult or impossible with SAM to test locally at the time I used it. You need a PHD in Dynamo just to use it as anything other than a key/value store and you need to plan the whole thing out ahead of time and never change it. It will take months or even years to discover all the gotchas and expect new products to not be fully supported by CloudFormation, CDK, SAM, etc for at least half a year. How much is your time worth to you, only to find out that it's worse in almost every way?
> 
> > **uNki23** • [4 points](https://reddit.com/r/serverless/comments/1b8zc51/comment/ktw1j7e/) •
> > 
> > Some valid points here, you know that you hit some nerves since you got downvoted :) I gave you a +1
> > 
> > Nevertheless, please also don’t forget that ECS Fargate is also serverless and that it’s not always API GW + Lambda.
> > 
> > And my personal opinion is that I‘d use API GW + Lambda if I really need a free HTTP API, and even the money for Fargate and ALB are too much. OR if I really need a super scalable API with heavily spiking traffic across endpoints. Usually a container in Fargate is my go to solution.
> > 
> > Where serverless also shines is event driven services like processing SNS / SQS messages in a scalable fashion.
> > 
> > Yeah.. right tool for the job.

> **Hot\_Butter\_Scotch** • [2 points](https://reddit.com/r/serverless/comments/1b8zc51/comment/ktsve0c/) •
> 
> I think you’re either expecting people to agree with you outright without ever doubting you, or they have to come up with some original argument to why server-less is not great other than the main talk points that everyone seem to agree with ( that’s why they become a talk point!)
> 
> they give you their opinion, and they give you reasons behind it. It is you who is close minded, and won’t take no for an answer. Cold start is a real valid argument, saying they are senior is not. The only thing you find attackable about them or what they say is that they are seniors and insinuating that they’re close minded. You can’t find a better argument against them other than categorizing them as closed mind seniors.
> 
> > **\[deleted\]** • [3 points](https://reddit.com/r/serverless/comments/1b8zc51/comment/ktt3x39/) •
> > 
> > Your edit is a very fair point and a credible argument against me. "You can’t find a better argument against them other than categorizing them as closed-minded seniors." I can't actually dispute this statement, and I acknowledge that perhaps my framing of their perspectives might have come across as dismissive and I do think using the term closed-minded seniors does make me the more closed minded one. I'll reflect on this.
> > 
> > I don't think that the full context of the conversations can't be highlighted in a Reddit post. I still hold to my original gut instinct that some of my peers were completely dismissing serverless as a useful technology for building applications and that concerns me. Maybe the fact that it concerns me is the bigger problem, though?
> > 
> > **\[deleted\]** • [2 points](https://reddit.com/r/serverless/comments/1b8zc51/comment/ktsxly1/) •
> > 
> > Several people are calling me closed minded and maybe the fault lies with me. I'll think about that. I'm open to the fact that I could be wrong. I don't expect people to agree with me. All I expect is a credible argument other than "serverless is junk" and has to many problems. Maybe it does, but that hasn't been my experience. If one of them had told be about a specific problem they had or an example, I would take that. The response weren't like, "we used it at xyz and it was a nightmare or we ran into this at my previous company", they were more like talking points.

> **druhlemann** • [1 points](https://reddit.com/r/serverless/comments/1b8zc51/comment/kttebhi/) •
> 
> yeah, I totally embrace serverless as an application architect, however mitigating things like cold starts and private networking, you're basically just running serverless hardware in a non-serverless manner. That being said there are still wins associated to it.

> **BarredButtonQuail** • [1 points](https://reddit.com/r/serverless/comments/1b8zc51/comment/ku03p55/) •
> 
> Well if you have a low latency service that constantly gets traffic serverless is shit. OTOH if you have large scale data processing/machine learning training etc it’s the model you want.
> 
> > **MyUsernamePls** • [1 points](https://reddit.com/r/serverless/comments/1b8zc51/comment/kuls7r3/) •
> > 
> > If your get constantly the same traffic then yes serverless is not the optimal solution.  
> > But if you get constant traffic and regular spikes in traffic then lambda with provisioned concurrency to serve the "constant minimum" is very much a viable solution. As you then get the cheaper pricing of provisioned concurrency to serve the constant traffic and AWS is able to scale instantly if there's a sudden surge in traffic.

> **Spare\_Pipe\_3281** • [1 points](https://reddit.com/r/serverless/comments/1b8zc51/comment/kuns4of/) •
> 
> I run a web SaaS on top of the Serverless framework, backed by AWS Lambda, S3 and DynamoDB. For this particular service we see that it has scattered use through the day and going serverless was a perfect fit. A more traditional architecture with the same level of reliability would have been a much more complicated endeavor. Yes costs are raising (almost linearly) with an ever growing user-base but our fixed cost (team, permanent infrastructure) to run all this is incredibly low.

> **Top\_Presentation8673** • [1 points](https://reddit.com/r/serverless/comments/1b8zc51/comment/labkwgc/) •
> 
> the issue with serverless is that you are very much designing your software around aws infrastructure. so you are basically catering to their system. if anything ever happens to aws your f\*\*ked. and you cant switch to another provider ever to cut costs running it locally or something. also the codebase becomes a scattered mess after a while with serverless.

> **Professional\_Hair550** • [1 points](https://reddit.com/r/serverless/comments/1b8zc51/comment/ktutt2q/) •
> 
> I am not a senior yet. But the value of serverless is not worrying about scaling the project. So If you suddenly get 10 times more traffic than usual then you won't lose any client because you are waiting for new instances to start running
> 
> > **Ceigey** • [3 points](https://reddit.com/r/serverless/comments/1b8zc51/comment/ktwco75/) •
> > 
> > This can be a downside if your application scales better than your budget (though the real hidden cost tends to be data egress rather than compute)