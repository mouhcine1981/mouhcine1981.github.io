## What do we know about the current state of the Wilmington economy?
# Data sources
There are two main data sources **-Current Employment Statistics (CES) and Local Area Unemployment Statistics (LAUS)-**  that provide information about the labor market performance of sub-state areas. For the purposes of this entry, we will use both of them to describe the structure of the Wilmington Economy, where it stands relative to the previous year and pre-Covid times, and how it compares to other Metropolitan areas.

Nonfarm payroll employment estimates come from the monthly
**(CES)**, which surveys
approximately 144,000 businesses and government agencies, representing approximately 697,000 individual worksites nationally. In addition to overall nonfarm payroll, The Current Employment Statistics (CES) program produces detailed industry estimates of nonfarm employment, hours, and earnings of workers on payrolls.

The **(LAUS)** program calculates the state-level monthly unemployment rate, a modelbased estimate, using inputs from multiple sources, including
the Current Population Survey **(CPS)**, the **CES** program, the **unemployment insurance system**, and the **U.S. census**. The unemployment rate is used to
capture a snapshot of the labor force by providing an estimate of the percentage of the civilian noninstitutional population aged 16 years and over that is unemployed and looking for work. 

There main differences between the two the program are that the (CES) counts jobs as it is an establishment survey and the (LAUS) counts people as it mainly relies on a household surey. Importantly, the (CES) only counts individuals on payroll and therefore does not include self-employed individuals while the (LAUS) includes both payroll and self-employed individuals.


# Where does the Wilmington economy stand? 
1. According to data from the Current Employment Survey (CES), the Wilmington Metropolitan area had 135,000 Wage and Salary jobs as of October, 2021. This means it has 7,100 or 5.6% more jobs than the same month in 2020. Given the significant labor market disruptions that occured as a result of the pandemic, we also compare the current level of Wage and Salary jobs to the same month in 2019 and find that it is 1,500 or 1.1% higher indicating that unlike many parts of the state and the country it has exceeded its pre-pandemic employment levels. 

2. The labor force has 4,400 or 2.9% more individuals than the same month last year and 1,584 or 1% more than the pre-pandemic level in 2019. The state of North Carolina's labor force participation rate, on the other hand, as of November 2021 is still 3.6% below the November 2019 level.

3. The unemployment rate, as of October, 2021, stands at 3.1% which is 2.7 percentage points below the same month in 2020. The unemployment is even below October 2019 -pre-pandemic- which was 3.2%. 

# Nonfarm Wage and Salary Employment: 
Employment in November 2021 has already exceeded November 2019 levels. The decline in 2020 was short lived and employment has been on a steady trajectory, since then. 

![wilmi, out.width = '40%'](https://user-images.githubusercontent.com/94587267/146865971-33df220f-98d2-4b66-9443-31a4c69beffd.png)
<details>
 {::options parse_block_html="true" /}
<Code>
<Code>Click to expand!</Code>


 ## Total non farm SMS37489000000000001 
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
</Code>
</details>
{::options parse_block_html="false" /}


In the figure below, we decompose employment by year and show that the job losses were significant in 2020 but short lived as the recovery looks V-shaped unlike most communities across the country. Employment in 2021 has been above both 2020 and 2019 since August.

![yemp](https://user-images.githubusercontent.com/94587267/147981196-5c62eb9e-9f0d-4879-ae5d-7d665c11d83d.png)

# What about the Labor Force?
There have been recently been concerns about the great resignation, sizeable drops in the labor force participation rate, and labor shortages. In the figure below, we show that the labor force and employment both declined sharply in April 2020 but bounced shortly thereafter and have since recovered as they are both above the same levels in 2019. 
![lfemp](https://user-images.githubusercontent.com/94587267/147981227-f8625d2d-975e-4780-8b50-460d7baa6d17.png)
Relative to 2020, unsurprisingly, both employment and the labor force are appreciably higher given that the base period was depressed due the onset of the pandemic and the subsequent changes in behavior, and restrictions.
![j2](https://user-images.githubusercontent.com/94587267/148267258-fee9c096-b591-45e8-a4ac-d791496227ed.png)

When we compare monthly employment and the labor force relative to 2019, we find a slightly more complicated picture. Monthy employment in 2021 was still below the same month in 2019 until July when it turned positive. The labor force has been higher than 2019 for all months except May and June. 
![j3](https://user-images.githubusercontent.com/94587267/148264037-88b40116-124d-454a-89c9-9769d34e5374.png)

# Where is the employment growth coming from?
To better understand the sources of growth, we present below employment growth in the last 6 month of 2021 relative to the same months in 2020. It is clear that the sectors showing the most significant increase -such as Leisure and Hospitality- are the same one that suffered the most in the first few months of the pandemic. With the exception of the Federal Government, all other sectors are above their 2020 levels further confirming continued improvement in the labor market.
![allsec](https://user-images.githubusercontent.com/94587267/148265513-18c9231d-3345-42a2-8ffb-db9c091f5dd1.png)

# Which sectors are doing better than in 2019?
Perhaps, unsurprisingly, when we compare the employment levels of each sector to the same period in 2019, we find that the sectors -Leisure and Hospitality, Retail- most sensitive to consumer spending and individuals gathering indoors are the ones still showing the most scars from the pandemic. In addition to those two sectors, State government, Other services, and Manufacturing are also below their 2019 levels. Professional and Business Services, Transportation/Utilities, and Construction are consideraly above their 2019 levels. 
![rel2019](https://user-images.githubusercontent.com/94587267/148268842-0e9d3486-3f25-46ad-bd94-c90b3ca91fc0.png)

#  How does the Wilmington economic performance compare to other Metropolitan areas?
The Wilmington economy as we showed above seems to have recovered most of the jobs it lost as a result of the pandemic. As of October, 2021, Wilmington has 5.6% more jobs than the same month in 2020. That is the highest job gain across the state and is 2.2 percentage points above the state average.
![allmetro](https://user-images.githubusercontent.com/94587267/148269911-a1e4f796-4c9f-4b6e-b950-f35a43b00308.png)

As of October, 2021, Wilmington is only one of five metropolitan areas to have more jobs than in October, 2019. There is considerable variation across metropolitan areas with Asheville having 5.2% fewer jobs and Greenville having 2.4% more jobs than the same month last year .  
![allmetro19](https://user-images.githubusercontent.com/94587267/148270158-1ab30fb8-410b-4af2-9e08-2d2b33fca2d5.png)

# Main takeaways
The Wilmington economy has weathered the pandemic related disruption and has outperfromed most other metropolitan areas over the past year. Consumer facing industries are, however, still smaller than they were pre-pandemic. 

<details>
  <summary>Click to expand!</summary>

  ## Heading
  1. A numbered
  2. list
     * With some
     * Sub bullets
  <details>
