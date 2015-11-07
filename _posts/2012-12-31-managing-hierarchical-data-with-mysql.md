---
layout: post
title: "Hierarchical data in MySQL"
description: "A simple approach on solving this complex problem of managing hierarchical data using MySQL."
category: "MySQL"
tags: ["MySQL", "Join", "Hierarchy"]
---
{% include JB/setup %}

One of the most complex problem I've seen as a developer is dealing with some kind of
hierarchy in a data model. It's not much about storing the relationship but more it comes
to generating reports that take into account this hierarchy.

In this post, I will expose a simple approach I've found while dealing with these issues.
For the sake of this demonstration we will use the following scenario:

> *You are managing an affiliate program in which you pay users $5 for each user they*
> *refer. On top of that you have a 3-tier commissions on purchases these users make. You*
> *pay 5% of the total amount to direct referrals, 2% on level 2 referrals and 1% on*
> *level 3 referrals.*


### Table Structure

Here is the table structure we will be using for this scenario.

#### Users
<table class="table table-bordered">
	<tr>
		<td><strong>id</strong></td>
		<td><strong>username</strong></td>
		<td><strong>referrer_id</strong></td>
	</tr>
	<tr>
		<td>1</td>
		<td>user.1</td>
		<td>NULL</td>
	</tr>
	<tr>
		<td>2</td>
		<td>user.2</td>
		<td>1</td>
	</tr>
	<tr>
		<td>3</td>
		<td>user.3</td>
		<td>NULL</td>
	</tr>
	<tr>
		<td>4</td>
		<td>user.4</td>
		<td>3</td>
	</tr>
	<tr>
		<td>5</td>
		<td>user.5</td>
		<td>2</td>
	</tr>
	<tr>
		<td>6</td>
		<td>user.6</td>
		<td>2</td>
	</tr>
	<tr>
		<td>7</td>
		<td>user.7</td>
		<td>4</td>
	</tr>
	<tr>
		<td>8</td>
		<td>user.8</td>
		<td>7</td>
	</tr>
	<tr>
		<td>9</td>
		<td>user.9</td>
		<td>6</td>
	</tr>
	<tr>
		<td>10</td>
		<td>user.10</td>
		<td>NULL</td>
	</tr>
</table>

#### Purchases
<table class="table table-bordered">
	<tr>
		<td><strong>id</strong></td>
		<td><strong>user_id</strong></td>
		<td><strong>timestamp</strong></td>
		<td><strong>amount</strong></td>
	</tr>
	<tr>
		<td>1</td>
		<td>1</td>
		<td>2012-12-01 08:00</td>
		<td>14.95</td>
	</tr>
	<tr>
		<td>2</td>
		<td>2</td>
		<td>2012-12-01 08:15</td>
		<td>24.95</td>
	</tr>
	<tr>
		<td>3</td>
		<td>4</td>
		<td>2012-12-01 08:25</td>
		<td>4.95</td>
	</tr>
	<tr>
		<td>4</td>
		<td>4</td>
		<td>2012-12-01 08:40</td>
		<td>9.95</td>
	</tr>
	<tr>
		<td>5</td>
		<td>8</td>
		<td>2012-12-01 08:55</td>
		<td>54.95</td>
	</tr>
	<tr>
		<td>6</td>
		<td>7</td>
		<td>2012-12-01 09:10</td>
		<td>34.95</td>
	</tr>
	<tr>
		<td>7</td>
		<td>10</td>
		<td>2012-12-01 09:20</td>
		<td>29.95</td>
	</tr>
	<tr>
		<td>8</td>
		<td>8</td>
		<td>2012-12-01 09:35</td>
		<td>39.95</td>
	</tr>
	<tr>
		<td>9</td>
		<td>2</td>
		<td>2012-12-01 09:40</td>
		<td>14.95</td>
	</tr>
	<tr>
		<td>10</td>
		<td>7</td>
		<td>2012-12-01 10:15</td>
		<td>19.95</td>
	</tr>
</table>


### The concept of a Lineage

The concept of a lineage is quite simple. You store as text the hierarchical
representation in your Database. For doing this, all we need to do is to add a column
called lineage in our users table. Then we need to fill this column.

The first query we will run will be to populate the lineage for users who do not have a
referrer.

	UPDATE `users`
	SET `lineage` = `id`
	WHERE `referrer_id` IS NULL
	
Then we will run the following query in a for loop until we have 0 affected rows. This
will populate the lineage for the other users.

	UPDATE `users` u1, `users` u2
	SET u1.`lineage` = CONCAT(u2.`lineage`, '-', u1.`id`)
	WHERE u1.`lineage` IS NULL
	AND u2.`id` = u1.`referrer_id`
	AND u2.`lineage` IS NOT NULL
	
The users table will look like the following once the lineage column has been populated.

<table class="table table-bordered">
	<tr>
		<td><strong>id</strong></td>
		<td><strong>username</strong></td>
		<td><strong>referrer_id</strong></td>
		<td><strong>lineage</strong></td>
	</tr>
	<tr>
		<td>1</td>
		<td>user.1</td>
		<td>NULL</td>
		<td>1</td>
	</tr>
	<tr>
		<td>2</td>
		<td>user.2</td>
		<td>1</td>
		<td>1-2</td>
	</tr>
	<tr>
		<td>3</td>
		<td>user.3</td>
		<td>NULL</td>
		<td>3</td>
	</tr>
	<tr>
		<td>4</td>
		<td>user.4</td>
		<td>3</td>
		<td>3-4</td>
	</tr>
	<tr>
		<td>5</td>
		<td>user.5</td>
		<td>2</td>
		<td>1-2-5</td>
	</tr>
	<tr>
		<td>6</td>
		<td>user.6</td>
		<td>2</td>
		<td>1-2-6</td>
	</tr>
	<tr>
		<td>7</td>
		<td>user.7</td>
		<td>4</td>
		<td>3-4-7</td>
	</tr>
	<tr>
		<td>8</td>
		<td>user.8</td>
		<td>7</td>
		<td>3-4-7-8</td>
	</tr>
	<tr>
		<td>9</td>
		<td>user.9</td>
		<td>6</td>
		<td>1-2-6-9</td>
	</tr>
	<tr>
		<td>10</td>
		<td>user.10</td>
		<td>NULL</td>
		<td>10</td>
	</tr>
</table>


### Calculating the sign up commission for a single user

This case is quite simple. It can be done by executing the following query.

	SELECT u.`id`, u.`username`, COUNT(r.`id`) * 5 as `signup_commission`
	FROM `users` u
	LEFT JOIN `users` r ON u.`id` = r.`referrer_id`
	WHERE u.`id` = '1'
	
Basically, we simply join the users table on the referrer\_id and counting how many that
gives for a single user. In this case, it will give the following result.

<table class="table table-bordered">
	<tr>
		<td><strong>id</strong></td>
		<td><strong>username</strong></td>
		<td><strong>signup_commission</strong></td>
	</tr>
	<tr>
		<td>1</td>
		<td>user.1</td>
		<td>5</td>
	</tr>
</table>


### Calculating the sign up commission for all users

This case is quite similar to the previous case. All we have to do is remove the filter
for the specific user and grouping by the user id.

	SELECT u.`id`, u.`username`, COUNT(r.`id`) * 5 as `signup_commission`
	FROM `users` u
	LEFT JOIN `users` r ON u.`id` = r.`referrer_id`
	GROUP BY u.`id`
	
This would generate the following results.

<table class="table table-bordered">
	<tr>
		<td><strong>id</strong></td>
		<td><strong>username</strong></td>
		<td><strong>signup_commission</strong></td>
	</tr>
	<tr>
		<td>1</td>
		<td>user.1</td>
		<td>5</td>
	</tr>
	<tr>
		<td>2</td>
		<td>user.2</td>
		<td>10</td>
	</tr>
	<tr>
		<td>3</td>
		<td>user.3</td>
		<td>5</td>
	</tr>
	<tr>
		<td>4</td>
		<td>user.4</td>
		<td>5</td>
	</tr>
	<tr>
		<td>5</td>
		<td>user.5</td>
		<td>0</td>
	</tr>
	<tr>
		<td>6</td>
		<td>user.6</td>
		<td>5</td>
	</tr>
	<tr>
		<td>7</td>
		<td>user.7</td>
		<td>5</td>
	</tr>
	<tr>
		<td>8</td>
		<td>user.8</td>
		<td>0</td>
	</tr>
	<tr>
		<td>9</td>
		<td>user.9</td>
		<td>0</td>
	</tr>
	<tr>
		<td>10</td>
		<td>user.10</td>
		<td>0</td>
	</tr>
</table>

Alternatively, you can calculate the commission of all users of a same level with this
query.

	SELECT u.`id`, u.`username`, COUNT(r.`id`) * 5 as `signup_commission`, length(u.`lineage`)-length(replace(u.`lineage`,"-","")) as `level`
	FROM `users` u
	LEFT JOIN `users` r ON u.`id` = r.`referrer_id`
	GROUP BY u.`id`
	HAVING `level` = 0
	

### Calculating the purchases commission for a single user

This is where it starts to get complex and where the lineage becomes very useful.
Basically, we will join the users table for the referrals but this time we will join using
the lineage. It would give the following query.

	SELECT u.`id`, u.`username`, 
	COALESCE(SUM(CASE
		WHEN length(replace(r.`lineage`, u.`lineage`, ""))-length(replace(replace(r.`lineage`, u.`lineage`, ""),"-","")) = 1 THEN .05 * p.`amount`
		WHEN length(replace(r.`lineage`, u.`lineage`, ""))-length(replace(replace(r.`lineage`, u.`lineage`, ""),"-","")) = 2 THEN .02 * p.`amount`
		WHEN length(replace(r.`lineage`, u.`lineage`, ""))-length(replace(replace(r.`lineage`, u.`lineage`, ""),"-","")) = 3 THEN .01 * p.`amount`
		ELSE 0
	END), 0) as `purchases_commission`
	FROM `users` u
	LEFT JOIN `users` r ON r.`lineage` LIKE CONCAT(u.`lineage`, "-%")
	LEFT JOIN `purchases` p ON p.`user_id` = r.`id`
	WHERE u.`id` = '1'
	
In this case, it will give the following result.

<table class="table table-bordered">
	<tr>
		<td><strong>id</strong></td>
		<td><strong>username</strong></td>
		<td><strong>signup_commission</strong></td>
	</tr>
	<tr>
		<td>1</td>
		<td>user.1</td>
		<td>1.9950</td>
	</tr>
</table>


### Calculating the purchases commission for all users

This one is similar to the previous one except that we replace the filter for the single
user by grouping the users.

	SELECT u.`id`, u.`username`, 
	COALESCE(SUM(CASE
		WHEN length(replace(r.`lineage`, u.`lineage`, ""))-length(replace(replace(r.`lineage`, u.`lineage`, ""),"-","")) = 1 THEN .05 * p.`amount`
		WHEN length(replace(r.`lineage`, u.`lineage`, ""))-length(replace(replace(r.`lineage`, u.`lineage`, ""),"-","")) = 2 THEN .02 * p.`amount`
		WHEN length(replace(r.`lineage`, u.`lineage`, ""))-length(replace(replace(r.`lineage`, u.`lineage`, ""),"-","")) = 3 THEN .01 * p.`amount`
		ELSE 0
	END), 0) as `purchases_commission`
	FROM `users` u
	LEFT JOIN `users` r ON r.`lineage` LIKE CONCAT(u.`lineage`, "-%")
	LEFT JOIN `purchases` p ON p.`user_id` = r.`id`
	GROUP BY u.`id`
	
This would generate the following results.

<table class="table table-bordered">
	<tr>
		<td><strong>id</strong></td>
		<td><strong>username</strong></td>
		<td><strong>signup_commission</strong></td>
	</tr>
	<tr>
		<td>1</td>
		<td>user.1</td>
		<td>1.9950</td>
	</tr>
	<tr>
		<td>2</td>
		<td>user.2</td>
		<td>0.0000</td>
	</tr>
	<tr>
		<td>3</td>
		<td>user.3</td>
		<td>2.7920</td>
	</tr>
	<tr>
		<td>4</td>
		<td>user.4</td>
		<td>4.6430</td>
	</tr>
	<tr>
		<td>5</td>
		<td>user.5</td>
		<td>0.0000</td>
	</tr>
	<tr>
		<td>6</td>
		<td>user.6</td>
		<td>0.0000</td>
	</tr>
	<tr>
		<td>7</td>
		<td>user.7</td>
		<td>4.7450</td>
	</tr>
	<tr>
		<td>8</td>
		<td>user.8</td>
		<td>0.0000</td>
	</tr>
	<tr>
		<td>9</td>
		<td>user.9</td>
		<td>0.0000</td>
	</tr>
	<tr>
		<td>10</td>
		<td>user.10</td>
		<td>0.0000</td>
	</tr>
</table>


### Creating the final commission report for all users

This is basically combining the queries we've been running above. The query would be the
following.

	SELECT u.`id`, u.`username`, COUNT(DISTINCT(r1.`id`)) * 5 +
	COALESCE(SUM(CASE
		WHEN length(replace(r2.`lineage`, u.`lineage`, ""))-length(replace(replace(r2.`lineage`, u.`lineage`, ""),"-","")) = 1 THEN .05 * p.`amount`
		WHEN length(replace(r2.`lineage`, u.`lineage`, ""))-length(replace(replace(r2.`lineage`, u.`lineage`, ""),"-","")) = 2 THEN .02 * p.`amount`
		WHEN length(replace(r2.`lineage`, u.`lineage`, ""))-length(replace(replace(r2.`lineage`, u.`lineage`, ""),"-","")) = 3 THEN .01 * p.`amount`
		ELSE 0
	END), 0) as `total_commission`
	FROM `users` u
	LEFT JOIN `users` r1 ON u.`id` = r1.`referrer_id`
	LEFT JOIN `users` r2 ON r2.`lineage` LIKE CONCAT(u.`lineage`, "-%")
	LEFT JOIN `purchases` p ON p.`user_id` = r2.`id`
	GROUP BY u.`id`
	
This would generate the following results.

<table class="table table-bordered">
	<tr>
		<td><strong>id</strong></td>
		<td><strong>username</strong></td>
		<td><strong>signup_commission</strong></td>
	</tr>
	<tr>
		<td>1</td>
		<td>user.1</td>
		<td>6.9950</td>
	</tr>
	<tr>
		<td>2</td>
		<td>user.2</td>
		<td>10.0000</td>
	</tr>
	<tr>
		<td>3</td>
		<td>user.3</td>
		<td>7.7920</td>
	</tr>
	<tr>
		<td>4</td>
		<td>user.4</td>
		<td>9.6430</td>
	</tr>
	<tr>
		<td>5</td>
		<td>user.5</td>
		<td>0.0000</td>
	</tr>
	<tr>
		<td>6</td>
		<td>user.6</td>
		<td>5.0000</td>
	</tr>
	<tr>
		<td>7</td>
		<td>user.7</td>
		<td>9.7450</td>
	</tr>
	<tr>
		<td>8</td>
		<td>user.8</td>
		<td>0.0000</td>
	</tr>
	<tr>
		<td>9</td>
		<td>user.9</td>
		<td>0.0000</td>
	</tr>
	<tr>
		<td>10</td>
		<td>user.10</td>
		<td>0.0000</td>
	</tr>
</table>


To conclude this post, the lineage concept has been extremely useful to me. I've used it
on several project to create advanced reports and real-time dashboards.
