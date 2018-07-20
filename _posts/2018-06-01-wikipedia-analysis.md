---
layout: inner
title: 'Wikipedia Data Analysis using MapReduce'
date: 2018-06-01 08:00:00
categories: Tech
tags: Python Cloud MapReduce Hadoop AWS
featured_image: '/img/posts/thumbnails/01_map_reduce.jpeg'
project_link: 'https://github.com/shantanu27/Parallel-Analysis'
lead_text: 'Big Data Analytics on a large Wikipedia dataset done in the Cloud (AWS)'
---

# Big Data Analytics on a large Wikipedia dataset done in the Cloud (AWS)
<br>
Wikipedia is often a great reflector of current events. Pages that are being accessed with increasing frequency often indicate an event related to the topic of the page. Wikipedia serves as a fairly unbiased source of reporting of the news, and sometimes even reveals hidden patterns. This project involved processing and analysing 1 month (720 hours/135 GB) worth of traffic log from Wikipedia. The month chosen was November 2016 (ring a bell?).

### What did I use?
- Python
- MapReduce
- AWS (Amazon Web Services)
- Python Data Analysis Library (pandas)
- Jupyter Notebook

### Data Pre-processing

The amount of data to process was huge. Before it could be analysed, it needed to be filtered and process into a form which could be later fed to different analysis libraries (in my case, *pandas*). Some of the properties of interest were - only English pages, no malformed data, URL normalization and percent-encoding, only [Wikipedia namespaces](https://en.wikipedia.org/wiki/Wikipedia:Namespace), article title limitations, blacklisted file extensions, [Wikipedia Disambiguations](https://en.wikipedia.org/wiki/Wikipedia:Disambiguation) etc. 

As you can see, there were lot of cases which needed to be handled. To make it easier to incorporate these cases, debug, and easily scale, I decided to break all the pre-processing components into granular functions, where each function just had one role - tell whether the input could be filtered. I could then 'chain' the functions in any order desired, enabling me to add/remove conditions, and debug easily if something didn't work as expected. Like -

```python
def filter_malformed_data(input):
    # some code
    return <True or False>

def filter_english_pages(input):
    # some code
    return <True or False>

filters = [filter_malformed_data(), filter_english_pages()]

if all(f(data) for f in filters):
    # proceed further
```

### Map-Reduce Operation

Once data was filtered out, next big step was to write the mappers and reducers that would finally produce the daily view count for the month of November 2016. Since it was an academic project, I cannot discuss the map-reduce program in detail. The only topics of interest were the ones having more than 100,000 hits in total. 

To accomplish this task, I used the EMR (Elastic MapReduce) service provided by AWS. I had 3 nodes in the cluster - 1 master and 2 core (slave) nodes. The machines were of type **m4.large**. The entire job took around 2 hours to run (135 GB data, remember?). The final output of the mapreduce job was a file few megabytes in size and of the following format -

```
<Total Count> <Topic>   <Day 1 View Count>  <Day 2 View Count>  .....  <Day 30 View Count>
```

Example -

```
9016683 United_States_presidential_election_2016   108018  126420  127054  135039  134971  220274  375259  723502  2332337 811996  536615  354246  372025  305104  279582  239844  206588  153615  125084  124844  140998  134532  153956  146080  110354  124210  115150  120080  100642  78264
```

### Analysis

Once the data was ready, I used *pandas* to do some basic analyses. The most obvious question to ask was, "Which was the most searched topic of November 2016?". And the answer, unsurprisingly, was *Donald Trump*! There were almost 18 million hits on that topic. As expected, *United States Presidential Election 2016*, *Hillary Clinton*, *Melania Trump* and *Barack Obama* were in the most searched topics too!

