---
title: Legible SQL

weight: 2
bookToc: false
---

Writing legible code is seen as the hallmark of a competent developer, and for good reason.  If you can't read what you wrote, you can't change what you wrote. There are lots of organizations, especially once you get to larger companies where programming isn't the core competency or value center, that are prime examples of this. The result is a mess that not only impacts the code, but the entire company or business. 

There are great essays and books on reproducible code in almost every imaginable compiled and interpreted language, from Python, to Java, and great generalist essays, like Knuth's on writing reproducible code. 

But what about SQL, still the #1 data language of choice in 80% of the tech enterprise world?  

There are a couple problems with making SQL legible
---

SQL is often the redheaded stepchild of computer languages.

First, because it's a declarative language, it's hard to do the same kind of concepts that are super-easy in almost any other language: loops, recursion, functions, or even declaring variables. You can do these things in T-SQL, but learning T-SQL and optimizing queries in it is almost not worth it if you just need to pull x dataset for sampling and want to spend most of your time in Python or R analyzing said data,  or Y dataset with a,b,and c business logic requirements built in.  As a result, it's hard to see SQL as logical and amenable to logical constraints or exceptions, as in "regular" languages. The other issue is that SQL is often seen as a necessary evil and developers often try to force their way through it in order to write ORMs instead of seeing it as a value-add for data teams. 

Second, because it doesn't need to be written in a specific format, saved to an editor, or checked into repositories, SQL snippets often end up languishing in Oracle windows, Evernotes, text files in random project directories, and even in emails (guilty as charged -I've used Outlook as a SQL query storage mechanism before.)

Even though there is a lot of hype about NoSQL data systems, most data scientists will tell you that they spend a **LOT** of time getting data sets ready in MySQL or Oracle or SQL Server or Postgres as part of their data flow. 

Then these code snippets are usually stored away in everything from Word docs, to Evernotes, to text files in project directories. Because SQL is declarative and run one person at a time, most times, unless it's being run as part of a Python or R call through the respective APIs,  it's not even put into version control. 

But what about when SQL code needs to be stored or rerun? How to read the jumbled mess of nested subqueries and tons of case statements necessitated by the fact that the business logic for the data requires it? 

Here are a couple tips from painful personal experience. All of these are guidelines, by the way, and geared towards data scientists/analysts working alone on manual SQL tasks that are not part of an automated data flow (but even in that case, cleaning up code will make other components easier to read, too) 

The guidelines don't mean you have to use them every time. I've personally written some pretty ugly, functional SQL when it was crunch time. But they will make your life prettier and easier.  This is not specifically about optimizing SQL, which I'll cover in another post.   

How to make SQL legible to other humans
----

### 1. Make all declarative keywords their own new line if possible because it's easier to read and understand which table and which columns are being referenced: 

 ```sql

 SELECT * FROM tablex
 WHERE variabley=z 
 AND variablea=12
 AND variableb=15
 ```

is more legible than

 ```sql		

 select * from tablex where variabley=z AND variablea=12 AND variableb=15
```


### 2.  Make all declarative statements and DB functions uppercase

 ```sql

SELECT * FROM tablex
WHERE variabley=z 
AND variablea=12
AND variableb=15
ORDER BY RAND()
LIMIT 1
 ```

is more legible than

 ```sql		

 select * from tablex
 where variabley=z 
 AND variablea=12
 AND variableb=15
 order by RAND()
 limit 1
```

It's easier to read all uppercase. Or, alternatively, if you don't have time (and it is kind of annoying to remember), make them all lowercase. Just don't mix cases. And uppercase is preferable in any kind of documentation. The same is true for tablenames. Make them either all lower case or all uppercase. Lowercase is preferable to avoid mixing them up with  declaratives. 
 

### 3. Try to limit the amount of subqueries in your query.  

Yes, subqueries are much, much faster if you're aggregating across multiple data sets. Check out [this post](https://www.periscope.io/blog/use-subqueries-to-count-distinct-50x-faster.html) for more detail as to why. But when you get to the point where you have more than 4 subqueries, you probably want to do some optimization, or if possible, rebuild your tables in a way that makes more sense. If you find yourself CONSTANTLY having to write subqueries, it's a sign that your data schemas are architected incorrectly.  

### 4. Include comments.

  Both in the beginning of the code snippet to say what data the snippet pulls, the business logic it includes, and throughout, as much as possible, to reference subqueries. SQL makes it hard to read through an entire piece of code without running specific pieces one by one to get where you're going and comments will help jog the memory. Include them at the end of lines in code, or in blocks at the beginning of code. 

 ```sql	

 /*Pulls a sample of 1000 random Nutella orders
  by month from the Northeast and excludes 16-oz sized jars. 
 Orders without monetary transactions are excluded, too. 
 Pulls into R script for regression analysis. */	

SELECT * from trans a --transactions table 
JOIN region b --region table with 5 splits
ON a.trans_id=b.regionid
WHERE a.product_type='nutella'
AND b.regionid='NE' --NE=northeast
ORDER BY RAND() --selecting a random sample of 100
LIMIT 100
```

### 5. Use short but concise table aliases

You're probably doing a lot of joins. You want to see what those joins are doing. If you have a transactions_table, you don't want to alias it as transactions_table because that becomes too cluttered. Call it `trans`. `t` is too short and probably won't mean anything when you try to read the code back and translate it into business logic. 

### 6. Use indentation. If you have multiple subqueries, it's much easier to read where the subqueries came from. 


 ```sql

SELECT * FROM tablex
JOIN 
	(SELECT a, b 
	 FROM tabley
	 WHERE x='5') ysub
WHERE variabley=z 
AND variablea=12
AND variableb=15
ORDER BY RAND()
LIMIT 1
 ```

is more legible than

 ```sql		

 select * from tablex
 JOIN (select a, b from tabley where x='5') ysub
 where variabley=z 
 AND variablea=12
 AND variableb=15
 order by RAND()
 limit 1
```


### 7. Make new tables. 

If you find your query becoming longer than some random threshold you set for yourself, i.e. 15 lines or so AND you find yourself running it over and over, it's probably time to create a permanent table for your data. 

Here's another great post I referenced while writing this one:

+ [How do I make complex SQL queries easier to write? on Stack Overflow](http://programmers.stackexchange.com/questions/144602/how-do-i-make-complex-sql-queries-easier-to-write)
