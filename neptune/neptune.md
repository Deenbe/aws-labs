## Creating a simple recommendation engine with Amazon Neptune
In this lab you will learn the basics of how to use Amazon Neptune in order to create a recommendation system using collaborative filtering.

### How it works

![image](https://d1.awsstatic.com/Products/product-name/diagrams/product-page-diagram_Neptune.85fa1536c4a013c5f291b930985032d693dd8111.png)

-----

## Contents
1. Pricing & Length of Lab
2. Language & Framework Definitions
3. Creating your very first Amazon Neptune Cluster
4. Setting up a Neptune Notebook
5. Interacting with your Neptune Database  
	5.1 Navigating to a Neptune Notebook  
	5.2 Creating a new IPython Notebook  
	5.3 Confirming that you can connect to the Neptune Instance  
6. Graph Definitions
7. Graph Basics  
	7.1 Create a Vertex  
	7.2 Create a Vertex with a property  
	7.3 Show all Verticies  
	7.4 Drop a Vertex  
	7.5 Edges  
8. Create a simple recommendation engine  
	8.1 Creation of the Verticies  
	8.2 Adding more users  
	8.3 Friend Recommendation  
	8.4 Friend Strengths to Improve Recommendations  
9. Where to from here?
10. Extras  
    10.1 Data Import  
    10.2 Backups  
    10.3 High Availability & Failover  
    10.4 Dates  
    10.5 Iterators and next()  
11. Final Steps
12. References
13. Author & Feedback

----

## 1. Pricing & Length of Lab

Before you begin on this lab, be aware that the cost to run a Neptune cluster may be prohibitive to some users.

https://aws.amazon.com/neptune/pricing/

The provided CloudFormation script uses by default a db.r4.xlarge instance, which as of 03-APR-2020 is $0.84 per hour. 

This lab aims to take about 10-20 minutes, which equates to roughly <$0.44.

**Make sure to delete the CloudFormation Script after you have finished, otherwise you will incur additional costs.**

----

## 2. Language & Framework Definitions

### Gremlin ([Apache TinkerPop™](http://tinkerpop.apache.org)) / [SparQL](https://www.w3.org/TR/sparql11-overview/) (*Pronounced Sparkle*)
Gremlin and SparQL are the two main frameworks commonly used with modelling and interacting graph networks. Gremlin includes multiple language drivers for most programming languages.

SparQL is an RDF query language - semantic query language for databases.

<img src="https://st-summit-2020-resources.s3-ap-southeast-2.amazonaws.com/public/images/neptune-lab/gremlin-logo.png" height=150>

----

## 3. Creating your very first Amazon Neptune Cluster

An Neptune Cluster consists of one or more DB instances. A Neptune DB cluster instance can span multiple Availability Zones, which each AZ having a copy of the DB cluster data. Two types of DB instances make up a Neptune DB Cluster:

- Primary DB Cluster
    - A primary DB Cluster instance supports both write and read requests, and performs all the data modifications to the cluster volume. Each Neptune DB cluster has one primary DB instance
- Neptune replica
    - A replica instance connects to the same storage volume as a primary DB instance, but only supports read operations. Each Neptune DB cluster a number of replicas, in addition to the primary DB Instance. [Please refer to the official documentation for up-to-date supported replica numbers](https://docs.aws.amazon.com/neptune/latest/userguide/feature-overview-db-clusters.html).
    - It is recommended to at least have one replica in a different Availability Zone to support automatic fail-over in the event of an outage.
    - Fail-over priority can also be specified.

You may use the provided [CloudFormation script](https://docs.aws.amazon.com/neptune/latest/userguide/get-started-create-cluster.html) to create a Neptune Cluster in your AWS account.

*Please check that you have the appropriate prerequisites, otherwise the CloudFormation script will fail*

**Note: The above CloudFormation script requires you to have an existing SSH-keypair on your account**

The above CloudFormation script automatically generates a Neptune Cluster with 1 instance. The CloudFormation script also generates all required networking, IAM policy stacks and route tables to create Neptune resources within a different VPC. However, as per recommendations, please test on a different account with appropriate backups and review in detail before using within any production environment.

**Note: For Sydney Summit, we will have provisioned the accounts with an existing SSH-keypair already to save time.**

----

## 4. Setting up a Neptune Notebook

Interacting with the Neptune cluster can be performed through the Neptune workbench. The workbench uses Jupyter notebooks hosted by Amazon SageMaker.

[For more information, view the official documentation here](https://docs.aws.amazon.com/neptune/latest/userguide/notebooks.html)

Workbench resources are billed under Amazon SageMaker, separately from Neptune.

**Important**

In order to use the workbench, the security group that you attach in the VPC where Neptune is running must have an additional rule that allows inbound connections from itself - otherwise you will not be able to connect to the Neptune cluster.


**Note: For Sydney Summit, we will have provisioned the accounts with a Neptune Notebook to save time.**

----

[Please refer to the documentation for IAM Roles and IAM Policies required for the Notebooks](https://docs.aws.amazon.com/neptune/latest/userguide/notebooks.html)

## 5. Interacting with your Neptune Database

Interactions with the Neptune Database will primarily be managed using Neptune Notebooks and Gremlin Syntax. An alternative language is SparQL, however Gremlin syntax will be shown in this document.

### 5.1. Navigating to the Neptune Notebook

5.1.1 Start by navigating to the Amazon Neptune service by using the search field, or the product list on your AWS Web Console.

5.1.2. Within the Neptune service, click on the Sidebar to expand the menu.

![NeptuneSidebar](https://st-summit-2020-resources.s3-ap-southeast-2.amazonaws.com/public/images/neptune-lab/neptune_sidebar.png)

5.1.3. Click on the **Notebooks** option, and click on the **Neptune Notebook** that should have been created for you.

5.1.4. Click on **Open notebook** on the Jupyter notebook summary page. A new tab should appear.

### 5.2 Creating a new IPython Notebook (.ipynb) file

As we have successfully navigated to the Jupyter notebook, from the Amazon Neptune web console page. We will now create a new Jupyter Notebook, and run some queries against the Neptune cluster.

5.2.1. Clicking on the New dropdown, and select the **Python 3** option

![image](https://st-summit-2020-resources.s3-ap-southeast-2.amazonaws.com/public/images/neptune-lab/neptune-juypter-setup-01.png)

5.2.2. A new tab should appear with the document ready for input.

### 5.3 Confirming that you can connect to the Neptune Instance

In the first cell, type in `%status` and hit **run**.

![image](https://st-summit-2020-resources.s3-ap-southeast-2.amazonaws.com/public/images/neptune-lab/neptune_status.png)

If everything goes according to plan, you should see an output similar to:

```
{
    "status": "healthy",
    "startTime": "Wed Feb 12 07:21:17 UTC 2020",
    "dbEngineVersion": "1.0.2.1.R4",
    "role": "writer",
    "gremlin": {
        "version": "tinkerpop-3.4.1"
    },
    "sparql": {
        "version": "sparql-1.1"
    },
    "labMode": {
        "ObjectIndex": "disabled",
        "Streams": "disabled",
        "ReadWriteConflictDetection": "enabled"
    }
}
```

----

## 6. Graph Definitions

Before we really get stuck in, we need to go through some definitions which we will refer to during the rest of the lab.

### Vertex / Verticies

A vertex, or vertice, is a denotation representing a discrete object, such as a person, place or event.

*(Personally, i like to refer to a Vertex as a Node)*

### Edge

An edge denotes a relationship between verticies, such as relationships, likes, or having been to a place, etc.

### Properties
A vertex or edge may include properties, which express non-relational information. Examples include but are not limited to:

- A person vertex may include user and age properties
- A Edge may include timestamp and weight

![image](https://st-summit-2020-resources.s3-ap-southeast-2.amazonaws.com/public/images/neptune-lab/5_neptune.png)

----

## 7. Graph Basics

Now that we have connected to our Neptune Cluster, and learnt the basic terminologies on verticies and edges, we can start pushing data into the Graph Database.

We need to prefix each *cell* with the `%%gremlin` syntax so the processor understands that the next syntax is a Gremlin query.

Start by clearing all verticies
```
%%gremlin

g.V.drop().iterate() # Drops all nodes
```

and check that we don't have any verticies

```
%%gremlin

g.V()
```

### 7.1 Create a Vertex

```
%%gremlin

g.addV('person')
```

When adding a Vertex of type 'person', the graph database will automatically generate a unique identifier for that given vertex.

You can specify the exact ID that you wish, by defining the property on creation.

```
%%gremlin

g.addV('person').property(id, "bob")
```

The properties `id` and `label` are reserved attributes for verticies and edges, and therefore are not specified with string denotation.

### 7.2 Create a Vertex with a property
```
%%gremlin

g.addV('person').property('name', 'bob')
```

This syntax allows you to create a vertex, and give it a custom property key and value. 

You can also add multiple properties using the following syntax.

```
%%gremlin

g.addV('person')
    .property('age', 17)
    .property('lname', 'jones')
```

### 7.3 Show all Verticies - the V is case sensitive!
```
%%gremlin

g.V() // V is upper case!
```

You can also show all Verticies and their values, by using the `valueMap` syntax.
```
%%gremlin

g.V().valueMap()
```

### 7.4 Drop a specific Vertex

You may need to drop a specific vertex, and can utilise query commands such as `has` to drop. 

```
%%gremlin

g.V('bob').drop()
```

```
%%gremlin

g.V().has('name', 'dan').drop()
```

```
%%gremlin

g.V().hasLabel('person')
```

### 7.5 Edges

This example creates an edge between two verticies
```
%%gremlin

g.V('1').addE('knows').to(g.V('2')).property('likes', 0.6)
```
Breakdown explanation:

1. We select a vertex using `g.V('1')`
2. We add an Edge using `.addE('knows')`
3. We define which node it is, using a selector `.to(g.V('2'))`
4. We define a property of `'likes'` with a value

----

## 8. Create a simple social recommendation engine

We are now going to attempt to create a simple social recommendation engine, which follows a couple of rules.

1. We have a number of users who are friends with each other
2. Friends of friends, may be recommended
3. For simplicity, we will start with a simple rule that *if number of mutual friends is > 2, recommend the friend*

### 8.1 Creation of the Verticies

The following code will generate a Graph Database with the following verticies, and edges.

<img src="https://st-summit-2020-resources.s3-ap-southeast-2.amazonaws.com/public/images/neptune-lab/6.1_simple_users.png" height=250 />

```
%%gremlin

g.addV('person').property(id, "bob")
 .addV('person').property(id, "jess")
 .V("bob").addE('FRIEND').to(g.V("jess"))
```

We can now check the edges by using "Bob" as a reference vertex, and viewing all `outbound edges`

```
%%gremlin

g.V("bob").outE()
```

```
Total Results: 1
1	e[b8b82641-08fd-dea8-5d35-0cbf5c2393a7][bob-FRIEND->jess]
```

### 8.2 Adding more users

We will expand upon our graph network by adding another user, and making them Jess's friend.

<img src="https://st-summit-2020-resources.s3-ap-southeast-2.amazonaws.com/public/images/neptune-lab/62_friend.png" height=300 />

```
%%gremlin

g.addV("person").property(id, "sarah")
 .addV("person").property(id, "charlotte")
 .V("jess").addE("FRIEND").to(g.V("sarah"))
 .V("jess").addE("FRIEND").to(g.V("charlotte"))

Total Results: 1
1	e[ceb82646-6ed7-1553-22e2-812b2172753d][jess-FRIEND->charlotte]
```

### 8.3 Friend Recommendation

We can now create a really simple friend recommendation.

```
%%gremlin

g.V("bob").out("FRIEND").out("FRIEND")

Total Results: 2
1	v[sarah]
2	v[charlotte]
```

What is happening here is that we are `traversing` from `Bob`, and navigating outwards to other Verticies that are `FRIENDS`. We are then traversing from those `Verticies`, out to other `Friends`.

<img src="https://st-summit-2020-resources.s3-ap-southeast-2.amazonaws.com/public/images/neptune-lab/63_traversal.png" height=300 />

### 8.4 Friend Strengths to Improve Recommendations

What if we wanted to improve our friend strength recommendations based on strengths - instead of just a simple friend value?

Well we can utilise a `strength property` to denote whether the traversal applies, and only traverse if the strength is greater than 1.


### 8.4.1 Start by clearing the graph database
```
%%gremlin

g.V().drop().iterate()
```

### 8.4.2 Create the following Vertex network
![image](https://st-summit-2020-resources.s3-ap-southeast-2.amazonaws.com/public/images/neptune-lab/7.4.2_neptune.png)

Try and create the above network, you can click below to reveal the code if you get stuck.

<details>
    <summary>Unveil Code</summary>

    %%gremlin

    g.addV('person').property(id, "bob")
    .addV('person').property(id, "jess")
    .addV("person").property(id, "sarah")
    .addV("person").property(id, "charlotte")
    .V("jess").addE("FRIEND").to(g.V("sarah")).property("strength", 0.5)
    .V("jess").addE("FRIEND").to(g.V("charlotte")).property("strength", 2)
    .V("bob").addE('FRIEND').to(g.V("jess")).property("strength", 1)
</details>

----

### 8.4.3 Evaluating paths based strength

We will now find outgoing edges from the "bob" vertex, given that "bob's" friendship is above a certain value.

```
%%gremlin

g.V("bob").outE("FRIEND").has("strength", P.gte(1))
```

The syntax `P` denotes that given an object, evaluate whether the result is `true` or `false`. In this example its "greater-than-or-equal to 1.

*Question: Will the query above traverse through to both vertexes? Or only through one if applied to the network below?*

![image](https://st-summit-2020-resources.s3-ap-southeast-2.amazonaws.com/public/images/neptune-lab/7.4.3_neptune.png)

----

```
%%gremlin

g.V("bob").outE("FRIEND").has("strength", P.gte(1)).otherV()
```

`otherV` = move to another vertex that is not the vertex it came from.

----

### 8.4.4 Build the traversal

We can now build the traversal to do the following:

- Given the start from "Bob", traverse outwards on Edges of type "FRIEND" and has "strength" of greater than 1. Move to another vertex that is not the vertex it came from, and then navigate outwards to edges that are also `FRIEND` and has a `strength of greater than 1`. 

This means that Sarah will not appear as the strength is only `0.5`, while Charlotte is `2`.

Try and write the above statement as a query, you can refer to the code below if you get stuck

<details>
    <summary>Click here to unveil</summary>
    <p>

```
%%gremlin

g.V("bob").outE("FRIEND")           # 1
    .has("strength", P.gte(1))      # 2
    .otherV().outE("FRIEND")        # 3
        .has("strength", P.gte(1))  # 4

e[4ab8264f-b637-ab4e-5001-596ea652439e][jess-FRIEND->charlotte]
```

![image](https://st-summit-2020-resources.s3-ap-southeast-2.amazonaws.com/public/images/neptune-lab/7.4.4_neptune.png)

Visualising this is easier if you break down the traversal process into 4 distinct steps.

1. From the vertex
2. Leaving the vertex via an edge
3. Comparing the edge properties if it fulfils a condition
4. Entering a vertex from an edge

![image](https://st-summit-2020-resources.s3-ap-southeast-2.amazonaws.com/public/images/neptune-lab/7.4.4_2_neptune.png)

For more information on graph traversals, [please refer to the Neo4j documentation.](https://neo4j.com/blog/graph-algorithms-neo4j-15-different-graph-algorithms-and-what-they-do/)
</p>
</details>

----

### 8.4.5 Changing the traversal

Try changing the `gte` value of the queries to see the traversal results change. 

First strength value is > 1, should return 0 results

<details>
    <summary>Click here to unveil</summary>

    %%gremlin

    g.V("bob").outE("FRIEND").has("strength", P.gte(2)).otherV().outE("FRIEND").has("strength", P.gte(0.4))

    Total Results: 0
</details>


Second strength value is > 0.4, should show 2 results
<details>
    <summary>Click here to unveil</summary>

    %%gremlin

    g.V("bob").outE("FRIEND").has("strength", P.gte(1)).otherV().outE("FRIEND").has("strength", P.gte(0.4))

    Total Results: 2
    1	e[08b8264f-b637-6bf2-04b4-de13b7682328][jess-FRIEND->sarah]
    2	e[4ab8264f-b637-ab4e-5001-596ea652439e][jess-FRIEND->charlotte]
</details>

----

## 9. Where to from here?

We have only scratched the surface of what we can accomplish with a Graph Database. We can build even more complex relationships - such as item and music recommendations, while including properties and other rules which change what Verticies are able to be traversed.

[Please refer to the Gremlin recipes link included for more inspiration.](http://tinkerpop.apache.org/docs/current/recipes/#recommendation)

----

## 10. Extras

----

## 10.1 Data Import

Amazon Neptune supports bulk loading of external files directly into the Neptune DB instance. This process supports executing a large number of `INSERT`, `addVertex`, `addEdge`, or other API calls.

The Neptune **Loader** supports both RDF (Resource Description Framework) and Gremlin data, and is designed for large datasets.

[Please refer to the official documentation to use the Neptune Loader](https://docs.aws.amazon.com/neptune/latest/userguide/bulk-load.html)

![Image](https://docs.aws.amazon.com/neptune/latest/userguide/images/load-diagram.png)

----

## 10.2 Backups
Amazon Neptune supports cluster snapshots and restoration.

[Please refer to the official documentation to learn more](https://docs.aws.amazon.com/neptune/latest/userguide/backup-restore.html)

----

## 10.3 High Availability & Failover

Amazon Neptune stores copies of the data in a DB cluster across multiple Availablity Zones in a single AWS Region. You must specify to create Neptune replicas across AZs, and if configured will automatically provision and maintain them synchronously.

If a DB cluster is in a single Availability Zone, you can make it Multi-AZ by adding a Neptune replica to a different Availability Zone.

As of 18-FEB-2020, Neptune supports up to 15 Neptune replicas to process read-only queries.

[Please refer to the official documentation to learn more](https://docs.aws.amazon.com/neptune/latest/userguide/feature-overview-availability.html)

----

## 10.4 Dates

Neptune does not support Java Date. Use the datetime() function instead. datetime() accepts an ISO8061-compliant datetime string.

It supports the following formats:

- YYYY-MM-DD
- YYYY-MM-DDTHH:mm
- YYYY-MM-DDTHH:mm:SS
- YYYY-MM-DDTHH:mm:SSZ.

```
%%gremlin

g.V().property(single, 'lastUpdate', datetime('2018-01-01T00:00:00'))
```

----

## 10.5 Iterators and next()

```
%%gremlin
​
g.V().hasLabel('person').properties('age').drop().iterate()
g.V('1').drop().iterate()
g.V().outE().hasLabel('created').drop()
```
Note

The `.next()` step does not work with `.drop()`. Use `.iterate()` instead.

----
## 11. Final Steps

**Make sure to delete the CloudFormation Script after you have finished this lab, otherwise you will incur additional costs.**

----

## Copy of Neptune Prebuilt (.ipynb) Resources

- [01 - About the Neptune Notebook](https://steven-neptune-notebooks.s3-ap-southeast-2.amazonaws.com/Notebooks/01-Getting-Started/01-About-the-Neptune-Notebook.ipynb)
- [02 - Access Graph with Gremlin](https://steven-neptune-notebooks.s3-ap-southeast-2.amazonaws.com/Notebooks/01-Getting-Started/02-Using-Gremlin-to-Access-the-Graph.ipynb)
- [03 - RDF and SparQL to access the graph](https://steven-neptune-notebooks.s3-ap-southeast-2.amazonaws.com/Notebooks/01-Getting-Started/03-Using-RDF-and-SPARQL-to-Access-the-Graph.ipynb)
- [04 - Social Network Recommendations](https://steven-neptune-notebooks.s3-ap-southeast-2.amazonaws.com/Notebooks/01-Getting-Started/04-Social-Network-Recommendations-with-Gremlin.ipynb)

----

## 12. References

- Official Documentation
    - https://docs.aws.amazon.com/neptune/latest/userguide/get-started-create-cluster.html
        - *A SSH key pair is required to access the EC2 instance, and run the CloudFormation script. Please ensure that you define an existing SSH file, when running the CloudFormation script.*

- Blog Post
    - https://aws.amazon.com/blogs/database/analyze-amazon-neptune-graphs-using-amazon-sagemaker-jupyter-notebooks/

- Best Practices
    - https://docs.aws.amazon.com/neptune/latest/userguide/best-practices.html

- Neptune Developer Resources
    - https://aws.amazon.com/neptune/developer-resources/

- Glue-Neptune
    - https://github.com/awslabs/amazon-neptune-tools/tree/master/glue-neptune

- Kinesis to Neptune
    - https://github.com/aws-samples/amazon-neptune-samples/tree/master/gremlin/stream-2-neptune

- .NET connector
    - https://docs.aws.amazon.com/neptune/latest/userguide/access-graph-gremlin-dotnet.html

- Gremlin Recipes
    - http://tinkerpop.apache.org/docs/current/recipes/#recommendation

- Neo4j Traversal Algorithms
    - https://neo4j.com/blog/graph-algorithms-neo4j-15-different-graph-algorithms-and-what-they-do/

<img src="https://st-summit-2020-resources.s3-ap-southeast-2.amazonaws.com/public/images/neptune-lab/gremlin-lab-coat.png" height=200>

*Special thanks to [Neo4j](https://neo4j.com/), [Apache TinkerPop&trade;](http://tinkerpop.apache.org/), [W3C](https://www.w3.org/) and [Ketrina Yim](https://ketrinayim.tumblr.com/), who is the designer behind Gremlin and his TinkerPop friends.*

----

## 13. Author & Feedback

If you have any feedback, concerns or would like to have a chat, please send me an email.

Steven Tseng (stetseng@amazon.com)

Solutions Architect - Digital Natives MEL/SYD