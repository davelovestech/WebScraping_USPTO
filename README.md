# Using R to Webscrape the USPTO

### My Process for Writing the Code

I went to the USPTO website and searched for "telomerase inhibitor." Telomerase is heavily activated in cancer, so it's an interesting therapeutic target! My search result had a link of: 

* https://search.usa.gov/search?utf8=%E2%9C%93&affiliate=web-sdmg-uspto.gov&sort_by=&query=telomerase+inhibitor

I played around with the R rvest package and print command until I figured out the code for printing each patent's name and hyperlink. That was all done in R Studio, which is kinda like a Jupyter notebook cause of how easy it is to play with and get variables without fighting in the terminal.

Once I got the code working in R Studio, I turned it into an R Script. **I have heavily commented each component of the code, so you should be able to follow along with how it works!** Here's that code:

### Functional R Script Code
```r
#!/usr/bin/Rscript
#^ that line tells my Ubuntu Linux computer that this is an R script

# the rvest library is for webscraping
library('rvest')

# I searched for "telomerase inhibitor" at the USPTO webpage. Telomerase is used by cancer to divide indefinitely, 
# so telomerase inhibitors are an interesting target for cancer therapies! Here's the link my search returned (saved as 'url' variable)
url <- "https://search.usa.gov/search?utf8=%E2%9C%93&affiliate=web-sdmg-uspto.gov&sort_by=&query=telomerase+inhibitor"

# I used the rvest "read_html" to parse the htmlo data
webpage <- read_html(url)

# I used the Google Chrome "SelectorGadget" to identify "h4 a" as the CSS for the patent links
# the "%>%" is for applying the CSS selectors to the webpage variable
patent_links <- webpage %>%
  html_nodes("h4 a")

# this is a while loop that cycles through all of the h4 a patent names & links
# i is the iterator of all the patent names and links
i <- 1

# the print statement is just to make things easier to read (this print only happens once)
print("==============================================================================================")

# I used a while loop to cycle through the length of the patent_links variable
while (i < length(patent_links)){
  #################################### FIND PATENT NAMES #################################################
  
  # I used print to identify that every patent name is preceded by only one '>'
  # I used gregexpr to store the starting point of the '>'
  # gregexpr identifies all instances of the pattern. This section uses gregexpr cause the end_of_name 
  # variable, which you'll see in a few lines, has multiple instances of the searched pattern
  location_great <- gregexpr('>', patent_links[i])
  
  # the location_great variable has a lot of attributes. I'm only interested in the starting point number,
  # which I grabbed with applying the selector of [[1]][1]
  start_of_name <- location_great[[1]][1]
  
  # I used print to identify that end of name is at the second '<' greater than sign
  location_less <- gregexpr('<', patent_links[i])
  
  # the problem is that there are multiple instances of the '<' character
  # I specifically grabbed the second instance, the one at the end of the name, with the [[1]][2] selector
  end_of_name <- location_less[[1]][2]
  
  # In previous lines, I identified the start and stop character positions of the name. This gets the name with that information
  # I used the substr argument to get the name. start_of_name has +1 because the '>' isn't what we're interested in ... 
  # I want +1 of that. end_of_name has -1 because the '<' isn't what we're interested in ... we want -1 of that for the text
  name <- substr(patent_links[i], start_of_name+1, end_of_name-1)
  
  # this prints the name of the patent!
  print(paste("Patent Name is: ", name))
  
  #################################### FIND PATENT HYPERLINKS #################################################
  # I used print to identify that links start with 'href="'
  # regexpr identifies the first instance of the pattern ... there is only one instance of the pattern, so it's easier!
  link_start <- regexpr('href="', patent_links[i])
  
  # links end with ".html" OR ".pdf"... that'd require two selectors, not happening, haha what about '">'?
  # okay, yes! That works :) I used regexpr again to grag the end position
  link_end <- regexpr('">', patent_links[i])
  
  # I used the substr argument to grab the link that we are interested in. link_start begins with 'href=', so we
  # want +6 of that. link_end starts with '">', which we don't want. SO link_end has -1 applied to it.
  link <- substr(patent_links[i], link_start+6, link_end-1)
  
  # this prints the full link name
  print(paste("Link is: ", link))
  
  # we want to run this code for each instance of the patent link and name, so it gets iterated + 1 each time
  # note that i was initialized to 1 right before the code block
  i <- i + 1
  
  # the print statement is just to make things easier to read (this one gets called with every loop)
  print("==============================================================================================")
  
}
```

### Webscraping Code Output
I ran the code in my terminal with the command "./webscrape_USPTO_Script.r". Here is the output:

```console
[14:00] david@Ryzen WebScraping_USPTO/ >  ./webscrape_USPTO_Script.r 
Loading required package: xml2
[1] "=============================================================================================="
[1] "Patent Name is:  CPC Scheme - C12Q MEASURING OR TESTING PROCESSES INVOLVING ENZYMES, NUCLEIC ACIDS OR MICROORGANISMS ; COMPOSITIONS OR TEST PAPERS THEREFOR; PROCESSES OF PREPARING SUCH COMPOSITIONS; CONDITION-RESPONSIVE CONTROL IN MICROBIOLOGICAL OR ENZYMOLOGICAL PROCESSES"
[1] "Link is:  https://www.uspto.gov/web/patents/classification/cpc/html/cpc-C12Q.html"
[1] "=============================================================================================="
[1] "Patent Name is:  CPC Scheme - C12N MICROORGANISMS OR ENZYMES; COMPOSITIONS THEREOF ; PROPAGATING, PRESERVING OR MAINTAINING MICROORGANISMS ; MUTATION OR GENETIC ENGINEERING; CULTURE MEDIA"
[1] "Link is:  https://www.uspto.gov/web/patents/classification/cpc/html/cpc-C12N.html"
[1] "=============================================================================================="
[1] "Patent Name is:  CPC Scheme - C12Y ENZYMES"
[1] "Link is:  https://www.uspto.gov/web/patents/classification/cpc/html/cpc-C12Y.html"
[1] "=============================================================================================="
[1] "Patent Name is:  SYNOPSIS OF APPLICATION OF UTILITY GUIDELINES"
[1] "Link is:  https://www.uspto.gov/sites/default/files/web/menu/utility.pdf"
[1] "=============================================================================================="
[1] "Patent Name is:  ENZYMES"
[1] "Link is:  https://www.uspto.gov/web/patents/classification/cpc/pdf/cpc-scheme-C12Y.pdf"
[1] "=============================================================================================="
[1] "Patent Name is:  MICROORGANISMS OR ENZYMES; COMPOSITIONS THEREOF (<U+2060>biocides, pest repellants or attractants, or plant growth regulators, containing microorganisms, viruses, microbial fungi, enzymes, fermentates or substances produced by or extracted from microorganisms or animal material A01N 63/00; food compositions A21, A23; medicinal preparations A61K; chemical aspects of, or use of materials for, bandages, dressings, absorbent pads or surgical articles A61L; fertilisers C05<U+2060>); PROPAGATING, PRESERVING OR MAINTAINING MICROORGANISMS (<U+2060>preservation of living parts of humans or animals A01N 1/02<U+2060>); MUTATION OR GENETIC ENGINEERING; CULTURE MEDIA (<U+2060>microbiological testing media C12Q<U+2060>)"
[1] "Link is:  https://www.uspto.gov/web/patents/classification/cpc/pdf/cpc-scheme-C12N.pdf"
[1] "=============================================================================================="
[1] "Patent Name is:  MICROORGANISMS OR ENZYMES; COMPOSITIONS THEREOF (biocides, pest repellants or attractants, or plant growth regulators, containing microorganisms, viruses, microbial fungi, enzymes, fermentates or substances produced by or extracted from microorganisms or animal material A01N 63/00; food compositions A21, A23; medicinal preparations A61K; chemical aspects of, or use of materials for, bandages, dressings, absorbent pads or surgical articles A61L; fertilisers C05); PROPAGATING, PRESERVING OR MAINTAINING MICROORGANISMS (preservation of living parts of humans or animals A01N 1/02); MUTATION OR GENETIC ENGINEERING; CULTURE MEDIA (microbiological testing media C12Q)"
[1] "Link is:  https://www.uspto.gov/web/patents/classification/cpc/pdf/cpc-definition-C12N.pdf"
[1] "=============================================================================================="

```
