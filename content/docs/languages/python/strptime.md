---
title: strptime and strftime
weight: 2
bookToc: false
---

# One weird trick about Python date formats

I've been working with date manipulation in Python as part of[ this Reddit challenge](https://www.reddit.com/r/dailyprogrammer/comments/49aatn/20160307_challenge_257_easy_in_what_year_were/), which asks the programmer to find the year(s) in history when the most US presidents were alive given each president's birth and death date. 

Usually, when [working with dates and times in programming](http://infiniteundo.com/post/25326999628/falsehoods-programmers-believe-about-time), it's best to convert to object types that inherently have some assumptions about time and date already built in to allow for easy addition, subtraction, and date conversion.  

In Python, the [datetime library](https://docs.python.org/2/library/datetime.html) provides convenient ways to work with date and time data, as well as converting data to and from those formats. 

There used to be a [module called time](http://effbot.org/librarybook/time.htm) that was not based exclusively on object-oriented patterns. Datetime is an update of time. 

Datetime has several object types, (as seen in the Python REPL):

```python
	>>>import datetime
	>>> dir(datetime)
	['MAXYEAR', 'MINYEAR', '__doc__', '__file__', '__name__', '__package__', 'date', 'datetime', 'datetime_CAPI', 'time', 'timedelta', 'tzinfo']
```

I was particularly interested in coverting all the years to datetime or date objects and working with them that way. 

A datetime object looks like this, where you're given back year-month-day-hour-minute-second-microcecond. 

```python

	>>>datetime.datetime.now()
	>>>datetime.datetime(2016, 3, 9, 16, 32, 12, 385142)
	

	>>>now = datetime.datetime.now()
	>>> type(now)
	>>><type 'datetime.datetime'>

A date object is very similar, but does not include the time. 

	>>>nowday = datetime.date.today()
	>>> nowday
	>>>datetime.date(2016, 3, 9)
```

To convert strings to these types of object, Python's datetime has strptime, a function that has an equivalent in most modern languages. 

I always get strptime and strftime confused, so here are silly mnemonics I can remember:

+ **strptime** - converts strings to datetime objects (stri**p**s strings to datetime)
+ **strftime** - converts datetime objects to specific human-readable format while still maintaining their datetimey-ness.  (F is for formatting)
 
So let's say I have a date that's a string: 
 ```python
	x = "19 Aug 2015"
	type(x)
	<type 'str'>
```
And I want to read it into a datetime object: 
```python
	datetime.datetime.strptime(x, '%d %b %Y')
	>>>datetime.datetime(2015, 8, 19, 0, 0)
```

Then I can add days, years, subtract days, etc. The ability to automatically do math with differing units of time is what makes this module so convenient.  

I can then spit that calculation back out in any format I want with strftime, for example, I wanted to see the entire month: 
```python
	>>> y = datetime.datetime.strptime(x, '%d %b %Y')
	>>> datetime.datetime.strftime(y, '%B-%d-%Y')
	'August-19-2015'
```
There are a couple caveats that make strftime and strptime really annoying.  
	
## Strptime and formatting: 

The first is that the formatting of the input string needs to match exactly, otherwise the assignment throws an exception. This was from when I omitted the spaces from the formatting paramter:  

	ValueError: time data '19 Aug 2015' does not match format '%d%b%Y'

This can be extremely irritating if you have data in the same column that changes formats. In this case, you have to write `try/except` blocks or find some way of working around this "limitation."

## Strftime and Python 2/3: 

The other VERY annoying thing is that strptime does not work on data earlier than 1900. 

I came across one when I was trying to add and subtract historical dates of presidents born before 1900. [You get the following](http://stackoverflow.com/questions/10263956/use-datetime-strftime-on-years-before-1900-require-year-1900): 

	ValueError: year=1601 is before 1900; the datetime strftime() methods require year >= 1900

The [documentation states](https://docs.python.org/2/library/datetime.html#strftime-behavior), and I'm guessing it's a carryover from C that is no longer relevant to currnent computing memory constraints. [Excel](http://exceluser.com/formulas/earlydates.htm) has the same issue. 

>The exact range of years for which strftime() works also varies across platforms. Regardless of platform, years before 1900 cannot be used.

The easy fix for this is to use Python 3, which [accounts for it](https://bugs.python.org/issue1777412):

Here's my simple script, `datetimenew.py`: 

```python
	from datetime import datetime
	x = "19 Aug 1500"
	y = datetime.strptime(x, '%d %b %Y')
	outformat = datetime.strftime(y, '%Y')
	print (outformat)

If I try to run it in Python 2:
 
	vboykis$ python datetimenew.py
	Traceback (most recent call last):
  		File "datetimenew.py", line 5, in <module>
    	outformat = datetime.strftime(y, '%Y')
	ValueError: year=1500 is before 1900; the 	datetime strftime() methods require year >= 1900
```

And 3: 

```python
	vboykis$ python3 datetimenew.py
	1500
```python

Yet another reason to finally make that switch ;). 

Or to use isoformat from date instead:
```python
	date.isoformat()
	Return a string representing the date in ISO 8601 format, ‘YYYY-MM-DD’. For example, date(2002, 12, 4).isoformat() == '2002-12-04'
```	
