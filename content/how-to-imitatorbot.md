Title: Building ImitatorBot
Date: 2017-09-20 18:46
Category: ImitatorBot

# What is ImitatorBot?
ImitatorBot is a [SlackBot]() that generates nonsense messages that are written in the same style as the co-worker of your choice. It does this by creating n-gram models for each of your co-workers from their respective message histories. [What are n-grams?](http://text-analytics101.rxnlp.com/2014/11/what-are-n-grams.html)

# Usage
ImitatorBot is a [bot user](https://api.slack.com/bot-users) that can be added to channels in your company's slack.

To request an imitation you either DM ImitatorBot or mention it in a channel or group chat. If the message includes `imitate` as well as a mention of another user then your imitation should be on its way!

For example: `@imitatorBot` imitate `@nigel`

# Architecture Overview
On the surface ImitatorBot is quite simple. It needs to...

1. build n-gram models for the users in your company
2. parse a request to generate a message for a particular user
3. generate the message and respond in the same channel or chat

Lets unpack these requirements and see just how deep the rabbit hole goes...

## Scraping Message History
A message history presents some unique challenges as a corpus. As it is being updated regularly, we need a strategy to keep our n-gram models up to date. One approach might be to run a job on some interval that scrapes the complete history of every channel that the bot is a member of. For small companies without too much chatter this could be viable, but once you have a couple hundred employees you will very quickly start going over [Slack's rate limits](https://api.slack.com/docs/rate-limits). Also, as tempting as it is to train the model on the entire data set, you can get satisfying results from a relatively small sample.

To accommodate these constraints I implemented an "on-demand" strategy. Whenever a request is made for an imitation, before responding ImitatorBot scrapes the history only from the channel or group from which the request was made and scrapes only the number of messages as set by an environment variable. The first time a channel is scraped, it is from the present into the past. On subsequent requests on the same channel, the job runs from the latest scraped to the present and then from the earliest scraped to the past. There is also an environment variable that sets the number of messages to be scraped after the response in the background.

_See my post on the [implementation of this strategy]() (Coming soon)._

This approach enables the manager of the bot to limit the number of requests that are made to the slack API. The number of requests that are made are also dependent on the popularity of the bot. The more it is used, the stronger the models get. If it is not used at all, then there are no requests to the API.

## Training N-gram models
The n-gram models are conceptually simple. ImitatorBot uses 2-grams so a model for a particular user can be thought of as a list of every pair of words they ever typed in succession with a probability indicating the likelihood that a


Later articles:

Implementing and Persisting weighted lists
Implementing the On-demand scraping
