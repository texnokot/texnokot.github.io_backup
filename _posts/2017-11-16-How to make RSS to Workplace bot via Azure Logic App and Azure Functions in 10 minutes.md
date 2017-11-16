

## Introduction
Let's make simple notification bot, which sends notifications from Azure health page to Facebook Workplace group by getting RSS feeds. Components are used:
* RSS from [Azure Health page](https://azure.microsoft.com/en-us/status/)
* Azure Logic App: will listen for a new RSS feed and trigger Azure Function
* Azure Function: will get RSS feed title and summary and will create formatted post in dedicated group 
* Facebook Workplace in two words is Facebook for company usage. More information [here](https://www.facebook.com/workplace)


## Facebook Workplace
Let's start with Facebook Workplace. Choose or create a group, where notifications shall land. Next step is to make custom integration ([more](https://developers.facebook.com/docs/workplace/integrations/custom-integrations/apps)). 
NOTE: To be able to create and manage access tokens for apps you should be *System Administrator*.


Make custom integration:
![Create app](https://publicbw.blob.core.windows.net/articlerss/workplace_app_0.png)

Give proper rights to an app:
![Rights app](https://publicbw.blob.core.windows.net/articlerss/workplace_app_1.png)

Select to which group notifications should go:
![Group app](https://publicbw.blob.core.windows.net/articlerss/workplace_app_2.png)

Remember Group ID and Access Token for an application. These will be used in Azure Function.


## Azure Function
Next step is to make a function, which will get RSS title and summary from Logic App. The code is written in JavaScript. We will use a generic webhook for this and give a name:
![Azure function](https://publicbw.blob.core.windows.net/articlerss/Azure_function_create.png)

Create a function:

```javascript
module.exports = function (context, data) {
    context.log('Webhook was triggered!');
    context.log('Feed title = ' + data['title']);
    context.log('Feed summary = '+ data['summary'] )
    var request = require('request');
    var graphapi = request.defaults({ 
        baseUrl: 'https://graph.facebook.com',
        auth: {
                'bearer': process.env.ACCESS_TOKEN
            }
    });
    graphapi({
                method: 'POST',
                url: '/'+ process.env.TARGET_GROUP +'/feed',
                qs: {
                    formatting : 'MARKDOWN',
                    'message': '# '+data['title']+'\n'+data['summary']
                }
            }, function (error, response, feed) {
                if (error) {
                    context.error(error);
                } else {
                    context.log(Date.now());
                }
            });
    context.done();
}
```


## Logic App

Next step is to create Logic App:
![Azure Logic App](https://publicbw.blob.core.windows.net/articlerss/Create_logic_app.png)

After Logic App is created, choose Blank Logic App. Take action RSS and configure RSS feed address and interval:
![Azure Logic App feed](feed_publishing)

Add connector and choose Azure Functions, select your function. Your function is a web hook and will expect a JSON payload with two fields as an input.  Be sure that payload is exactly same as shown:
![Azure Logic App feed](https://publicbw.blob.core.windows.net/articlerss/feed_config.png)

So the result should look like:
![Azure Logic App feed](https://publicbw.blob.core.windows.net/articlerss/feed_result.png)

Now everything is done! When Azure health status publishes some changes App Logic will trigger Azure function and post on Facebook Workplace will be created.
![Workplace feed](https://publicbw.blob.core.windows.net/articlerss/workplace_result.png)


## Summary
* Now we have bot publishing Azure Health status into Facebook Workplace
* The creation of bot took less time than writing this post


## Bonus track
If you want to have Azure Health notifications into Slack, just integrate it by going into Slack - taking Apps and choose RSS.
Done:
![Slack](https://publicbw.blob.core.windows.net/articlerss/slack.png)








