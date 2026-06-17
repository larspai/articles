# ESB and Business process articles
## [Enterprise Architecture Value](EnterpriseArchitectureValue.md)
An approach to enterprise architecture that is less concerned with tooling and complex model building and more with what creates value short- and long term.

## [ESB Business Process Relations](ESBBusinessprocessRelations.md)
How an ESB may efficiently support business processes

This article tries to contextualise the use of an event driven, publish-subscribe architecture, to support business processes and to make key object data readily available across the enterprise.

## [ESB Logging and Monitoring](ESBLoggingAndMonitoring.md)
While implementing an ESB is not too difficult, it is often forgotten to produce a reporting framework around it, to allow its stakeholders to see exactly what transactions are flowing between the systems, what data was actually sent and what was received. This is, however, a rather important component, both for you (for debugging and for exposing and advertising your service), for the users/stakeholders, but not least in a compliance/control/audit context.

## [ESB Routing](ESBRouting.md)
In this article, I will illustrate with a Minimum Viable Product for a truly tiny, yet efficient ESB, just how little you need, to implement the basic principles of the ESB, and I will discuss the routing part and suggest some workable guidelines. I will also touch upon the subscription and transformation but keep a deeper discussion for another time.

## [ESB Message Format](ESBMessageFormat.md)
In this article, I will discuss the messages, their format and transformation. In a later article, I intend to do a broader discussion of homegrown vs. commercial implementations.

## [ESB Buy or Build](ESBBuyorBuild.md)
In this article, I will discuss how to choose technology, and more specifically if you should code your own ESB or if you should choose a commercial solution like MS BizTalk, Mulesoft, WSO2, Azure Services, Tibco etc. - a discussion that will fall into two perspectives: a technical one and a legal, compliance or corporate one.

## [ESB An Even Simpler Bus](ESBAnEvenSimplerBus.md)
In this article, I suggest publishing to happen through a generic web service. The routing is handled by RabbitMQ exchanges and queues, and subscription is done by consumers listening for messages on queues.
