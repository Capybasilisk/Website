---
title: "Web Scraping: Building A Large Science Fiction And Fantasy Dataset"
date: 2020-04-02T00:09:04Z
---


As part of my ongoing quest to learn AI, I recently joined Kaggle, the Google-owned machine learning competition site. While I'm nowhere near ready to enter the contests hosted on there, I thought I might contribute an original dataset for other coders to use.

The question was, what kind of data to gather and compile? I knew I didn't want to make another hotel bookings dataset as those have already been done to death. I thought of doing something related to trilobites, or exoplanets, but ultimately decided to start simpler.

The Internet Speculative Fiction Database is a project that aims to catalog every published literary work in the science fiction and fantasy genres. From what I can tell, they've basically succeeded in that task. As someone drawn more to the possible than the actual, I've been a science fiction fan all my life, and making a Kaggle dataset from ISFDB data seemed like a great idea.

Unfortunately, ISFDB doesnt provide a convenient API to get the data, so I knew I'd have to write a script to scrape what I needed from the website. Seeing as the site has an unfamiliar cataloging scheme, in addition to the fact that it's been gradually added to since 1995, I knew this wasn't going to be a pleasant process. Indeed, the final dataset I ended up with is fairly basic. Just book title, author, publication date, and type (e.g. novel, short story, anthology etc.)

Several entries had interesting possible data categories such as thematic tags (e.g. lost colony), but as only some of the titles had these, including them in the final dataset would have led to a ton of missing values.

&nbsp;

Below is the script I wrote to scrape the site and build the dataset. After completion, you'll have a CSV file with the metadata of about 125,000 books. It takes several hours to run on a basic Linux server: 

&nbsp;


```Python

import requests
import bs4
import clevercsv
import re




def catalog():
    
    """Scrapes metadata of science fiction and fantasy literary works 
    from The Internet Speculative Fiction Database and stores them in 
    a CSV file. If site structure changes significantly, code may stop 
    functioning properly.
    """

    card = 0
    
    for entry in range(199000):
        
        try:
        
            card += 1

            
            page = requests.get(
                f"http://www.isfdb.org/cgi-bin/title.cgi?{card}")
            parsed = bs4.BeautifulSoup(
                page.content, 
                "html.parser")
            content = parsed.find(
                id = "content").text.split("\n")
            content_string = "##".join(content)

            
            content_title = re.search(
                "Title:\s+[^#]+", content_string).group(0)
            title = content_title.split(": ")[1] 

            content_author = re.search(
                "Author:##[^#]+", content_string).group(0)
            author = content_author.split("##")[1]

            content_date = re.search(
                "Date:\s+\d+\-\d+\-\d+", content_string).group(0)
            pubdate = content_date.split("  ")[1]

            content_type = re.search(
                "Type:\s+[^#]+", content_string).group(0)
            booktype = content_type.split(": ")[1]


            accepted_booktype = [
                "NOVEL",  
                "SHORTFICTION", 
                "COLLECTION", 
                "ANTHOLOGY", 
                "OMNIBUS", 
                "POEM",
                "NONFICTION", 
                "ESSAY"]


            with open(
                "SFF_Dataset.csv", "a", encoding="UTF-8") as sff:
                dataset = clevercsv.writer(sff)

                if sff.tell() == 0:
                    
                    dataset.writerow(
                        ["Title",
                        "Author", 
                        "Publication Date", 
                        "Type"])

                if booktype in accepted_booktype:

                    dataset.writerow(
                        [title, 
                        author, 
                        pubdate,
                        booktype])


        except:

            print(
                f"Skipping entry no. {card}: Empty article.", "\n" *4)

            continue  



if __name__ == "__main__":

    catalog()


```


&nbsp;

Below is how the dataset looks in Calc from LibreOffice:

&nbsp;

![Dataset](/static/sff_dataset.png)

&nbsp;

[Full code repo available on Github](https://github.com/Capybasilisk/SFF-Scraper)

&nbsp;

Update: [I've coded a Reddit bot that uses the dataset](https://capybasilisk.com/posts/2020/04/speculative-fiction-bot/). It watches for comments containing mentions of books in the dataset, than searches YouTube for audiobook versions of the mentioned book.

&nbsp;

Related posts:

[Speculative Fiction Bot](https://capybasilisk.com/posts/2020/04/speculative-fiction-bot/)

[EXP-RTL: Exponential Retaliation In Iterated Prisoner's Dilemma Games](https://capybasilisk.com/posts/2020/04/exp-rtl-exponential-retaliation-in-iterated-prisoners-dilemma-games/)

[Square Neighborhood Algorithm: Balancing Exploration And Exploitation In Optimization](https://capybasilisk.com/posts/2020/06/square-neighborhood-algorithm-balancing-exploration-and-exploitation-in-optimization/)

[Interleaved Neighborhood Algorithm: Fully Exploratory Optimization](https://capybasilisk.com/posts/2020/06/interleaved-neighborhood-algorithm-fully-exploratory-optimization/)

&nbsp;

[About Me](https://capybasilisk.com/about/)
