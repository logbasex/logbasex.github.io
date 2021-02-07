## Chap 1

If you want to provide search for your data, you’ll have to deal with all these issues:
returning relevant search results, returning statistics, and doing all that quickly. This is
where search engines like Elasticsearch come into play because they’re built to meet
exactly those challenges. You can deploy a search engine on top of a relational database to create indices and speed up the SQL queries. Or you can index data from your
NoSQL data store to add search capabilities there. You can do that with Elasticsearch,
and it works well with document-oriented stores like MongoDB because data is represented in Elasticsearch as documents, too. Modern search engines like Elasticsearch
also do a good job of storing your data so you can use it as a NoSQL data store with
powerful search capabilities.
Elasticsearch is open-source and distributed, and it’s built on top of Apache
Lucene,
an open-source search engine library, which allows you to implement search
functionality in your own Java application. Elasticsearch takes this Lucene function
and extends it to make storing, indexing, and searching faster, easier, and, as the
name suggests, elastic. Also, your application doesn’t need to be written in Java to
work with Elasticsearch; you can send data over HTTP in JSON to index, search, and
manage your Elasticsearch cluster.

1.1.1 **Quick search**
- Elasticsearch using **inverted index** (it
  creates a data structure where it keeps a list of where each word belongs, and it's much faster)
-  An inverted index is appropriate for a search engine when it comes to relevance,
   too. Let’s say you search for “Elasticsearch in Action.” and a document contains
   the word “in”—along with a million other documents. At this point, you know that
   “in” is a common word (so-called stop word), and the fact that this document matched doesn’t say much
   about how relevant it is to your search.   In contrast, if it contains “Elasticsearch” along
   with a hundred others, you know you’re getting closer to relevant documents
- The tradeoff for improved search performance and relevancy is that the
     index will take up disk space and adding new blog posts will be slower because you
     have to update the index after adding the data itself. On the upside, tuning can make
     Elasticsearch faster, both when it comes to indexing and searching

1.1.2 **Ensuring relevant results**
- **Relevancy score**: relevancy score is a number assigned to each document that matches your
  search criteria and indicates how relevant the given document is to the criteria. For
  example, if a blog post contains “elections” more times than another, it’s more likely
  to be about elections
  
- **[TF-IDF algorithm](https://viblo.asia/p/thuat-toan-danh-gia-score-trong-elasticsearch-OeVKBY7y5kW)**: By default, the algorithm used to calculate a document’s relevancy score is TF-IDF
  > **TF-IDF** stands for term frequency–inverse document frequency, which are the two factors that influence relevancy score.
  >> **Term frequency** - The more times the words you’re looking for appear in a document, the higher the score.
  > 
  >> **Inverse document frequency** - The weight of each word is higher if the word is
      uncommon across other documents.
  > 
- In addition to choosing an algorithm, Elasticsearch provides many other built-in
  features to influence the relevancy score to suit your needs. For example, you can
  “boost” the score of a particular field, such as the title of a post, to be more important
  than the body

1.1.3 **Searching beyond exact matches**
- Handing typos
    > You can configure Elasticsearch to be tolerant of variations instead of looking for only exact matches. A fuzzy query can be used so a search for “bicycel” will match a blog
post about bicycles.
- Supporing derivatives
    > Elasticsearch understand that
  a blog with “bicycle” in its title should also match queries that mention “bicyclist” or
  “cycling.”
- Statistic
- Providing suggestions
    > Once users start typing, you can help them discover popular searches and results.
  You can use suggestions to predict their searches as they type, as most search
  engines on the web do
  > 

1.2 **Exploring typical Elasticsearch use cases**
