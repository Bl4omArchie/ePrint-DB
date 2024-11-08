# ePrint Database Project

Goal : getting every ePrint papers in a database the fastest as possible.

Technologies :
- scripting : golang
- SGBD : postgreSQL
- concurrency

Result of retrieving datas from every 2024 papers :
```
Execution time: 4m43.4106983s
```

Result of retrieving datas from every 2024, 2023 and 2022 papers in concurrency :
```
Total execution time = ~10m40s
```
For this test, I launched three goroutines one for each year. I think it is a poor concurrency design and I can find better.

Notes : 
- too much goroutines making simultaneous request at one server lead to this error:
read: connection reset by peer

- Find a better design for my concurrency

# Design ideas

Stages :
1) Retrieve data about a paper : title, pdf url, category
2) Download the PDF
3) store the raw binary in the database

Draw of the process :
```

Start: GetPapersYear -> For each papers: RetrieveDataPaper -> DownloadPaper -> InsertBinary (into the dabase)

```

First idea :
- One goroutine for each years
- A fixed number N of goroutines for stages 1, 2 and 3 <br/>
(ie : I create 100 goroutines for each stages and when they have finished one task, they continue with the next one)

Second idea :
- One goroutine for each years
- N goroutines for each stages that cannot exceed a limit P of goroutines. <br/>
(ie : I create goroutines for my task until I reached the limit P. Then I wait that some of them has done to create a new ones) 

More ideas : <br/>
Basically the same but using a pipeline with channels. A fourth idea idea could be to use custom rating limit with work-stealing queue.
I shall explore and test those ideas.


# Statistics 

In order to anticipate rate limit issue (from hardware or the ePrint server), I decided to make a small analysis of how many request I will need, how many insertion in my database etc

There is the volume of paper for each years :
```
"2024":1799, "2023":1971, "2022":1781, "2021":1705, "2020":1620,
"2019":1498, "2018":1249, "2017":1262, "2016":1195, "2015":1255, "2014":1029, "2013":881, "2012":733, "2011":714, "2010":660, 
"2009":638, "2008":545, "2007":482, "2006":485, "2005":469, "2004":375, "2003":265, "2002":195, "2001":113, "2000":69,
"1999":24, "1998":26, "1997":15, "1996": 16,

```

Years between 2014 and 2024 have more than one thousand papers which I consider to be the years who need more goroutines.
In the other case years, between 1996 and 2013, there is only less than one thousand papers or even a few dozen which means we don't need too much goroutines.