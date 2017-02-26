---
layout: post
title:  "NLTK Analysis of Asimov's Foundation Series (part II)"
date:   2017-02-01  20:41:20
categories: blog
---

This is the second part of my previous [post][partI]. I will write about the second method of my 
NLTK analysis on Isaac Asimov's Foundation Series.


# Collocation Frequency
In this method I used the collocations function of the NLTK library to find 30 collocations in each of the books and 
in a similar way as in the first method, summarized the collocations into a list. Vectors of 
normalized collocation frequencies for each book were created. 

First, I made a function that stores the output of the collocations function into a list as it prints it
directly on the screen: 
{% highlight R %} 
...
def collocation_list(self, tokenized_text, num, window_size):
    f = io.StringIO()
    with redirect_stdout(f):
        tokenized_text.collocations(num=num, window_size=window_size)
        collocation = f.getvalue()
        collocation = collocation.replace("\n", " ")
        collocation = collocation.strip()
        return collocation.split("; ")
{% endhighlight %}
Then the book texts were transformed into lists of bigram words and the collocation frequencies 
were counted and vectorized: 
{% highlight R %}
...
def text_bigrams(self, tokenized_text):
    book_bigrams = list(bigrams(tokenized_text))
    bigrams_combined = [element[0]+" "+element[1] for element in book_bigrams]
    return bigrams_combined

def create_vector(self, nltk_books):
    all_book_collocations = []
    one_list_collocations = []
    for book in nltk_books:
        all_book_collocations.append(self.collocation_list(book, 30, 4))
        one_list_collocations = one_list_collocations + self.collocation_list(book, 30, 4)        
    collocation_tf_vector = numpy.zeros((len(nltk_books), len(one_list_collocations)))
    i = 0  
    for book in nltk_books:
        bi = self.text_bigrams(book)
        collocation_tf_vector[i] = [bi.count(word)/len(bi) for word in one_list_collocations]
        i = i + 1
    return collocation_tf_vector
{% endhighlight %}
The vectors were returned to the main code instead of the TFIDF vectors in case of where the collocation 
method was chosen. Then the cosine similarities between the vectors were calculated in the same way. 

Here I printed the nearest three books for each books by this method.
{% highlight R %}
{'Contact':                 [('Foundation and Empire', '0.415'),
                             ('Foundation', '0.371'),
                             ('Forward the Foundation', '0.357')],
 'Forward the Foundation':  [('Foundation', '0.696'),
                             ('Prelude to Foundation', '0.604'),
                             ('Foundation and Empire', '0.56')],
 'Foundation':              [('Forward the Foundation', '0.696'),
                             ('Foundation and Empire', '0.641'),
                             ('Contact', '0.371')],
 'Foundation and Earth':    [("Foundation's Edge", '0.447'),
                             ('Contact', '0.25'),
                             ('Foundation and Empire', '0.231')],
 'Foundation and Empire':   [('Foundation', '0.641'),
                             ('Forward the Foundation', '0.56'),
                             ('Second Foundation', '0.533')],
 "Foundation's Edge":       [('Second Foundation', '0.746'),
                             ('Foundation and Earth', '0.447'),
                             ('Foundation and Empire', '0.297')],
 'I, Robot':                [('Prelude to Foundation', '0.181'),
                             ('Contact', '0.125'),
                             ('Moby Dick', '0.0956')],
 'Moby Dick':               [('Foundation and Empire', '0.277'),
                             ('Contact', '0.229'),
                             ('Forward the Foundation', '0.212')],
 'Prelude to Foundation':   [('Forward the Foundation', '0.604'),
                             ('Foundation', '0.316'),
                             ('I, Robot', '0.181')],
 'Second Foundation':       [("Foundation's Edge", '0.746'),
                             ('Foundation and Empire', '0.533'),
                             ('Forward the Foundation', '0.311')]}
{% endhighlight %}
My curiosity was how collocation frequencies method is comparing to the TFIDF method. 
The results were quite similar: It predicted 10 out of 12 cases of the previous and 
the next book in author's chronological order and all the cases of the timeline. 
It makes sense that the book similarities appear more likely in the books own timeline order rather 
than the author's order to publish as the books  may contain similar vocabulary. 

On the other hand, closely related books does not seem to be as I thought or at least different 
than the TFIDF methods one. Foundation's Edge has a 
high cosine similarity with the Second Foundation (0.746) whereas not very close to the Foundation and 
Earth (0.447). Forward the Foundation is more close to the Foundation (0.696) than the 
Prelude to Foundation (0.604). Well, maybe it is so.

At this point I can not say which of the methods I used were better. But I am satisfied by the results. 

[partI]: http://data.altai.se/jekyll/update/2017/01/04/Foundation_books_I.html
