
EDA for FOIA requests
---------------------

.. code:: python

    %pylab inline

.. parsed-literal::

    Populating the interactive namespace from numpy and matplotlib


.. code:: python

    # Dependencies
    import pandas
    import re
    import random
    
    from gensim import corpora, models, similarities
    from itertools import chain
    from nltk import word_tokenize, clean_html
    from nltk.corpus import stopwords
    from operator import itemgetter
Loading Data
~~~~~~~~~~~~

.. code:: python

    data = pandas.read_csv('foia_online_requests.csv',  error_bad_lines=False, encoding="ISO-8859-1")
    data = data.append(pandas.read_csv('foia_online_requests_2.csv',  error_bad_lines=False, encoding="ISO-8859-1"))
    data = data.drop_duplicates('Tracking Number')
    data = data.dropna(subset = ['Description/Basis for Appeal'])

.. parsed-literal::

    Skipping line 32: expected 8 fields, saw 9
    Skipping line 33: expected 8 fields, saw 9
    Skipping line 42: expected 8 fields, saw 9
    Skipping line 55: expected 8 fields, saw 9
    Skipping line 64: expected 8 fields, saw 17
    Skipping line 67: expected 8 fields, saw 9
    Skipping line 73: expected 8 fields, saw 9
    Skipping line 115: expected 8 fields, saw 9
    Skipping line 216: expected 8 fields, saw 27
    Skipping line 219: expected 8 fields, saw 10
    Skipping line 231: expected 8 fields, saw 10
    Skipping line 263: expected 8 fields, saw 9
    Skipping line 355: expected 8 fields, saw 9
    Skipping line 456: expected 8 fields, saw 12
    Skipping line 458: expected 8 fields, saw 9
    Skipping line 486: expected 8 fields, saw 9
    Skipping line 493: expected 8 fields, saw 9
    Skipping line 650: expected 8 fields, saw 20
    Skipping line 697: expected 8 fields, saw 9
    Skipping line 703: expected 8 fields, saw 12
    Skipping line 710: expected 8 fields, saw 9
    Skipping line 721: expected 8 fields, saw 9
    Skipping line 740: expected 8 fields, saw 9
    Skipping line 755: expected 8 fields, saw 13
    Skipping line 793: expected 8 fields, saw 10
    Skipping line 810: expected 8 fields, saw 9
    Skipping line 813: expected 8 fields, saw 18
    Skipping line 832: expected 8 fields, saw 9
    Skipping line 858: expected 8 fields, saw 10
    Skipping line 878: expected 8 fields, saw 11
    Skipping line 884: expected 8 fields, saw 13
    Skipping line 932: expected 8 fields, saw 19
    Skipping line 967: expected 8 fields, saw 15
    Skipping line 968: expected 8 fields, saw 10
    Skipping line 985: expected 8 fields, saw 9
    Skipping line 1049: expected 8 fields, saw 9
    Skipping line 1050: expected 8 fields, saw 9
    Skipping line 1062: expected 8 fields, saw 9
    Skipping line 1074: expected 8 fields, saw 19
    Skipping line 1080: expected 8 fields, saw 9
    Skipping line 1090: expected 8 fields, saw 9
    Skipping line 1098: expected 8 fields, saw 11
    Skipping line 1115: expected 8 fields, saw 15
    Skipping line 1133: expected 8 fields, saw 9
    Skipping line 1220: expected 8 fields, saw 12
    Skipping line 1262: expected 8 fields, saw 9
    Skipping line 1263: expected 8 fields, saw 13
    Skipping line 1277: expected 8 fields, saw 12
    Skipping line 1306: expected 8 fields, saw 12
    Skipping line 1308: expected 8 fields, saw 10
    Skipping line 1323: expected 8 fields, saw 24
    Skipping line 1361: expected 8 fields, saw 10
    Skipping line 1407: expected 8 fields, saw 9
    Skipping line 1442: expected 8 fields, saw 9
    Skipping line 1472: expected 8 fields, saw 9
    Skipping line 1484: expected 8 fields, saw 11
    Skipping line 1487: expected 8 fields, saw 11
    Skipping line 1503: expected 8 fields, saw 13
    Skipping line 1553: expected 8 fields, saw 29
    Skipping line 1569: expected 8 fields, saw 9
    Skipping line 1603: expected 8 fields, saw 15
    Skipping line 1693: expected 8 fields, saw 15
    Skipping line 1717: expected 8 fields, saw 10
    Skipping line 1762: expected 8 fields, saw 12
    Skipping line 1782: expected 8 fields, saw 9
    Skipping line 1794: expected 8 fields, saw 9
    Skipping line 1814: expected 8 fields, saw 11
    Skipping line 1893: expected 8 fields, saw 10
    Skipping line 1924: expected 8 fields, saw 9
    Skipping line 1959: expected 8 fields, saw 17
    Skipping line 1973: expected 8 fields, saw 9
    Skipping line 2009: expected 8 fields, saw 11
    Skipping line 2048: expected 8 fields, saw 17
    Skipping line 2049: expected 8 fields, saw 15
    Skipping line 2050: expected 8 fields, saw 13
    Skipping line 2083: expected 8 fields, saw 10
    Skipping line 2088: expected 8 fields, saw 9
    Skipping line 2107: expected 8 fields, saw 10
    Skipping line 2134: expected 8 fields, saw 10
    Skipping line 2148: expected 8 fields, saw 10
    Skipping line 2177: expected 8 fields, saw 11
    Skipping line 2191: expected 8 fields, saw 9
    Skipping line 2213: expected 8 fields, saw 14
    Skipping line 2241: expected 8 fields, saw 9
    Skipping line 2282: expected 8 fields, saw 16
    Skipping line 2289: expected 8 fields, saw 13
    Skipping line 2316: expected 8 fields, saw 10
    Skipping line 2320: expected 8 fields, saw 9
    Skipping line 2353: expected 8 fields, saw 10
    Skipping line 2363: expected 8 fields, saw 28
    Skipping line 2365: expected 8 fields, saw 16
    Skipping line 2375: expected 8 fields, saw 10
    Skipping line 2388: expected 8 fields, saw 15
    Skipping line 2390: expected 8 fields, saw 10
    Skipping line 2434: expected 8 fields, saw 10
    Skipping line 2468: expected 8 fields, saw 16
    Skipping line 2473: expected 8 fields, saw 14
    Skipping line 2491: expected 8 fields, saw 12
    Skipping line 2497: expected 8 fields, saw 16
    
    Skipping line 7: expected 8 fields, saw 9
    Skipping line 8: expected 8 fields, saw 18
    Skipping line 57: expected 8 fields, saw 27
    Skipping line 60: expected 8 fields, saw 10
    Skipping line 91: expected 8 fields, saw 9
    Skipping line 136: expected 8 fields, saw 12
    Skipping line 200: expected 8 fields, saw 18
    Skipping line 228: expected 8 fields, saw 15
    Skipping line 245: expected 8 fields, saw 10
    Skipping line 249: expected 8 fields, saw 9
    Skipping line 251: expected 8 fields, saw 9
    Skipping line 299: expected 8 fields, saw 9
    Skipping line 311: expected 8 fields, saw 9
    Skipping line 318: expected 8 fields, saw 12
    Skipping line 328: expected 8 fields, saw 9
    Skipping line 346: expected 8 fields, saw 14
    Skipping line 377: expected 8 fields, saw 11
    Skipping line 401: expected 8 fields, saw 14
    Skipping line 437: expected 8 fields, saw 12
    Skipping line 457: expected 8 fields, saw 9
    Skipping line 499: expected 8 fields, saw 20
    Skipping line 510: expected 8 fields, saw 10
    Skipping line 534: expected 8 fields, saw 13
    Skipping line 554: expected 8 fields, saw 16
    Skipping line 557: expected 8 fields, saw 9
    Skipping line 593: expected 8 fields, saw 9
    Skipping line 739: expected 8 fields, saw 9
    Skipping line 748: expected 8 fields, saw 12
    Skipping line 777: expected 8 fields, saw 9
    Skipping line 786: expected 8 fields, saw 31
    Skipping line 791: expected 8 fields, saw 10
    Skipping line 821: expected 8 fields, saw 9
    Skipping line 836: expected 8 fields, saw 9
    Skipping line 860: expected 8 fields, saw 9
    Skipping line 923: expected 8 fields, saw 10
    Skipping line 943: expected 8 fields, saw 9
    Skipping line 947: expected 8 fields, saw 12
    Skipping line 973: expected 8 fields, saw 9
    Skipping line 985: expected 8 fields, saw 12
    Skipping line 1038: expected 8 fields, saw 9
    Skipping line 1064: expected 8 fields, saw 15
    Skipping line 1123: expected 8 fields, saw 9
    Skipping line 1223: expected 8 fields, saw 9
    Skipping line 1251: expected 8 fields, saw 13
    Skipping line 1261: expected 8 fields, saw 10
    Skipping line 1271: expected 8 fields, saw 10
    Skipping line 1294: expected 8 fields, saw 11
    Skipping line 1306: expected 8 fields, saw 9
    Skipping line 1316: expected 8 fields, saw 9
    Skipping line 1364: expected 8 fields, saw 9
    Skipping line 1379: expected 8 fields, saw 10
    Skipping line 1381: expected 8 fields, saw 10
    Skipping line 1383: expected 8 fields, saw 9
    Skipping line 1385: expected 8 fields, saw 9
    Skipping line 1390: expected 8 fields, saw 9
    Skipping line 1445: expected 8 fields, saw 27
    Skipping line 1456: expected 8 fields, saw 10
    Skipping line 1467: expected 8 fields, saw 11
    Skipping line 1520: expected 8 fields, saw 9
    Skipping line 1523: expected 8 fields, saw 10
    Skipping line 1543: expected 8 fields, saw 10
    Skipping line 1559: expected 8 fields, saw 17
    Skipping line 1561: expected 8 fields, saw 10
    Skipping line 1566: expected 8 fields, saw 9
    Skipping line 1602: expected 8 fields, saw 9
    Skipping line 1605: expected 8 fields, saw 15
    Skipping line 1658: expected 8 fields, saw 9
    Skipping line 1661: expected 8 fields, saw 11
    Skipping line 1731: expected 8 fields, saw 11
    Skipping line 1833: expected 8 fields, saw 12
    Skipping line 1848: expected 8 fields, saw 9
    Skipping line 1918: expected 8 fields, saw 9
    Skipping line 1944: expected 8 fields, saw 17
    Skipping line 1965: expected 8 fields, saw 10
    Skipping line 2039: expected 8 fields, saw 14
    Skipping line 2080: expected 8 fields, saw 12
    Skipping line 2196: expected 8 fields, saw 10
    Skipping line 2198: expected 8 fields, saw 10
    Skipping line 2200: expected 8 fields, saw 9
    Skipping line 2216: expected 8 fields, saw 10
    Skipping line 2228: expected 8 fields, saw 13
    Skipping line 2310: expected 8 fields, saw 37
    Skipping line 2323: expected 8 fields, saw 14
    Skipping line 2346: expected 8 fields, saw 9
    Skipping line 2404: expected 8 fields, saw 11
    Skipping line 2436: expected 8 fields, saw 9
    Skipping line 2460: expected 8 fields, saw 9
    Skipping line 2506: expected 8 fields, saw 12
    Skipping line 2520: expected 8 fields, saw 9
    Skipping line 2546: expected 8 fields, saw 9
    Skipping line 2571: expected 8 fields, saw 9
    Skipping line 2573: expected 8 fields, saw 9
    Skipping line 2609: expected 8 fields, saw 12
    Skipping line 2622: expected 8 fields, saw 16
    Skipping line 2639: expected 8 fields, saw 12
    Skipping line 2657: expected 8 fields, saw 9
    Skipping line 2665: expected 8 fields, saw 36
    Skipping line 2668: expected 8 fields, saw 10
    


Request Lengths
~~~~~~~~~~~~~~~

.. code:: python

    # getting lengths
    docs = [[re.sub("[^a-z]","",word) for word in document.lower().split()]
            for document in data['Description/Basis for Appeal']]
    lengths = []
    for doc in docs:
        lengths.append(len(doc))
    lengths = pandas.Series(lengths)
    
    # subsetting sample
    rows = random.sample(lengths.index, 1000)
    lengths_sample = lengths.ix[rows]
.. code:: python

    # summary stats
    lengths_sample.describe()



.. parsed-literal::

    count    1000.000000
    mean       86.359000
    std        73.981669
    min         2.000000
    25%        29.000000
    50%        65.000000
    75%       126.000000
    max       331.000000
    dtype: float64



.. code:: python

    lengths_sample.hist().set_title('Hist of Request Lengths')



.. parsed-literal::

    <matplotlib.text.Text at 0x118208b90>




.. image:: output_8_1.png


It appears that requests tend to be short. The plurality are below 50
words. Question - is there a relationship between length and response?

Repeat requesters
~~~~~~~~~~~~~~~~~

.. code:: python

    # Getting the values counts of requesters
    requesters = data['Requester'].value_counts()
    requesters.head()



.. parsed-literal::

    Manuel E. Solis        139
    Kristine Savona         89
    Under Agency Review     77
    Connie Marini           53
    Greg Oberlohr           48
    dtype: int64



.. code:: python

    requesters.hist().set_title('Hist of Requester Counts')



.. parsed-literal::

    '\nprint(data[\'Requester Organization\'].describe())\nprint("\n\n\n")\nprint(data[\'Requester Organization\'].isnull().sum())\n'




.. image:: output_12_1.png


.. code:: python

    # Requestor info
    print( "% of single requesters \t" +  str(float(requesters[requesters == 1].shape[0]) / float(data.shape[0]) * 100))
    print("% of requesters without org: " +  str(float(data['Requester Organization'].isnull().sum()) / float(data.shape[0]) * 100))

.. parsed-literal::

    % of single requesters 	42.8610957388
    % of requesters without org: 17.100166021


Top Words
~~~~~~~~~

.. code:: python

    words = ' '.join(data['Description/Basis for Appeal']).lower().split(' ')
    words = [re.sub("[^a-z]","",word) for word in words if word and len(word) > 3]
    words = pandas.Series(words)
    top_words = words.value_counts()
    top_words



.. parsed-literal::

                     7020
    information      3198
    request          2965
    this             2693
    that             1931
    please           1844
    environmental    1821
    site             1707
    records          1700
    would            1423
    with             1398
    your             1363
    documents        1315
    property         1142
    regarding        1141
    ...
    chapeks                    1
    sitesepa                   1
    simeon                     1
    specialistenvironmental    1
    stands                     1
    pristine                   1
    vantage                    1
    experiment                 1
    prncwe                     1
    soughtthe                  1
    espresso                   1
    tomer                      1
    facilitiesfdocuments       1
    suitablefor                1
    swrpr                      1
    Length: 17592, dtype: int64



LDA Model
~~~~~~~~~

.. code:: python

    texts = [[re.sub("[^a-z]","",word) for word in document.lower().split() if len(re.sub("[^a-z]","",word)) > 3] for document in data['Description/Basis for Appeal']]
    dictionary = corpora.Dictionary(texts)
    corpus = [dictionary.doc2bow(text) for text in texts]
.. code:: python

    tfidf = models.TfidfModel(corpus)
    corpus_tfidf = tfidf[corpus]
.. code:: python

    n_topics = 10
    lda = models.LdaModel(corpus_tfidf, id2word=dictionary, num_topics=n_topics)
    
    for i in range(0, n_topics):
        temp = lda.show_topic(i, 10)
        terms = []
        for term in temp:
            terms.append(term[1])
        print "Top 10 terms for topic #" + str(i) + ": "+ ", ".join(terms)
     

.. parsed-literal::

    WARNING:gensim.models.ldamodel:too few updates, training might not converge; consider increasing the number of passes or iterations to improve accuracy


.. parsed-literal::

    Top 10 terms for topic #0: description, agency, review, under, this, request, site, street, property, environmental
    Top 10 terms for topic #1: site, storage, environmental, property, reports, related, information, would, records, following
    Top 10 terms for topic #2: environmental, kansas, information, hazardous, records, site, property, regarding, city, have
    Top 10 terms for topic #3: site, property, landfill, environmental, troy, decree, information, waste, cleanup, with
    Top 10 terms for topic #4: detained, near, around, records, like, would, have, grant, review, environmental
    Top 10 terms for topic #5: description, agency, review, under, this, request, documents, from, copy, requesting
    Top 10 terms for topic #6: records, request, this, foia, that, documents, please, information, with, site
    Top 10 terms for topic #7: photos, fingerprint, detained, taken, around, were, review, nemours, records, have
    Top 10 terms for topic #8: responses, documents, response, supplemental, information, environmental, asbestos, version, correspondence, records
    Top 10 terms for topic #9: your, records, environmental, emissions, information, project, please, properties, need, additional


Examples of topic 9
~~~~~~~~~~~~~~~~~~~

.. code:: python

    topic_dict = {}
    for doc in range(0,1000):
        topic_number =  str(max(lda[corpus[doc]],key=itemgetter(1))[0])
        if topic_dict.get(topic_number):
            topic_dict[topic_number].append(" ".join(texts[doc]))
        else:
            topic_dict[topic_number] = [" ".join(texts[doc])]
    
    i = 0
    for doc in topic_dict["9"]:
        i += 1
        print (doc + "\n\n")
        if i > 6:
            break


.. parsed-literal::

    information regarding corps engineers case corps engineers foia ronnie alleged mechanized land clearing wetlands tract vacant land located northwest intersection sleepy hollow post road montgomery county texas property
    
    
    would like copy customs border protection background investigations report investigation regarding myself
    
    
    whom concern through freedom information requesting following listing rcra corrective action sites corracts regions would like data cdrom email would like data access excel format cannot send access excel format like file comma delimited format information would like region facility name location street address location city location state location zipcode location county number area name corrective action event code original scheduled date scheduled date actual date ncaps ranking code naics code please include documentation needed read data guarantee payment cost cost will exceed please know thank your assistance sincerely connie marini wheelers farms road milford cmariniedrnetcom
    
    
    requesting little more information about national monitoring plan pesticides
    
    
    hello through foia request requesting following region excel format listing underground storage tank site locations indian land information would like site tankid tankstatusdesc dateoftankstatuschange substancedesc overfillinstalled spillinstalled tribe locname locstr city state locphone dateinstalled thank christian ampuero christianareccheckcom
    
    
    hello through foia request requesting following region excel format listing underground storage tank site locations indian land information would like altfacilityid tribe locname latitude longitude locstr tblfacilitycity tblfacilitycounty tblfacilitystate tblfacilityzip facilitydesc tankid substancedesc tankstatusdesc tankmatdesc pipematdesc spillinstalled overfillinstalled leakdetectedorcleanclosure dateinstalled dateclosed name address tblownercity tblownercounty tblownerstate tblownerzip
    
    
    researching mine sites that will require longterm water treatment particularly those that will require treatment years into perpetuity meet water quality standards
    
    


.. code:: python

    # Getting professionals (people who have sumitted more than 10)
    professionals = data.Requester.value_counts()
    professionals = professionals[professionals > 15].index
    data.loc[data.Requester.isin(professionals), 'Professional'] = True
    # Getting length of request
    data['request_length'] = data['Description/Basis for Appeal'].apply(lambda row: len(row.split(" ")))
.. code:: python

    print(data.loc[data.Professional==True,'request_length'].describe())
    data.loc[data.Professional==True,'request_length'].hist().set_title('Hist of Len of Professional Requests')

.. parsed-literal::

    count    674.000000
    mean      75.547478
    std       59.048208
    min        1.000000
    25%       23.000000
    50%       65.000000
    75%      125.750000
    max      316.000000
    Name: request_length, dtype: float64




.. parsed-literal::

    <matplotlib.text.Text at 0x1108db050>




.. image:: output_23_2.png


.. code:: python

    print(data.loc[data.Professional!=True,'request_length'].describe())
    data.loc[data.Professional!=True,'request_length'].hist().set_title('Hist of Len of Non-Professional Requests')

.. parsed-literal::

    count    2940.000000
    mean       86.681293
    std        77.036560
    min         2.000000
    25%        29.000000
    50%        62.000000
    75%       122.000000
    max       724.000000
    Name: request_length, dtype: float64




.. parsed-literal::

    <matplotlib.text.Text at 0x110964f50>




.. image:: output_24_2.png


.. code:: python

    from scipy.stats import ttest_ind
    t_value, p_value = ttest_ind(data.loc[data.Professional!=True,'request_length'], data.loc[data.Professional==True,'request_length'])
    print "T-test P =", p_value

.. parsed-literal::

    T-test P = 0.000433175715683


