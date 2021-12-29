---
weight: 1
bookFlatSection: true
title: "What is machine learning engineering?"
---

# What is machine learning engineering?



Once, on a crisp cloudless morning in early fall, a machine learning engineer left her home to seek the answers that she could not find, even in the [newly-optimized](https://blog.google/products/search/search-language-understanding-bert/) Google results.

She closed her laptop, put on her backpack and hiking boots, and walked quietly out her door and past her mailbox, down a dusty path that led past a stream, until the houses around her gave way to broad fields full of ripening corn. 

She walked past farms where cows grazed peacefully underneath enormous data silos, until the rows of crops gave way to a smattering of graceful pines and oaks, and she found herself in a forest clearing, headed into the woods. She went deeper through the decision trees and finally stopped near a data stream around midday to have lunch and stretch her legs. 

The sun made its way through the sky and eventually, she walked further, out of the forest. Finally, she found a path that started snaking its way up a mountainside, and she started to hike upwards, through the red rocks. After several hours she stopped and took a drink from her Klean Kanteen as she surveyed the sprawling random forest, the valley spread out below her and the sparkling data lake in the distance. 

Here, on the upper slopes of the mountain, the filepath became erratic and truncated, and now she was mostly climbing, using all of the muscles in her haunches to gain her footing. The sun suddenly lost its heat and shadows started rising in the slopes of the hills. 

Finally, after several more hours, she ascended to the top of the mountain’s gradient. At the peak, there was a low, flat platform. On the platform was a bench, and on the bench, a solitary figure sat, drinking a cold brew and looking out at the sunset. 

She realized she was in the Presence of a Staff Engineer.  Although he could not have been older than thirty-six, his face was already careworn with the wrinkles of a man who had been on   PagerDuty more than once in the last fiscal quarter. 

“Oh wise one,” she said, prostrating herself before him and offering him a sacred token of respect, her YubiKey.  “I have so many questions about machine learning engineering,” she said.  

The man looked at her wearily. “This isn’t about that PR from Tuesday, is it? I have it in my Jira backlog, I just haven’t gotten to it yet. That’s why I’m up here. No WiFi means emails don’t exist here.”

“Oh no, no,” she said. “Far be it from me to Block you, Your Staffness. I have just been a machine learning engineer for a very long time now, since even before Tensorflow 2, and I have been wondering about the meaning of it all.”

The man immediately relaxed. “Oh good, at least it’s not a question about [bundling Python executables](http://veekaybee.github.io/2018/09/24/the-case-of-the-broken-lambda/). Come sit down?”

She sat near him on the bench.

“What is it,” he said.

“Well,” she said, hesitating. “I’ve been doing machine learning engineering for a long time now, and I keep wondering when I’m actually going to get to the...machine learning engineering.”

“What do you mean? What do you do on a day-to-day basis?”

“Last year, my manager tasked me with building a machine learning model to predict the churn for our B2B startup, which specializes in selling mattresses as a service to [startups that sell mattresses](https://vicki.substack.com/p/your-mattress-as-a-service) to B2C consumers.”

“Ok...”

“So I knew I’d need to build a model that would use [XGBoost](https://ieeexplore.ieee.org/document/9103818) and collect the features we needed to use to predict churn. For example, we’d realized that we needed to know the age of the cohort when the customer signed up because newer customers tended to churn more quickly, how many times they visited our site, how many times they hovered over the “subscribe” button, and how many times they called support. 

But first, [we didn’t even know that customers were churning](https://vicki.substack.com/p/all-numbers-are-made-up-some-are) and our explanatory variable was complete noise,  because some of our customers cancelled and then restarted using our service. We’d mark the cancellation in one column, ‘cancelled_service’, and the restart in a second column, ‘new_customer’,  so I had to do feature engineering and join two columns from our Salesforce data which we batch via Sqoop into Hive on HDFS on a daily basis. I thought I was good to go, but that day, the batch job failed and so I didn’t have the latest data, so I had to wait.”

“When the data didn’t hit the next day, I looked at the code responsible for populating those tables,I could check the cron scheduler for the Sqoop job. The cron scheduler for the Sqoop job had somehow become broken (we’re only in the middle of our Airflow migration), and I had to patch it. After a couple hours, we started getting data again and I was able to finally get that response field. Now, I could do some feature engineering to pull the rest of the variables. “

“The only problem was that, while the billing data was in our legacy HDFS, the newer, clickstream app data, was being streamed via Kinesis into S3, and then into Athena. The other issue with the clickstream data was that it had a ton of non-standard JSON fields that I had to extract specifically to get the events I needed to understand how many times the customer had hovered over the subscribe button and clicked through, and create sessions for user data. I used jq to check those fields and then wrote them up in our metadata dictionary.”

“Once I had that data available to me, I needed a place that our security team allowed so that I could combine the two data sources. They suggested using Spark on AWS to write out to S3, which they’d then send back to HDFS. Our Spark workloads run on K8s on AWS Spot Instances, but some of the pods were hanging, so I used kubectl to take a look at them and figure out what the issue was.”

{{< tweet 1438900066949939206 >}}

“Now, I had all of the data in a single place, and it was time to build my XGBoost model and iterate on it . But the Python available to me on the machine that could access the HDFS server didn’t have all the dependencies I needed to run XGBoost, so I-”

The Staff Engineer held up a palm as if to stop the torrent of pain that flowed from the machine learning engineer. “I’ve heard enough,” he said, sighing. 

“I didn’t even get to the part where I’ll need to surface the serialized results of the churn model into a frontend UI for sales and marketing to consume and make decisions on. The latency-”

The machine learning engineer sighed deeply herself and stopped, as if whatever she said next would break her.  They sat in silence for a bit. The Staff Engineer sipped his cold brew thoughtfully through a biodegradable straw. 

Finally, the machine learning engineer said, in a very small voice, “I still haven’t gotten to machine learning engineering.”

The Staff Engineer scratched his head. “Well, hang on.  Let’s list out all of the things that you just said. You said you were:”


* Working with serialization formats
* Evaluating modeling requirements and selecting the right model for the business case
* Using jq and getting what you need from JSON fields
* Handling cron and bash scripting and getting ETL jobs to run 
* Tuning and optimizing SQL
* Working on containers and orchestration debugging
* Shaping, understanding, and describing your data
* Reasoning about distributed systems
* Being defensive around data you ingest that impacts your ML pipeline
* Integration testing across distributed file stores
* Working with HTTP verbs, networking, simple issues that could go wrong when you’re working across machines, SSH, nohup, screen, port forwarding, exposing ports

“And, most importantly, you were":

* Architecting a scalable, resilient system to deliver results to key stakeholders using machine learning insights surfaced into an accessible front-end layer

“I have some (potentially bad) news for you, you’re doing machine learning engineering.”

“But I haven’t even touched a model yet.”

“Have you read the [black box paper](https://proceedings.neurips.cc/paper/2015/file/86df7dcfd896fcaf2674f757a2463eba-Paper.pdf)? The ecosystem of the model is always greater than the model itself.”

The machine learning engineer frowned. “But how can it be?”

“Machine learning systems are new. [We’re still in the steam-powered days of machine learning](https://vicki.substack.com/p/were-still-in-the-steam-powered-days), and yet machine learning is not simply machine learning. It is, at this stage, more [engineering](http://veekaybee.github.io/2019/02/13/data-science-is-different/) than simply machine learning. We’re building more and more on older systems, [abstracting away complexity and in the process creating newer and newer levels of it](http://veekaybee.github.io/2019/04/11/attic-compsci/) that we now have to manage and hold in our heads. Many of the algorithms have been written. Much of the work we do, both in machine learning, and in development today in general, will be [glue work and vendor work.](https://rachelbythebay.com/w/2020/08/14/jobs/) 

{{< tweet 1113512466795900929 >}}

“So what does this mean for me?”

“Nothing, keep doing what you’re doing. Keep grinding away. You’ll get to XGBoost eventually, and then, after working with the model for a brief period, you’ll have a whole new set of very boring, non model-specific problems related to: 

* Figuring out what online and offline metrics should be, where you'll store them, and how to analyze them
* [Tuning hyperparameters over and over again](https://sagemaker-examples.readthedocs.io/en/latest/introduction_to_applying_machine_learning/xgboost_customer_churn/xgboost_customer_churn.html) 
* [Cleaning up lots and lots of notebooks](http://veekaybee.github.io/2020/02/25/secrets/)
* Forgetting to shut down your notebooks and incurring cloud costs 

“So what do I do?” asked the machine learning engineer, distressed. “How do I continue in this field?”

“Nothing,” the Staff engineer said, leaning back, taking the last sip of his coffee. “You make peace with it. This is the job, writing and gluing together the code that makes drastically different systems speak to each other in data-oriented language.”

“The beautiful part, though, is when you finally connect all these systems and your database is talking to your streaming platform and your model is reading from the database and you have a new model every day and, finally, one day, someone from sales will come to you and say, we just prevented a customer from churning because we offered them a hypoallergenic organic mattress based on their previous browsing behavior and we kept the customer, and then you can look back through the decision trees, to see the forest of what you have built. The [working system in production is our reward](http://veekaybee.github.io/2020/06/09/ml-in-prod/), and we always move towards that.”

The machine learning engineer looked at the sunset thoughtfully. 

The Staff Engineer said, “Let me ask you something. Did you enjoy the walk here, even though it was long, hard, and annoying?”

“Well, yeah,” the MLE paused.  “The world is beautiful.”

“That’s it. In machine learning engineering, the journey, ultimately, is the destination,” the Staff Engineer said, neatly depositing his iced coffee container in the recycling bin located next to the bench, and checked his phone. 

“Shit, PagerDuty,” he exclaimed, and ran to his backpack, swiftly pulling out a MacBook Pro.  As he was descending further down the mountain where there was WiFi he turned to the machine learning engineer and said, “[See you in prod](http://veekaybee.github.io/2021/06/20/the-ritual-of-the-deploy/),” and then he vanished out of view.  

