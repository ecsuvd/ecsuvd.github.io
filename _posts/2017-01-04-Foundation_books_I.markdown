---
layout: post
title:  "NLTK Analysis of Asimov's Foundation Series (part I)"
date:   2017-01-04  20:41:20
categories: jekyll update
---

I am recently learning about text processing and analysing. 
Then I have decided to analyze books which I have read recently using python's NLTK package. 
My recent books include Isaac Asimov's Foundation Series, Robot Series, Empire Series 
(I admit that I read quite systematically...), and Carl Sagan's Contact. 
Among these books I chose the 7 books of the Foundation Series: 
Foundation (1951), Foundation and Empire (1952), Second Foundation (1953), 
Foundation's Edge (1982), Foundation and Earth (1986), Prelude to Foundation (1988), 
and Forward the Foundation (1993). 

I listed these books in a chronological order written by the author. 
However the story of the books chronological order is a bit different: 
The last two books are the prequels of Foundation (1951) or the Foundation trilogy 
(first three books in the list above) and are closely connected to each other, 
centered around one main character. Moreover, Foundation's Edge and Foundation and 
Earth are the sequels of the Foundation trilogy and are also closely connected books, 
a continuous story of the same main character. On the other hand, the Foundation trilogy itself is more like 
a collection of stories happening in different timelines of history on different characters. 

I was little bit curious about how much NLTK would reveal to me about the books. 
Would it identify the previous and the next books of a book in author's chronological order 
or stories own timeline order? Would it show connection between closely related stories? 

I was also curious about how it would be if I include different books from the same or different 
genres and authors. Thus I added Asimov's I, Robot as a comparison from the same author, 
and Sagan's Contact as a comparison from the same genre in sci-fi space travelling. 
In addition, one more book: Herman Melville's Moby Dick; or, The Whale from a completely 
different genre was included. So, totally it became ten books. 

For doing the analysis I applied two different methods: term frequency and collocation frequency.  
I was also curious about how different the results would be by different approaches. 

I have got the outcome of my analysis now and the code is uploaded on my 
[github][github-bookmatcher] repository.  
Let me explain how I did it now.  

The code was done using python3. Firstly, the book texts were imported, tokenized and converted into nltk analyzable forms. 
{% highlight R %}
...
    def get_text(self, path):
        fileptr = open(path)
        print("Opening " + path)
        return fileptr.read()

    def text_nltk(self, raw):
        tokens = word_tokenize(raw)
        return nltk.Text(tokens)
...
        book_dir = "books/"
        books = []
        book_titles = []
        for book in os.listdir(book_dir):
            book_titles.append(book)
            books.append(self.get_text(book_dir + book))

        nltk_books = []
        for book in books:
            nltk_books.append(self.text_nltk(book))
{% endhighlight %}
Here, the books were placed in a folder called "books". 

After that, an option to choose the method between term frequency and collocation was given: 
{% highlight R %}
       if calc_method == "collocation":
            mymethod = src.collocation_method.Collocation_Method()
       elif calc_method == "tfidf":
            mymethod = src.tfidf_method.TFIDF_Method()
{% endhighlight %}

In case of term frequency method was chosen how the code works is explained below.

# Term Frequency

In this analysis, first I calculated TFIDF (Term Frequency and Inverse Document Frequency) values for the 100 most
frequent words in each book. Then summarized them into a list and a TFIDF 
vector was created for each book using it. 

To calculate the TFIDF values a basic text cleaning function was created and used to transform the 
texts into lowercase letters, remove non-alphabetical words, stopwords and stemmize. 
This is for preparing the texts for frequency analysis. I did not go for an advanced text cleaning. 
I thought it would be enough for my book analysis. 
{% highlight R %}
...
    def clean_text(self, text, language):
        lowercase_text = [word.lower() for word in text if word.isalpha()]
        stop = set(stopwords.words(language))
        text_without_stopwords = [word for word in lowercase_text if word not in stop]
        porter = PorterStemmer()
        return [porter.stem(w) for w in text_without_stopwords]
...
        cleaned_books = []
        for book in nltk_books:
            cleaned_books.append(self.clean_text(book, "english"))
{% endhighlight %}
After this, a function which returns most frequent words in a text was created. With this function 
lists of frequent words were made.  
{% highlight R %}
...
    def most_freq_words(self, text, number):
        word_freq = FreqDist(text)
        words_counts = word_freq.most_common(number)
        words = [pair[0] for pair in words_counts]
        return words
...
        freq_words = []
        for book in cleaned_books:
            freq_words.append(self.most_freq_words(book, 100))
{% endhighlight %}
From the frequent word lists a list of all words was created:
{% highlight R %}
...
        all_words = []
        for words in freq_words:
            for word in words:
                if word not in all_words:
                    all_words.append(word)
{% endhighlight %}

Then, functions to estimate TFIDF values were created. Formula for the IDF term was taken from the [Data Science for Business][book-url] book. It is equal to scilearns:
`` smooth idf=false.``

{% highlight R %}
...
    def tf(self, word, book):
        return (book.count(word) / float(len(book)))

    def n_docs_containing(self, word, booklist):
        return sum(1 for book in booklist if book.count(word) > 0)

    # 
    def idf(self, word, booklist):
        return (1 + math.log(len(booklist) / self.n_docs_containing(word, booklist)))

    def tfidf(self, word, book, booklist):
        return self.tf(word, book) * self.idf(word, booklist)
{% endhighlight %}
Subsequently, the TFIDF values in the collection list of the most frequent 100 words were calculated 
and a vector was assigned for each book.
{% highlight R %}
        tfidf_vector = numpy.zeros((len(cleaned_books), len(all_words)))
        j = 0
        for book in cleaned_books:
            i = 0
            for word in all_words:
                tfidf_vector[j][i] = self.tfidf(word, book, cleaned_books)
                i = i + 1
            j = j + 1
{% endhighlight %}

These vectors were returned to the main code for further analysis.
{% highlight R %}
vectors = mymethod.create_vector(nltk_books)
{% endhighlight %}
Once the main code got the vectors cosine similarities between them were estimated:
{% highlight R %}
        cosines_matrix = numpy.zeros((len(nltk_books), len(nltk_books)))
        for i in range(len(nltk_books)):
            for j in range(len(nltk_books)):
                cosines_matrix[i][j] = 1 - spatial.distance.cosine(vectors[i], vectors[j])
{% endhighlight %}

Finally, the first three books which have the highest cosine similarity with a book was found. 
{% highlight R %}
...
    def closest_texts(self, vector, text_titles):
        max3 = heapq.nlargest(4, enumerate(vector), key=lambda x: x[1])
        indexes = [x[0] for x in max3[1:4]]
        return [(text_titles[i], vector[i]) for i in indexes]
...
        closest3 = {}
        for i, item in enumerate(cosines_matrix):
            closest3[book_titles[i]] = self.closest_texts(cosines_matrix[i], book_titles)
{% endhighlight %}
That's it. I printed list of the books together with the nearest three books ranking by
their cosine values. Below, I show the outcome of the code:

 {% highlight R %}
{'Contact': [('Foundation', '0.443'),
             ('Forward the Foundation', '0.414'),
             ('Second Foundation', '0.406')],
 'Forward the Foundation': [('Prelude to Foundation', '0.819'),
                            ('Foundation', '0.516'),
                            ('Foundation and Empire', '0.442')],
 'Foundation': [('Foundation and Empire', '0.521'),
                ('Forward the Foundation', '0.516'),
                ('Prelude to Foundation', '0.504')],
 'Foundation and Earth': [("Foundation's Edge", '0.798'),
                          ('Prelude to Foundation', '0.328'),
                          ('Foundation', '0.317')],
 'Foundation and Empire': [('Second Foundation', '0.567'),
                           ('Foundation', '0.521'),
                           ('Forward the Foundation', '0.442')],
 "Foundation's Edge": [('Foundation and Earth', '0.798'),
                       ('Foundation', '0.45'),
                       ('Second Foundation', '0.45')],
 'I, Robot': [('Foundation', '0.331'),
              ('Contact', '0.327'),
              ('Prelude to Foundation', '0.311')],
 'Moby Dick': [('Foundation', '0.352'),
               ('Foundation and Empire', '0.34'),
               ('Contact', '0.337')],
 'Prelude to Foundation': [('Forward the Foundation', '0.819'),
                           ('Foundation', '0.504'),
                           ('Foundation and Empire', '0.432')],
 'Second Foundation': [('Foundation and Empire', '0.567'),
                       ('Foundation', '0.48'),
                       ("Foundation's Edge", '0.45')]}
 {% endhighlight %}

I had several questions when I begin and now I can give answers to myself:
1. Can the list of the nearest three books predict the previous or the next book of a book in either 
author's chronological or in books own timeline order?   
+ It actually could. Following the author's timeline order, 11 out of 12 cases are in the nearest 
book lists whereas all cases are correctly included according to the books timeline. 
2. Can it identify closely related books, direct continuations of a story?  
+ Yes. They appear with quite high cosine similarities. The cosine between 
Foundation's Edge and Foundation and Earth is 0.798 and 0.819 for Prelude to Foundation and Forward 
the Foundation. It is noticebly higher than the cosines between other books. 
Next time I read a sequel or a prequel of a book, I may test by my code to know how close they are. 
3. Does a book by the same author relate close by cosine?  
+ Not really, I, Robot is not as close as Contact to the Foundation books. In fact, Contact has higher cosine similarities with the Foundation books than the I, Robot.  
4. Can the same genre be predicted?   
+ Maybe, space travel sci-fi seems to be recognized by having a higher cosine similarities, 
but not sci-fi in general. I,Robot and Moby Dick have about the same cosine rankings with the books.  

TFIDF did give me satisfiable results. Now, I am curious about the collocation method and whether 
it will be better or not than TFIDF. I will write about it in my next [post][partII].

[github-bookmatcher]: https://github.com/ecsuvd/book-matcher
[partII]: http://data.altai.se/jekyll/update/2017/02/01/Foundation_books_II.html
[book-url]: https://www.goodreads.com/book/show/17912916-data-science-for-business
