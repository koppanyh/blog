# Reading a Wikipedia Dump

Friday, July 17, 2015

---

So I wanted to embark on a journey to find out if I could make a natural language processor by analysing how grammar works, but I needed some examples. I needed to see how sentences were put together in terms of the parts of speech and also see which words can be removed from a sentence without altering the meaning but making it easier to process. I figured Wikipedia has the largest source of text and context that I could simply download and scan to answer my questions. Problem is, the [Wikipedia Dump](https://dumps.wikimedia.org/) is encoded in XML so that it can be easily parsed and used offline, but is not what I am looking for to analyse the language.

My solution to the problem was to parse the Wikipedia dump myself and turn each page into its own file. In order to do the initial testing, I decided to use a dump for Simple Wiki since it's quite a few gigabytes less. I also wrote a small python script to go through the XML file at intervals of 1000 characters at a time. This allowed me to get some insight into how the XML file was structured.

Between the `<page>` and `</page>` tags was the page which included the title and sections and subsections and references and all those things. The title of each page was found between the `<title>` and `</title>` tags. The main text, which houses relevant information, is kept between the `<text>` and `</text>` tags. Words surrounded by three single quotes `''' '''` appeared to be what signalled bolded words. Words between two sets of square brackets `[[ ]]` are links to other pages. Sections and subsections are signalled by words between two `== ==` and three `=== ===` equal signs consecutively. The asterisk symbol `*` indicated a bullet point. And it appeared that `{{ndash}}` was simply a dash `-`.

Knowing all this, I was able to construct a program that made Python list of every page in the dump and then made a text file with the title of the page title and the content of the page text.

Turns out some of the text files only contained text that was only one file and described a redirect. Otherwise those files were empty, so I had to write a little Python script that went through and removed all of those, leaving only files with actually useful text.

Otherwise, I had to go through all the files individually and remove files with names in foreign languages and names of things that aren't useful for my purposes... 140,000+ text files... it's fun...

Anyway, the code for the program that got each page and turned it into a text file is at the far bottom and the general algorithm is below this block of text. The program was designed to parse the entire 400+ MB simplewiki file without loading it all into memory... not sure why... made sense at the time. But it scans through the whole file line by line to find the page, title and text tags. Just remember to actually download the Wiki dump if you want to try this, use SimpleWiki since it's quite a few gigabytes less than the actual EnWiki dump.

```
open simplewiki.xml
set text variable to " "
while lines haven't run out{
    page content = ""
    while you haven't found the <page> tag{
        text variable = readline()
    }
    while you haven't hit the </page> tag{
        add the current line to the page text variable
    }
    set a title variable equal to the page title by searching between <title> tags
    open a file for writing with the title variable's title
    set a text variable equal to the  text between <text> tags
    write that text to the file
    close the file}
close the simplewiki.xml file
```

```python
#!/usr/bin/env python
f = open("simplewiki.xml")
x = " "
while x != "": #while still reading
    page = "" #hold page
    while x.find("<page") == -1: #skip through lines until find page start
        x = f.readline()
        if x == "": break
    while x.find("</page") == -1: #get lines between <page> and </page>
        x = f.readline()
        page += x
        if x == "": break
    title = page[page.find("<title")+7:page.find("</title")].replace("/","-") #get title
    g = open(title+".txt","w") #open file with title
    text = page[page.find("<text"):page.find("</text")]
    g.write(text)
    g.close()
f.close()
print "Done" #give confirmation of finishing
```

Here are pictures. This first one is a screenshot of the finished code in my favorite text editor: Gedit. I also use Ninja-IDE in extreme cases where I'm working on complicated projects.

![](./images/Screenshot%20from%202015-07-17%2018_52_30.png)

This second picture is a picture of all of my processed text files. Since there are so many, I actually sorted them out based on the letter they start with and put them in folders. All the text files you can see have weird/foreign characters at the start and have been sorted manually after this picture was taken. But there you go, I now house every topic of SimpleWiki on my computer in text file format... come at me internet blackout.

![](./images/Screenshot%20from%202015-07-17%2018_52_53.png)

<sub>Posted by [koppanyh](https://github.com/koppanyh) on 2015/07/17 at 7:01 PM PDT</sub>