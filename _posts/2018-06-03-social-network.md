---
layout: inner
title: 'Social Networking Timeline with Heterogenous Backends'
date: 2018-06-03 08:00:00
categories: Tech
tags: Java MySQL MongoDB HBase MapReduce AWS
featured_image: '/img/posts/thumbnails/03_social_timeline.jpg'
project_link: 'https://github.com/shantanu27/Social-Network'
lead_text: 'Social Networking Timeline with Heterogeneous Backends'
---

# Social Networking Timeline with Heterogeneous Backends

<header class = "titleimage_social_network">
	<img src="{{ '/img/posts/03_social_timeline.png' | site.baseurl }}" alt="Social Networking Timeline" title="Source: Cloud Computing - Carnegie Mellon University">
</header>

### Scenario

The proposed idea was to build a social networking website which used the data from Reddit.com to allow people to view other people's comments in a timeline manner. This offered a view for users to notice what their friends were interested in those days. With a budget of $15, I was required to build a prototype for the backend of this social networking website.

### What did I use?
- Java
- MySQL (through Amazon RDS)
- MongoDB
- HBase
- MapReduce
- Terraform

### Architectural Components (Setup)

The frontend for the website was provided, which called different backend services to load various components in the frontend. The project required me to implement the following four functionalities:

**1. Implementing Basic Login with MySQL on Amazon RDS**
<br>
User profiles had username, password, and profile image urls. Since this was highly structured data, MySQL was the best way to store this data. Amazon RDS provides Database-as-a-Service (DBaaS) allowing users to easily configure and deploy relational databases. The database was deployed in a *t2.micro* instance. I wrote a corresponding Java service which provided endpoints to authenticate the user using JDBC drivers. 

**2. Storing Social Graph using HBase**
<br>
A user can follow other users and has followers too. When considering the graph with respect to all the users in the database, it is a spare graph. One of the efficient ways of representing this data is in the form of adjacency lists. HBase (which emulates Google's BigTable) makes a suitable candidate for storing social graph. 

The data provided to me was in the format ```followee->follower```. To construct an adjacency list representation of the data, I needed a MapReduce program which would convert it into that format and store in the following format in HBase:

![HBase Followee-Follower]({{ "/img/posts/03_follower_followee.png" | site.baseurl }} "Source: Cloud Computing - Carnegie Mellon University")

The database was deployed in using a cluster having 3 nodes (1 master and 2 slaves) of type *m4.large*. After storing the data in HBase, I wrote the corresponding Java service to return list of followers given a user_id using the HBase Java libraries.

**3. Building Homepage using MongoDB**
<br>
Next step involved storing and displaying user comments on the timeline. A document store like MongoDB becomes an ideal choice for storing comments, since it allows us to quickly query the data as if it had a structure, but does not enforce a schema like a relational database. I was given a json file containing comments made by a user. Each line in the json had data in the following structure:
```
{
    "cid":"xxx",                          // Comment ID

    "parent_id":"xxx",                    // The parent comment of this comment

    "uid":"xxxxx",                        // User name of the comment

    "timestamp":"xxxxxxxxxx",             // When post is posted

    "content":"xxxxxx",                   // Comment content

    "subreddit":"xxxx",                   // Subreddit category of this comment

    "ups":xx                              // Number of ups received

    "downs":xx                            // Number of downs received 
}
```
I built a service endpoint, which would return all the comments of a given user sorted in descending order of "upvotes".

**4. Putting Everything Together into a Timeline**
<br>
Apart from the information assembled above, we were also required to develop a service that would return the 30 most popular comments by the followees. And in those comments, it returned them in the nesting level of parent and grandparent. And of course, the comments were sorted in decreasing order of popularity. 	

<style>
pre {
  white-space: pre !important;
  overflow-y: scroll !important;
  height: 35vh !important;
}
</style>

```
returnRes({
  "followers": [
    {
      "profile": "https://farm4.staticflickr.com/3733/10498671606_caba28c5a6_m.jpg",
      "name": "Fortanono"
    },
    {
      "profile": "https://farm6.staticflickr.com/5779/20996394095_e3a527d780_m.jpg",
      "name": "That_Zeffia_guy"
    },

    (More followers...)

  ],
  "comments": [
    {
      "uid": "MacerV",
      "downs": 0,
      "parent_id": "t3_34u8s5",
      "ups": 466,
      "_id": {
        "$oid": "589a39bc36b221496362e1c1"
      },
      "subreddit": "KerbalSpaceProgram",
      "content": "Just started chuckling uncontrollably at the gif. Good choice.",
      "cid": "t1_cqy3fxn",
      "timestamp": "1430757711"
    },
    {
      "uid": "MacerV",
      "downs": 0,
      "parent_id": "t3_37afc6",
      "ups": 197,
      "_id": {
        "$oid": "589a39bc36b221496362e0c6"
      },
      "subreddit": "hockey",
      "content": "Well thats a rather redundant statement.",
      "cid": "t1_crl0nvs",
      "timestamp": "1432613962"
    },
    {
      "grand_parent": {
        "uid": "IAteOkayZebraVulva",
        "downs": 0,
        "parent_id": "t1_cr4lbp4",
        "ups": 421,
        "_id": {
          "$oid": "589a35d936b22149639aac54"
        },
        "subreddit": "creepyPMs",
        "content": "Yup. She also threatened to kill herself and blamed my friend. But you know, she was the best big she could be.",
        "cid": "t1_cr4lxec",
        "timestamp": "1431276739"
      },
      "uid": "onthefence928",
      "parent": {
        "uid": "Das_Perderdernerter",
        "downs": 0,
        "parent_id": "t1_cr4lxec",
        "ups": 102,
        "_id": {
          "$oid": "589a3afa36b2214963a31b98"
        },
        "subreddit": "creepyPMs",
        "content": "As someone not in the American schooling system, what's a \"Big\"?",
        "cid": "t1_cr4oi20",
        "timestamp": "1431281880"
      },
      "downs": 0,
      "parent_id": "t1_cr4oi20",
      "ups": 173,
      "_id": {
        "$oid": "589a37fe36b22149630ab79e"
      },
      "subreddit": "creepyPMs",
      "content": "In the greek fraternity system they gave a tradition where more senior members play big sibling to younger members, it's supposed to work like mentorship. ",
      "cid": "t1_cr4omt3",
      "timestamp": "1431282139"
    },
    {
      "grand_parent": {
        "uid": "Ragexz",
        "downs": 0,
        "parent_id": "t3_35e6or",
        "ups": 250,
        "_id": {
          "$oid": "589a3de236b221496339b3fe"
        },
        "subreddit": "KerbalSpaceProgram",
        "content": "Most linear conversation ever.",
        "cid": "t1_cr3kudr",
        "timestamp": "1431181222"
      },
      "uid": "MacerV",
      "parent": {
        "uid": "drillgorg",
        "downs": 0,
        "parent_id": "t1_cr3kudr",
        "ups": 185,
        "_id": {
          "$oid": "589a35f236b22149639ff424"
        },
        "subreddit": "KerbalSpaceProgram",
        "content": "It's how us engineering students communicate.",
        "cid": "t1_cr3l0ay",
        "timestamp": "1431181614"
      },
      "downs": 0,
      "parent_id": "t1_cr3l0ay",
      "ups": 143,
      "_id": {
        "$oid": "589a39bc36b221496362e0db"
      },
      "subreddit": "KerbalSpaceProgram",
      "content": "I can confirm. You learn quickly to speak clearly and effectively because communication errors cause mistakes; sometimes lethal ones. ",
      "cid": "t1_cr3lsxq",
      "timestamp": "1431183463"
    },

    (More comments...)

  "profile": "https://farm2.staticflickr.com/1519/26254768136_09c360b4c1_m.jpg",
  "name": "Branibor"
})
```

### Conclusion

To speed-up the development and testing process, I also used *terraform* to spin-up/terminate instances. And since they were done using these scripts, it became much more reliable, reducing the scope of human error. This was a really informative project as it provided so many insights on how companies manage different kinds of data efficiently, and make informed choices on the backend type/design. 
