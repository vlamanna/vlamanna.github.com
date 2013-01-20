---
layout: post
title: "Real time data segmentation"
description: "Achieve better performance when doing data segmentation in MySQL."
category: "MySQL"
tags: ["MySQL", "Join", "Segmentation"]
---
{% include JB/setup %}

Recently, I had to work on improving real-time data segmentation. We had a solution in
place but the performance were not so great. In some cases, getting a response back was
taking a couple of minutes. In this post, I will share with how I managed to improve this.

Here is the scenario we will use to explore this problem.

> *You have a list of users and track their activities. You want to segment your list of users based on their activities dynamically.*


### Table Structure

Here is the table structure we will be using for this post.

#### Users
<table class="table table-bordered">
	<tr>
		<td><strong>id</strong></td>
		<td><strong>username</strong></td>
		<td><strong>firstname</strong></td>
		<td><strong>lastname</strong></td>
	</tr>
	<tr>
		<td>1</td>
		<td>vincent.lamanna</td>
		<td>Vincent</td>
		<td>Lamanna</td>
	</tr>
	<tr>
		<td>2</td>
		<td>john.doe</td>
		<td>John</td>
		<td>Doe</td>
	</tr>
	<tr>
		<td>3</td>
		<td>jane.doe</td>
		<td>Jane</td>
		<td>Doe</td>
	</tr>
</table>

#### Activities
<table class="table table-bordered">
	<tr>
		<td><strong>id</strong></td>
		<td><strong>user_id</strong></td>
		<td><strong>timestamp</strong></td>
		<td><strong>activity</strong></td>
	</tr>
	<tr>
		<td>1</td>
		<td>1</td>
		<td>2012-12-01 08:00:00</td>
		<td>signup</td>
	</tr>
	<tr>
		<td>2</td>
		<td>1</td>
		<td>2012-12-01 08:02:00</td>
		<td>confirm_signup</td>
	</tr>
	<tr>
		<td>3</td>
		<td>1</td>
		<td>2012-12-01 08:03:00</td>
		<td>login</td>
	</tr>
	<tr>
		<td>4</td>
		<td>2</td>
		<td>2012-12-03 12:54:00</td>
		<td>signup</td>
	</tr>
	<tr>
		<td>5</td>
		<td>3</td>
		<td>2012-12-05 15:23:00</td>
		<td>signup</td>
	</tr>
	<tr>
		<td>6</td>
		<td>3</td>
		<td>2012-12-05 15:24:00</td>
		<td>confirm_signup</td>
	</tr>
	<tr>
		<td>7</td>
		<td>1</td>
		<td>2012-12-05 18:04:00</td>
		<td>login</td>
	</tr>
</table>


### A Common Naive Approach

Let's say we want to segment our list of users to get a list of users who have signed up
but haven't login or confirm their signup yet. A common approach I've seen people using
would be the following query.

	SELECT `id`, `username`
	FROM `users`
	WHERE `id` IN (
		SELECT DISTINCT(`user_id`)
		FROM `activities`
		WHERE `activity` = 'signup'
	) AND (
		`id` NOT IN (
			SELECT DISTINCT(`user_id`)
			FROM `activities`
			WHERE `activity` = 'login'
		)
		OR `id` NOT IN (
			SELECT DISTINCT(`user_id`)
			FROM `activities`
			WHERE `activity` = 'confirm_signup'
		)
	)

The issue with this approach is that it uses sub-queries and causes MySQL to performance
a full table scan on the main table you are segmenting. Here is the MySQL Explain you
would get running the above query.

<table class="table table-bordered">
	<tr>
		<td><strong>id</strong></td>
		<td><strong>select_type</strong></td>
		<td><strong>table</strong></td>
		<td><strong>type</strong></td>
		<td><strong>possible_keys</strong></td>
		<td><strong>key</strong></td>
		<td><strong>key_len</strong></td>
		<td><strong>ref</strong></td>
		<td><strong>rows</strong></td>
	</tr>
	<tr>
		<td>1</td>
		<td>PRIMARY</td>
		<td>users</td>
		<td>ALL</td>
		<td>NULL</td>
		<td>NULL</td>
		<td>NULL</td>
		<td>NULL</td>
		<td>3</td>
	</tr>
	<tr>
		<td>4</td>
		<td>DEPENDENT SUBQUERY</td>
		<td>activities</td>
		<td>index_subquery</td>
		<td>user_id</td>
		<td>user_id</td>
		<td>4</td>
		<td>func</td>
		<td>1</td>
	</tr>
	<tr>
		<td>3</td>
		<td>DEPENDENT SUBQUERY</td>
		<td>activities</td>
		<td>index_subquery</td>
		<td>user_id</td>
		<td>user_id</td>
		<td>4</td>
		<td>func</td>
		<td>1</td>
	</tr>
	<tr>
		<td>2</td>
		<td>DEPENDENT SUBQUERY</td>
		<td>activities</td>
		<td>index_subquery</td>
		<td>user_id</td>
		<td>user_id</td>
		<td>4</td>
		<td>func</td>
		<td>1</td>
	</tr>
</table>

When working with a small to medium dataset, this is usually not a problem if you have a
decent server. However, it can become a real bottleneck when you have a large dataset.
Simple segments could take minutes to return a result. It can really become a problem
when you want to achieve real-time data segmentation.


### A Better Approach

After some iterations, here is the approach I found was best to achieve good performance
on solving this problem.

	SELECT u.`id`, u.`username`
	FROM `users` u
	LEFT JOIN `activities` q1_1 ON u.`id` = q1_1.`user_id` AND q1_1.`activity` = 'signup'
	LEFT OUTER JOIN `activities` q1_2 ON (u.`id` = q1_2.`user_id` AND q1_2.`activity` = 'signup' AND
		(q1_1.`id` < q1_2.`id` OR q1_1.`id` = q1_2.`id` AND q1_1.`id` < q1_2.`id`))
	LEFT JOIN `activities` q2_1 ON u.`id` = q2_1.`user_id` AND q2_1.`activity` = 'confirm_signup'
	LEFT OUTER JOIN `activities` q2_2 ON (u.`id` = q2_2.`user_id` AND q2_2.`activity` = 'confirm_signup' AND
		(q2_1.`id` < q2_2.`id` OR q2_1.`id` = q2_2.`id` AND q2_1.`id` < q2_2.`id`))
	LEFT JOIN `activities` q3_1 ON u.`id` = q3_1.`user_id` AND q3_1.`activity` = 'login'
	LEFT OUTER JOIN `activities` q3_2 ON (u.`id` = q3_2.`user_id` AND q3_2.`activity` = 'login' AND
		(q3_1.`id` < q3_2.`id` OR q3_1.`id` = q3_2.`id` AND q3_1.`id` < q3_2.`id`))
	WHERE q1_2.`id` IS NULL AND q2_2.`id` IS NULL AND q3_2.`id` IS NULL
	AND (q1_1.`id` IS NOT NULL AND (q2_1.`id` IS NULL OR q3_1.`id` IS NULL))
	
Looking at the MySQL Explain, we can see this is a much better approach.

<table class="table table-bordered">
	<tr>
		<td><strong>id</strong></td>
		<td><strong>select_type</strong></td>
		<td><strong>table</strong></td>
		<td><strong>type</strong></td>
		<td><strong>possible_keys</strong></td>
		<td><strong>key</strong></td>
		<td><strong>key_len</strong></td>
		<td><strong>ref</strong></td>
		<td><strong>rows</strong></td>
	</tr>
	<tr>
		<td>1</td>
		<td>PRIMARY</td>
		<td>u</td>
		<td>ALL</td>
		<td>NULL</td>
		<td>NULL</td>
		<td>NULL</td>
		<td>NULL</td>
		<td>3</td>
	</tr>
	<tr>
		<td>1</td>
		<td>SIMPLE</td>
		<td>q1_1</td>
		<td>ref</td>
		<td>user_id</td>
		<td>user_id</td>
		<td>4</td>
		<td>application.u.id</td>
		<td>1</td>
	</tr>
	<tr>
		<td>1</td>
		<td>SIMPLE</td>
		<td>q1_2</td>
		<td>ref</td>
		<td>user_id</td>
		<td>user_id</td>
		<td>4</td>
		<td>application.q1_1.user_id</td>
		<td>1</td>
	</tr>
	<tr>
		<td>1</td>
		<td>SIMPLE</td>
		<td>q2_1</td>
		<td>ref</td>
		<td>user_id</td>
		<td>user_id</td>
		<td>4</td>
		<td>application.q1_1.user_id</td>
		<td>1</td>
	</tr>
	<tr>
		<td>1</td>
		<td>SIMPLE</td>
		<td>q2_2</td>
		<td>ref</td>
		<td>user_id</td>
		<td>user_id</td>
		<td>4</td>
		<td>application.q1_1.user_id</td>
		<td>1</td>
	</tr>
	<tr>
		<td>1</td>
		<td>SIMPLE</td>
		<td>q3_1</td>
		<td>ref</td>
		<td>user_id</td>
		<td>user_id</td>
		<td>4</td>
		<td>application.q1_1.user_id</td>
		<td>1</td>
	</tr>
	<tr>
		<td>1</td>
		<td>SIMPLE</td>
		<td>q3_2</td>
		<td>ref</td>
		<td>user_id</td>
		<td>user_id</td>
		<td>4</td>
		<td>application.q1_1.user_id</td>
		<td>1</td>
	</tr>
</table>

Here is how it works. First, we need to build the dataset we will be using to segment.
For this part, we use double JOIN statements along with a WHERE statement. This is to
ensure that if a record as more than 1 activity of the same time it is not duplicating
the user in the final table.

	LEFT JOIN `activities` q1_1 ON u.`id` = q1_1.`user_id` AND q1_1.`activity` = 'signup'
	LEFT OUTER JOIN `activities` q1_2 ON (u.`id` = q1_2.`user_id` AND q1_2.`activity` = 'signup' AND
		(q1_1.`id` < q1_2.`id` OR q1_1.`id` = q1_2.`id` AND q1_1.`id` < q1_2.`id`))
	...
	WHERE q1_2.`id` IS NULL

Once we have the dataset, we can then apply the segmentation rule using a where statement.

	WHERE (q1_1.`id` IS NOT NULL AND (q2_1.`id` IS NULL OR q3_1.`id` IS NULL))
		
The reason why we do this instead of simple INNER JOIN statement or LEFT OUTER JOIN
statement is to allow for complex segmentation rule such as the following.

	(Criteria 1 OR Criteria 2) AND (Criteria3 OR Criteria 4) OR Criteria 5
	
	
To conclude this post, MySQL is great for doing real-time data segmentation. We just need
to make sure we do it the proper way.