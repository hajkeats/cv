---
title: "Fantasy Premier League Bot"
date: 2021-01-05T18:03:57Z
---

### [Repository](https://github.com/hajkeats/fpl_bot)


## A crude notification bot

This Christmas break I got bored. With coronavirus rife across the UK, and tier 4 restrictions in place, I had little to do but sit about watching football on TV, occasionally fiddling with my [Fantasy Premier League](https://fantasy.premierleague.com/) team, something that I'm pretty good at. In fact... I've won my fantasy league each year for as long as I remember. Currently I sit at 79,498th out of 7,791,110 places, and top of both the classic and head-to-head style leagues that 5 of my friends and I participate in. That said, I'm [not quite Magnus Carlsen](https://www.theguardian.com/sport/2019/dec/16/chess-champion-magnus-carlsen-top-of-world-fantasy-football-rankings-premier-league), and so have decided to leverage the FPL API to help elevate my skills to GrandMaster level.

{{< image src="/images/magnus-carlsen.png" alt="Magnus Carlsen Twitter" >}}

One issue that I face is easily solved. Although I usually do better than my friends at staying interested throughout the season, I still occasionally forget to change my team in time for a new gameweek. This can lead to disastrous consequences, and is very easy to do with the midweek fixtures around the middle of the season. In order to remind myself and the other members of the fantasy league, I decided to build a facebook messenger bot for our group chat, with 2 functions:

1. Remind the chat that a gameweek is about to start.
1. Update the chat on results of head-to-head games.

### Challenges

Unfortunately, the platform used for our FPL chat group is Facebook messenger, which has [recently demoted chatbots](https://techcrunch.com/2020/02/28/messenger-removes-discover/) in favour of a simpler and more streamlined app. Chatbots are now mainly considered as marketing tools tied to pages. Creating such a bot is fairly simple, and would be possible with a combination of AWS [API Gateway](https://aws.amazon.com/api-gateway/) and [Lambda](https://aws.amazon.com/lambda/), but it is not possible to add a page to a group chat.

Instead, I used Lambda and the [`fbchat`](https://github.com/fbchat-dev/fbchat) Python library.

_"fbchat works by emulating the browser. This means doing the exact same GET/POST requests and tricking Facebook into thinking it’s accessing the website normally. Therefore, this API requires the credentials of a Facebook account."_

The `fbchat` repository is currently unmaintained, hence the hacky workaround required [in the code](https://github.com/hajkeats/fpl_bot/blob/master/src/h2h_bot.py#L16-L18). 

The other main challenge came in accessing the FPL API. All the endpoints seemed to work without login initially, then it became apparent that the head-to-head league endpoint only allowed the user to access the first 50 records. When this was noticed I sought to incorporate the [fpl](https://fpl.readthedocs.io/en/latest/) wrapper. Being an asynchronous implementation, it didn't work naturally with lambda which does not support asynchronous handlers. To get around this issue, the following code was used to call an asynchronous function from a synchronous one:

```python
asyncio.get_event_loop().run_until_complete(get_gameweek_fixtures(event_id))
```

`get_gameweek_features` being the async function:
```python
async def get_gameweek_fixtures(event_id):
    """
    Returns the fixtures for a given league and gameweek
    :param event_id: the identifier for the gameweek
    :return: fixtures for the gameweek
    """
    async with aiohttp.ClientSession() as session:
        fpl = FPL(session)
        await fpl.login(email=environ['FPL_EMAIL'], password=environ['FPL_PASSWORD'])
        h2h_league = await fpl.get_h2h_league(environ['LEAGUE_ID'])
        fixtures = await h2h_league.get_fixtures(gameweek=event_id)
    return fixtures
```

To deploy the code, a submodule of the [`python_lambda_template` repository](https://github.com/hajkeats/python_lambda_template) (which I talk about [here](/projects/python_lambda_template/)) was incorporated. Terraform is then used to launch the Lambda. Once the code is deployed in AWS, the Lambda function is triggered by a Cloudwatch Event at 9AM every morning, and depending on whether fixtures are upcoming or have just finished, a set of messages are sent to the group chat.

{{< image src="/images/fpl_bot.png" alt="Another victory" >}}
Another good gameweek for Giroud - Sandstorm!