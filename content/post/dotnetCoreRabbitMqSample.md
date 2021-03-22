+++
author = "Emre Erkoca"
title = "Dotnet Core and RabbitMQ"
date = "2021-03-20"
description = "Dotnet Core RabbitMQ Sample with MassTransit Framework"
tags = [
    "Dotnet",
    "Dotnet Core",
    ".NET",
    ".NET Core",
    "RabbitMQ",
    "Message Broker",
    ".NET Core RabbitMQ",
    "MassTransit"
]
categories = ["Dotnet Core"]
+++
RabbitMQ is an open-source message broker. When it’s explained with words it might be confusing. I’ll try to explain via a diagram.	
<!--more-->

![Resources](/images/ExplainingProducerAndConsumer.png)

**Producer:** Producer publishes messages about events. The producer is an app in this diagram. When the user posts a new article system creates an article and publishes a message (ArticleCreatedEvent).

**Consumer:** Consumers are listening to messages. When a message publishes, consumers trigger and read the message context. When the ArticleCreatedEvent message publishes, NotificationSenderConsumer and ArticlePriceCalculatorConsumer trigger in this diagram. NotificationSenderConsumer runs SendNotificationCommand and it sends notifications to followers of the author. 
ArticlePriceCalculatorConsumer runs CalculateArticlePriceCommand and it calculates the price of the article. 

I made a sample project to explain this subject. https://github.com/emreerkoca/dotnet-core-rabbitmq-sample

If you want to write sample code about RabbitMQ, you can download and install RabbitMQ.
https://www.rabbitmq.com/ 

I used MassTransit in this project. MassTransit open-source distributed application framework for .NET. 

https://masstransit-project.com/

I’ll explain the project step by step. I suppose you installed RabbitMQ.

**Create a Dotnet Core Web Application (Web API)** 

Add MassTransit package to your project:

{{< highlight bash >}}
$ cd DotnetCoreRabbitMqSample.Api
$ dotnet add package MassTransit
{{< /highlight >}}

If you don't work with commands you can install the package from Nuget Package Manager.

Create MembershipStartedEvent class. It's an event. It'll keep your information to publishing.

{{< highlight csharp >}}
public class MembershipStartedEvent
{
    public MembershipResponse MembershipResponse { get; set; }
}
{{< /highlight >}}

Create IMembershipService interface:
{{< highlight csharp >}}
public interface IMembershipService
{
    MembershipResponse PostMembership(PostMembershipRequest postMembershipRequest);
}
{{< /highlight >}}

Create MemberShipService class: 
{{< highlight csharp >}}
using AutoMapper;
using DotnetCoreRabbitMqSample.Api.Contracts.Events;
using DotnetCoreRabbitMqSample.Api.Contracts.Requests;
using DotnetCoreRabbitMqSample.Api.Contracts.Responses;
using MassTransit;

namespace DotnetCoreRabbitMqSample.Api.Services
{
    public class MembershipService : IMembershipService
    {
        private readonly IBusControl _busControl;
        private readonly IMapper _mapper;

        public MembershipService(IBusControl busControl, IMapper mapper)
        {
            _busControl = busControl;
            _mapper = mapper;
        }

        public MembershipResponse PostMembership(PostMembershipRequest postMembershipRequest)
        {
            //We suppose we saved user's data
            MembershipResponse membershipResponse = _mapper.Map<MembershipResponse>(postMembershipRequest);

            _busControl.Publish<MembershipStartedEvent>(new MembershipStartedEvent
            {
                MembershipResponse = membershipResponse
            });

            return membershipResponse;
        }
    }
}
{{< /highlight >}}

PostMembership method just publishes event. We suppose it already saved the user's data. You can publish a new event like this. 
When an event publishes, it process through a queue. A consumer consumes your message and it works your commands. When it got an error while working your message will be sent to an error queue. When you have messages on error queues you can move and process them.

You can open your local RabbitMQ panel from this URL: http://localhost:15672/ 

A queue:

![Resources](/images/messageConsuming.png)

**Ready:** Queued message count for waiting to process.

**Unacked:** Count of processed messages but it's not clear. So, the consumer is saying "Yes, I'll consume them" but there is no information about the process. When you see decreasing total message count you can understand they're consuming. 

An error queue:

![Resources](/images/errorQueue.png)    

Now add a new Controller (MembershipController.cs)

{{< highlight csharp >}}
using DotnetCoreRabbitMqSample.Api.Contracts.Requests;
using DotnetCoreRabbitMqSample.Api.Services;
using Microsoft.AspNetCore.Mvc;

namespace DotnetCoreRabbitMqSample.Api.Controllers
{
    [Route("membership")]
    [ApiController]
    public class MembershipController : ControllerBase
    {
        IMembershipService _membershipService;
        public MembershipController(IMembershipService membershipService)
        {
            _membershipService = membershipService;
        }

        [HttpPost("")]
        public ActionResult Post([FromBody] PostMembershipRequest popstMembershipRequest)
        {
            var result = _membershipService.PostMembership(popstMembershipRequest);

            return Ok(result);
        }
    }
}
{{< /highlight >}}

If you confused about IMemberService you can search "dependency injection" and "dependency injection constructor injection"
I created a service class and I used it. There is no business or data access layer because it's not a software architecture tutorial. 

PostMembershipRequest method sends postMembershipRequest object to PostMembership method.

It's not necessary to explain each part. You can check the repository for request classes, model classes, or response classes. 

The consumer class looks like this. When the message published it comes to here:

{{< highlight csharp >}}
public class MembershipStartedEventConsumer : IConsumer<MembershipStartedEvent>
{
    ILogger<MembershipStartedEventConsumer> _logger;

    public MembershipStartedEventConsumer(ILogger<MembershipStartedEventConsumer> logger)
    {
        _logger = logger;
    }

    public async Task Consume(ConsumeContext<MembershipStartedEvent> context)
    {
        MembershipResponse membershipResponse = context.Message.MembershipResponse;
        EmailModel emailModel = new EmailModel
        {
            EmailAddress = membershipResponse.Email,
            Subject = "Welcome to X Community",
            Body = "You're officially an X community member. Congrats :)"
        };

        await SendEmail(emailModel);
    }

    public async Task SendEmail(EmailModel emailModel)
    {
        //Send email
        _logger.LogInformation("Email: " + JsonSerializer.Serialize(emailModel));
    }
}
{{< /highlight >}}

**MassTransit Configuration**


{{< highlight csharp >}}
using System;
using System.Threading.Tasks;
using EventContracts;
using MassTransit;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddMassTransit(x =>
        {
            x.AddConsumer<MembershipStartedEventConsumer>();

            x.SetKebabCaseEndpointNameFormatter();

            x.UsingRabbitMq((context, cfg) =>
            {
                cfg.ReceiveEndpoint("MembershipStartedEventConsumerQueue", e =>
                {
                    e.ConfigureConsumer<MembershipStartedEventConsumer>(context);
                });
            });
        });

        services.AddMassTransitHostedService();
    }
}
{{< /highlight >}}

It's adding consumers and defining queue names.

If you created other parts or downloaded the repository, it's ready to run. 
Put the breakpoint on Consume method and send post requests. You'll see your messages will come to Consumer.

Post membership operations diagram:

![Resources](/images/PostMembership.png)

I can make mistakes. Because I'm not a machine. If you see any mistake in this article you can tell me. I hope it might be helpful. 





