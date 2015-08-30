---
layout:     post
title:      Scrapying Greek Laws from UltraCl@rity
date:       2015-06-22 00:37:19
summary:    In this post we provide a tutorial about how to downlad all Greek laws from the unofficial UltraCl@rity service, using the popular open source framework Scrapy.
categories: legislation code
---

In this post we present a step by step tutorial describing the procedure we followed to download all PDF documents containing law texts that form Greek legislation.

Laws in Greece are published by the [National Printing Service (NPS)](http://www.et.gr) as PDF files, a format that is, from a technological point of view, unfriendly for their processing. Since 2010 these files are available to download for free. Unfortunately, NPS does not provide any API that would allow software applications to request and grant access to these documents.

Another service that could be used to obtain these documents is [UltraCl@rity](https://yperdiavgeia.gr/), which is an unofficial and voluntarily developed search engine for public sector decisions. This service apart from decisions published in the governmental [Cl@rity](https://diavgeia.gov.gr/) website, contains also the total set of Greek legislation. While it provides an API for querying content published in Cl@rity website, it does not support queries for the total set of Greek laws. 

To deal with this situation, our approach was to implement a web crawler. So, here comes the powerful framework "Scrapy".

###What is Scrapy?
[Scrapy](http://scrapy.org/) is a web crawling framework written in Python. It is a fast and open-source tool for extracting data from websites. Using Scrapy we implemented a crawler that automatically downloads all law PDF files from the UltraCl@rity website. Our preference for this unofficial service instead of the NPS website results from the fact that URLs follow a consistent pattern making it easy to form the required http requests. 
For example:

{% highlight ruby %}
    https://yperdiavgeia.gr/laws/search/year_from:XXXX/year_to:YYYY/teuxos:A
{% endhighlight %}

returns an html page with links to all law documents published between year XXXX and year YYYY. Adding an extra *page* parameter at the end of the URL allows to easily deal with pagination.

On the other hand, the NPS website provides a form (available [here] (http://www.et.gr/index.php/2013-01-28-14-06-23/2013-01-29-08-13-13)) that limits search to the first 200 results, making the overall procedure more complex, plus the fact that pagination is implemented client-side with JavaScript.

###How do we use Scrapy?
To get a feeling of how scrapy works there is an online tutorial plus installation instructions available [here](http://doc.scrapy.org/en/0.24/). In this post we assume that Scrapy is already installed and a new project (named ultraclarity) has been created, following the structure as described in the tutorial. 

Before starting the development of the spider it is necessary to define the data that we want to scrape. This is done by modifying the *items.py* file. We create [Items](http://doc.scrapy.org/en/0.24/topics/items.html) which are described as containers that will be loaded with the scraped data. Inside items.py file a class named *UltraclarityItem* has been automatically created that takes a scrapy item object as a parameter and inside that class we define some atrributes (according to what we need to download from the website) as shown below:
 
{% highlight python %}
  class UltraclarityItem(scrapy.Item):
    title = scrapy.Field()
    url = scrapy.Field()
    desc = scrapy.Field()
    pass
{% endhighlight %}

The above model defines three attributes used to capture the *title* (in our case this will be used as the name of the file that the data will be saved to), the *URL* (where is the item that we want to scrape), and the description referred as *desc* (later it will become clear that this should contain the content of the PDF file). 

Next, we have to declare our Spider. Spiders are user-written classes that must be well defined. With the term well-defined we mean that it is important to instantiate some mandatory fields, for example its unique name etc, create the URLs to download and define through methods how content is parsed to extract items. There are some basic methods to use but scrapy also allows to create or override functions in order to obtain a more customized behaviour. All spiders are set up inside a folder named *spiders*.

In our project we used CrawlSpider which is the most common spider used to crawl regular websites. Inside spiders folder we create a new file named ultraclarityspider.
As we will developing the spider, packages need to be imported to avoid errors when compiling the code. For example to be able to make http requests from scrapy.http we need to import Request. These packages are written in ultraclarityspider and are shown below:

{% highlight python %}
import scrapy
from scrapy.spiders 	import CrawlSpider
from scrapy.selector    import Selector 
from ultraclarity.items import UltraclarityItem
from scrapy.http    	import Request
{% endhighlight %}

We are now ready to begin developing the spider. Firstly, we create a class named *UltraclaritySpider* which subclasses the CrawlSpider and afterwards we specify its mandatory fields: a *name* that we will use to call from the terminal, an optional list named *Allowed_domains* which is a list of domains where the spider will begin to crawl from, when no particular URLs are specified and finally a list of URLs where the Spider will begin to crawl from. The code below illustrates the above description.

{% highlight python %}
class UltraclaritySpider(CrawlSpider):
    name = "ultraclarity"
    allowed_domains = ["yperdiavgeia.gr"]
        
    start_urls = []
       
    for year in range(xxxx, yyyy):
        start_urls.append("http://yperdiavgeia.gr/laws/search/year_from:" + str(year) + "/year_to:" + str(year) + "/teuxos:A"))
{% endhighlight %}

In order to define *start_urls* we need to know which years of publications of laws we intend to search for. So, we use a *for* loop where *xxxx* and *yyyy* is the range of years to work with. 

Inside *UltraclaritySpider* class we also implement three methods to help us scrape data. The first one, *parse* is a basic method, a default callback used by Scrapy to process downloaded responses. 

{% highlight python %}
def parse(self, response):
    sel = Selector(response)
    num_pages = pages(int(sel.xpath('//div[@id="total-results"]/text()').extract()[0]))
    
    for n in range(1, num_pages + 1):
        request = Request(response.url + "/page:" + str(n), callback=self.parse_objects)
        yield request
{% endhighlight %}

The above code shows that we have used a mechanism called [Selector](http://doc.scrapy.org/en/0.24/topics/selectors.html) to select which data we are going to extract. Selectors select certain parts of the HTML document specified either by [XPath](http://www.w3.org/TR/xpath/) or CSS expressions. XPath (which we are using) is a language for selecting nodes in XML and HTML documents. In this method we use selectors to deal with pagination. First, we follow the path to the element that contains the total number of results according to our search, then we use a *pages* function that divides this number with the number of results per page (UltraCl@rity Service by default uses ten documents per page) and returns the number of pages. As mentioned above we can use an extra *page* parameter to construct the urls dynamically. Multiple requests are then processed with this tecnhique and for such requests we implement a callback function named "parse_objects". 

Now that we have created all requests for every page, we need to fill the items that we created in UltraclarityItem class. Spiders are expected to return their scraped data inside *item* objects and by taking advantage of selectors and following the tree structure of every page we can easily spot the information we need. In our case, documents are stored in divs having class *law* or *law alt* so for every path we follow we call an instance of the UltraclarityItem and we fill the item title (following the same pattern to all files e.g *government-gazzette-issue-num\_issue-type\_year-of-publication*) and the URL that points to the PDF file. As this URL points directly to the law document, we just need to perform requests based on these URLs, define a callback function named *parse_urls* and just pass the item as metadata so that we can use it in the callback function:

{% highlight python %}
def parse_objects(self, response):
    sel = Selector(response)
    paths = sel.xpath('//div[@class="law"] | //div[@class="law alt"]')
    
    for paths in paths:
        item = UltraClarityItem()
        title = paths.xpath('a[@class="title"]/text()').extract()
        item['title'] = title[0].split()[2].replace("/","_").replace(",","")+'_'+title[0].split()[3]
        item['url'] = paths.xpath('a[@class="title"]/@href').extract()
        request = Request(item['url'][0], callback = self.parse_urls)
        request.meta['item'] = item
        yield request
{% endhighlight %}

The last method of the class is used to store the file in the item's description.

{% highlight python %}
def parse_urls(self,response):
    item = response.meta['item']
    item['desc'] = response.body
    yield item
{% endhighlight %}

Finally, in the last step we need to store every item that contains information locally to our computer. After an item has been scraped by a spider, it is sent to the item pipeline which processes it through several components that are executed sequentially. Scrapy provides a placeholder file when you create a project in *project_name/project_name/pipelines.py* that one can modify. Our pipeline is a class (*UltraclarityPipeline*) that implements a simple method named *process_item* which stores every item in a file based on the item's title. All we need to do is just open files and store in them the text. For better organisation, we store files in folders based on their year of publication.

{% highlight python %}
from ultraclarity.items import UltraclarityItem
import os

class UltraclarityPipeline(object):
    def process_item(self, item, spider):
        if not os.path.exists('laws/'+item['title'].split('_')[2]+'/'):
            os.makedirs('laws/'+item['title'].split('_')[2]+'/')  
        
        with open('laws/'+item['title'].split('_')[2]+'/'+item['title'], 'w') as f:
            f.write(item['desc'])
            
        return item
{% endhighlight %}

This is it! Just calling our spider's name from the terminal will do the job!
