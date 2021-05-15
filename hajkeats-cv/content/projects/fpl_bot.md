---
title: "Fantasy Premier League Bot"
date: 2021-01-04T15:00:00Z
---

### [Repository](https://github.com/hajkeats/fpl_bot)


## A crude notification bot

This Christmas break I got bored. With coronavirus rife across the UK, and tier 4 restrictions in place, I had little to do but sit about watching football on TV, occasionally fiddling with my [Fantasy Premier League](https://fantasy.premierleague.com/) team, something that I'm pretty good at. In fact... I've won my fantasy league each year for as long as I remember. Currently I sit at 79,498th out of 7,791,110 places, and top of both the classic and head-to-head style leagues that 5 of my friends and I participate in. That said, I'm [not quite Magnus Carlsen](https://www.theguardian.com/sport/2019/dec/16/chess-champion-magnus-carlsen-top-of-world-fantasy-football-rankings-premier-league), and so have decided to leverage the FPL API to help elevate my skills to GrandMaster level.

{{< image src="/images/magnus-carlsen.png" alt="Magnus Carlsen Twitter" >}}

One issue that I face is easily solved. Although I usually do better than my friends at staying interested throughout the season, I still occasionally forget to change my team in time for a new gameweek. This can lead to disastrous consequences, and is very easy to do with the midweek fixtures around the middle of the season. In order to remind myself and the other members of the fantasy league, I decided to build a facebook messenger bot for our group chat, with 2 functions:

1. Remind the chat that a gameweek is about to start.
1. Update the chat on results of head-to-head games.

### Challenges

#### Use of fbchat

Unfortunately, the platform used for our FPL chat group is Facebook messenger, which has [recently demoted chatbots](https://techcrunch.com/2020/02/28/messenger-removes-discover/) in favour of a simpler and more streamlined app. Chatbots are now mainly considered as marketing tools tied to pages. Creating such a bot is fairly simple, and would be possible with a combination of AWS [API Gateway](https://aws.amazon.com/api-gateway/) and [Lambda](https://aws.amazon.com/lambda/), but it is not possible to add a page to a group chat.

Instead, I used Lambda and the [`fbchat`](https://github.com/fbchat-dev/fbchat) Python library.

_"fbchat works by emulating the browser. This means doing the exact same GET/POST requests and tricking Facebook into thinking it’s accessing the website normally. Therefore, this API requires the credentials of a Facebook account."_

The `fbchat` repository is currently unmaintained, hence the hacky workaround required [in the code](https://github.com/hajkeats/fpl_bot/blob/master/src/h2h_bot.py#L12-L15). 

#### Asynchronous FPL library

The other main challenge came in accessing the FPL API. All the endpoints seemed to work without login initially, then it became apparent that the head-to-head league endpoint only allowed the user to access the first 50 records. When this was noticed I sought to incorporate the [fpl](https://fpl.readthedocs.io/en/latest/) wrapper. Being an asynchronous implementation, it didn't work naturally with lambda which does not support asynchronous handlers. To get around this issue, the following code was used to wrap the coroutines so they could be treated synchronously:

```
def run_sync(func):
    """
    Call an asynchronous function from a synchronous one.
    :param func: The asynchronous function to be called
    """
    def sync_wrapper(*args, **kwargs):
        return asyncio.get_event_loop().run_until_complete(func(*args, **kwargs))
    return sync_wrapper
```

`get_final_gameweek_fixture_date` being an example coroutine, here used with an `run_sync` decorator:
```
@run_sync
async def get_final_gameweek_fixture_date(gameweek_id):
    """
    Gets the date of the final fixture of a gameweek.
    :param gameweek_id: The id of a gameweek
    :return: the date of the last fixture
    """
    async with aiohttp.ClientSession() as session:
        fpl = FPL(session)
        await fpl.login(email=environ['FPL_EMAIL'], password=environ['FPL_PASSWORD'])
        fixtures = await fpl.get_fixtures_by_gameweek(gameweek=gameweek_id)
        return fixtures[-1].kickoff_time
```

Now functions can be called synchronously:
```
last_match = parse(get_final_gameweek_fixture_date(previous_gameweek.id))
```

### Deployment

To deploy the code, a submodule of the [`python_lambda_template` repository](https://github.com/hajkeats/python_lambda_template) (which I talk about [here](/projects/python_lambda_template/)) was incorporated. Terraform is then used to launch the Lambda. Once the code is deployed in AWS, the Lambda function is triggered by a Cloudwatch Event at 9AM every morning, and depending on whether fixtures are upcoming or have just finished, a set of messages are sent to the group chat.

{{< image src="/images/fpl_bot.png" alt="Another victory" >}}
Another good gameweek for Giroud - Sandstorm!


## Update - February

### Facebook Login issues

A new problem emerged a couple weeks into bot deployment where it failed to login to facebook. Manually logging into the account revealed that the security features of facebook had considered the bot's activity as suspicious and had locked the account. A password reset was required.

In order to prevent this issue occuring in future, a [dynamodb](https://aws.amazon.com/dynamodb/) table was added through the terraform, which stores the most recent set of session cookies. Rather than logging in every time, the lambda attempts to use the session cookies instead to get the fbchat client.

## Update - March

### RIP fpl_bot

_Jan 2021 - Mar 2021_

FBChat no longer successfully logs into Facebook, and as such, the bot has been retired.