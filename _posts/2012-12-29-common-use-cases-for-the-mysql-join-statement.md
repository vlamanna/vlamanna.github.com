---
layout: post
title: "Common use cases for the MySQL Join statement"
description: "Demystifying the MySQL Join statement for building better datasets."
category: "MySQL"
tags: ["MySQL", "Join"]
---
{% include JB/setup %}

Since I started working as a software engineer, I've seen a lot of people misusing or
trying to avoid MySQL Join statement. These statements can actually be quite handy when
used properly.

In this post, I will demonstrate how to effectively use the Join statement with some
common use cases. The scenario will be the following:

> *You have a list of users and track their activities. You want to build an internal reporting tool.*


### Table Structure

Here is the table structure we will be using for these common use cases.

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
	<tr>
		<td>8</td>
		<td>3</td>
		<td>2012-12-05 19:45:00</td>
		<td>login</td>
	</tr>
</table>


### Use Case #1: Getting the list of users with their signup date

For this use case, all we have to do is select everything from the users table and use a
left join with the activities table. Using a left join will ensure that if some users
don't have a signup activity they will still show up in the list with a null value for
the signup date.

The most important thing when doing a join is to ensure that we have indexes on the column
we use in the join. In this scenario, we have a primary index on users.id and an index
on activity.user_id.

Here's the query:

	SELECT u.`id`, u.`username`, a.`timestamp` as `signup_date`
	FROM `users` u
	LEFT JOIN `activities` a
		ON u.`id` = a.`user_id`
		AND a.`activity` = 'signup'

Here's the results set:
<table class="table table-bordered">
	<tr>
		<td><strong>id</strong></td>
		<td><strong>username</strong></td>
		<td><strong>signup_date</strong></td>
	</tr>
	<tr>
		<td>1</td>
		<td>vincent.lamanna</td>
		<td>2012-12-01 08:00:00</td>
	</tr>
	<tr>
		<td>2</td>
		<td>john.doe</td>
		<td>2012-12-03 12:54:00</td>
	</tr>
	<tr>
		<td>3</td>
		<td>jane.doe</td>
		<td>2012-12-05 15:23:00</td>
	</tr>
</table>

Here's the explain:
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
		<td>SIMPLE</td>
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
		<td>a</td>
		<td>ref</td>
		<td>user_id</td>
		<td>user_id</td>
		<td>4</td>
		<td>application.u.id</td>
		<td>1</td>
	</tr>
</table>

As you can see, our join used the index we defined. Having properly set indexes ensure
optimal performance.
		

### Use Case #2: Getting the list of users with their signup and confirmation date

This use case is very similar to our first use case. All we will have to add is a second
left join to the same activities table but filtering the activity to be confirm\_signup.

Here's the query:

	SELECT u.`id`, u.`username`, a1.`timestamp` as `signup_date`, a2.`timestamp` as `confirm_date`
	FROM `users` u
	LEFT JOIN `activities` a1
		ON u.`id` = a1.`user_id`
		AND a1.`activity` = 'signup'
	LEFT JOIN `activities` a2
		ON u.`id` = a2.`user_id`
		AND a2.`activity` = 'confirm_signup'
		
Here's the results set:
<table class="table table-bordered">
	<tr>
		<td><strong>id</strong></td>
		<td><strong>username</strong></td>
		<td><strong>signup_date</strong></td>
		<td><strong>confirm_date</strong></td>
	</tr>
	<tr>
		<td>1</td>
		<td>vincent.lamanna</td>
		<td>2012-12-01 08:00:00</td>
		<td>2012-12-01 08:02:00</td>
	</tr>
	<tr>
		<td>2</td>
		<td>john.doe</td>
		<td>2012-12-03 12:54:00</td>
		<td>NULL</td>
	</tr>
	<tr>
		<td>3</td>
		<td>jane.doe</td>
		<td>2012-12-05 15:23:00</td>
		<td>2012-12-05 15:24:00</td>
	</tr>
</table>

Here's the explain:
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
		<td>SIMPLE</td>
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
		<td>a1</td>
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
		<td>a2</td>
		<td>ref</td>
		<td>user_id</td>
		<td>user_id</td>
		<td>4</td>
		<td>application.u.id</td>
		<td>1</td>
	</tr>
</table>

This use case shows us how useful the left join can be. We can easily generate a report
showing us the users who signed up but who haven't confirmed their sign up.


### Use Case #3: Getting the list of users with the timestamp and type of their last activity

This use case is a bit trickier. It's also referenced as the greatest-n-per-group
problem. In our scenario, it can be solved by joining the activities table twice. The
first join is a normal left join but the second join is a left outer join. This will have
the effect of discarding all the rows except the latest one for each user.

Here's the query:

	SELECT u.`id`, u.`username`, a1.`timestamp`, a1.`activity`
	FROM `users` u
	LEFT JOIN `activities` a1 ON u.`id` = a1.`user_id`
	LEFT OUTER JOIN `activities` a2 ON (u.`id` = a2.`user_id` AND
		(a1.`timestamp` < a2.`timestamp` OR a1.`timestamp` = a2.`timestamp` AND a1.`id` < a2.`id`))
	WHERE a2.`id` IS NULL

Here's the results set:
<table class="table table-bordered">
	<tr>
		<td><strong>id</strong></td>
		<td><strong>username</strong></td>
		<td><strong>timestamp</strong></td>
		<td><strong>activity</strong></td>
	</tr>
	<tr>
		<td>1</td>
		<td>vincent.lamanna</td>
		<td>2012-12-05 18:04:00</td>
		<td>login</td>
	</tr>
	<tr>
		<td>2</td>
		<td>john.doe</td>
		<td>2012-12-03 12:54:00</td>
		<td>signup</td>
	</tr>
	<tr>
		<td>3</td>
		<td>jane.doe</td>
		<td>2012-12-05 19:45:00</td>
		<td>login</td>
	</tr>
</table>

Here's the explain:
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
		<td>SIMPLE</td>
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
		<td>a1</td>
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
		<td>a2</td>
		<td>ref</td>
		<td>PRIMARY,user_id,timestamp</td>
		<td>user_id</td>
		<td>4</td>
		<td>application.u.id</td>
		<td>1</td>
	</tr>
</table>

This double join can be very handy when generating reports. An interesting thing to note
as well, is that the index that is being used on the second join is still the user\_id.


### Use Case #4: Getting the list of users with their total number of activities

This last use case is very simple and can be achieved using only one left join. You will
also need to use the count aggregator as well as a group by to have a list of users.

Here's the query:

	SELECT u.`id`, u.`username`, COUNT(a.`id`) as `num_activities`
	FROM `users` u
	LEFT JOIN `activities` a
		ON u.`id` = a.`user_id`
	GROUP BY u.`id`

Here's the results set:
<table class="table table-bordered">
	<tr>
		<td><strong>id</strong></td>
		<td><strong>username</strong></td>
		<td><strong>num_activities</strong></td>
	</tr>
	<tr>
		<td>1</td>
		<td>vincent.lamanna</td>
		<td>4</td>
	</tr>
	<tr>
		<td>2</td>
		<td>john.doe</td>
		<td>1</td>
	</tr>
	<tr>
		<td>3</td>
		<td>jane.doe</td>
		<td>3</td>
	</tr>
</table>

Here's the explain:
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
		<td>SIMPLE</td>
		<td>u</td>
		<td>index</td>
		<td>NULL</td>
		<td>PRIMARY</td>
		<td>NULL</td>
		<td>NULL</td>
		<td>3</td>
	</tr>
	<tr>
		<td>1</td>
		<td>SIMPLE</td>
		<td>a</td>
		<td>ref</td>
		<td>user_id</td>
		<td>user_id</td>
		<td>4</td>
		<td>application.u.id</td>
		<td>1</td>
	</tr>
</table>

As you can see in this last use case, the explain is a bit different from the other use
cases for the users table. It's actually because of the group by which causes MySQL to
use the primary key.


### Creating a report with all of the above use cases

Now is the time to sum everything up into one report.

Here's the query:

	SELECT u.`id`, u.`username`,
		a1.`timestamp` as `signup_date`, a2.`timestamp` as `confirm_date`,
		a3.`timestamp` as `last_date`, a3.`activity` as `last_activity`,
		COUNT(a5.`id`) as `num_activities`
	FROM `users` u
	LEFT JOIN `activities` a1
		ON u.`id` = a1.`user_id`
		AND a1.`activity` = 'signup'
	LEFT JOIN `activities` a2
		ON u.`id` = a2.`user_id`
		AND a2.`activity` = 'confirm_signup'
	LEFT JOIN `activities` a3 ON u.`id` = a3.`user_id`
	LEFT OUTER JOIN `activities` a4 ON (u.`id` = a4.`user_id` AND
		(a3.`timestamp` < a4.`timestamp` OR a3.`timestamp` = a4.`timestamp` AND a3.`id` < a4.`id`))
	LEFT JOIN `activities` a5
		ON u.`id` = a5.`user_id`
	WHERE a4.`id` IS NULL
	GROUP BY u.`id`
	
Here's the results set:
<table class="table table-bordered">
	<tr>
		<td><strong>id</strong></td>
		<td><strong>username</strong></td>
		<td><strong>signup_date</strong></td>
		<td><strong>confirm_date</strong></td>
		<td><strong>last_date</strong></td>
		<td><strong>last_activity</strong></td>
		<td><strong>num_activities</strong></td>
	</tr>
	<tr>
		<td>1</td>
		<td>vincent.lamanna</td>
		<td>2012-12-01 08:00:00</td>
		<td>2012-12-01 08:02:00</td>
		<td>2012-12-05 18:04:00</td>
		<td>login</td>
		<td>4</td>
	</tr>
	<tr>
		<td>2</td>
		<td>john.doe</td>
		<td>2012-12-03 12:54:00</td>
		<td>NULL</td>
		<td>2012-12-03 12:54:00</td>
		<td>signup</td>
		<td>1</td>
	</tr>
	<tr>
		<td>3</td>
		<td>jane.doe</td>
		<td>2012-12-05 15:23:00</td>
		<td>2012-12-05 15:24:00</td>
		<td>2012-12-05 19:45:00</td>
		<td>login</td>
		<td>3</td>
	</tr>
</table>


To conclude this post, don't be afraid of using joins in MySQL but use them wisely and
make sure your indexes are properly set by running explains on your queries.