---
layout: post
title: Creating Pivot reports in MySQL
excerpt: With some tricks you can create dynamic pivot reports in Mysql. This is a step by step guide to achieve same result as PIVOT.
---

## Intro


<script type="text/javascript" src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js" async=""></script>
<img class="alignright" align='right' style="margin-left: 20px; margin-right: 20px;" title="Bar chart" alt="barchart" src="http://www.boynux.com/wp-content/uploads/2013/11/barchart-300x171.png" width="300" height="171" />
Honestly, one of nice feature that I know from MSSQL and missing in MySQL is PIVOT, it's very handy when generating pivot report from grouped data. Just imagine you have polling system in your website and you want to show the result in bar chart, like this:

<div class="ads"> <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins></div>
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>

	+-------------------------------------------------------+------------+----------+
	| title                                                 | answer     | count(*) |
	+-------------------------------------------------------+------------+----------+
	| Which SQL database do you prefer?                     | MSSQL      |        3 |
	| Which SQL database do you prefer?                     | MySQL      |       10 |
	| Which SQL database do you prefer?                     | PostgreSQL |        3 |
	| Which SQL database will you use in your next project? | MSSQL      |        7 |
	| Which SQL database will you use in your next project? | MySQL      |       25 |
	| Which SQL database will you use in your next project? | PostgreSQL |        9 |
	+-------------------------------------------------------+------------+----------+

Now you can use this information to create you bar charts, but isn't it better if I could fetch data in this format? 

	+-------------------------------------------------------+-------+-------+------------+
	| title                                                 | MSSQL | MySQL | PostgreSQL |
	+-------------------------------------------------------+-------+-------+------------+
	| Which SQL database do you prefer?                     |     3 |    10 |          3 |
	| Which SQL database will you use in your next project? |     7 |    25 |          9 |
	+-------------------------------------------------------+-------+-------+------------+

This form of data needs less processing at backend and you can directly feed this into Highcharts Javascript or similar libraries with very little manipulation to render, and this is what PIVOT does, Pivot Reports, it actually turns row values into column titles with corresponding values. 

## MySQL alternative 

Unfortunately MySQL does not support PIVOT but with some hacks you can achieve the same result. Before I start to show you how to do it, I'll give you initial table schema and sample data to create a sample data structure as I have to be able to follow this tutorial step by step. 

<div class="ads"> <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="rectangle"></ins> </div>
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>

Table schema: 

	CREATE TABLE `question` (
	   `id` int(11) NOT NULL AUTO_INCREMENT,
	   `title` varchar(256) NOT NULL,
	    PRIMARY KEY (`id`),
	    UNIQUE KEY `title` (`title`)
	) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=latin1

	CREATE TABLE `answer` (
	   `id` int(11) NOT NULL AUTO_INCREMENT,
	   `question_id` int(11) NOT NULL,
	   `answer` varchar(256) NOT NULL DEFAULT 'Not Answered',
	   PRIMARY KEY (`id`),
	   KEY `question_id` (`question_id`),
	   KEY `answer` (`answer`),
	   CONSTRAINT `answer_ibfk_1` FOREIGN KEY (`question_id`) REFERENCES `question` (`id`)
	) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=latin1

That's it, I created two tables to make things a little more complex, because in reality you might have many tables joining together so I show that you can adapt this idea into different situations. And now sample data: 

	INSERT INTO `answer` VALUES (1,1,'MySQL'),(2,1,'MySQL'),(3,1,'MySQL'),(4,1,'MSSQL'),(5,1,'PostgreSQL'),(6,1,'MySQL'),(9,1,'MySQL'),(10,1,'MySQL'),(11,1,'MSSQL'),(12,1,'PostgreSQL'),(13,1,'MySQL'),(16,1,'MySQL'),(17,1,'MySQL'),(18,1,'MSSQL'),(19,1,'PostgreSQL'),(20,1,'MySQL'),(23,2,'MSSQL'),(29,2,'MySQL'),(30,2,'MySQL'),(31,2,'MySQL'),(32,2,'MySQL'),(33,2,'MySQL'),(34,2,'PostgreSQL'),(35,2,'PostgreSQL'),(36,2,'PostgreSQL'),(52,2,'MSSQL'),(53,2,'MSSQL'),(54,2,'MSSQL'),(55,2,'MSSQL'),(56,2,'MSSQL'),(57,2,'MSSQL'),(58,2,'MySQL'),(59,2,'MySQL'),(60,2,'MySQL'),(61,2,'MySQL'),(62,2,'MySQL'),(63,2,'MySQL'),(64,2,'MySQL'),(65,2,'MySQL'),(66,2,'MySQL'),(67,2,'MySQL'),(68,2,'MySQL'),(69,2,'MySQL'),(70,2,'MySQL'),(71,2,'MySQL'),(72,2,'MySQL'),(73,2,'MySQL'),(74,2,'MySQL'),(75,2,'MySQL'),(76,2,'MySQL'),(77,2,'MySQL'),(78,2,'PostgreSQL'),(79,2,'PostgreSQL'),(80,2,'PostgreSQL'),(81,2,'PostgreSQL'),(82,2,'PostgreSQL'),(83,2,'PostgreSQL');

	INSERT INTO `question` VALUES (1,' Which SQL database do you prefer?'),(2,'Which SQL database will you use in your next project?');

OK, we have same data sample to start. In order to get same result as `PIVOT` in MySQL you must initiate a query like this: 

	SELECT
	  title,
	  SUM(IF(answer = 'MySQL', 1, 0)) AS 'MySQL',
	  SUM(IF(answer = 'MSSQL', 1, 0)) AS 'MSSQL',
	  SUM(IF(answer = 'PostgreSQL', 1, 0)) AS 'PostgreSQL'
	FROM
	  question
	INNER JOIN
	  answer ON answer.question_id = question.id
	GROUP BY
	  question.id,  title

Let's explain this a little bit more, what this query actually does is using aggregate functions along with `IF` statement to create new `SELECT` columns and increment their values by one every time it finds that value in grouping same as the column titles. Output of this query is what you've seen at the beginning of this page. But there is a problem with this query! what if I add a new value to answer table, like 'Oracle'? Obviously I must go and change query to add new column named 'Oracle' and it's not very practical! One solution is to use backend code to generate a dynamic query, that's fine but it's not what I'm looking for. 

## Solution: 

Playing a little bit with MySQL functions and documentation will lead to an acceptable solution for this problem. So, I break that  query apart into two sections to make it more clear what we need to create: 

***1.***

	SUM(IF(answer = 'MySQL', 1, 0)) AS 'MySQL',
	SUM(IF(answer = 'MSSQL', 1, 0)) AS 'MSSQL',
	SUM(IF(answer = 'PostgreSQL', 1, 0)) AS 'PostgreSQL'

***2.***

	SELECT
	  title,
	  ...
	FROM
	  question
	INNER JOIN
	  answer ON answer.question_id = question.id
	GROUP BY
	  question.id,  title

First I create the number one, I'm going to use `GROUP_CONCAT` and `CONCAT` functions and puting the result into '@answers' variable to achieve that: 

	SELECT
	    GROUP_CONCAT(
		CONCAT('SUM(IF(answer='', answer, '',1 ,0)) AS '', answer, '''), 'n'
	    ) INTO @answers
	FROM (
	    SELECT DISTINCT answer FROM answer
	) A

The second part is a very simple SQL query and its not something special, but we need to find a way to combine these two parts together. For this I use `CONCAT` again to create another string by concatenating those two: 

	SET @query := 
	  CONCAT(
	    'SELECT title, ', @answers, 
	    ' FROM question INNER JOIN  answer ON answer.question_id = question.id  GROUP BY question.id, title'
	  );

Well, now '@query' variable contains exactly the same query that we are looking for. and final step is to find a way to run this query, this is the most simple part we can use `PREPARE` and `EXECUTE` commands. To make it even more easy I create a stored procedure to do all the job: 

	DROP PROCEDURE IF EXISTS pivot_question;
	CREATE PROCEDURE pivot_question()
	BEGIN
	  SELECT
	    GROUP_CONCAT(
		CONCAT('SUM(IF(answer='', answer, '',1 ,0)) AS '', answer, '''), 'n'
	    ) INTO @answers
	  FROM (
	    SELECT DISTINCT answer FROM answer
	  ) A;

	  SET @query := 
	    CONCAT(
	      'SELECT title, ', @answers, 
	      ' FROM question INNER JOIN  answer ON answer.question_id = question.id  GROUP BY question.id, title'
	    );

	  PREPARE statement FROM @query;
	  EXECUTE statement;
	END;

Voilà, That's it, and if I call `pivot_question` in MySQL I get exactly what I was looking for and in addition if a  new answer added into answer table it will be automatically shown into the result set: 

	mysql> call pivot_question;;
	+-------------------------------------------------------+-------+-------+------------+
	| title                                                 | MSSQL | MySQL | PostgreSQL |
	+-------------------------------------------------------+-------+-------+------------+
	| Which SQL database do you prefer?                     |     3 |    10 |          3 |
	| Which SQL database will you use in your next project? |     7 |    25 |          9 |
	+-------------------------------------------------------+-------+-------+------------+
	2 rows in set (0.01 sec)

	Query OK, 0 rows affected (0.01 sec)

In addition it is `Stored Procedure` and you can add inputs like `question ID`  in order to make it more customized, no need to say you can still use those query parts as simple SQL statements without using `Stored Procedure`s. 

Final note: Thank you and I hope you enjoyed reading it much as I enjoyed writing and I would be very happy to hear your opinion and comments about this article. 

<div class="ads"> <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> </div>
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
<div class="ads"> <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins></div>
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
