---
Title: "K Means Clustering"
Author: "Shreya Goel"
Date: "25 October 2016"
Output: html_document
---

# K-means clustering algorithm

https://www.cs.uic.edu/~wilkinson/Applets/cluster.html

```{r}
install.packages("ggplot2", "dplyr", "tidyr")
```

```{r}
K1 <- read.table("Class_Motivation.csv", header = TRUE, sep = ",")
```

This file contains the self-reported motivation scores for a class over five weeks. We are going to look for patterns in motivation over this time and sort people into clusters based on those patterns.

But before we do that, we will need to manipulate the data frame into a structure that can be analyzed by our clustering algorithm.

The algorithm will treat each row as a value belonging to a person, so we need to remove the id variable.

```{r}
K2 <- dplyr::select(K1, 2:6)
```

It is important to think about the meaning of missing values when clustering. We could treat them as having meaning or we could remove those people who have them. Neither option is ideal. What problems do you foresee if we recode or remove these values?
Recode - It can include our biases and skew our data and data interpretations
Remove - We could loose important data points


We will remove people with missing values for this assignment.

```{r}
K3 <- na.omit(K2) #It "omits" all rows with missing values, also known as a "listwise deletion". EG - It runs down the list deleting rows as it goes.
```
Another pre-processing step used in K-means is to standardize the values so that they have the same range. We do this because we want to treat each week as equally important - if we do not standardise then the week with the largest range will have the greatest impact on which clusters are formed. We standardise the values by using the "scales()" command.

```{r}
K3 <- scale(K3)
```

Now we will run the K-means clustering algorithm. 
1) The algorithm starts by randomly choosing some starting values 
2) Associates all observations near to those values with them
3) Calculates the mean of those clusters of values
4) Selects the observation closest to the mean of the cluster
5) Re-associates all observations closest to this observation
6) Continues this process until the clusters are no longer changing

Notice that in this case we have 5 variables and in the other case we only had 2. It is impossible to vizualise this process with 5 variables. Also, we need to choose the number of clusters we think are in the data. We will start with 2.

```{r}
fit <- kmeans(K3, 2)
```

We have created an object called "fit" that contains all the details of our clustering including which observations belong to each cluster. We can access the list of clusters by typing "fit$cluster", the top row corresponds to the original order the rows were in. Notice we have deleted some rows.

```{r}
fit$cluster
```

We can also attach these clusters to the original dataframe by using the "data.frame" command to create a new data frame called K4.

```{r}
K4 <- data.frame(K3, fit$cluster)
```

Have a look at the K4 dataframe. Lets change the names of the variables to make it more convenient with the names() command.
c() stands for concatonate and it creates a vector of anything, in this case a vector of names.

```{r}
names(K4) <- c("1", "2", "3", "4", "5", "cluster") 
```

Now we need to visualize the clusters we have created. To do so we want to play with the structure of our data. What would be most useful would be if we could visualize average motivation by cluster, by week. To do this we will need to convert our data from wide to long format. We use tidyr and dplyr!

First lets use tidyr to convert from wide to long format.
 - we make 2 new columns namely week and motivation
 - week column contains the column heads
 - motivation column contains the values under each column which have been put to long data format

```{r}
K5 <- tidyr::gather(K4, "week", "motivation", 1:5)
```

Now lets use dplyr to average our motivation values by week and by cluster.
 - %>% is the pipe function

```{r}
library (dplyr)
K6 <- K5 %>% group_by(week, cluster)
```

or

```{r}
library (dplyr)
K6 <- dplyr::group_by(K5, week, cluster)
```

Summarise the motivation data from K6

```{r}
K6 <- summarise(K6, avg = mean(motivation))
```

Now it's time to do some visualization! 
https://www.cs.uic.edu/~wilkinson/TheGrammarOfGraphics/GOG.html
http://docs.ggplot2.org/current/

```{r}
K6$week <- as.numeric(K6$week)
K6$cluster <- as.factor(K6$cluster)
```

### Rplot1-1

```{r}
library(ggplot2)
ggplot(K6, aes(week, avg, colour = cluster)) + geom_line() + xlab("Week") + ylab("Average Motivation")

#- The first argument in a ggplot is the dataframe we are using: K6
#- Next is what is called an aesthetic (aes), the aesthetic tells ggplot which variables to use and how to use them. Here we are using the variables "week" and "avg" on the x and y axes and we are going color these variables using the "cluster" variable
#- Then we are going to tell ggplot which type of plot we want to use by specifiying a "geom()", in this case a line plot: geom_line()
#- Finally we are going to clean up our axes labels: xlab("Week") & ylab("Average Motivation")
```

It would be useful to determine how many people are in each cluster. We can do this easily with dplyr.

To get the number of clusters:

```{r}
K7 <- dplyr::count(K4, cluster)
```

Look at the number of people in each cluster.

Now 3 clusters instead of 2

### Rplot1-2

```{r}
K8 <- na.omit(K2)
K8 <- scale(K8)

fit1 <- kmeans(K8, 3) 
fit1$cluster

K9 <- data.frame(K8, fit1$cluster)

names(K9) <- c("1", "2", "3", "4", "5", "cluster")

K10 <- tidyr::gather(K9, "week", "motivation", 1:5)

K11 <- K10 %>% group_by(week, cluster)


K11 <- summarise(K11, avg = mean(motivation))


K11$week <- as.numeric(K11$week)
K11$cluster <- as.factor(K11$cluster)

library(ggplot2)
ggplot(K11, aes(week, avg, colour = cluster)) + geom_line() + xlab("Week") + ylab("Average Motivation")


K11 <- dplyr::count(K9, cluster)
```

I found the Rplot1-2 makes more sense or is more informative. SInce, there are 3 clusters we can differntiate them more and on comparing we see 
#- the group 1 , in red is sinking on the level of motivation over the weeks
#- the group 3 in blue almost in the middle, started with a negative motivation. Its trying hard, fluctuating but motivation level seems to improve.
#- the group 2 in green, at the bottom is not motivated to a normal level that should be there, but is rather negative. 

Hence, group 2 need more attention and external motivation may be. Group 3 is trying good.
Group 1 needs to pull its strings, because it started good but is sinking negatively over the weeks.












# Extension Exercise
Data [collected in class](https://tccolumbia.qualtrics.com/SE/?SID=SV_6RRqlSAM6lZWYQt). We want to create two groups of clusters, the answers to the questions and regions where people grew up. Then we create a visualization that shows the overlap between the two groups of clusters.

```{r}
install.packages("ggplot2")
install.packages("cluster")
install.packages("ggmap")

library(dplyr)
library(tidyr)
library(ggplot2)
library(cluster)
library(ggmap)

```

```{r}
P1 <- read.table("cluster-class-data.csv", header = TRUE, sep = ",")
```

name the head labels in P1 data frame

```{r}
names(P1) <- c("duration", "name1", "name2", "cats", "giff_jiff", "nylife", "siblings", "sports", "distancefromtc", "androidfriends", "movies", "classesthissem", "placesvisited", "city", "state", "country")
```

### Location variable
the location variables from P1 into a new data frame

```{r}
P7 <- dplyr::select(P1, city, state, country)
```

clean the data if you like. I haave just cleaned the country data. This step can be omited.

```{r}
levels(P7$country)
levels(P7$country)[levels(P7$country) == "United States "] <- "United States"
levels(P7$country)[levels(P7$country) == "United States of America"] <- "United States"
levels(P7$country)[levels(P7$country) == "The United States of America"] <- "United States"
levels(P7$country)[levels(P7$country) == "United States of America "] <- "United States"
levels(P7$country)[levels(P7$country) == "USA"] <- "United States"
levels(P7$country)[levels(P7$country) == "china"] <- "China"
levels(P7$country)[levels(P7$country) == "CHINA"] <- "China"
levels(P7$country)[levels(P7$country) == "Viet Nam "] <- "Vietnam"
levels(P7$country)[levels(P7$country) == "chile"] <- "Chile"
```

change the data type of each variable to charachter

```{r}
P7$country <- as.character(P7$country)
P7$state <- as.character(P7$state)
P7$city <- as.character(P7$city)
```

unite all the variables under 1 variable.

```{r}
P7 <- tidyr::unite(P7, location, city, state, country, sep = ",")
```

use ggmap to get the longtude and latitude of each loation.

```{r}
library(ggmap)
P7 <- mutate_geocode(P7, location)
```

make a new data frame that contains only thelongitude and latitude variables

```{r}
P8 <- dplyr::select(P7, 2:3)
```

use kmeans to make 2 clusters may be

```{r}
fit <- kmeans(P8, 2)
```

add a variable cluster to fit

```{r}
fit$cluster
```

make a new data frame that contains 

```{r}
P9 <- data.frame(P8, fit$cluster)
```

make the cluster plot
### Rplot2_location

```{r}
library(cluster)
clusplot(P9, P9$fit.cluster, labels = 2, color= TRUE)
```

Interpretation of Cluster plot - Location
Cluster 1 - students' location includes: united states and areas around it
Cluster 2 - students' location includes: asia and near by - China, Chile, India etc 

add a new variable named id, 1 through 22

```{r}
P8$id <- c(1:22)
```

### Response dataframe

make another dataframe that contains data related to questions answered in the survey.

```{r}
P3 <- dplyr::select(P1, sports, placesvisited)
```

change the data type to numeric 

```{r}
P3$sports <- as.numeric(P3$sports)
P3$placesvisited <- as.numeric(P3$placesvisited)
```

use kmeans tomake 3 clusters this time 

```{r}
fit1 <- kmeans(P3, 3)
```

add a variabe cluster

```{r}
fit1$cluster
P4 <- data.frame(P3, fit1$cluster)
```

plot the cluster plot
### Rplot2_responses

```{r}
library(cluster)
clusplot(P4, P4$fit1.cluster, labels = 2, color = TRUE)
```
This cluster plot depicts 
cluster 2 - students who visited places from 9 to 15
cluster 3 - students who visted more than 15 places
cluster 1 - students who visited less than 9 places
This implies thst playing sports and visiting places is not related to a big extent
add the variable id 1 through 22

```{r}
P3$id <- c(1:22)
```

merge the 2 data frames p3 and p8 on the basis of common variable id

```{r}
Q <- merge(P8, P3,by="id")
```

change the data type of id to numeric

```{r}
Q$id <- as.numeric(Q$id)
```

```{r}
fit2 <- kmeans(Q, 2)
```

change the 2 above for number clusters 

```{r}
fit2$cluster
q1 <- data.frame(Q, fit2$cluster)
```

final clusterplot
### Rplot2 - location+responses

```{r}
library(cluster)
clusplot(q1, q1$fit2.cluster, labels = 2, color = TRUE)
```

1. see how we did not change the data type to numeric in location cluster and we had to in the response data frame. this is because longitude and latitude are already in numeric format in location data frame

2. location cluster plot reveals how the locations of the participants have been divided into 2 groups or clusters

3. response cluster plot reveals the 4 cluster based the response types chosen

4. the final cluster is the integration of the location as well as the responses into 2 clusters.


```{r}
install.packages("vcd", dependencies = TRUE)
library(vcd)
```

```{r}
fit1$cluster <- as.factor(fit1$cluster)
fit$cluster <- as.factor(fit$cluster)
```

Make anew table that contains only the cluster values

```{r}
N <- table(fit1$cluster, fit$cluster)
```

Plot a mosaic table

```{r}
mosaicplot(N, main = "Mosaic Plot", sub = NULL, xlab = "Responses", ylab = "Location", sort = NULL, off = NULL, dir = NULL, color = blues9, shade = FALSE, margin = NULL, cex.axis = 0.6, las = par("las"), border = NULL, type = c("pearson", "deviance", "FT"))

#Interpretation of Mosaic Plot - 
#The clusters on y axis are locations and there are 2 main clusters: "1" around Asia and "2" around United States. Which implies that students from Asia have visited lesser number of places than people from United States in general.
```

Extra - for future refrence of the author (to learn)
```{r}
#plot q1 - only for future reference (this plot does not really mean anything in specific)
mosaicplot(q1, main = deparse(substitute(q1)), sub = NULL, xlab = NULL, ylab = NULL, sort = NULL, off = NULL, dir = NULL, color = blues9, shade = FALSE, margin = NULL, cex.axis = 0.6, las = par("las"), border = NULL, type = c("pearson", "deviance", "FT"))

#name of the plot is Rplot2-mosaicplot
```
