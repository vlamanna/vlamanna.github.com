---
layout: post
title: "Deriving a status"
description: "Efficiently derive statuses from a log table in MySQL."
category: "MySQL"
tags: ["MySQL", "Join", "Logs"]
---
{% include JB/setup %}

We've been working hard lately at CakeMail on building the new version of the application. While building
this new version, we have been rebuilding the API from scratch, reviewing every queries. A set of these
queries were to derive the status of a subscriber based on the logs of a campaign.


### Table Structure

Here is the table structure we will be using for this post.

#### Logs
<table class="table table-bordered">
	<tr>
		<td><strong>id</strong></td>
		<td><strong>subscriber_id</strong></td>
		<td><strong>timestamp</strong></td>
		<td><strong>action</strong></td>
	</tr>
	<tr>
		<td>1</td>
		<td>1</td>
		<td>2013-10-01 08:00:00</td>
		<td>sent</td>
	</tr>
	<tr>
		<td>2</td>
		<td>2</td>
		<td>2013-10-01 08:00:00</td>
		<td>sent</td>
	</tr>
	<tr>
		<td>3</td>
		<td>3</td>
		<td>2013-10-01 08:00:00</td>
		<td>sent</td>
	</tr>
	<tr>
		<td>4</td>
		<td>4</td>
		<td>2013-10-01 08:00:00</td>
		<td>sent</td>
	</tr>
	<tr>
		<td>5</td>
		<td>1</td>
		<td>2013-10-01 08:15:00</td>
		<td>open</td>
	</tr>
	<tr>
		<td>6</td>
		<td>1</td>
		<td>2013-10-01 08:16:00</td>
		<td>click</td>
	</tr>
	<tr>
		<td>7</td>
		<td>2</td>
		<td>2013-10-01 08:30:00</td>
		<td>open</td>
	</tr>
	<tr>
		<td>8</td>
		<td>3</td>
		<td>2013-10-01 08:45:00</td>
		<td>click</td>
	</tr>
</table>


### Getting the list of subscribers that were sent the campaign

Deriving this status is pretty simple. All we have to do is to select all the logs that have the
action "sent". This would be the query we would use.

    SELECT `subscriber_id`, 'sent'
    FROM `logs`
    WHERE `action` = 'sent'

And to experience the best performance, we would have an index on the field "action".


### Getting the list of subscribers who have clicked on a link in the campaign

Again, deriving this status is pretty simple as it would be the same as the above query but we would
filter the action "click". This would be the query we would use.

    SELECT `subscriber_id`, 'clicked'
    FROM `logs`
    WHERE `action` = 'click'
    GROUP BY `subscriber_id`

Note that in this last query, we added a group by. This is to ensure that if a subscriber clicked
on more than one link in the campaign, they won't appear twice in the list of subscribers who have
clicked on a link.


### Getting the list of subscribers who have opened the campaign

Even if this one seems simple, it's a bit more complicated because of the way we detect that someone
has opened the campaign. Basically, the way we detect that someone has opened the campaign is by adding
a web beacon at the end of every email we send. This web beacon is actually a 1x1 pixel image.

Therefore, if someone doesn't render the images of the campaign we won't detect they have opened the
campaign.

Oh wait! What if they clicked on one of the link in the campaign. In such cases we wouldn't have a log
with the action "open" but we would have one with the action "click".

Seems pretty simple right, all we have to do is to filter every action "open" or "click". The
query would look like this.

    SELECT `subscriber_id`, 'opened'
    FROM `logs`
    WHERE `action` = 'click' OR `action` = 'open'
    GROUP BY `subscriber_id`

There is one problem with this query. The "OR" statement is quite slow in MySQL as it cannot use
the index as efficiently as an "AND" statement. The solution to this would be to actually use
joins. Here would be a more optimal query.

    SELECT COALESCE(o.`subscriber_id`, c.`subscriber_id`) AS `subscriber_id`, 'opened'
    FROM `logs` o
    FULL OUTER JOIN `logs` c ON (o.`subscriber_id` = c.`subscriber_id` AND c.`action` = 'click')
    WHERE o.`action` = 'open'
    GROUP BY `subscriber_id`


### Getting the list of subscribers who haven't opened the campaign

This one is the most complicated as we need to check for every subscriber to whom we've sent the campaign
but who haven't opened the campaign nor clicked on any link in the campaign. The worst solution
I've seen for this problem is the following.

    SELECT `subscriber_id`, 'unopened'
    FROM `logs`
    WHERE `action` = 'sent'
    AND `subscriber_id` NOT IN (
        SELECT `subscriber_id`
        FROM `logs`
        WHERE `action` = 'open' OR `action` = 'click'
    )

On large campaigns, this solution actually performs very badly because of the NOT IN statement.
Here is the solution I've found to this problem that is performing much better using joins.

    SELECT s.`subscriber_id`, 'unopened'
    FROM `logs` s
    LEFT OUTER JOIN `logs` o
        ON (s.`subscriber_id` = o.`subscriber_id` AND o.`action` = 'open')
    LEFT OUTER JOIN `logs` c
        ON (s.`subscriber_id` = c.`subscriber_id` AND c.`action` = 'click')
    WHERE `action` = 'sent'
    AND o.`id` IS NULL AND c.`id` IS NULL
    GROUP BY s.`subscriber_id`


It had been a while since the last time I posted on my blog but when we came across this problem,
I thought it would be good to do a post on this to share the solutions we found. Hopefully this
will be useful to you.
