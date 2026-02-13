---
layout: post
title: "Extreme Carpaccio"
date: 2016-01-07 00:00:00 +0000
---

At the end of 2014, I started working for a well known French investment bank as Software Craftsmanship evangelist. About 2 or 3 years before, an ambitious program started being deployed across the organisation in a huge movement towards Continuous Delivery. I joined the agile and technical coaches' team in charge of the program as Craftsman Coach (as they use to call the role themselves). My job was to help development teams to improve software quality, stability and the delivery process, by introducing concepts like TDD, clean code, continuous refactoring, BDD, continuous integration, simple design, and so on.

On my day-to-day work, I strongly used coding exercises, better know as code katas, in order to help developers to improve their skills. Most of the exercises I used can be easily found on the internet. Nevertheless, sometimes I was led to create some exercises by my own to better address specific team needs. On that context, while trying to figure out how to better explain feature slicing and iterative development, [Radwane Hassen](https://twitter.com/radwane_h) (a colleague also working as tech coach) and I have created Extreme Carpaccio, which was inspired by two great existing exercises: [Extreme Startup](http://chatley.com/posts/05-27-2011/extreme-startup/) and [Elephant Carpaccio](http://alistair.cockburn.us/Elephant+Carpaccio+exercise).

## Extreme Startup

Extreme Startup is a coding exercise [created by Robert Chatley and Matt Wynne](http://chatley.com/posts/05-27-2011/extreme-startup/). The first time I played Extreme Startup was during a [Xebia Knowledge Exchange (XKE)](http://blog.xebia.fr/2012/07/19/extreme-startup-chez-xebia/) in a session organised by [Sébastian Le Merdy](http://sebastian.lemerdy.name/), [Nicolas Demengel](http://ndemengel.github.io/) and [Jean-Laurent de Morlhon](http://happycleancoding.com/jeanlaurent.html).

During an Extreme Startup session the facilitator uses his own computer to send HTTP requests with puzzles to participants' computers. Participants should figure out how to solve each puzzle and code to respond server's requests. For every good answer, the participant earn points and for wrong answers, he loses.

Extreme Carpaccio uses the same principle: a server simulates a market and sends requests to participants who need to code the response. It turns out this approach is very funny, since it creates a competitive atmosphere and people try their best to win. However, counter to Extreme Startup, in an Extreme Carpaccio session most of the problem is unveiled at the beginning and teams should decide what to implement first.

## Elephant Carpaccio

Elephant Carpaccio is a slicing [exercise created by Alistair Cockburn](http://alistair.cockburn.us/Elephant+Carpaccio+exercise). Many colleagues use this exercise to work with teams and help them to improve their slicing skills.

In an Elephant Carpaccio session, the facilitator exposes the problem and asks participants to slice it into small pieces, prioritize and implement iteratively. At the end of each iteration, teams demonstrate how they devised the problem and what they achieved. The goal is to slice the whole problem into a bunch of small ones, in such a way that each sub-problem could bring feedback and value (either knowledge value, customer value, or both). The opposite approach (waterfall) would require a huge amount of time during the development phase and only allows delivering to production once everything is done. This approach has many drawbacks and one of them is that it postpones feedback.

Unlike Elephant Carpaccio, Extreme Carpaccio doesn't guide participants through slicing, iterations and demonstrations. However, the exercise was designed to promote teams that slice the whole problem through a value perspective. The more you slice, the earlier you go to production, and the earlier you earn points. If you only go live when the whole problem is implemented (waterfall), people that took an approach based on iterations will start earning points before you.

## Problem to be solved

Based on Elephant Carpaccio, the problem consists in a company which sends to each team's computer (sellers) a purchase order. The order contains prices and quantities of each product being sold, the buyer's country and the reduction (discount) to be applied. Using this information, teams need to figure out how to calculate the order's total amount and send back to the server a bill containing the amount. If the answer is correct, the team earns points equivalent to the order's amount. Otherwise, they lose half of the order's amount. For each wrong answer, the server sends a feedback to the client with the right amount value.

Following an example of request the server sends to participants:

```json
{
  "prices": [65.6, 27.26, 32.68],
  "quantities": [6, 8, 10],
  "country": "FR",
  "reduction": "STANDARD"
}
```

The number of purchase orders each country sends varies depending on the country who sent it. This number follows the population density of each country. Germany, UK and France, for instance, will send more requests than Luxembourg, Cyprus or Malta. That information is not mentioned explicitly to encourage people to exchange with the facilitator. The goal here is to favour communication and the facilitator(s) should encourage then teams to exchage with him/her/them. It is also possible to figure that out by looking at the messages sent by the server. Nevertheless, as in real life, asking questions to the right person can save you time.

## Session Plan

Extreme Carpaccio was designed to be played with developers and product owners (or business analysts) together. Nevertheless, it is possible to run the exercice only with developers too.

During an Extreme Carpaccio session:

1. participants get organised into teams of 2 or 3, with 1 product owner and at least 1 developer each (2 is better) – this should take around 5 minutes
2. the facilitator exposes the problem and announces the next steps – takes generally between 5 and 10 minutes
3. the facilitator asks teams to start slicing and prioritizing user stories – between 10 and 20 minutes
4. the facilitator starts the server and asks teams to register themselves via the server's dashboard – can take from 5 to 15 minutes, depending on the number of participants, network issues, etc.
5. once everyone is registered, the facilitator allows people to start coding – takes between half and one hour
6. (optional) if teams are fast and quickly solve the exercise, the facilitator can activate some new behaviour or traps in the servers which might shake the score and add more challenge
7. retrospectives – around 15 minutes

Book 1h30 to 2h for the exercise. During the session, the facilitator should walk among teams in order to make sure everyone is on the run. Most of the problem is exposed at the beginning, but not all of it (see reductions' section). Teams should try to decouple the problem based on their own strategy with the information they have.

At the end, take some minutes to do a retrospective and ask participants what they've learnt and if they have something to share. I generally ask people who finished at the top which slicing strategy they chose. You can also ask about technical difficulties, success, etc. Retrospectives are frequently very rich and I'm frequently surprised to discover new ways to deal with the problem.

## Changing Behaviour And Adding Constraints On The Fly

In the middle of a session, generally things start to stabilize and we will probably have some teams at the top and the others in the middle and down in the ranking. At this moment, in order to challenge teams on top and shake the score, the facilitator can change the order's reduction type without saying anything. Since the reduction rules used to calculate the new amount aren't described anywhere, people need to stop doing what they are doing and try to figure out how the new reduction works (this can be easily achieved by looking at the new purchasing orders and feedback coming from the server). By doing this, we allow people which were in the mid-raking to challenge people on the top. This breaks with the "predictable" aspect of the exercise, that happens when some teams perform much better than the others from the beginning.

Beside the reduction, other behaviour/constraints are available to the facilitator if he/she/them want to add more challenge during the session or shake the scoreboard. It is possible, for instance, to start charging teams with penalties when they go offline. This requires teams to think "zero-downtime" and keep an eye on production (contribution from [Julien Jakubowski](https://twitter.com/jak78)). It is also possible to send corrupted orders, making teams' servers crash. This tends to encourage testing, mainly in boundaries. Finally, the facilitator can use a JavaScript function to overrite the formula used to compute taxes for individual countries, making taxes computation more dynamic (and breakes the dictionnary pattern most teams use to keep countries' taxes) (contributions from [Arnauld Loyer](http://arnauld.github.io/)).

## Interesting Things I've Faced Running Extreme Carpaccio

Something interesting I've noticed running Extreme Carpaccio sessions is how teams slice when someone is playing the role of product owner and when there are only developers in the audience. Generally, developers tend to release once the whole implementation is ready. Product Owners usually gather developers around a slicing strategy and prioritize user stories in order to earn money (points) as soon as possible.

However, an interesting fact that sometimes happens when there are mainly developers in the audience is that they try to hack the server as well as other teams. Personally, I find this great because it pushes people to think out of the box. Generally, participants try to hack their own friends and someone they don't know only when we've established together from the beginning that "everything is possible". Some hacks I've faced were DDoS against the server, man in the middle or someone sending fake requests trying to usurp server's identity.

Extreme Carpaccio sources are available on [Github](https://github.com/dlresende/extreme-carpaccio). If you want to contribute, create an issue on the Github or make a pull request. If you run an Extreme Carpaccio session share your feedback on a blog post or on [Twitter](https://twitter.com/search?q=%22extreme%20carpaccio%22%20OR%20%22Xtreme%20carpaccio%22%20OR%20%23ExtremeCarpaccio&src=typd) and let me know.

*Updated on 11/12/2016: add changing behaviour/constraints on the fly.*
