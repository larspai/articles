# ESB - Buy or Build?
First published 31 Aug 2018
A discussion of commercial vs. home grown ESB solutions
## Introduction
In earlier articles, I have discussed general ESB concepts, routing and message formats, to provide a pragmatic approach to the ESB pattern. In this article, I will discuss how to choose technology, and more specifically if you should code your own ESB or if you should choose a commercial solution like MS BizTalk, Mulesoft, WSO2, Azure Services, Tibco etc. - a discussion that will fall into two perspectives: a technical one and a legal, compliance or corporate one.

## What is an ESB in the First Place
First of all however, I will spend a little time discussing what an ESB is, mainly because one member of CodeProject pointed out, that not explaining what the key acronym of an article is, is not acceptable (thanks!), but also because that provides a nice frame for the following discussion.

An ESB (Enterprise Service Bus) is a mechanism that allows data to be shared among systems, typically business systems like ERP, CRM, etc., in a real-time, but asynchronous manner. The basic principle, or pattern, is that of publish-subscribe, meaning that systems publish data (in the form of messages, typically related to the change of some object/entity), while other systems subscribe to the data and are delivered the messages in a form understandable to them.

It is, in other words, typically a one-to-many communication and therefore the source systems will only know that its published messages were received by the ESB, not whether it was successfully delivered to any destination systems.

This pattern has a lot of advantages, but not surprisingly also a couple of caveats.

First of all, it is a way to prevent integrations from being point-to-point, which will lead to the slow death of any application landscape (SaaS or on-prem, regardless of whether the integrations are homemade or commercial plugins), but it also offers a lot of opportunities to manage, monitor, throttle, log and regulate communication, that would otherwise be difficult, even impossible.

On the other side, the decoupling nature of the ESB means that it can be hard to know, if systems actually were updated, so it may require a reporting framework and reconciliation routines to cover for these shortcomings (something I have discussed in an earlier article).

## Technical Perspective
Because of the asynchronous nature of an ESB - of the publish-subscribe pattern really, an ESB may be heavily componentised. The ESB consists of:

- receive mechanisms
where source systems publish their data and the ESB receives it
- a routing mechanism
where messages are copied and distributed to all relevant subscribers
- transformation mechanisms
which transform the messages into messages relevant to the subscribing destination system and
- delivery mechanisms
which are able to deliver messages to the destination systems

Each of these components may be implemented as completely independent systems and they may even run on their own server instances in each of their own environment. Therefore, they may also be implemented in each of their own technology - though this probably only very rarely is desirable.

In larger organisations, the ESB - or integrations in general, are typically created and maintained by a dedicated team, often with a fairly uniform skillset. The logical consequence is to implement in technologies inside this. However choices may have been made to, i.e., implement all externally facing end-points in some proven robust and heavily monitored technology. The ESB easily adapts to that, as this will affect the receiving mechanism only.

The list of components still looks daunting though, it’s a lot of functionality to implement, but as I have shown in earlier articles, it may not be quite as bad as it seems. First of all, you should always try to only implement functionality that is particular to your needs - if you have the choice.

Looking at the list of components, the routing mechanism is an example of functionality that already exists and reimplementing this will in most cases only lead to a less efficient result than choosing some of-the-shelve solution, like a traditional database (like BizTalk uses SQL Server), an in-memory database with particular pub-sub functionalities (like Redis) or my personal favorite AMQP message queue RabbitMQ.

Most development teams probably have database skills, and a database solution may seem the obvious choice - and a database has a lot of compelling qualities, like the possibility to look into the details of the data that is in there at any given time.

It is my experience however that, even though a simple distribution of messages seems like a doable thing (insert row, copy row, read row, mark row as read, delete or move read rows), it turns out to be a challenge to keep the database empty (all messages delivered), and as time goes by, it becomes increasingly difficult to find out, what has happened, what to do, how important the data in the database is, etc. It becomes an operations problem.

I also strongly believe that the routing mechanism should be event driven, and with a traditional RDBMS, this requires writing triggers. Design complexity increases.

For the use cases I have experienced, it has always been a key feature of the ESB to provide a high level of delivery reliability. For that very same reason, I have never considered the Redis pub-sub functionality an option, as it (to my knowledge) requires the receiver to be there at all times, to be sure to receive all messages. There may well be use cases for this though.

This leaves us with the message queue. It has (optional) persistence, it is event driven and its simplicity leads to fairly simple operations scenarios, simple monitoring, alerting, etc. and it has simple, metadata based, routing key mechanisms. It is my preferred routing mechanism. (Check out www.rabbitmq.com).

This still leaves the receive location, the transformation and the delivery components to be implemented by hand! Yes, it does. This is where your particular integration needs become visible, becomes the requirement specification for the work to be done.

Both receiving locations and consumers may be individually implemented. As mentioned, it may even be in mixed technologies. While this probably is not desirable, it does open for experiments with suitable technologies. A consumer of a particular message type written in Go may well live together with a Mulesoft flow consuming the same message type.

So what do you get with the commercial products? Well, they differ!, but in many cases, you get a kind of an application server; often with that a management console, and sometimes a development environment.

I don’t believe much good can be said about application servers(!). Often, they represent an extra single point of failure, but even worse, they may let errors in one component spill over to all the others. I have experienced one Mulesoft flow succeed in crashing in a way that took the entire server down, and thus all the other running flows with it. For this reason, I would suggest running each flow with its own server instance - as it is also offered by the cloud version of Mulesoft, or I would go in a much leaner direction and choose some technology that does not include an application server.

Talking about Mulesoft, this offers an IDE where flows may be build in a graphical user interface, basically wiring functional micro components together. I’m a programmer and don’t particularly fancy not knowing exactly what code is running and not having access to it. I will say however that it is easy to understand these graphical representations of flows and they are more easily accessed by someone who did not create them.

If I were to reason for an application server, I would emphasize their management and reporting capabilities. They are often able to give detailed information about running components, and offer the handles to start and stop them, something you will otherwise want to implement somehow.

I have not experienced significant differences in productivity or development speed by choosing a commercial product, but if you are facing your first implementation, it may be beneficial to choose an environment where things are done in one particular way. You will get requests along the way and it can be difficult to stay true to your design if you have limited experience with it.

So from a technical point of view, there is no right answer. You have to look at your organization's development skillset - can you stay inside it or do you have to add to it? If you choose to buy a commercial solution, you will have to educate your developers to use it properly. If you choose to build your own solution, you should educate your developers in your design and architecture.

Similarly, you should think about your operations environment. What will the deliverable be like, how may it be monitored? What can your operations team do when problems occur, etc.? What is already in your technology stack?

## Legal/Compliance Perspective
ESBs are rarely implemented as hobby projects, but on the contrary often are part of an enterprise application landscape and therefore must fit in a number of ways - the available skillset(s) as mentioned, operational possibilities, legal/compliance requirements, etc.

The legal and compliance aspects are little recognized, but actually at least as important as the technical aspect.

In an audit/compliance context, internally developed solutions may at best be a time consuming experience. Auditors will consider internally developed code a liability and will have to dive into deep details to understand what is happening and not happening to data, what the risk and security levels are, etc. You may want to consider “auditability” as a non-functional requirement in your design; basically, this would be about providing check points for auditors to see what is going on, and logging at relevant points (primarily what comes in and what goes out).

Similarly, your legal department will need to know everything about the licensing of all libraries that you may be depending on. If you happen to infringe or compromise some code ownership in some library deep down in your implementation, the cost consequences may be very serious.

Therefore, you should at least have the design well documented and all licenses kept readily accessible and in an updated state, reflecting the version of the various code libraries that you are using (including dependencies you may have “inherited” with the libraries).

This is typically not the case with commercial products. The situation is a little more uncertain with open source products. These may not guarantee against dependencies on third party code and therefore are seldom welcomed for mission critical software by a legal department. With most mature open source projects, companies exist who sell and support the product and who take the legal responsibility. You will have to consider for yourself in each instance how you value this sort of guarantee.

Bottom line in that discussion is that while it is not at all impossible to become compliant with PCI or SOX or CFR part 11 or similar frameworks with your own code, it will complicate the process.

## Conclusion
Overall, the same principle applies to the choice of ESB as to the individual components of it: you should only spend development efforts on differentiating processes. There is however not much that is standard in an ESB - besides the routing mechanism and some connection components. I believe the choice of buy or build really mainly depends on your organization. I hope however, that I have provided some views here that may help to qualify your choice.