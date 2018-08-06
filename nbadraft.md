NBA draft report, with googleVis
================
Yoon Sung Hong
11/18/2017

1. Introduction
---------------

This project aims to elaborate on, in particular, the visual representation part. For example, we want to be able to see how the data extracted applies in a more relatable form for everyone; if you're someone who doesn't understand R, but wants to know about the proportion of rookies drafted into the NBA from Cal, it would be easier for you to learn about this through a more diagrammatic report. By the end of the post, we want to be able to present advanced data in a more tangible and complex ways and analyze the visualized data accordingly, in particular using the package `googleVis`. Further, we should be able to utilize some aspects of packages such as `rjson` and `RCurl`.

You may be asking, *why is this important*? Think from a strategic standpoint. First and foremost, it is easier to **visualize and strategize data in a relatable manner**; often, we want to get people to understand the trend with data, and this requires a language that is more relatable to them, which in this case is visual representation. Second, we will learn how to extract data from online sources when neat csv files are not made available. `rjson` package will allow us to retrieve such data, in particular those in JSON format.

2. Background knowledge
-----------------------

`googleVis` is a package that brings in Google Charts API, allowing one to make interactive charts similar to `ggVis`. `rjson` is a package that allows one to bring in JSON (JavaScript Object Notation) objects into R objects. This will be the package we will be utilizing to *import data from NBA's API*, as they do not *directly offer any prepared data files*.

I will be using mostly data related to NBA (basketball). For those who may not watch basketball every day, some of these concepts we discuss in this post may come across as strange to you. Thus, before we go further with the post, I will explain some of the concepts that may seem confusing later in the post.

We will be considering the 2017 NBA drafts dataset when learning to use `rjson` package and `googleVis` package. The NBA drafts happen annually, to bring new faces into the league with hopes of developing them into the next Michael Jordan's and Lebron James's. There are in total 30 teams in the NBA, and fundamentally, **each of the teams walks out with 2 players each year at the draft**; 1 in the first round and 1 in the second round. However, teams can include these draft picks (for future seasons) for trade with other teams so some teams may end up with other teams' draft picks, ending up drafting more than 2 or less than 2 rookies. These players are drafted from high school, college, and European leagues.

3. Preparation
--------------

#### 3.1. Loading packages

We will first load `googleVis`, `rjson`, `sqldf`, and `dplyr` as packages to help us.

``` r
library(googleVis)
library(rjson)
library(dplyr)
library(sqldf)
```

#### 3.2. Fetching the data

We will only be considering the 2017 NBA draft, where nationally known players such as *Lonzo Ball* were drafted. <img src="http://images.performgroup.com/di/library/sporting_news/ed/41/nba-draft-2017-ftr-062317jpg_pflanj8jvxqg1pf0d4ws1qjsr.jpg?t=-741473956" style="width:50.0%" />

The picture above was the picture taken after the 2017 NBA draft, with the chief NBA commissioner [Adam Silver](https://en.wikipedia.org/wiki/Adam_Silver) and the rookies drafted. NBA.com provides the statistics for NBA rookie draft every year, but it is visible through their API. This requires us to first find the **json** format of the statistics, then use `rjson` package to import the data into R. I will explain this through screenshots and instructions. For your information, my browser is Google Chrome.

First, click on the view tab, then Developer, and Developer Tools on your browser

<img src="~/previous-projects/R-projects/nbadraft_visualization/images/cap1.png" style="width:50.0%" />

Second, look at the right half of the screen and click the Network tab, then XHR tab right underneath it.

<img src="~/previous-projects/R-projects/nbadraft_visualization/images/cap2.png" style="width:50.0%" />

If you already had your target page open, it may not show anything under the tabs after you click them. **Don't worry**, you just need to hit **refresh page**.

When you hit the refresh page, the page now should look something like this.

<img src="~/previous-projects/R-projects/nbadraft_visualization/images/cap3.png" style="width:50.0%" />

In our case, you'll see that there are four different "objects" listed. One of those is likely to be the object with all the data stored. If you click on these objects, you'll learn that it opens new tabs, like ones in the diagram below.

<img src="~/previous-projects/R-projects/nbadraft_visualization/images/cap4.png" style="width:50.0%" />

Use the preview tab to see what is inside the object. The data you'll be looking for will have a lot of information stored similar to what you see in the API (i.e. objects like **Year**, **Player Names** such as **Lonzo Ball**). Therefore, it is important that you make the comparison between preview tab and original page's data.

Through trial & error, you can find that the 2nd object labeled "drafthistory" is the one storing all the data. You may be asking: how do I know this?

<img src="~/previous-projects/R-projects/nbadraft_visualization/images/cap5.png" style="width:50.0%" />

You can see that with the *drafthistory* object, a lot of the data in the **rowSet** part resembles what we see in the website's API. Therefore, it is highly likely that *drafthistory* object is the one we should be using to import the data into our post.

Now, we need to get the URL for this object that we can then use with `rjson` feature to import into R. We can find this in the tab next to *Preview*, at *Headers*.

<img src="~/previous-projects/R-projects/nbadraft_visualization/images/cap6.png" style="width:50.0%" />

4. Data processing and presenting
---------------------------------

#### 4.1. Importing and cleaning the data

Now, using the url, we can import the data with `fromJSON` function in `rjson` package. Let's first create an object for the URL, then import the JSON data. Note that the imported data will be in a list format. The imported data will then be in a form of a list, with list inside a list. In the list of the list, 2nd component is the data collected and 3rd component is the names of the columns. This will make more sense in the code chunk presented.

``` r
#2017 Drafting data
URL <- paste("https://stats.nba.com/stats/drafthistory",
             "?College=&LeagueID=00&Overall",
"Pick=&RoundNum=&RoundPick=&Season=2017&TeamID=0&TopX=",
sep = "")
dat <- fromJSON(file = URL, method = "C")
rook <- data.frame(matrix(unlist(dat$resultSets[[1]][[3]]),
                          ncol = 12, 
                          byrow = TRUE))
colnames(rook) <- dat$resultSets[[1]][[2]]
```

Let's use `dplyr` to arrange the data so that it can be oragnized to be used for the Sankey diagram. To do this, **data must be grouped and aggregated in terms of the number of people that each university has sent to different teams**, since some teams sent more than one player to a team (for example, Los Angeles Lakers drafted 2 rookies from UCLA). You can see the code chunk below to see what I mean.

``` r
rook$FULLTEAM <- rep(0, nrow(rook))
for(i in 1:nrow(rook)) {
  rook$FULLTEAM[i] <- paste(rook$TEAM_CITY[i], rook$TEAM_NAME[i], sep = " ")
}
rook$COUNT <- rep(1, nrow(rook))
data <- rook %>% 
  select(FULLTEAM, ORGANIZATION_TYPE, ORGANIZATION, COUNT) %>%
  group_by(ORGANIZATION,FULLTEAM) %>%
  summarise(count = sum(COUNT))
```

#### 4.2. Sankey Diagram

Now that the data is ready, let's try using the `googleVis` and its Sankey diagram feature through `gvisSankey` function. **from** part of the function should be the university/institute rookie is from, and **to** part of the function should be the team that the rookie was drafted into. It will be weighed by the aggregate numbers we got from above to show the difference in numbers when necessary. I also added some color, opacity, and resizing formats in the function. **Note that I am using print function to make sure that the gvisSankey is displayed in the same html document, not as a separate pop up.**

``` r
# Sankey diagram using googleVis
print(gvisSankey(data, from="ORGANIZATION", to="FULLTEAM", weight="count",
    options=list(height=800, width=850,
    sankey="{
     link:{color:{fill: 'lightgray', fillOpacity: 0.7}},
     node:{nodePadding: 5, label:{fontSize: 12}, interactivity: true, width: 20},
   }")
  ), 'chart')
```

<!-- Sankey generated in R 3.4.2 by googleVis 0.6.2 package -->
<!-- Tue Jul 24 11:41:45 2018 -->
<!-- jsHeader -->
<script type="text/javascript">
 
// jsData 
function gvisDataSankeyID2a033ab505ac () {
var data = new google.visualization.DataTable();
var datajson =
[
 [
"Adelaide 36ers (Australia)",
"Oklahoma City Thunder",
1
],
[
"Arizona",
"Boston Celtics",
1
],
[
"Arizona",
"Minnesota Timberwolves",
1
],
[
"BC Zalgiris (Lithuania)",
"Houston Rockets",
1
],
[
"California",
"Boston Celtics",
1
],
[
"California",
"Orlando Magic",
1
],
[
"California-Los Angeles",
"Indiana Pacers",
2
],
[
"California-Los Angeles",
"Los Angeles Lakers",
1
],
[
"CB Gran Canaria (Spain)",
"Orlando Magic",
1
],
[
"Clemson",
"San Antonio Spurs",
1
],
[
"Colorado",
"San Antonio Spurs",
1
],
[
"Creighton",
"Chicago Bulls",
1
],
[
"Duke",
"Boston Celtics",
1
],
[
"Duke",
"Charlotte Hornets",
1
],
[
"Duke",
"Detroit Pistons",
1
],
[
"Duke",
"Portland Trail Blazers",
1
],
[
"FC Barcelona Basquet (Spain)",
"Brooklyn Nets",
1
],
[
"Florida State",
"New Orleans Pelicans",
1
],
[
"Florida State",
"Orlando Magic",
1
],
[
"Gonzaga",
"Sacramento Kings",
1
],
[
"Gonzaga",
"Utah Jazz",
1
],
[
"Houston",
"New York Knicks",
1
],
[
"Indiana",
"Toronto Raptors",
1
],
[
"Indiana",
"Utah Jazz",
1
],
[
"Iowa State",
"Denver Nuggets",
1
],
[
"Kansas",
"Phoenix Suns",
1
],
[
"Kansas",
"Sacramento Kings",
1
],
[
"Kansas State",
"Orlando Magic",
1
],
[
"Kentucky",
"Charlotte Hornets",
1
],
[
"Kentucky",
"Miami Heat",
1
],
[
"Kentucky",
"Sacramento Kings",
1
],
[
"KK Mega Leks (Serbia)",
"Atlanta Hawks",
1
],
[
"KK Mega Leks (Serbia)",
"Denver Nuggets",
1
],
[
"KK Mega Leks (Serbia)",
"New York Knicks",
1
],
[
"Louisville",
"Denver Nuggets",
1
],
[
"Miami (FL)",
"Phoenix Suns",
1
],
[
"Michigan",
"Milwaukee Bucks",
1
],
[
"Nanterre 92 (France)",
"Philadelphia 76ers",
1
],
[
"North Carolina",
"Los Angeles Lakers",
1
],
[
"North Carolina",
"Portland Trail Blazers",
1
],
[
"North Carolina State",
"Dallas Mavericks",
1
],
[
"Oklahoma State",
"Philadelphia 76ers",
1
],
[
"Oregon",
"Atlanta Hawks",
1
],
[
"Oregon",
"Chicago Bulls",
1
],
[
"Oregon",
"Houston Rockets",
1
],
[
"Purdue",
"Portland Trail Blazers",
1
],
[
"Red Star Belgrade (Serbia)",
"Philadelphia 76ers",
1
],
[
"SIG Strasbourg (France)",
"New York Knicks",
1
],
[
"South Carolina",
"Milwaukee Bucks",
1
],
[
"Southern Methodist",
"Boston Celtics",
1
],
[
"Southern Methodist",
"Philadelphia 76ers",
1
],
[
"Syracuse",
"Utah Jazz",
1
],
[
"Texas",
"Brooklyn Nets",
1
],
[
"Utah",
"Brooklyn Nets",
1
],
[
"Valparaiso",
"Phoenix Suns",
1
],
[
"Villanova",
"Utah Jazz",
1
],
[
"Wake Forest",
"Atlanta Hawks",
1
],
[
"Washington",
"Philadelphia 76ers",
1
],
[
"Xavier",
"New Orleans Pelicans",
1
] 
];
data.addColumn('string','ORGANIZATION');
data.addColumn('string','FULLTEAM');
data.addColumn('number','count');
data.addRows(datajson);
return(data);
}
 
// jsDrawChart
function drawChartSankeyID2a033ab505ac() {
var data = gvisDataSankeyID2a033ab505ac();
var options = {};
options["width"] = 850;
options["height"] = 800;
options["sankey"] = {
     link:{color:{fill: 'lightgray', fillOpacity: 0.7}},
     node:{nodePadding: 5, label:{fontSize: 12}, interactivity: true, width: 20},
   };

    var chart = new google.visualization.Sankey(
    document.getElementById('SankeyID2a033ab505ac')
    );
    chart.draw(data,options);
    

}
  
 
// jsDisplayChart
(function() {
var pkgs = window.__gvisPackages = window.__gvisPackages || [];
var callbacks = window.__gvisCallbacks = window.__gvisCallbacks || [];
var chartid = "sankey";
  
// Manually see if chartid is in pkgs (not all browsers support Array.indexOf)
var i, newPackage = true;
for (i = 0; newPackage && i < pkgs.length; i++) {
if (pkgs[i] === chartid)
newPackage = false;
}
if (newPackage)
  pkgs.push(chartid);
  
// Add the drawChart function to the global list of callbacks
callbacks.push(drawChartSankeyID2a033ab505ac);
})();
function displayChartSankeyID2a033ab505ac() {
  var pkgs = window.__gvisPackages = window.__gvisPackages || [];
  var callbacks = window.__gvisCallbacks = window.__gvisCallbacks || [];
  window.clearTimeout(window.__gvisLoad);
  // The timeout is set to 100 because otherwise the container div we are
  // targeting might not be part of the document yet
  window.__gvisLoad = setTimeout(function() {
  var pkgCount = pkgs.length;
  google.load("visualization", "1", { packages:pkgs, callback: function() {
  if (pkgCount != pkgs.length) {
  // Race condition where another setTimeout call snuck in after us; if
  // that call added a package, we must not shift its callback
  return;
}
while (callbacks.length > 0)
callbacks.shift()();
} });
}, 100);
}
 
// jsFooter
</script>
<!-- jsChart -->
<script type="text/javascript" src="https://www.google.com/jsapi?callback=displayChartSankeyID2a033ab505ac"></script>
<!-- divChart -->

You can specifically click on the categories, to see the distribution of the players to the selected teams (for the case of this html file you're reading, you cannot. But if you make the googleVis sankey yourself, you will be able to). For example:

<img src="~/previous-projects/R-projects/nbadraft_visualization/images/cap7.png" style="width:50.0%" />

In the above, we selected Cal as the organization, and we see that two players from Cal were drafted into the NBA in this year's draft, with one going to Orlando Magic and another going to Boston Celtics. Now, let's see how many players the Philadelphia 76ers have drafted this season.

<img src="~/previous-projects/R-projects/nbadraft_visualization/images/cap8.png" style="width:50.0%" />

We can see that 5 players were drafted into the 76ers, which is an unusually high number. However, this makes sense as 76ers are known to be rebuilding their team around young players in the recent years.

5. Conclusion
-------------

It is clear that there is no right or wrong way to present the data visually. googleVis and its diverse features offer different ways to quantify and visually present the data. I think through this post, it was realizable that it is important to try to present different forms of graphs and compare them to see which ones apply best to the given data set. I am certain that different cases with different data frames offer alternate graphical solutions that work best for the analysis; if the purpose of the analysis was to see the different colleges sending their rookies to NBA every season, a Sankey diagram would be more ideal than a scatterplot. Therefore, a case-by-case representation of diagrams would be necessary.

In terms of rjson package, it shows and proves that there are many sources where data can be scraped, particularly using the API's set up on the websites.

In terms of the analysis itself relating to NBA rookie draft, it seems to be that same/similar list of colleges send their students to the NBA as rookies every year. In other words, it seems to be that basketball-prestigious colleges always seem to send some players to the NBA every year. This is likely because talented high school basketball players tend to choose their colleges based on their reputations in relation to basketball, creating a cycle between incoming athletes and sustained reputation of these colleges.
