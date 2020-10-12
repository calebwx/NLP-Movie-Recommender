# NLP-Movie-Recommender
A system to recommend movies using only user-input free-form text requests for movies.

# Notebooks:

- 01: Data Collection.  This notebook provides functions built around the PushShift API to easilty pull data and comments from specific subreddits. Some code is designed to work around times that the API does not work well.

- 02: Data Cleaning. This notebook cleans and merges the IMDB databases and filters the list of available titles to the most popular movies in the US region. The IMDB files are too large to be hosted on github, so the "data" folder is not in the repository. However, these files can be found at www.imdb.com/interfaces : movies.tsv is from the file "title.basics.tsv.gz", and movies_2.tsv is from the file "title.akas.tsv.gz".

- 03: Building Database.  This notebook takes requests collected from the 01 notebook and movies filtered by the 02 notebook and builds a database of request : suggestions in formats that can be readily parsed.

- 04: Features and Models.  Although the database is formatted well, it cannot save one-hot enocding or other complex data types. Because of this, all of the feature creation and modeling is in the same notbook. This contains feature creation (TFIDF, LDA), spaCy nlp, neural networks, and some visualization functions to view how the databases look with different features.

# Problem Statement:

Recommender systems for movies usually rely either on user similarity or pre-defined features that a user can select. A more flexible type of recommender could take in a sentence, a paragraph, or even a few key words, and return movie recommendations. This project is an attempt to create such a recommender, using data scraped from public internet conversations about movie suggestions. This could potentially result in a powerful recommender that can find connections between movies that a traditional recommender cannot, and can adapt to meet users' specific requests. The goal is to have a working system that takes a user's query and returns a handful of movie titles by predicting the most likely human responses to the given text query.

# Data:

Data was scraped from www.reddit.com/r/moviesuggestions, where users can make requests for movie suggestions from other users. From this subreddit, 11,329 posts were marked 'Request', providing the desired request-suggestion format for building the database. Movie data was compiled from IMDB's freely available data, and titles from the most popular movies in this database were used to find suggestions among the comments. Some requests did not yeild any suggestions, either because none were given, or none were found. From this data, 8,686 requests were matched with a total of 188,964 non-duplicate suggestions, representing 8,580 movies. The request text and suggested titles form the database used for this project.

The files in the "movies_data" folder are:

- **movies** : contains data on movies: year, title, identifier, and other basic info from IMDB
- **requests_with_suggestions** : request text, suggestion lists, scores
- **saved_comments_list** : only for checking the progress of the code in notebook 01 for problems with API functioniality
- **slow_comments, slow_df** : precursor data to requests_with_suggestions for the same difficulties in notebook 01
- **tfidf_girdsearch_reults** : tfidf vectorizer parameters and the results as found in notebook 04.
- **title_corrections** : List of titles that return a large amount of false positives (words like "it") when matching

# Executive Summary

Submissions and their respective comments were scraped using the PushShift API. The IMDB data contained well over 500,000 titles, so that was filtered down to include only the most popular (votes > 5000) movies in the U.S. region, narrowing the number of films down to less than 12,000. The titles, including alternate titles, were matched to comments using spaCy phrasematcher and regular expressions to pick up on poorly formatted titles. However, mis-spelled or abbreviated titles (which are not listed among the alternate titles) cannot be matched. 8,580 of these movies were matched in at least one comment. Comments were not explored past the top level of comments to avoid matching on unrelated discussion, and top-level comments by bots and request submitters were ignored. Comments with scores less than 1 were also excluded, as that is an indication that the suggestion is not appropriate for the given request.

For natural language processing, the corpus is composed of the request titles and selftext, combined for each submission to create a single document. The data is split into training and testing sets, so that the recommender can be tested on unseen data by treating the testing documents as if they were queries for the system. To evaluate the system, recall and sometimes accuracy (in situations such as Keras' multilabel accuracy score where they are the same metric) are used as the metric of choice when comparing machine to human recommendations. While this metric can inform feature selection and model tuning, a good recommender does not necessarily match human recommendations. This will always be a limitation, and some intuition will have to be applied to evaluate the system alongside more rigorous metrics.

To match text queries to movies, each title is treated as a document. Each title is matched to the request documents (for which that title was recommended by humans) and the title is represented as an aggregate of the vectors of the request documents. The vectors are aggregated by mean, but making better aggregations is a likely avenue for improvements to the recommender system.

As of this update (Oct 12, 2020), the best performing model is a simple cosine distance document comparison with TFIDF vectorization of the training corpus. Simple feedforward or convolutional neural networks have not performed well, likely due to the huge number of titles to predict on a relatively small number of requests.

Latent dirichlet allocation (LDA) with gensim provides another way to vectorize documents for comparison, or can be used for transfer learning, or classifying documents by topic.This will likely improve with better methods of filtering the database prior to ranking documents. This will also benefit from mutliple sets of better stopwords, applied at different steps of the process of creating tokens and n-grams.

# Current State and Future Work

TFIDF vectorization provides useful and interesting results. The system is not likely to recommend popular movies, which is by no means a bad thing. The reason for this is likely because the most popular titles are represented by vectors that more closely resemble the corpus, rather than vectors that are similar to individual queries. Reintroducing repeated suggestions, or weighting suggestions by post score, could help to better shape the vectors and make them more distinct.

Further cleaning and munging on the data will remove false positives (such as common words) that are incorrectly labeled as suggestions. Using a spaCy matcher that finds "Title" + "Year," or more sophisticated mathing strategies, will allow the system to reintroduce titles that have been manually removed.

One possible feature of the system, once in production, is to allow users to select genres to narrow their results - providing both a good filter for the data, and giving users an interesting tool to explore the system.