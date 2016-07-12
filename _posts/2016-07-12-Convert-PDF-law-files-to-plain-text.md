---
layout:     post
title:      Convert PDF law files to plain text using pdftotext
date:       2016-07-12 13:15:00
summary:    This is a short tutorial on converting pdf files to plain text
categories: legislation code
--- 

In a [previous post](http://www.openlaws.gr/legislation/code/2015/06/22/scrapying-greek-laws/), we briefly presented a tutorial for downloading greek legislative texts using Scrapy. Having acquired and organized all law documents based on their year of publication, it is important to transform them in plain text format in order to start their analysis using natural language processing techniques. PDF is a format that is not easily processable and in order to perform the task of transformation, we introduce [pdftotext](http://linux.die.net/man/1/pdftotext), an open source utility for Linux, appropriate for such operations.

Below we present a short description of the code which implements the required transformation. The first step is to define the source (where the files exist) and the destination folder (where the plain texts will be stored). For example:

{% highlight python %}
    src = '/path/to/laws/folder/'
    text_dest = '/path/to/destination/'
{% endhighlight %}

Next, we traverse all the files one by one in the source path and we call [subprocess](https://docs.python.org/2/library/subprocess.html), a module that allows us to spawn new processes (it provides the same functionality as if someone was writing commands in the terminal). Here is a short description of the code:

{% highlight python %}
    for root, dirs, files in os.walk(src, topdown=False):
        for name in files:
            subprocess.call(["pdftotext", "-raw", os.path.join(root,name)])
{% endhighlight %}

Although the *-raw* is a parameter that is no longer recommended according to the tutorial, it is an important option in our transformation as our analysis showed that it keeps text in content stream order (Greek legislative texts are written in two columns). In the image below we present an example of legal content in a PDF file (left part) and its transformation in plain text (right part). 

![Example of PDF transformation to plain text]({{ site.baseurl }}/images/pdftotext.jpg)

Our further analysis in the generated plain texts also designated some problems that could have a detrimental effect in further analysis and processing, some of which are:

1. __Absence or wrong text encoding.__ Some PDF files lack information about text encoding and as a result the generated texts have wrong characters. This problem occur mainly in laws published between 2000 and 2005. An extra step of character matching is needed in order to acquire the proper legal text. There are also PDF files that did not provide encoding at all (e.g. Law 4028/2011) and no character matching could be beneficial.
2. __Scanned Documents.__ Unfortunately, legislative texts before 2004 are published as scanned documents or text in other languages (e.g. laws approving international agreements) and as a result an OCR (Optical Character Recognition) processing is necessary for their transformation. These images are often of low quality, sometimes with skew positioning and contrast variation. Such a process usually introduce errors in the produced text files and require other techniques such as training other tools (e.g. tessaract) which is out of the scope of this post. 
3. __Table data.__ Tables in PDF are of special concern due to the fact that there is no information about them in a format like PDF. PDF files provides no information about where headers, rows and columns begin or end, and how data is organized (as it happens for example with HTML pages through the use of tags). Table transformation revealed that data was scattered, in an unorganized way or even missing from the document, a major effect as correct and up-to-date legal data is important for further content analysis techniques. 
