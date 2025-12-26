!!! example "Topics according to exam"
    For AWS Developer exam: the main thing to know is:

    - Not many questions on the developer one mainly if you had to pick a SQL database

    For AWS Solution Architect exam: the main thing to know is:
    
    - Aurora vs RDS 
    - Multi-az 

**SQL Database**

**<span style="color: #0057FF;">Advantages:</span>**

* Continuous backups \- restore to specific time   
* Read replicas  
* Scaling vertical-horizontal  
* Multi AZ  
* Storage backed by EBS

**<span style="color: #0057FF;">Storage Autoscaling:</span>**

* RDS scales automatically if free storage like < 10% or low for 5 min  
* Have to set Max storage

**Read Replica** mainly for read :

* Up to 15   
* cross AZ or cross Region  
* Replication *ASYNC* **eventually consistent**  
* Replica can be promoted to **their own master DB manually**  
* Read replica *free same region \- cross region $$*

**<span style="color: #0057FF;">RDS Multi-AZ Acid compliant \- Only for FAILOVER:</span>**

* **SYNC Replication**  
* for high availability \- NOT scaling  
* Cost more then RDS  
* Automatic failover- **the standby gets promoted automatically failure**  
* Also **no need to stop the DB** takes internal snapshot and move to the other  
* **60-120s**  

**<span style="color: #0057FF;">Aurora:</span>**


* Mainly for high availability  
* **Read heavy** workload  
* Faster automatic backup  
* Serverless option automatic scaling  
* Doesn't support SQL Server , Oracle , MariaDB  
* **Replication aurora is faster <10ms**  
* **Read replica are not acid as per**   
* Automatic Failover **<30s** compare to standards RDS 1-2 min  
* 20% more expensive RDS

**<span style="color: #0057FF;">Custom endpoint:</span>**

You can use specific endpoints to do queries in one or analytical queries in another 

**<span style="color: #0057FF;">Aurora Global database:</span>**
   
You can promote another region for disaster recovery <1 minute RTO  
**Exam loves this**: Aurora replicas support automatic failover (unlike RDS read replicas)  
Cross region replication takes <1 second 

**Aurora ML:**  
Integration with ML   
Amazon <span style="color: #FF1493; font-size: 16px;">SageMaker</span> (any ML model)  
Amazon   (for sentiment analysis ) 

**Babelfish:**  
Allows Microsoft SQL to talk Aurora PostgreSQL

**Aurora Multi-Master**:Multiple write nodes (all are primary)

Database Cloning

Copy-on-write- create a new DB from an existing one \- for staging

* Lazy Loading: all the read data is cached, data can become stale in cache  
* Write Through: Adds or update data in the cache when written to a DB (no stale data)  
* Session Store: store temp data in a cache TTL

<span style="color: #FF1493; font-size: 16px;">Neputure</span> → graph database \-\> like post comments  
<span style="color: #FF1493; font-size: 16px;">Redshift</span> → Analytics data warehouse  
<span style="color: #FF1493; font-size: 16px;">MongoDB</span> → DocumentDB

**<span style="color: #0057FF;">ElasticCache:</span>**
 
*Involve heavy application code changes*

* Read-heavy application workloads (such as social networking, gaming, media sharing, leaderboard, and Q&A portals)   
* Or compute-intensive workloads (such as a recommendation engine) by allowing you to store the objects that are often read in the cache.

* Associate to reduce the replication lag as much as possible with minimal changes to the application code or the effort required to manage the underlying resources.

Changing from **RDS to Aurora its actually not that bad** just change of endpoint   
Especially because Aurora has fast retrieval 10ms in comparison of RDS that need 1s min  
You could add caching but it will be more architecture 