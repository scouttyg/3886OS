# Search for the masses

Search is the touchstone of the Internet; without it, the Internet wouldn’t be all that useful. But search hasn’t always been that easy nor affordable to implement. That's where Elasticsearch shines. Elasticsearch adds not only a simple API for adding and searching content, but does it in a distributed manner. With infinitesimal arm grease, you can set up a search cluster that smears your data and resultant queries across a series of nodes. Not only is this resultant architecture fast, but it’s easy to set up and extremely affordable as search nodes can run on commodity hardware. 

Using this repository, I'll show you how to get up and running with Elasticsearch. You'll learn how to create a cluster, how to index and search documents via its RESTful API and how to use a few handy client libraries. You'll be bringing search to the masses in no time!

## So what's the big idea?

When I was in high school, google was just a noun representing an incredibly large number. Today, we sometimes use google as a verb synonymous with online browsing and searching, and we also use it to refer to the eponymous company. It is common to invoke Google as an answer for almost any question: "Just google it!" It follows that application users expect to be able to search the data (files, logs, articles, images, and so on) that an application stores. For software developers, the challenge is to enable search functionality quickly and easily, without losing too much sleep, or cash, to do it.

User queries are becoming more complex and personalized over time, and much of the data required to deliver an appropriate response is inherently unstructured. Where once an `SQL LIKE` clause was good enough, today's usage sometimes calls for sophisticated algorithms. Fortunately, a number of open source and commercial platforms address the need for pluggable search technology, including Lucene, Sphinx, Solr, Amazon's CloudSearch, and Xapian. 

## Enter Elasticsearch

"ElasticSearch is a distributed, RESTful, free/open source search server based on Apache Lucene."


[Elasticsearch](http://www.elasticsearch.org/) is one of a number of open source search platforms. Its service is to offer an additional component (a searchable repository) to an application that already has a database and web front-end. Elasticsearch provides the search algorithms and related infrastructure for your application. You simply upload application data into the Elasticsearch datastore and interact with it via RESTful URLs. You can do this either directly or indirectly via a library like cURL or a client library, like Jest for Java.

The architecture of Elasticsearch is distinctly different from its predecessors in that it is expressly built with horizontal scaling in mind. Unlike some other search platforms, Elasticsearch is designed to be distributed. This feature dovetails quite nicely with the rise of cloud and big data technologies. Elasticsearch is built on top of one of the more stable open source search engines, Lucene, and it works similarly to a schema-less JSON document datastore. Its singular purpose is to enable text-based searching.

> "ElasticSearch is a distributed, RESTful, free/open source search server based on Apache Lucene."

Finally, Elasticsearch is written in Java so it requires you have a JVM, healthy memory and enough space to index documents. Basically think of it as you would a database where I/O is fundamentally important. 

For more information, see:

  * [The Democratization of Search](http://thediscoblog.com/blog/2013/05/14/the-democratization-of-search/)
  * [Java development 2.0: Scalable searching with ElasticSearch](http://www.ibm.com/developerworks/java/library/j-javadev2-24/index.html?ca=drs-)

### Installing Elasticsearch

Elasticsearch is easy to install. And you have a few options ranging from installing the Elasticsearch binary locally on your computer to [running Elasticsearch in AWS](http://thediscoblog.com/blog/2013/05/17/elasticsearch-on-ec2-in-less-than-60-seconds/) to firing up a [Vagrant box already running Elasticsearch](http://thediscoblog.com/blog/2013/11/25/elasticsearch-in-a-box/). 

The simplest installation, which assumes you have access to a terminal (i.e. you are on a Linux or OSX machine), is to download the binary, unzip it, change directories into the install directory and type:

``` 
> bin/elasticsearch -f
```

Boom, you are done!

#### Elasticsearch-in-a-box

Running Elasticsearch locally is easy, but I like to keep my local machine slim and instead rely on VMs. Accordingly, if you are familiar with Vagrant, then you can download [Elasticsearch-in-a-box](https://github.com/aglover/coffer), which is a freely available Vagrant base box. Here are the required steps:

First, add the Elasticsearch-in-a-box box definition:

```
> vagrant box add esinabox https://s3.amazonaws.com/coffers/esinabox.box
```

Then initialize it (you can provide any name you'd like):

```
> vagrant init 'esinabox'
```

Finally, fire it up!

```
> vagrant up
```

Now Elasticsearch is up and running!

For more information, see:

  * [ElasticSearch on EC2 in Less Than 60 Seconds](http://thediscoblog.com/blog/2013/05/17/elasticsearch-on-ec2-in-less-than-60-seconds/)
  * [Elasticsearch in a Box](http://thediscoblog.com/blog/2013/11/25/elasticsearch-in-a-box/)

### Clustering out-of-the-box

[ElasticSearch supports clustering](http://thediscoblog.com/blog/2013/09/03/effortless-elasticsearch-clustering/); that is, you can have a series of distinct ElasticSearch instances work in a coordinated manner without much administrative intervention at all. Clustering ElasticSearch instances (or nodes) provides data redundancy as well as data availability.

Best of all, clustering in ElasticSearch, by default, doesn't require any configuration -- nodes discover each other. To see this in action, it's easiest to run multiple Elasticsearch instances locally. For instance, take the Elasticsearch binary from [elasticsearch.com](http://www.elasticsearch.org/) and copy it into 3 different directories; for example, I've called my directories `node-1`, `node-2`, and `node-3`. 

Open up three terminal windows and start the first one, `node-1`, using the command:

``` 
> bin/elasticsearch -f
```

You should see some output in the terminal window regarding `cluster.service` stating that this instance is the new master. 

Next, fire up the other two nodes (`node-2` & `node-3`) one by one using the same `elasticsearch` command. You should see the two new nodes discover `node-1` in log statements labeled as `detected_master`. 

In fact, to verify your cluster is working correctly, open up a 4th terminal window and type the following command:

```
curl -XGET 'http://localhost:9200/_cluster/nodes?pretty=true'
```

You should see a JSON response listing 3 Elasticsearch nodes in your cluster. 

Just for fun, Ctrl+C one of your nodes (`node-1` is a fun one!) and watch the log outputs -- you should see the cluster notice one node missing and if you happened to kill the master, you'll see a new master elected! Not bad, eh?

For more information, see:

  * [Effortless ElasticSearch Clustering](http://thediscoblog.com/blog/2013/09/03/effortless-elasticsearch-clustering/)

### Elasticsearch terminology

Before I get to the Elasticsearch API, it's helpful to understand a few concepts related to search technology. 

An _index_ is basically a database or schema. A _type_ is essentially a database table. And finally, the process of _indexing_ is inserting documents into an index. In that way, you can think of Elasticsearch as a database; what's more, layer on top of that analogy RESTful concepts and you can quickly see the following relationships: 

  * Indexing (or inserting) documents is done via an HTTP PUT
  * Searching (or selecting) documents is done via an HTTP GET
  * Updating indexes or types or documents is done via HTTP PUTs
  * Deleting is done via HTTP DELETEs

In that way, you can interact with Elasticsearch via HTTP with tools like cURL and in fact, this is how I recommend you first become familiar with Elasticsearch. All client libraries use Elasticsearch's HTTP API under the covers so understanding the base API is rather important.


### The Elasticsearch HTTP API

Sadly, lots of early Internet beer recipes aren't necessarily in an easily digestible format; that is, these recipes are _unstructured_ intermixed lists of directions and ingredients often originally composed in an email or forum post.

So while it's hard to easily put these recipes into traditional data stores (ostensibly for easier searching), they're perfect for Elasticsearch in their current form.

#### Index creation

Accordingly, I am going to create an Elasticsearch index full of beer recipes. To create index, you use an HTTP PUT (because you are updating Elasticsearch to create an index) like so: 

```
curl -XPUT 'http://localhost:9200/beer_recipes/'
```

where `beer_recipes` is the name of the index. 

#### Indexing

Next, I need to add some recipes. In this repository, I've created a few JSON documents that represent beer recipes. I will show you how to add a JSON document manually and then I'll show you a short cut script that'll add all JSON documents in the `recipes` directory.

To manually add on recipe, you use an HTTP PUT and provide a path to the index (named `beer_recipes`); plus, you provide a _type_, which in this case named `beer`. Thus, the URL becomes: 

```
curl -XPOST 'http://localhost:9200/beer_recipes/beer' --data @es-book/beers/wit_1.json
```

The `--data` part posts the contents of the `wit_1.json` document, which looks like this:

```
 { 
   "name": "Todd Enders' Witbier", 
   "style": "wit, Belgian ale, wheat beer", 
   "ingredients": "4.0 lbs Belgian pils malt, 4.0 lbs raw soft red winter wheat, 0.5 lbs rolled oats, 0.75 oz coriander, freshly ground Zest from two table oranges and two lemons, 1.0 oz 3.1% AA Saaz, 3/4 corn sugar for priming, Hoegaarden strain yeast"
 }
```

In the `beers` directory of this repository, there is a script dubbed `post_all_beers.sh` -- you can run that and it'll post all beers in the `recipes` directory one-by-one.

##### Bulk Indexing

Elasticsearch [supports bulk indexing](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/docs-bulk.html), which is much more efficient that adding one document at a time. The format of the document for beer recipes is as follows:

```
{"index": { "_index" : "beer_recipes", "_type" : "beer"} }
{ "name": "AAA Westvleteren 12 (D-180 variation)", "style": "Belgian Ale", "ingredients": "12.5 lbs Dingemann's Belgian Pilsner, 2.00 lbs Dingemann's Belgian Pale, 0.10 lbs Belgian Debittered Black, 0.15 lbs Belgian Special B, 3.00 lbs D-180 Candi Syrup, 8.0 gallons (filtered), Brewers Gold 1.00 oz 60 min, Hersbrucker 1.00 30 min, Styrian Goldings 1.00 15 min, White Labs WL 530 Westmalle strain (2.0 liter starter), 1 cap of servomyces, 2 tsp gypsum"}
{"index": { "_index" : "beer_recipes", "_type" : "beer"} }
{ "name": "Belgian Dubbel", "style": "Belgian ale, Trappist, dubbel", "ingredients": "9.5 lbs pale malt, 4 oz. Crystal malt (20 deg L), 4 oz. Brown malt, 3/4 lbs Sugar, 1 oz. Styrian (5% alpha) (bittering), .3 oz. Hallertauer (bittering), .3 oz Saaz (aroma), 3 oz priming sugar or 2-2.5 volumes of CO2, trappist ale yeast starter"}
 ...
```

You'll find this JSON file in the `bulk` directory -- it's named `bulk.json` and you can index the entire document as follows:

```
curl -XPOST 'http://localhost:9200/beer_recipes/beer/_bulk'  --data-binary @./bulk/bulk.json
```  

#### Searching

Searching is a huge subject; indeed, entire books have been written on it as well as numerous degrees earned on search alone. Nevertheless, I'm going to show you a few searches and I encourage you to dig deeper if you want to learn more. The Elasticsearch documentation is quite extensive. 

Provided you've added, I mean, indexed a few documents either one-by-one or via the bulk API, you can quickly get a count of indexed documents like so:

```
curl -XGET 'http://localhost:9200/beer_recipes/beer/_count' 
```

You should see a JSON response resembling this:

```
{"count":8,"_shards":{"total":5,"successful":5,"failed":0}}
```

Where 8 is the total count of documents in the `beer_recipes` index. 

You can suggest that Elasticsearch format the response JSON nicely by appending `pretty=true` to any RESTful request. 

```
curl -XGET 'http://localhost:9200/beer_recipes/beer/_count?pretty=true' 
```

In this case, the response is:

```
{
  "count" : 8,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  }
}
```

Search queries are just JSON documents. There are a whole host of different _types of queries_; suffice it to say that `term` queries are the easiest to grasp. Via a `term` query, you select a field in the  document and provide a search term. 

If I want to find all beer documents that have the term "wit" in their `style` field then I can format the following query: 

```
{
    "query" : {
        "term" : { "style" : "wit" }
    }
}
```

I can then issue this query like so:

```
curl -XGET 'http://localhost:9200/beer_recipes/beer/_search?pretty=true' -d @./search/search_for_wits.json
```

where the contents of my query reside in the file `search_for_wits.json`. Note the usage of an HTTP GET against the `beer_recipes` index and `beer` type. 

The response should be as follows: 

```
{
  "took" : 44,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 3,
    "max_score" : 0.74075186,
    "hits" : [ {
      "_index" : "beer_recipes",
      "_type" : "beer",
      "_id" : "CDTBeF_QSf2awDKOFjxJSA",
      "_score" : 0.74075186, "_source" :  { "name": "Firefly Witbier", "style": "Wit, Belgian ale, wheat beer", "ingredients": "6lb Belgian Pils, 4.6lb White Summer Wheat, 9.6oz Flaked Oats, 1.0oz 5.2% EK Goldings - 60min, 0.5oz 5.2% Styrian Goldings - 10min, 0.5oz 5.2% Styrian Goldings - @knockout, 0.5oz Curacao Orange Peel, 0.5oz Sweet Orange Peel, 2.0oz Coriander Seed, 5ml Lactic Acid, Wyeast #3944 Belgian White Ale Yeast, Several Fireflies"}
    }, {
      "_index" : "beer_recipes",
      "_type" : "beer",
      "_id" : "gDpseUicTYutB6tZqiRqBw",
      "_score" : 0.4375, "_source" :  { "name": "Todd Enders' Witbier", "style": "wit, Belgian ale, wheat beer", "ingredients": "4.0 lbs Belgian pils malt, 4.0 lbs raw soft red winter wheat, 0.5 lbs rolled oats, 0.75 oz coriander, freshly ground Zest from two table oranges and two lemons, 1.0 oz 3.1% AA Saaz, 3/4 corn sugar for priming, Hoegaarden strain yeast"}
    }, {
      "_index" : "beer_recipes",
      "_type" : "beer",
      "_id" : "P_YsFg3VS--EWxmRl2GYkw",
      "_score" : 0.4375, "_source" :  { "name": "Wit", "style": "wit, Belgian ale, wheat beer", "ingredients": "4 lbs DeWulf-Cosyns 'Pils' malt, 3 lbs brewers' flaked wheat (inauthentic; will try raw wheat nest time), 6 oz rolled oats, 1 oz Saaz hops (3.3% AA), 0.75 oz bitter (Curacao) orange peel quarters (dried), 1 oz sweet orange peel (dried), 0.75 oz coriander (cracked), 0.75 oz anise seed, one small pinch cumin, 0.75 cup corn sugar (priming), 10 ml 88% food-grade lactic acid (at bottling), BrewTek 'Belgian Wheat' yeast"}
    } ]
  }
}
```

As you can see, 3 results were returned and the returned JSON response will provide the matching content via the `_source` field in the `hits` section.

Of course, you can search different fields in a document:

```
{
    "query" : {
        "term" : { "ingredients" : "saaz" }
    }
}
```

In this case, the `ingredients` field is term searched for "saaz". 

You can add an explain clause to any query as well: 

```
{
    "explain": true,
    "query" : {
        "term" : { "ingredients" : "styrian" }
    }
}
```

The resultant JSON document will contain an explanation section that'll describe how a particular result was determined to match. Like a SQL explain clause, interpreting the results requires some specialized search knowledge. 

You can search all fields in a document using the `match` query and the specialized `_all` field matcher. For example, to search everything for the word "pale" you can issue the following query: 

```
{
    "query" : {
        "match" : { "_all" : "pale" }
    }
}
```

More complex searching

```
{"explain": true,
    "query": 
    { "filtered" : 
        { "filter" : 
            {  
            "query" : { "term" : { "ingredients" : "styrian" } } 
            },
            "query" : { "term" : {"ingredients" : "gold"} } ,
            "query" : { "term" : {"style" : "wit"} } 
        } 
     }
   }
```


## Helpful Resources

  * [The Disco Blog](http://thediscoblog.com/)
    * [All Elasticsearch articles](http://thediscoblog.com/blog/categories/elasticsearch/)
  * [Java development 2.0: Scalable searching with ElasticSearch](http://www.ibm.com/developerworks/java/library/j-javadev2-24/index.html?ca=drs-)
  * [elasticsearch.org](http://www.elasticsearch.org/)


## License

<a rel="license" href="http://creativecommons.org/licenses/by-sa/3.0/deed.en_US"><img alt="Creative Commons License" style="border-width:0" src="http://i.creativecommons.org/l/by-sa/3.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/3.0/deed.en_US">Creative Commons Attribution-ShareAlike 3.0 Unported License</a>.    
