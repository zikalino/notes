# Creating Azure IoT Hub Bot

## Background

This occured to me just a few days ago.

Currently there are no applications for my iPhone that would allow me communicating with Azure IoT Hub.
There is **Azure App** for iPhone, but the only thing you can do is to see a list of your IoT Hubs. Or maybe even this is not possible. So I was planning for a long time to make my contribution to **Azure App** and add IoT Hub related functionality, but now somehow occured to me that it would be even easier just to create **Azure IoT Hub Bot** and enable access to IoT Hub via many more channels than just iPhone or Android.

## So what's out there?

I started my research and found there's already available **Azure Bot**:

https://bots.botframework.com/bot?id=azurebot

It's more a sample bot as functionality is limited, as stated in overview:

    Be more productive with your own Microsoft Azure subscriptions using a natural language. Current support includes starting, stopping, and listing Azure VMs and starting Azure Automation runbooks. Note: Only supports Azure AD accounts, not Microsoft / MSA accounts.

It works with:
- Skype
- Slack
- GroupMe (I have to admit I have never heard about GroupMe)
- Microsoft Teams

I have tried the bot, and it already has one function I will need for my **Azure IoT Hub Bot**, and that is authentication. It works pretty nicely.

User can just type **login** and then click **Please click to sign in**, which causes redirection to standard Azure authentication interface in the browser. After logging he just has to copy the token back to Skype. And that's it.

![Docker VSTS](images/azure-bot-skype.png)

## Deploying Azure Bot

The simplest approach to create **Azure IoT Hub Bot** is to build on existing project, which is available here (MIT License):

https://github.com/Microsoft/AzureBot

The only disadvantage is that the project is done in C# for ASP.NET. I would rather prefere Node.js and deploy it in Docker, as I have some other plans for the future, but for now it's the simplest way to build the prototype.

First step is obviously to clone, build and deploy the bot as it is just to see how the process works.

