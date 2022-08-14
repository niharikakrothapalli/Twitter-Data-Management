# Twitter-Data-Management

Introduction/Motivation
Twitter is a social networking service on which users post about any topic, which users can like the tweet, reply to the tweet and also retweet the tweet. This Twitter search application was used for finding the tweets by text, hashtags and also by author. The primary motivation of this project is to understand the usage of different database technologies, learn how to store, retrieve the data from the databases, and caching the data for faster retrieval of data. The dataset corona-out-2,corona-out-3 had been provided by Professor John.  

Dataset
The dataset provided was in the JSON format. There are a total of around 112k documents combined. Each document represents a tweet and contains information about the contents of the tweet, the user who tweeted it, and other tweets related to it.. There are a lot of fields inside this dataset for both the user data and the tweet data. The dataset was edited in Notepad ++  to make sure the whole data is inside square brackets(‘[‘data’]’) and also to put commas after each document to aid with ease of ingestion.

Data Storage and Technologies Used
As the twitter data is very large, and complex in a way. Different database technologies should be used for storing the data, analyzing the data and also for retrieving the data.  For doing all these processes, a MySQL database is used for storing the user data which is contained in the field named ‘user’ in each document. Similarly,  a MongoDB database is used for storing the Tweets data of the dataset. All the above steps are done through jupyter notebook, an open source integrated development environment (IDE) for Python using their corresponding libraries. The diagram for how the tweet data is organized into the two databases can be seen in Figure 3.1

Storing User Data
MySQL which is an open source relational database which performs exceptionally well during the management of the database. MySQL is very popular, globally renowned for being the most secure and reliable database management system. MySQL is used for storing the user data as it is built for extensibility, scalability and high performance to lower the query waiting time.
As MySQL should be performed through Jupyter notebook, mysql.connector python library must be installed for connecting with the MySQL database. MySQL workbench is also downloaded as it enables users to graphically administer MySQL databases and also be able to visualize the database structures.
Certain fields from the User data provided are selected as the rest of the fields are irrelevant for the search application. In the MySQL database, a table, user, is created with fields and datatypes as seen in Figure 4.1

The fields such as Screen_Name, Location, and Description which are in the VARCHAR format, are in the form of “ ‘ ’ ’ ”, while inserting these particular fields into the data it raises an error. To avoid this problem a new function, escape_from_string, is created which replaces certain characters in the string so that it can be inserted (as seen in Figure 4.2)
The data is inserted into the database one document at a time. First, it extracts the user data from the document. Then, it checks if the user is already in the database, and if not, adds it. It also checks whether the text of a tweet starts with ‘RT’, and if yes then it checks the retweeted_status field and inserts the user data from the retweeted_status in the same way the user data is inserted into the database if the user has not already been added.
Having the user data in a separate SQL database optimizes for space because the user information does not have to be embedded into each tweet. In addition, in the future relationships could be added between users based on following or friends if that information is accessible, and also if user information changes it can be changed in one place instead of having to go through every single tweet. Furthermore, it makes user information specific searches much more efficient. However, since the data is separate from the tweet data, it means that in the search application, the MySQL database has to be queried every single time a tweet is displayed with user information, which is a tradeoff.

Storing Tweet Data
MongoDB greatly simplifies tweet storage, search, and recall eliminating the need of a tweet parser. Its dynamic schema makes it best fit for storing tweets as every tweet might not hold all the fields. Pymongo library in Python is used to connect to and communicate with MongoDB. In addition, to interact with the database Compass GUI is used - comes with the MongoDB Standalone edition. 
Firstly, on establishing connection to the database the whole data is loaded as a json file. After looking at the various fields in the twitter data the eight fields in Figure 5.1 are found to be relevant to the search application and are stored into the MongoDB database.
In the original documents, if the tweet is a retweet, the information about the original tweet is embedded in the data for the retweet. However, for this search application, retweets are only used when looking up authors or retweet specific information. Hence, having the tweet data embedded would be using a lot of extra space. Also, in the dataset provided, some tweets are only contained as the original tweet of a retweet, without being its own document.  Thus,  retweets are handled using a simple algorithm that can be found in Figure 5.2. While pushing the data into MongoDB, retweets are identified by checking if the text of the tweets is starting with ‘RT’. Upon checking for the retweet id in the database, if it is found then the retweet count of the original tweet is incremented by 1, otherwise the tweet is added.
An example of what a document for a tweet looks like can be found in Figure 5.3. Note that the Retweet_ID is 0 if the tweet is an original tweet, a fact which is checked for in the search application. This structure prioritizes space as it includes only the required information and does not use embedded documents, only an array object for the hashtags.
Some indexes were also created to help facilitate the searching process. In particular, a Text Index was created on the text of the tweet, which allows for the usage of MongoDB’s special text search functionality
Search Application Design
The search application itself is built using a Python Flask backend and a basic HTML/CSS frontend. Using these two tools allow for the creation of a web application which has multiple pages and search tools. What proceeds in this section is an explanation of the various features included in the implementation of this application.
First is the actual search function. As can be seen in Figure 6.1, the user can input any search term, such as the word “banana” into the search bar. From there they are given a choice of 3 radio buttons, Text, Author, or Hashtag. Each of these options performs a different kind of query on the Mongo database which makes use of the Mongo Aggregation Pipeline. For Text, the query takes advantage of the Text index to search for the search term in the text of the tweets. Then, the name and screen name is gathered for each tweet from the MySQL database and appended to the search results. For Author, the search application expects a screen name as the search term, so it first looks through the MySQL database to find the User ID associated with the screen name. Once that is found, a query is performed which searches through the Mongo database for any tweets made by that user. Finally, Hashtag is similar to Text, but instead of checking the text of the tweet, it checks the entities within the hashtag array for a match. For each of these search functions, they are also sorted by Relevance, which is also simply the number of retweets that the tweet got. Furthermore, the user can specify a date range from which the tweets will be retrieved. To accommodate caching, the date range is not taken into consideration in the actual query, but instead taken care of after the data has already been retrieved.
Figure 6.2 shows a snapshot of what the results web page looks like. After the search has been completed, the results are sorted by Relevance (retweet count) by default, but the user can select one of the radio buttons and then click Sort to arrange the results by order of post date, be it most recent or least recent. This rearrangement does not requery the database but rather simply orders the data already retrieved, which allows for more efficient sorting. Furthermore, as can be seen in Figure 6.2, the screen name and number of retweets are both hyperlinks. These are the implemented drill-down features.

When the user clicks on the screen name hyperlink, they are directed to a page which displays information about the user as well as all of the tweets and retweets made by the user ordered by most recent (as seen in Figure 6.3). This information is gathered through two queries, one which retrieves the user information from the MySQL database and one which performs the same Author search as before. For the retweets drill-down, clicking it brings the user to a page which lists out all of the users who have retweeted the original tweet, ordered by the time they retweeted (as seen in Figure 6.4). This information is gathered by querying the Mongo database for tweets which have a Retweet ID which matches the ID of the original tweet. 
In addition to searches, sorting, and drill-downs, the search application also has some high level “Top 10” metrics. On the home page there are two hyperlinks which lead you to either a list of the top 10 users by follower count or the top 10 tweets by retweet count (as seen in Figure 6.4). These users and tweets are gathered from the database using simple queries which first sort by the desired metric and then limit the results to the top 10.
					
Cache
Given the amount of data that has been provided for this project, at first it may seem the indices, the Mongo and the MySQL databases by itself are pretty remarkable in terms of query performance, but it would be short-sighted to not look at this whole application in a real-time setting. As the amount of data increases, searches passed to the database may take longer to retrieve relevant data. And it happens with fair certainty that there are a bunch of search terms that are quite popular and are being passed as an input to the search application over and over. In this case, the search application should not query the databases for the same results repeatedly. It is redundant, and a major weakness in application design and implementation. But, with the design discussed up until now, it simply isn’t possible to avoid this issue since recent results aren’t being stored anywhere. So an addition is made to store results for popular search keys in memory to increase the efficiency and speed of the application. And this is done by implementing a local cache. 

Below is a definition and outline of the current caching needs: 
The application is working with both structured and unstructured data, so the cache needs to be able to accommodate any structure/type of data. And since results correspond to a certain search key, it should be capable of storing results in terms of key-value pairs;
The cache essentially needs to be an in-memory database that can be easily integrated with the search application and returns results much faster; and
The service needs to be open source with plenty of documentation and configuration control especially for scaling.

Redis fits these requirements very well, and the design of this cache is discussed below.

In addition to what has been discussed above already with regards to storage of results to popular search terms provided as input, the current caching design additionally stores data for the top ten users and top ten tweets. The application provides a user easy direct-click buttons to look up this information since these searches are widely common. In fact, in a real-world scenario, the results for these are often computed periodically, and caching them increases the efficiency and performance of the application. But, as the data scales out, it is smarter to set this up in a different location. Turns out, Redis provides architects with the option of creating multiple instances holding different databases that can be active in-memory at the same time. So, the design looks like the figure 7.1


And the two instances created to store these caches are shown in figure 7.2
 

The instance ‘r’ contains the ‘Dynamic Cache’ which allows for storage of popular search key/terms and results entered by users that keep updating constantly. Currently, the memory of this database has been limited to 10 MB, exceeding which 20% of the oldest search keys and their results are disposed of. Search keys are in the format 'search key <term> category <class>’ and the values are stored as JSON string objects. Every time a search is made, queries first get redirected to this database, if the key exists, the value is instantaneously returned or else the search query is passed onto the Mongo and MySQL databases and the retrieved results are later pushed into this cache. 


As can be seen in figure 7.3 above, getting is a fairly straightforward lookup and pushing new data is subject to checks for current cache memory. If required, the function clear_20_percent disposes the oldest 20% keys before pushing new data into the database. 

The Periodic Cache is contained in the instance ‘p’ and currently does not have any limitations such as those for memory. The push and get functionalities are very similar to the kind above with the only difference being that there are no memory checks before pushing data.  

Results
Part of the goal of this project is to test the efficiency of the search application, and the efficacy of the caching system, so a timing tool is included in the search application. The time of a search is defined as the time it takes to query the actual database, be it Redis, MySQL, or Mongo (or some combination of them) and return the results. It does not include time to filter out the specific date range because that is separate from the databases and hence not affected by the design.

Search Term
Pre Cache
Post Cache
Number of Results
government (T)
0.37763190269470215
0.0039670467376708984
430
sleep (T)
0.11521148681640625
0.0009980201721191406
39
BarackObama (A)
0.7046241760253906
0.001058340072631836
1
ArvindKejriwal (A)
0.7861380577087402
0.0010013580322265625
5
#corona (H)
0.7830114364624023
0.010030746459960938
1014
#pandemic (H)
0.27484130859375
0.0009834766387939453
28

The table in Figure 8.1 shows the pre-caching time against the post-caching time of various test searches. It is clear that the cache is serving its purpose well, as the post-caching searches are generally at least 100 times faster. It is also worth noting that for text and hashtag searches, the number of results also seems to have a correlation with the time of the search. This is likely due not only to the query having to handle a larger amount of data, but also to the repeated queries that need to be done to retrieve the name and screen name from the MySQL database for every result (which is not needed in the author searches). Another observation is that the pre-caching text based searches are in general faster than the searches of the other two categories. This is because of the Text Index that was added to the Mongo Database which makes text search more efficient, which is also good because it is the search option most likely to be used.

Conclusions
This project explores data ingestion from a JSON file into MySQL and MongoDB, building a search application through Python Flask, and implementing a caching system with Redis. The experimentation shows that using LRU caching based on search term and category is very effective in reducing query times, and also that indexing can be a powerful tool for reducing query times. This project was a great opportunity to learn how to use a plethora of tools and technologies in conjunction with each other and also to see firsthand how the design elements of a database system can affect the efficient usage of said system.

