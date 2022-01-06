## What do we know about the current state of the Wilmington economy?
# Data sources
There are two main data sources **-Current Employment Statistics (CES) and Local Area Unemployment Statistics (LAUS)-**  that provide information about the labor market performance of sub-state areas. For the purposes of this entry, we will use both of them to describe the structure of the Wilmington Economy, where it stands relative to the previous year and pre-Covid times, and how it compares to other Metropolitan areas.

Nonfarm payroll employment estimates come from the monthly
**(CES)**, which surveys
approximately 144,000 businesses and government agencies, representing approximately 697,000 individual worksites nationally. In addition to overall nonfarm payroll, The Current Employment Statistics (CES) program produces detailed industry estimates of nonfarm employment, hours, and earnings of workers on payrolls.

The **(LAUS)** program calculates the state-level monthly unemployment rate, a model-based estimate, using inputs from multiple sources, including
the Current Population Survey **(CPS)**, the **CES** program, the **unemployment insurance system**, and the **U.S. census**. The unemployment rate is used to
capture a snapshot of the labor force by providing an estimate of the percentage of the civilian noninstitutional population aged 16 years and over that is unemployed and looking for work. 

There main differences between the two the program are that the (CES) counts jobs as it is an establishment survey and the (LAUS) counts people as it mainly relies on a household surey. Importantly, the (CES) only counts individuals on payroll and therefore does not include self-employed individuals while the (LAUS) includes both payroll and self-employed individuals.

# Where does the Wilmington economy stand? 
1. According to data from the Current Employment Survey (CES), the Wilmington Metropolitan area had 135,000 Wage and Salary jobs as of October, 2021. This means it has 7,100 or 5.6% more jobs than the same month in 2020. Given the significant labor market disruptions that occured as a result of the pandemic, we also compare the current level of Wage and Salary jobs to the same month in 2019 and find that it is 1,500 or 1.1% higher indicating that unlike many parts of the state and the country it has exceeded its pre-pandemic employment levels. 

2. The labor force has 4,400 or 2.9% more individuals than the same month last year and 1,584 or 1% more than the pre-pandemic level in 2019. The state of North Carolina's labor force participation rate, on the other hand, as of November 2021 is still 3.6% below the November 2019 level.

3. The unemployment rate, as of October, 2021, stands at 3.1% which is 2.7 percentage points below the same month in 2020. The unemployment rate is even below October 2019 -pre-pandemic- which was 3.2%. 

# Nonfarm Wage and Salary Employment: 
Employment in November 2021 has already exceeded November 2019 levels. The decline in 2020 was short lived and employment has been on a steady trajectory, since then. 

![wilmi, out.width = '40%'](https://user-images.githubusercontent.com/94587267/146865971-33df220f-98d2-4b66-9443-31a4c69beffd.png)

 <details>
<summary>Code</summary>

<pre><code lang="R">
library(rjson)
 library(blsAPI)
 library(ggplot2)
 
 ## Pull the data via the API
 payload <- list(
   'seriesid'=c('SMS37489000000000001'),
   'startyear'=2019,
   'endyear'=2021)
 response <- blsAPI(payload)
 json <- fromJSON(response)
 
 ## Process results
 apiDF <- function(data){
   df <- data.frame(year=character(),
                    period=character(),
                    periodName=character(),
                    value=character(),
                    stringsAsFactors=FALSE)
   
   i <- 0
   for(d in data){
     i <- i + 1
     df[i,] <- unlist(d)
   }
   return(df)
 }
 
 total.df <- apiDF(json$Results$series[[1]]$data)


 ## Change value type from character to numeric
 total.df[,4] <- as.numeric(total.df[,4])
 
 ## Rename value prior to merging
 names(total.df)[4] <- 'Nonfarm'


 total.df$date <- as.POSIXct(strptime(paste0('1',total.df$periodName,total.df$year), '%d%B%Y'))
 
 ## Beginning and end dates for the Great Recession (used in shaded area)
 gr.start <- as.POSIXct(strptime('1March2020', '%d%B%Y'))
 gr.end <- as.POSIXct(strptime('1March2021', '%d%B%Y'))
 ## Plot the data
 ggplot(total.df) + geom_rect(aes(xmin = gr.start, xmax = gr.end, ymin = -Inf, ymax = Inf), alpha =20, fill="#DDDDDD") + geom_line(aes(date, Nonfarm)) + ylab('Nonfarm employment')  + xlab('1 year post-COVID') + labs(title="Nonfarm wage employment, Wilmington (Jan 2019 to October 2021)",subtitle="Shaded area=March 2020 to March 2021") + theme_bw()
 ggsave("wilmi.png")
 
 
</code></pre>
</details>

In the figure below, we decompose employment by year and show that the job losses were significant in 2020 but short lived as the recovery looks V-shaped unlike most communities across the country. Employment in 2021 has been above both 2020 and 2019 since August.

![yemp](https://user-images.githubusercontent.com/94587267/147981196-5c62eb9e-9f0d-4879-ae5d-7d665c11d83d.png)

<details>
<summary>Code</summary>

<pre><code lang="R">
 colors <- c("blue", "black", "gold1")
 

 
 p <- ggplot(data=total.df, aes(x=period, y=Nonfarm,group=year,color=year)) +  theme_bw()+
   geom_line(,size=2)
 p
 p+geom_line(data=subset(total.df, year == "2019"), color="blue", size=1.5) +
   geom_line(data=subset(total.df, year == "2020"), color="grey50", size=1.5) +
   geom_line(data=subset(total.df, year == "2021"), color="gold1", size=1.5) +
   scale_x_discrete(labels=c("M01" = "January", "M02" = "February",
                             "M03" = "March", "M04"="April", "M05"="May","M06"="June","M07"="July","M08"="August","M09"="September","M10"="October","M11"="November","M12"="December")) +
   theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust=1)) +
   labs(title="Total Nonfarm wage employment in 2019, 2020, and 2021",x = "Month",
        y = "Nonfarm employment (thousands)")  +
   scale_color_manual(values = c("blue","grey50","gold1")) 
 
 ggsave("yemp.png")
 
</code></pre>
</details>

# What about the Labor Force?
There have been recently been concerns about the great resignation, sizeable drops in the labor force participation rate, and labor shortages. In the figure below, we show that the labor force and employment both declined sharply in April 2020 but bounced shortly thereafter and have since recovered as they are both above the same levels in 2019. 
![lfemp](https://user-images.githubusercontent.com/94587267/147981227-f8625d2d-975e-4780-8b50-460d7baa6d17.png)
Relative to 2020, unsurprisingly, both employment and the labor force are appreciably higher given that the base period was depressed due the onset of the pandemic and the subsequent changes in behavior, and restrictions.
![j2](https://user-images.githubusercontent.com/94587267/148267258-fee9c096-b591-45e8-a4ac-d791496227ed.png)

<details>
<summary>Code</summary>

<pre><code lang="R">
 ######API SERIES
 ###LABOR FORCE LAUMT374890000000006
 ###EMPLOYMENT 	LAUMT374890000000005
 ####UNEMPLOYMENT 	LAUMT374890000000004
 ###UR rate LAUMT374890000000003
 
 library(rjson)
 library(blsAPI)
 library(ggplot2)
 
 ## Pull the data via the API
 lf <- list(
   'seriesid'=c('LAUMT374890000000006', 'LAUMT374890000000005'),
   'startyear'=2019,
   'endyear'=2021)
 response <- blsAPI(lf, 2)
 json <- fromJSON(response)
 
 ## Process results
 apiDF <- function(data){
   df <- data.frame(year=character(),
                    period=character(),
                    periodName=character(),
                    value=character(),
                    stringsAsFactors=FALSE)
   
   i <- 0
   for(d in data){
     i <- i + 1
     df[i,] <- unlist(d)
   }
   return(df)
 }
 
 lf.df <- apiDF(json$Results$series[[1]]$data)
 emp.df <- apiDF(json$Results$series[[2]]$data)
 
 ## Change value type from character to numeric
 lf.df[,4] <- as.numeric(lf.df[,4])
 emp.df[,4] <- as.numeric(emp.df[,4])
 
 ## Rename value prior to merging
 #names(lf.df)[4] <- 'Labor.force'
 #names(emp.df)[4] <- 'Employment'
 
 ###Create names of the individual variables
 emp.df$name <- c("Employment")
 lf.df$name<- c("Labor force")
 appendedDf <- rbind(emp.df,lf.df)
 
 #######Think about the way to graph this for a second
 appendedDf$value <- as.numeric(as.character(appendedDf$value/1000))
 
 p <- ggplot(data=appendedDf, aes(x=period, y=value)) +  theme_bw()+
  geom_bar(stat="identity")
 p+
   facet_wrap(~name+year)
 
 plot <- ggplot(appendedDf, aes(period, value, fill=name))
 plot <- plot + geom_bar(stat = "identity", position = 'dodge') +
   facet_wrap(~year) +
   theme_bw () +
   scale_x_discrete(labels=c("M01" = "January", "M02" = "February",
                             "M03" = "March", "M04"="April", "M05"="May","M06"="June","M07"="July","M08"="August","M09"="September","M10"="October","M11"="November","M12"="December")) +
   theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust=1)) +
   labs(title="Labor force and total employment in 2019, 2020, and 2021",x = "Month",
        y = "Employment and Labor force in thousands") +
   theme(legend.title = element_blank())
 plot
 ggsave("lfemp.png")
</code></pre>
</details>

When we compare monthly employment and the labor force relative to 2019, we find a slightly more complicated picture. Monthy employment in 2021 was still below the same month in 2019 until July when it turned positive. The labor force has been higher than 2019 for all months except May and June. 
![j3](https://user-images.githubusercontent.com/94587267/148264037-88b40116-124d-454a-89c9-9769d34e5374.png)

# Where is the employment growth coming from?
To better understand the sources of growth, we present below employment growth in the last 6 month of 2021 relative to the same months in 2020. It is clear that the sectors showing the most significant increase -such as Leisure and Hospitality- are the same one that suffered the most in the first few months of the pandemic. With the exception of the Federal Government, all other sectors are above their 2020 levels further confirming continued improvement in the labor market.
![allsec](https://user-images.githubusercontent.com/94587267/148265513-18c9231d-3345-42a2-8ffb-db9c091f5dd1.png)
 
 <details>
<summary>Code</summary>

<pre><code lang="R">

library(devtools)
library(rjson)
library(blsAPI)
library(ggplot2)

## Pull the data via the API
payload <- list(
  'seriesid'=c('SMU37489001500000001', 'SMU37489003000000001','SMU37489004200000001','SMU37489004100000001','SMU37489004300000001','SMU37489005000000001','SMU37489005500000001','SMU37489006000000001','SMU37489006500000001','SMU37489007000000001','SMU37489008000000001','SMU37489009091000001','SMU37489009092000001','SMU37489009093000001'),
  'startyear'=2019,
  'endyear'=2021)
response <- blsAPI(payload)
json <- fromJSON(response)

## Process results
apiDF <- function(data){
  df <- data.frame(year=character(),
                   period=character(),
                   periodName=character(),
                   value=character(),
                   stringsAsFactors=FALSE)
  
  i <- 0
  for(d in data){
    i <- i + 1
    df[i,] <- unlist(d)
  }
  return(df)
}

mining.df <- apiDF(json$Results$series[[1]]$data)
manuf.df <- apiDF(json$Results$series[[2]]$data)
retrade.df <- apiDF(json$Results$series[[3]]$data)
whotrade.df <- apiDF(json$Results$series[[4]]$data)
transp.df <-  apiDF(json$Results$series[[5]]$data)
info.df<-apiDF(json$Results$series[[6]]$data)
fin.df <- apiDF(json$Results$series[[7]]$data)
prof.df <- apiDF(json$Results$series[[8]]$data)
edu.df<-apiDF(json$Results$series[[9]]$data)
les.df<-apiDF(json$Results$series[[10]]$data)
oth.df<-apiDF(json$Results$series[[11]]$data)
fed.df<- apiDF(json$Results$series[[12]]$data)
st.df<-apiDF(json$Results$series[[13]]$data)
loc.df<-apiDF(json$Results$series[[14]]$data)

## Change value type from character to numeric
mining.df[,4] <- as.numeric(mining.df[,4])
manuf.df[,4] <- as.numeric(manuf.df[,4])
retrade.df[,4] <- as.numeric(retrade.df[,4])
whotrade.df[,4] <- as.numeric(whotrade.df[,4])
transp.df[,4] <- as.numeric(transp.df[,4])
info.df[,4] <- as.numeric(info.df[,4])
fin.df[,4] <- as.numeric(fin.df[,4])
prof.df[,4] <- as.numeric(prof.df[,4])
edu.df[,4] <- as.numeric(edu.df[,4])
les.df[,4] <- as.numeric(les.df[,4])
oth.df[,4] <- as.numeric(oth.df[,4])
fed.df[,4] <- as.numeric(fed.df[,4])
st.df[,4] <- as.numeric(st.df[,4])
loc.df[,4] <- as.numeric(loc.df[,4])


###Identify the variable
mining.df$pos <- ( "Mining/Logging/Construction")
manuf.df$pos <- ("Manufacturing")
retrade.df$pos <- ("Retail Trade")
whotrade.df$pos <- ("Wholesale Trade")
transp.df$pos <- ("Transportation and Utilities")
info.df$pos <-("Information")
fin.df$pos <-("Finance")
prof.df$pos <-("Professional and Business Services")
edu.df$pos <-("Education and Healthcare Services")
les.df$pos <-("Leisure and Hospitality")
oth.df$pos <-("Other Services")
fed.df$pos <- ("Federal Government")
st.df$pos <- ("State Government")
loc.df$pos <- ("Local Government")


####This puts all the individual sectors into the same panel
new <- rbind(mining.df, manuf.df,retrade.df,whotrade.df,transp.df,info.df,fin.df,prof.df,edu.df,les.df,oth.df,fed.df,st.df,loc.df)

####Splitting the month variable into numeric and character
allsec<-data.frame(new, character = substr(new$period, 1, 1), numeric = substr(new$period, 2, nchar(new$period)))  
###turn month variable into numeric
library(data.table)
setDT(allsec) 
allsec[, mn := as.numeric(numeric)]
### combine month and year

library(zoo)
allsec$Date <- as.yearmon(paste(allsec$year, allsec$numeric), "%Y %m")

#### Difference relative to 12 months prior
library(dplyr)
allsec<-allsec %>%
  group_by(pos) %>%
  arrange(Date) %>%
  mutate(diff = value - lag(value,12))

allsec$yesno <- ifelse(allsec$diff>0, "Yes", "No")


####Growth rate
library(dplyr)
allsec<-allsec %>%
  group_by(pos) %>%
  arrange(Date) %>%
  mutate(perc = diff / lag(value,12))

################################Now graph percentages
p2<-ggplot(data=subset(allsec, year == "2021" & mn>4 & mn<11), aes(x=reorder(periodName,mn), y=perc,fill = yesno,label = scales::percent(perc))) +
  geom_bar(stat="identity") +
  ylab("Year over year percentage change in employment") +
  theme_bw()+
  theme(axis.text.x = element_text(angle = 90,size=10, vjust = 0.5, hjust=1))+
  labs(title="Percentage change in employment in 2021 relative to the same month in 2020",x = "Month",
       y = "Percentage change") +
  theme(legend.title = element_blank()) +
  theme(legend.position = "none",
        panel.grid = element_blank(),
        panel.background = element_blank())+
  scale_y_continuous(labels = scales::percent_format(accuracy = 1L))+
  geom_text(aes(label = sprintf("%0.1f",perc*100,"%" )),vjust=-0.6,size=3)+
  scale_fill_discrete(name = "", labels = c("Decline", "Growth"))
p2


p2+
  facet_wrap(~pos) 

ggsave("allsec.png") 

###############################################Relative to the same month in 2019 (AS A SHARE)#####################

####Growth rate
library(dplyr)
allsec<-allsec %>%
  group_by(pos) %>%
  arrange(Date) %>%
  mutate(rat = value / lag(value,24))
</code></pre>
</details>

 
# Which sectors are doing better than in 2019?
Perhaps, unsurprisingly, when we compare the employment levels of each sector to the same period in 2019, we find that the sectors -Leisure and Hospitality, Retail- most sensitive to consumer spending and individuals gathering indoors are the ones still showing the most scars from the pandemic. In addition to those two sectors, State government, Other services, and Manufacturing are also below their 2019 levels. Professional and Business Services, Transportation/Utilities, and Construction are consideraly above their 2019 levels. 
![rel2019](https://user-images.githubusercontent.com/94587267/148268842-0e9d3486-3f25-46ad-bd94-c90b3ca91fc0.png)

<details>
<summary>Code</summary>

<pre><code lang="python">
###############################################Relative to the same month in 2019 (AS A SHARE)#####################

####Growth rate
library(dplyr)
allsec<-allsec %>%
  group_by(pos) %>%
  arrange(Date) %>%
  mutate(rat = value / lag(value,24))
############################################Relative to same month in 2019
allsec$yes <- ifelse(allsec$rat>0.9999, "Yes", "No")


p2<-ggplot(data=subset(allsec, year == "2021" & mn>4 & mn<11), aes(x=reorder(periodName,mn), y=rat,fill=yes,label = scales::percent(rat))) +
  geom_bar(stat="identity") +
  ylab("Year over year percentage change in employment") +
  theme_bw()+
  theme(axis.text.x = element_text(angle = 90,size=10, vjust = 0.5, hjust=1))+
  labs(title="Ratio of employment in 2021 relative to the same month in 2019",x = "Month",
       y = "Ratio-Employment in 2021/Employment in 2019") +
  theme(legend.title = element_blank()) +
  theme(legend.position = "none",
        panel.grid = element_blank(),
        panel.background = element_blank())+
  scale_y_continuous(labels = scales::percent_format(accuracy = 1L))+
  scale_y_continuous(limits = c(0, 1.3), breaks = seq(0,1.3, .4),labels = scales::percent_format(accuracy = 1L))+
  scale_fill_discrete(name = "", labels = c("Employment in 2021 is below the same month in 2019", "Employment in 2021 is above the same month in 2019")) +
  geom_text(aes(label = sprintf("%0.1f",rat*100,"%" )),vjust=-0.3,size=3) +
  theme(legend.position=c(0.7,0.05))
  
p2


p2+
  facet_wrap(~pos) 

ggsave("rel2019.png") 
</code></pre>
</details>





#  How does the Wilmington economic performance compare to other Metropolitan areas?
The Wilmington economy as we showed above seems to have recovered most of the jobs it lost as a result of the pandemic. As of October, 2021, Wilmington has 5.6% more jobs than the same month in 2020. That is the highest job gain across the state and is 2.2 percentage points above the state average.
![allmetro](https://user-images.githubusercontent.com/94587267/148269911-a1e4f796-4c9f-4b6e-b950-f35a43b00308.png)

<details>
<summary>Code</summary>

<pre><code lang="python">
###############################################Weekly earnings/hourly earnings/ and weekly hours
####Weekly hours SMU37489000500000002
####Weekly earnings 	SMU37489000500000011
####Hourly earnings SMU37489000500000003

 ####################Decomposition by sector (North Carolina)
 ###total nonfarm (SMU37000000000000001)
 ##Goods producing (SMU37000000600000001)
 ##Mining and logging (SMU37000001500000001)
 ##Manufacturing (SMU37000003000000001)
 ##Trade/transportation/utilities (SMU37000004000000001)
 ##Information (SMU37000005000000001)
 ##Financial activities (SMU37000005500000001)
 ##Professional and Business services (SMU37000006000000001)
 ##Education and Healthcare services (SMU37000006500000001)
 ##Leisure and Hospitality (SMU37000007000000001)
 ##Other services (SMU37000008000000001)
 ##Government(SMU37000009000000001)
 ##total private (SMU37000000500000001)
 ##Goods producing (SMU37000000600000001)
 ##Service providing (SMU37000000700000001)
 ##Wholesale trade (SMU37000004100000001)
 ##Food and drinking places (SMU37000007072200001)
 ##Federal gov (SMU37000009091000001)
 ## State gov (SMU37000009092000001)
 ##local gov (SMU37000009093000001)
 
 ##Retail trade (SMU37000004200000001)
 
 
 ##### List of Metropolitan areas (Charlotte, Raleigh, Greensboro,Winston-salem,Durham,Fayetvile,Asheville,Hickory,Jacksonville,Greenville,Burlington,Rocky Mount,New Bern,Goldsboro)
 ###########Now pull Total non-farm by Metro 


#################Decomposition by sector (All BLS DATA) (WILMINGTON)
library(devtools)
library(rjson)
library(blsAPI)
library(ggplot2)

## Pull the data via the API
payload <- list(
  'seriesid'=c('SMU37000000000000001', 'SMU37117000000000001','SMU37155000000000001','SMU37167400000000001','SMU37205000000000001','SMU37221800000000001','SMU37241400000000001','SMU37246600000000001','SMU37247800000000001','SMU37258600000000001','SMU37273400000000001','SMU37351000000000001','SMU37395800000000001','SMU37405800000000001','SMU37489000000000001','SMU37491800000000001'),
  'startyear'=2019,
  'endyear'=2021)
response <- blsAPI(payload)
json <- fromJSON(response)

## Process results
apiDF <- function(data){
  df <- data.frame(year=character(),
                   period=character(),
                   periodName=character(),
                   value=character(),
                   stringsAsFactors=FALSE)
  
  i <- 0
  for(d in data){
    i <- i + 1
    df[i,] <- unlist(d)
  }
  return(df)
}

state.df <- apiDF(json$Results$series[[1]]$data)
ash.df <- apiDF(json$Results$series[[2]]$data)
Burlington.df <- apiDF(json$Results$series[[3]]$data)
char.df <- apiDF(json$Results$series[[4]]$data)
durham.df <-  apiDF(json$Results$series[[5]]$data)
faye.df<-apiDF(json$Results$series[[6]]$data)
gold.df <- apiDF(json$Results$series[[7]]$data)
green.df <- apiDF(json$Results$series[[8]]$data)
greenville.df<-apiDF(json$Results$series[[9]]$data)
hickory.df<-apiDF(json$Results$series[[10]]$data)
jackson.df<-apiDF(json$Results$series[[11]]$data)
newbern.df<- apiDF(json$Results$series[[12]]$data)
Raleigh.df<-apiDF(json$Results$series[[13]]$data)
rocky.df<-apiDF(json$Results$series[[14]]$data)
wilm.df<-apiDF(json$Results$series[[15]]$data)
winston.df<-apiDF(json$Results$series[[16]]$data)
## Change value type from character to numeric
state.df[,4] <- as.numeric(state.df[,4])
ash.df[,4] <- as.numeric(ash.df[,4])
Burlington.df[,4] <- as.numeric(Burlington.df[,4])
char.df[,4] <- as.numeric(char.df[,4])
durham.df[,4] <- as.numeric(durham.df[,4])
faye.df[,4] <- as.numeric(faye.df[,4])
gold.df[,4] <- as.numeric(gold.df[,4])
green.df[,4] <- as.numeric(green.df[,4])
greenville.df[,4] <- as.numeric(greenville.df[,4])
hickory.df[,4] <- as.numeric(hickory.df[,4])
jackson.df[,4] <- as.numeric(jackson.df[,4])
newbern.df[,4] <- as.numeric(newbern.df[,4])
Raleigh.df[,4] <- as.numeric(Raleigh.df[,4])
rocky.df[,4] <- as.numeric(rocky.df[,4])
wilm.df[,4] <- as.numeric(wilm.df[,4])
winston.df[,4] <- as.numeric(winston.df[,4])


###Identify the variable
state.df$pos <- ( "Statewide")
ash.df$pos <- ("Asheville")
Burlington.df$pos <- ("Burlington")
char.df$pos <- ("Charlotte")
durham.df$pos <- ("Durham")
faye.df$pos <-("Fayetville")
gold.df$pos <-("Goldsboro")
green.df$pos <-("Greensboro")
greenville.df$pos <-("Greenville")
hickory.df$pos <-("Hickory")
jackson.df$pos <-("Jacksonville")
newbern.df$pos <- ("New Bern")
Raleigh.df$pos <- ("Raleigh")
rocky.df$pos <- ("Rocky Mount")
wilm.df$pos <- ("Wilmington")
winston.df$pos <- ("Winston Salem")




####This puts all the individual sectors into the same panel
metro <- rbind(state.df, ash.df,Burlington.df,char.df,durham.df,faye.df,gold.df,green.df,greenville.df,hickory.df,jackson.df,newbern.df,Raleigh.df,rocky.df,wilm.df,winston.df)

####Splitting the month variable into numeric and character
allmetro<-data.frame(metro, character = substr(metro$period, 1, 1), numeric = substr(metro$period, 2, nchar(metro$period)))  
###turn month variable into numeric
library(data.table)
setDT(allmetro) 
allmetro[, mn := as.numeric(numeric)]
### combine month and year

library(zoo)
allmetro$Date <- as.yearmon(paste(allmetro$year, allmetro$mn), "%Y %m")

#### Difference relative to 12 months prior
library(dplyr)
allmetro<-allmetro %>%
  group_by(pos) %>%
  arrange(Date) %>%
  mutate(diff = value - lag(value,12))

allmetro$yesno <- ifelse(allmetro$diff>0, "Yes", "No")


####Growth rate
library(dplyr)
allmetro<-allmetro %>%
  group_by(pos) %>%
  arrange(Date) %>%
  mutate(perc = diff / lag(value,12))


area.color <- c("withcolor", NA, NA, NA,NA,NA,NA,NA,NA,NA,NA,NA,NA,NA,"withcolor",NA)
################################Now graph percentages
p2<-ggplot(data=subset(allmetro, year == "2021" & mn==10), aes(x=reorder(pos,perc), y=perc,fill = area.color,label = scales::percent(perc))) +
  geom_bar(stat="identity") +
  ylab("Year over year percentage change in employment") +
  theme_bw()+
  theme(axis.text.x = element_text(angle = 90,size=10, vjust = 0.5, hjust=1))+
  labs(title="Percentage change in employment in October 2021 relative to the same month in 2020",x = "Metropolitan area",
       y = "Percentage change") +
  theme(legend.title = element_blank()) +
  theme(legend.position = "none",
        panel.grid = element_blank(),
        panel.background = element_blank())+
  scale_y_continuous(labels = scales::percent_format(accuracy = 1L))+
  geom_text(aes(label = sprintf("%0.1f",perc*100,"%" )),vjust=-0.4,size=4)+
  scale_fill_discrete(name = "", labels = c("Decline", "Growth"))
p2




ggsave("allmetro.png") 

 
</code></pre>
</details>

As of October, 2021, Wilmington is only one of five metropolitan areas to have more jobs than in October, 2019. There is considerable variation across metropolitan areas with Asheville having 5.2% fewer jobs and Greenville having 2.4% more jobs than the same month last year .  
![allmetro19](https://user-images.githubusercontent.com/94587267/148270158-1ab30fb8-410b-4af2-9e08-2d2b33fca2d5.png)

<details>
<summary>Code</summary>

<pre><code lang="python">
###############################################Now relative to 2019
#### Difference relative to 24 months prior
library(dplyr)
allmetro<-allmetro %>%
  group_by(pos) %>%
  arrange(Date) %>%
  mutate(diff24 = value - lag(value,24))

allmetro$yesno <- ifelse(allmetro$diff>0, "Yes", "No")


####Growth rate
library(dplyr)
allmetro<-allmetro %>%
  group_by(pos) %>%
  arrange(Date) %>%
  mutate(perc24 = diff24 / lag(value,24))


area.color <- c("withcolor", NA, NA, NA,NA,NA,NA,NA,NA,NA,NA,NA,NA,NA,"withcolor",NA)
################################Now graph percentages
p2<-ggplot(data=subset(allmetro, year == "2021" & mn==10), aes(x=reorder(pos,perc24), y=perc24,fill = area.color,label = scales::percent(perc))) +
  geom_bar(stat="identity") +
  ylab("Year over year percentage change in employment") +
  theme_bw()+
  theme(axis.text.x = element_text(angle = 90,size=10, vjust = 0.5, hjust=1))+
  labs(title="Percentage change in employment in October 2021 relative to the same month in 2019",x = "Metropolitan area",
       y = "Percentage change") +
  theme(legend.title = element_blank()) +
  theme(legend.position = "none",
        panel.grid = element_blank(),
        panel.background = element_blank())+
  scale_y_continuous(labels = scales::percent_format(accuracy = 1L))+
  geom_text(aes(label = sprintf("%0.1f",perc24*100,"%" )),vjust=-0.3,size=4)+
  scale_fill_discrete(name = "", labels = c("Decline", "Growth"))
p2




ggsave("allmetro19.png") 

 
</code></pre>
</details>

# Main takeaways
The Wilmington economy has weathered the pandemic related disruption and has outperfromed most other metropolitan areas over the past year. Consumer facing industries are, however, still smaller than they were pre-pandemic. 


