# ESB - Business Process Relations
First published 3 Oct 2023

How an ESB may efficiently support business processes

This article tries to contextualise the use of an event driven, publish-subscribe architecture, to support business processes and to make key object data readily available across the enterprise.
## Introduction
Some time back, I wrote a couple of articles about various aspects to ESBs (Enterprise Service Bus). They were discussions of considerations you may have, when using a publish-subscribe mechanism for distributing event information between applications: What should the messages look like? What should govern the routing? How should you log the activities in the pub-sub mechanism?

In this article, I would like to contextualise and answer the questions: How do integrations tie into business processes and how do business processes determine the required type of integration? A slightly more enterprise architecture level take on event driven integrations.

## Business Processes?
When I try to think about business processes in a generalised way, I come to think of Porter’s value chain. It basically states that there is a primary production process and a number of supporting processes, like selling and bookkeeping. In order to be a little bit more explicit about how an ESB can solve integration requirements in an efficient way, I would like to introduce a process model I made about 10 years ago. It reflects the thinking in Porter’s model, but is a bit more specific in explaining what relations there could be between processes.

The model defines or describes three processes: Quote-to-Cash, Provisioning and Production.

It looks like this:

Image 1

In ordinary terms, the model is about three questions: How do we produce our product? How do we sell it? And how do we make sure the customers get what they paid for?

In a SaaS business, offering some web product, you would think of the service as the product, the Quote-to-Cash (Q2C) process is how your subscription is created, possibly also through the web site, and provisioning is how your system knows what the customers should be given access to.

Interestingly, the model has proven to be applicable to very different company types. Lately, I have used it to describe basic application relations in a publishing company. I call it my standard-H model - although it is an “H” lying on the side.

In a little more detail, the three processes are:

1. The **Quote-to-Cash** process, which is about the sale and related postings, and in this narrowed context is where the money is handled. It is a small part of the typical company finance processes, and will typically also include a payment flow, either from a purchase flow in a web frontend or from payments received from paid invoices.
In this process, an order is created in the first step. This could be in a managed customer flow, i.e., by a quoting process in Salesforce CPQ, or it could be initiated in an online order form, or any other purchase flow, like an in-app purchase in a mobile app. In any case, an order is created and handed to the eCommerce application, which creates the actual order, captures a payment, possibly creates a subscription, an invoice and anything else relevant to the order. The purchase is then reported to the ERP system with the details required for the books, revenue recognition, annual reporting, etc.

2. The **Provisioning** process, which is initiated with the creation of the order in the eCommerce system. This process is about making sure that the customer gets what they have purchased, and could be, i.e., by expressing the purchase in shape of user entitlements, that may inform some production system, about what the users should be allowed access to. It also could be a production order sent to a MES system (Manufacturing Execution System). Thus the last piece of the provisioning process is some part of the production system - in a web application, it would be the application frontend.

3. The **Production** process, which consists of some input system or backend, a business logic and presentation component and some delivery point. In a web application context, the components may be a CMS, a web server application and the client browser.
To come a little closer to the flows and integrations, and integration types, I will add some arrows and objects to the standard H model, to indicate what sort of data could be flowing between the components. I will also change the name of a few components: The model is generic and the important thing is to use it to get a mental image of the processes in the enterprise:

Image 2

## Event Driven or Request-Response?
Looking at the arrows, it is quite obvious which relations might be event driven, and which would be request-response type. Thinking about the processes, it makes sense that some flows happen as consequences to events: the order being created in the CRM (Customer Relation Management) system, causes the order to be sent to the eCommerce system, while others are triggered by requests: the user requesting for some data requires the API to ask the entitlement service, if this user has the right to get it.

It is only the event driven integrations that are relevant in an ESB context, but this also happens to be the quality of the majority of the integration points in the figure. In a context where your product is a web application, you may want to spend some thought on the fact, that most integrations in the backend are event driven, one-way, asynchronous, while the integrations involved with the customers tend to be request-response, two-way and synchronous.

I have discussed the main characteristics of the ESB in my other articles, the bottom line being that an ESB is a place where applications can publish data, for other applications to subscribe to. When an object is changed, the application may publish the new state of the object (preferably by publishing the entire object), and applications that need to know about the change can subscribe to the message and update their own data accordingly with no further ado.

But what exactly should the pub-sub infrastructure look like and what components would be involved?

I have earlier suggested that an ESB should offer:

- a receiving location
- a routing mechanism
- a transformation mechanism
- a delivery mechanism

These components create the basics in transporting and transforming a message from application A to application B:

Image 3

However, this simple setup (you only need to create the web service endpoint and the transformation/delivery component - the message queue is a commodity) is not enough, if you want to create an infrastructure for enterprise-wide distribution and reuse of key data objects like orders, subscriptions, users, products, payments, etc.

In an earlier article, I argued that applications should always deliver their messages on an exchange of its own. I would add to this that I believe that the application should deliver messages in a format that is suitable or even standard for it. It may make sense for some applications to subscribe to these messages as they are, and handle the particular format transformation on their own. Obviously, since there is no abstraction of the source system, the integration becomes a kind of asynchronous, point-to-point integration, but this may make sense in, i.e., migration scenarios, intermediate solutions or for logging or monitoring - in short: integrations that would go away with the source system.

Of course, a lot should be said here about contracts and versions - how to let changes happen to the delivered messages and still avoid that these changes break the downstream subscriber logic. I have touched on this in my other articles.

In an enterprise context, you probably want however, to work with more canonical formats for your key objects, for instance, to allow for more applications to produce messages of the same type: you might as mentioned earlier have orders created by sales people in your sales organisation as well as orders created by the customers themselves on your website. It should not necessarily matter to subscribing systems which application created an order, thus you might want to have a canonical order format for consuming applications to subscribe to.

To summarise, I suggest that:

- messages are published to source application specific exchanges, in source application specific formats
- messages may be transformed to canonical object messages and published to process specific exchanges

This suggests a somewhat more complex setup, but as mentioned in an earlier article, the components may be almost generic and only differ in i.e., the transformation logic. The design would look something like this:

Image 4

Note that in the above, I have omitted separate delivery components. This responsibility can be part of the transformation component, but it may be practical (i.e., to be able to have entirely generic transformation components) to separate the two responsibilities in each their own components.

Applying the design to the processes above, an implementation might look like this - as a starting point:

Image 5

With a landscape like this in place, any application in the enterprise may set up their own queues to listen for event messages describing the state of changed objects in the publishing applications. A couple of such consumers are illustrated in the figure, to show how, i.e., the ERP system may listen for invoices or the Business logic component listen to backend data input.

Often, you will see APIs build as such consumers, in order to add a request-response option, for applications to be able to request, i.e., invoice data for a given invoice ID. APIs will be simple caching mechanisms consisting of a transform and insert component, a database where messages are stored and an API endpoint that retrieves the objects from the database.

An example of such APIs could be the entitlement service and the production flow, if we were thinking of a SaaS application of some sort. The transformation components could be responsible for inserting the transformed messages into the API database:

Image 6

## Some Thoughts on Models
I have spent a lot of time on models in my time, from Yourdon to UML and I strongly believe in them. Actually, it is my belief that the way we understand our world is by model: we do not understand phenomena as they are, we create thought-models to represent them and to understand their characteristics and properties. Models are central to our understanding of things.

When creating models to convey the understanding of systems like the ones I have been discussing here, I believe there are two important criteria for them:

1. They must be intuitive. Using a standard like UML has some advantages when it comes to the density of the information that may be embedded in the model: Objects may take different shapes to express their type, and relations may have shapes and properties to do the same. However for that to work, it requires that the ones reading the model, know all of this. If they don’t, the extra information becomes noise, reducing the readability. And since most people don’t look at UML models every day, I tend to make models with only two or three shapes: bubbles, boxes and arrows. Arrows indicate a direction of flow, and the boxes may contain logic of any kind, functions or tasks that has to take place. Most people can grasp that without introduction.
And it is still possible to add to the shapes, properties that you read intuitively: in the model above, I have added lines in the exchange and queue boxes to indicate their functionality, and the web service in front of the message queue components is tiny, to indicate that it is merely a place to receive http requests - very little logic takes place there.

2. The scope of the model has to be right. It does not make sense to have overly simplistic models like “we produce stuff and then we sell it”. We all know that. On the other hand, it only rarely makes sense to have large and detailed models with lots of boxes and routes. I have sometimes used complex UML diagrams to illustrate, i.e., complex purchase flows, but they have often mainly been to gain an understanding of the flow myself, not for communicating what is going on. And if you get closer to code, it seldom makes sense to make models - the code often is the place to look anyway.
You want to hit the sweet spot, where the model is easily understood yet still brings information - documents design decisions, encapsulates roles and responsibilities, etc. The two models above do that (to my best understanding) at different levels: One gives an overall picture of what functionality goes where, while the other will allow you to understand, what components you will need to build, in order to i.e., add a consumer of a particular message, and where to put it.

The most important thing about models though is to create a common image of things. When we have a common image, conversation about issues related to the model scope becomes much simpler and much more precise.