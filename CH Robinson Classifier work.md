```R
# Initializations
library(ggplot2)
library(e1071)
library(nnet)
install.packages("ISLR")
install.packages("tree")
library(tree)
library(ISLR)
```


```R
# Data and notebook setup

#### import, inspect data elements
setwd("/Users/alecpaulson/Desktop")
raw <- read.csv("IMA Recommendation Simulation Data.csv")
raw <- subset(raw,select=-c(X))
head(raw,10)
names(raw)
dim(raw) # data shape

#### inspect levels of inferential target - there 7 non-missing levels of the inferential target
unique(raw$CurrentCondition)
raw1 <- raw[which(raw$CurrentCondition != ""),] # remove missing target level
100* table(raw1$CurrentCondition) / dim(raw1)[1]
```


```R
# Exploratory data analysis

#### is request_id unique?

length(unique(raw1$request_id)); dim(raw1) # appears to be one duplicate

# find the duplicate, inspect it
dups <- duplicated(raw1$request_id)

#### this is a true duplicate. let's remove one of these duplicated rows from the sample, keep the other.
raw1[raw1$request_id %in% unique(raw1$request_id[dups]),]
raw2 <- raw1[!dups,]
dim(raw2)

#### Is there any missing data in this data set? If so, which columns have missing data?
sapply(raw2, function(x) sum(is.na(x)))

####  following features have missing data: (1) order_distance, (2) order_origin_weight, 
####  (3) rate_nrom, (4) est_cost_norm

####  Are there any features you would consider highly correlated? Please limit this to two
####  pairs and describe why you selected the two pairs. Demonstrate this correlation
####  numerically and visually.

####  make a couple of the variables character, based on data dictionary
raw2$order_num_stops <- as.character(raw2$order_num_stops)
raw2$origin_dat_ref <- as.character(raw2$origin_dat_ref)
raw2$dest_dat_ref <- as.character(raw2$dest_dat_ref)
raw2$week_id <- as.character(raw2$week_id)

####  start with numeric variables
num.cols <- unlist(lapply(raw2, is.numeric))
names(raw2)[num.cols]
cor(raw2[complete.cases(raw2),num.cols])
####  ====> (1) miles/order_distance, (2) rate_nrom/est_cost_norm are highly correlated

#### plot numeric variable pairs exhibiting correlation

#### miles/order_distance appear to be linear functions of one another. But, there exists requests
####  for which the miles are greater than zero, but order distance is zero. This makes better sense on 
####  inspection of the data dictionary.
plotDat <-raw2[complete.cases(raw2),]
ggplot(plotDat, aes(x=order_distance, y=miles, color=CurrentCondition)) +
    geom_point() +
    ggtitle("Scatter plot of Miles and Order_Distance")

####  shipment costs correlate with customer payments, makes sense.
ggplot(plotDat, aes(x=rate_norm, y=est_cost_norm, color=CurrentCondition)) +
    geom_point() +
    ggtitle("Scatter plot of Rate_norm and Est_cost_norm")

#### assess correlations between target and categorical variables
getHist <- function(var){
    hist()
}
```


```R
# Classification modeling - 2 approaches

raw2$CurrentCondition.f <- as.factor(raw2$CurrentCondition)
raw3 <- subset(raw2, select = -c(CurrentCondition,request_id) )
raw4 <- raw3[complete.cases(raw3),]

set.seed(1212)
train.size <- 6/10
train.rows <- sample(1:dim(raw4)[1],round(dim(raw4)[1]*train.size),replace=FALSE)
train.data <- raw4[train.rows,]
test.data <- raw4[train.rows,]
names(train.data)

#### on this split order_num_stops has a single level. drop this variable
#### drop date references - these are treated as intelligence-free in the model
train.data <- subset(train.data,select = -c(order_num_stops))
test.data <- subset(test.data,select = -c(order_num_stops))
```


```R
## Try Naive Bayes

#### model
nb <- naiveBayes(CurrentCondition.f ~ .,data=train.data)
summary(nb)
table(predict(nb,test.data))
results <- cbind(test.data$CurrentCondition.f,predict(nb,test.data))

conf.mat.nb <- as.matrix(table(results[,1],results[,2]))
```


```R
100*(table(train.data[,14],train.data[,9]) / dim(train.data)[1])
```


```R
## Try multinomial regression

#### model 
mn.mod <- multinom(CurrentCondition.f ~ week_id+weekday+order_equipment_type+order_distance+
                   order_origin_weight+lead_days+color+est_cost_norm, data =train.data)
summary(mn.mod)
preds <- predict(mn.mod,test.data)
results <- as.data.frame(cbind(test.data$CurrentCondition.f,preds))

conf.mat.mn <- as.matrix(table(results[,1],results[,2]))
```


```R
### Results:
conf.mat.mn
print(paste("Error rate: ",1-(sum(c(50765,6998,15))/sum(conf.mat.mn)),sep=""))
conf.mat.nb
print(paste("Error rate: ",1-(sum(diag(conf.mat.mn)))/sum(conf.mat.mn)),sep="")
```
