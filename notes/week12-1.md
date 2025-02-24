---
title: Introduction to Topic Modeling
layout: post
output: 
  md_document:
    preserve_yaml: true
---

*Short description*

1.  Topic modeling is a type of dimensionality reduction method that is
    especially useful for high-dimensional count matrices. For example,
    it can be applied to,

         * Text data analysis, where each row is a document and each column is a word.
         The ijth entry contains the count of the jth word in the ith document.
         * Gene expression analysis, where each row is a biological sample and each
         column is a gene. The $ij^{th}$ entry measures the amount of gene j expressed in
         sample i.

    This week, we’ll specifically focus on the application to text data,
    since otherwise, we’ve covered relatively few visualization
    techniques that can be applied to this (very common) type of data.
    For the rest of these lectures, we’ll refer to samples as documents
    and features as words, even though these methods can be used more
    generally.

2.  These models are useful to know about because they provide a
    compromise between clustering and PCA.

         * In clustering, each document would have to be assigned to a single topic. In
         contrast, topic models allow each document to partially belong to several
         topics simultaneously. In this sense, they are more suitable when data do not
         belong to distinct, clearly-defined clusters.
         * PCA is also appropriate when the data vary continuously, but it does not
         provide any notion of clusters. In contrast, topic models estimate K topics,
         which are analogous to a cluster centroids (though documents are typically a
         mix of several centroids).

3.  Without going into mathematical detail, topic models perform
    dimensionality reduction by supposing, (i) each document is a
    mixture of topics and (ii) each topic is a mixture of words. For
    (i), consider modeling a collection of newspaper articles. A set of
    articles might belong primarily to the “politics” topic, and others
    to the “business” topic. Articles that describe a monetary policy in
    the federal reserve might belong partially to both the “politics”
    and the “business” topic. For (ii), consider the difference in words
    that would appear in politics and business articles. Articles about
    politics might frequently include words like “congress” and “law,”
    but only rarely words like “stock” and “trade.”

4.  A document is a mixture of topics, with more words coming from the
    topics that it is close to. More precisely, a document that is very
    close to a particular topic has a word distribution just like that
    topic. A document that is intermediate between two topics has a word
    distribution that mixes between both topics.

5.  Let’s see how to fit a topic model in R. We will use LDA as
    implemented in the topicmodels package, which expects input to be
    structured as a DocumentTermMatrix, a special type of matrix that
    stores the counts of words (columns) across documents (rows). In
    practice, most of the effort required to fit a topic model goes into
    transforming the raw data into a suitable DocumentTermMatrix.

6.  To illustrate this process, let’s consider the “Great Library Heist”
    example from the reading. We imagine that a thief has taken four
    books — Great Expectations, Twenty Thousand Leagues Under The Sea,
    War of the Worlds, and Pride & Prejudice — and torn all the chapters
    out. We are left with pieces of isolated pieces of text and have to
    determine from which book they are from. The block below downloads
    all the books into an R object.

            titles <- c("Twenty Thousand Leagues under the Sea",
                        "The War of the Worlds",
                        "Pride and Prejudice", 
                        "Great Expectations")
            books <- gutenberg_works(title %in% titles) %>%
              gutenberg_download(meta_fields = "title")
            books

7.  Since we imagine that the word distributions are not equal across
    the books, topic modeling is a reasonable approach for discovering
    the books associated with each chapter. Let’s start by simulating
    the process of tearing the chapters out. We split the raw texts
    anytime the word “Chapter” appears. We will keep track of the book
    names for each chapter, but this information is not passed into the
    topic modeling algorithm.

            by_chapter <- books %>%
              group_by(title) %>%
              mutate(
                chapter = cumsum(str_detect(text, regex("chapter", ignore_case = TRUE)))
              ) %>%
              group_by(title, chapter) %>%
              mutate(n = n()) %>%
              filter(n > 5) %>%
              ungroup() %>%
              unite(document, title, chapter)

8.  As it is, the text data are long character strings, giving actual
    text from the novels. To fit LDA, we only need counts of each word
    within each chapter – the algorithm throws away information related
    to word order. To derive word counts, we first split the raw text
    into separate words using the `unest_tokens` function in the
    tidytext package. Then, we can count the number of times each word
    appeared in each document using count, a shortcut for the usual
    `group_by` and `summarize(n = n())` pattern.

        word_counts <- by_chapter %>%
          unnest_tokens(word, text) %>%
          anti_join(stop_words) %>%
          count(document, word)

9.  These words counts are still not in a format compatible with
    conversion to a DocumentTermMatrix. The issue is that the
    DocumentTermMatrix expects words to be arranged along columns, but
    currently they are stored across rows. The line below converts the
    original “long” word counts into a “wide” DocumentTermMatrix in one
    step. Across these 4 books, we have 65 chapters and a vocabulary of
    size

10. 

<!-- -->

    ```r
        chapters_dtm <- word_counts %>%
          cast_dtm(document, word, n)
    ```

1.  Once the data are in this format, we can use the LDA function to fit
    a topic model. We choose K=4 topics because we expect that each
    topic will match a book. Different hyperparameters can be set using
    the control argument. There are two types of outputs produced by the
    LDA model: the topic word distributions (for each topic, which words
    are common?) and the document-topic memberships (from which topics
    does a document come from?). For visualization, it will be easiest
    to extract these parameters using the tidy function, specifying
    whether we want the topics (beta) or memberships (gamma).

        chapters_lda <- LDA(chapters_dtm, k = 4, control = list(seed = 1234))
        chapters_lda

        topics <- tidy(chapters_lda, matrix = "beta")
            memberships <- tidy(chapters_lda, matrix = "gamma")

2.  This tidy approach is preferable to extracting the parameters
    directly from the fitted model (e.g., using `chapters_lda@gamma`)
    because it ensures the output is a tidy data.frame, rather than a
    matrix. Tidy data.frames are easier to visualize using ggplot2.

            # highest weight words per topic
            topics %>%
              arrange(topic, -beta)

            # topic memberships per document
            memberships %>%
              arrange(document, topic)
