# Coding Camp Health Center Records 

My summer camp is revising its communicable disease protocol for summer 2021. In addition to public health guidance and expert advice, we hope to study our own health center records to see what we can learn from illnesses at camp in the past. This project develops a method for cleaning and classifying messy text-centered reports. It yields a tidy dataset and recommendations for further investigation.

## Contents
1.	[Introduction](https://github.com/amcgaha/coding-health-records#introduction)
2.	[Data Source](https://github.com/amcgaha/coding-health-records#data-source)
3.	[Methods & Tools](https://github.com/amcgaha/coding-health-records#methods--tools)
4.	[Procedure](https://github.com/amcgaha/coding-health-records#procedure)
5.	[Products](https://github.com/amcgaha/coding-health-records#products)
6.	[Next Steps](https://github.com/amcgaha/coding-health-records#next-steps)

## Introduction
The COVID-19 pandemic has caused many organizations to reevaluate their communicable disease protocols. My organization, a summer camp that did not open in 2020, is working on a safety plan to open this summer, which requires a careful study of best practices and risk management techniques.

We always followed basic measures, like washing hands and sanitizing surfaces. However, our knowledge of best practices has advanced significantly in the past year. In particular, we have learned the importance of social distancing, wearing masks, and isolating patients who are suspected to have a communicable disease. 

While they are now solidifying as social norms, these public health protocols seemed over-serious, even foreign, in the years before COVID-19. We certainly took any illness seriously, especially those accompanied by fever. However, a cold or cough was not a trigger for immediate isolation, mask wearing, or distancing, as would likely now be the case. As a result, we suspect our communicable disease protocol may not have been as robust as it needs to be during this and following summers.

Anecdotally, staff members -- including me -- can recall that getting a cold or stomach bug in the past has seemed normal. We have gotten sick, took some medication for relief of symptoms, and often kept working. The COVID-19 pandemic has shown us that these norms ignored some basic ways we can help reduce the spread of communicable diseases at camp.

As part of our broader planning strategy, we would like to know something about how diseases have manifested and spread at camp in the past. Knowing differences between staff and campers, for example, could help us generate smarter policies for each group. Watching how diseases manifest and change over time may also yield useful insights. 

To explore the distribution and character of communicable diseases at camp, we want to consult our health center visit records. We have data going back to 2013, which has been hand-entered by medical staff into our online system. Because this data is quite messy, considerable effort needs to be spent cleaning and processing before we can move on.

This project demonstrates the method for cleaning and classifying our health center records. Then, an overview of cold and flu symptoms is explored to generate some questions and ideas for further study.

Once we have the robust dataset, we can then conduct a more detailed study of cold and flu symptoms. We can also answer many more questions about health at our camp. We can identify trends, risks, and problem areas across many health categories, including injuries, bug bites, and even staff burnout or camper homesickness.

__Privacy Note__: Health records are extremely sensitive and regulated by law. Therefore, the data itself will remain completely private. Only the documentation of my process will be published.


## Data Source
The primary data source is the health center records downloaded from our data management system. This table includes records of all visits to the camp infirmary for the years 2013 – 2019. The number of records is 3,798.

This data is messy, inconsistent, and – thanks to a blank text box for entering notes – filled with shorthand, spelling errors, and other tough text problems. However, this data also contains a wealth of information about patient symptoms, treatment, temperature, time, and date and whether the patient was a camper or staff member.

In the early stages of investigation, an important fact was observed. While an average summer has about 573 health center visits recorded, one summer appears to have a serious problem of underreporting. The plot below shows that in 2016, medical staff reported about 223 fewer records than the average. This hole in the data should be considered when aggregating or comparing data from different summers.
 
![Image](https://github.com/amcgaha/coding-health-records/blob/main/total_records_image.png)

## Methods & Tools
Health center records are processed in __Python__ using the __pandas__ library. While the tables are censored for privacy, the process is documented in __Jupyter Notebooks__ so readers can follow along.

One script also flags records that require manual adjustment. These adjustments were made in __Microsoft Excel__, using filters and conditional formatting to make manual cleaning easier.

Once processed and classified, the completed dataset is uploaded to an existing __PostgreSQL__ database with information about years, sessions, campers, and households. (I created this database beforehand.) 

This project begins by adding information on staff to the database, because both campers and staff visit the health center as patients. The project ends by uploading a completed dataset to the database using __SQL__ commands in __DataGrip__.

## Procedure
### 1.	Add Staff to Database 
The data includes health records for both camper and staff visits to the health center. It is important to be able to differentiate between camper and staff visits, as well as connect other information, such as names and birthdates, to verify the data or explore other questions. However, the existing PostgreSQL database I created earlier only included information on campers at the time. Before we could work with the health records, staff needed to be added to the database.

After downloading a table with staff information from our commercial data management system, the following script was applied to clean and prepare the table. 

[View Notebook](https://github.com/amcgaha/coding-health-records/blob/main/add_staff_to_db.ipynb)

The table was then uploaded to the database and verified using SQL commands in DataGrip.

### 2.	Explore Health Center Records 
Next, the health center records were explored in Excel. The first step was to load the data in Excel and check for obvious errors, keywords to remember, and problems to expect in programming. 

One error that was fixed manually was a discrepancy in how temperatures were recorded. This is evidently a text box in our data management system rather than a dropdown or other controlled entry point. The data was reported in both Fahrenheit and Celsius, and as both floats and integers. Since there were only a few dozen reported temperatures, it was easy to adjust all records to be floats reported in Fahrenheit. 

Another error observed was an apparent limit in the character limit of text boxes in the data management system’s medical report form, which may not have been indicated to the medical staff completing reports. Each time a text note went longer than a certain character count, an error was produced. After the limit was exceeded, the system accidentally stored the extra text in other columns, such as temperature. This needed to be fixed with painstaking manual effort.

### 3.	Combine & Clean Records 
With the main errors fixed, the next step was to combine all the records together and create some uniformity with how records are stored in the database. The following script was applied to concatenate records together, add a few new columns, enforce data types, and rename columns to match the database format.

[View Notebook](https://github.com/amcgaha/coding-health-records/blob/main/preparing_records.ipynb)

### 4.	Fix Negative Report Bias 
After studying the data, a pervasive source of error became clear. Medical staff often report a _lack_ of symptoms in their notes in addition to positive reports. Because this project aims to classify records based on the text in the medical notes, this habit presents a problem. It would be easy to overestimate records for nausea, for example, because as many as half – or more – might be from medical staff reporting that a patient “denies nausea” or “states no nausea.” This problem impacts more than a third of all records in some way, which is over 1,000 records.

Several options were considered for how to deal with this challenge programmatically. One method, for example, would involve cutting text after certain keywords are raised. These types of solutions would save time and effort. However, keeping data integrity was the highest priority.  Any option considered would trade data integrity for efficiency, and it was important to be confident in the data in the end. Therefore, these errors were manually adjusted in Excel.

The following script was used to flag records for manual review.

[View Notebook](https://github.com/amcgaha/coding-health-records/blob/main/health_flagger.ipynb)

### 5.	Generate Keyword Dictionary 
While adjusting negative reports, notes were taken on how the medical reports tend to describe symptoms. From these notes, a dictionary of terms and synonyms was generated. 

[View Keywords](https://github.com/amcgaha/coding-health-records/blob/main/keyword_dictionary)


### 6.	Classify Records 
Using the keyword dictionary created in the last step, the following script was applied to the health center records. It searches the text for keywords and turns columns to True if it finds a match.

[View Notebook](https://github.com/amcgaha/coding-health-records/blob/main/health_classifier.ipynb)

### 7.	Add Records to Database 
The resulting dataset was uploaded to the database and verified using SQL commands. The verification process revealed an error on the first attempt, which was corrected. Steps 5-6 were edited and performed again. The error was that the classification code was accidentally left to the default setting, which made the text search case-sensitive. It therefore missed a range of lower- and upper-case combinations of text. With this error fixed, the verification tests were passed.

### 8.	Group for Exploration 
The following script was applied, which queries the database for a subset of the larger dataset focused on cold and flu symptoms. It then groups the resulting data by categories like year, day, and camper status, and exports a csv of the data for exploration in other platforms.

[View Notebook](https://github.com/amcgaha/coding-health-records/blob/main/grouping_records_for_viz%20(1).ipynb)

### 9.	Visualize Cold Symptoms 
Finally, cold and flu symptoms were visualized to offer some initial ideas and questions for further research.

## Products
### Database Update
The first product is an augmented PostgreSQL relational database. It now holds staff details and health center records from 2013 to 2019 in addition to the original information on dates, camp sessions, enrollment, and customer households. This product is useful because it provides a way to access health records that can be both private and more detailed. 

As demonstrated in this project, health records can now be downloaded in a completely anonymous form. This allows for simpler discussions about health data with wider audiences. For example, a version of the graphs presented here could be shared with staff members to emphasize the importance of washing hands before meals. 

Oppositely, the data can also be connected with more personal information in the database, as long as it is kept to strict internal use, to enrich further studies. It would allow, for example, a study on how infections have occurred among campers in specific cabins, or gender discrepancies in injuries. Connecting these data to the health records will now only require a few simple joins in SQL.

### Cold & Flu Symptom Subset
The second product is a subset of the data focused on cold and flu symptoms reported in health center visits. One question motivating this project was about the differences between camper and staff respiratory infections. This topic warrants a more detailed investigation, which is outside the scope of this project. However, to help demonstrate the utility of the new health records dataset, as well as generate ideas and questions for further exploration, cold and flu symptoms are visualized and described briefly in this section.

The cold and flu symptom dataset reports visits to the health center in which symptoms of colds or flu were mentioned in the report text. A diagnosis cannot be made retroactively based on symptoms. Cold and flu infections are similarly impossible to distinguish in this way. However, plotting the symptoms can provide an idea of how many people were feeling sick enough to seek medical care, and the patterns associated with those facts through time and between certain categories. 

The symptoms included in the dataset are:
* Cough 
* Shortness of breath or chest discomfort
* Fever or chills
* Headache
* Sore Throat
* Congestion
* Fatigue, feeling tired, body aches 

To begin the exploration process, two plots are presented below. The first displays the number of health center visits that have, on average, reported one cold or flu symptom. These reports are called Possible Infections. Because a visit only needs to report one symptom to count, there can only be low to medium confidence that a report actually captures a visit with a respiratory infection.

The plot shows this number for each day across the camp season, and it reflects the average for that day from 2013 – 2019. A line between the points shows a seven-day moving average. 

Because the camp schedule has remained the same during the entire data history, days can be standardized (Summer Day) to reveal patterns in how visits manifest during certain days. An overlay of camp session (Staff Training, Session 1, etc.) also shows which days belong to which sessions.

![Image](https://github.com/amcgaha/coding-health-records/blob/main/possible_infections.png)

While Possible Infections capture more data, they can also be skewed by single symptoms, such as simple headaches or fatigue. These are important symptoms of colds or the flu, but they can also have many causes. 

The second plot employs mostly the same parameters as the first. However, instead of Possible Infections, the second chart shows Likely Infections -- those that report at least two cold or flu symptoms. This plot shows visits that we can be more confident show real respiratory infections.

![Image](https://github.com/amcgaha/coding-health-records/blob/main/likely_infections.png)

In both charts, some interesting trends are apparent through time. In particular three “waves” of infections seem to occur toward the end of Session 2, 3, and 4. 

Differences between staff and campers are also observed. Most obvious is that a “third wave” of infections is less clear for staff. 

While these trends are certainly interesting, they should not lead to any conclusions just yet. A simple check of the rest of the dataset shows that all health center visits follow the same pattern, even with Possible Infections removed. The plot below shows that all other visits have a similar pattern: three distinct “waves” of visits toward the end of the middle sessions. 
This is a clue that the data requires careful study before drawing conclusions. 
 
![Image](https://github.com/amcgaha/coding-health-records/blob/main/visits_minus_possible.png)

## Next Steps
### 1.	Address Reporting Errors 
The data shows that visits to the health center were significantly underreported in 2016. While it is possible that this happened due to chance (i.e. fewer visits actually happened that year) the number reported falls below three standard deviations from the average number reported. That suggests another explanation is more likely. 

One possibility is that computer errors corrupted the data. Another possibility is a failure in the training process or materials for new medical staff. Both of these avenues should be investigated to prevent reporting errors in the future. It may also be smart to check records in the first weeks of camp, or in the first days of a new staff member’s arrival, to ensure their records are complete and accurate.

### 2.	Replicate with Treatment Text 
This project categorized only part of each health visit record, the text box labeled ‘ailment_text’, which describes symptoms. There is another text box that could be explored: the ‘treatment_text’ box. This includes categorical data (ex: ‘Rest’ or ‘Wound Care’) intermingled with text descriptions. The same methodology could be applied to transform this text box into useful categories. These could also be combined with symptoms to paint a more detailed picture.

### 3.	Describe Individual Years 
So far, only the average visits per camp day across all years has been explored. This method can provide a useful overview that is not skewed by the unique qualities of each summer. However, it may be that the unique qualities of each summer provide a richer story of how respiratory infections appear and spread over time. For example, while the average might show three peaks, looking at individual years might show that each year tends to have one big peak, which raises the average in that spot and creates a misleading overview.

### 4.	Investigate Wave Pattern 
One of the interesting observations that appears in the data is that the camper data shows three distinct waves of infections. Many epidemiological explanations can be applied to help understand this pattern. However, it is puzzling that all health center visits show this pattern -- even when possible infections are removed. This paradox must be resolved before the respiratory infection pattern can be properly understood.  

### 5.	Study Other Ailments 
One method to solve the wave paradox will be describing individual years as mentioned above. Another solution could be to explore the distribution of other categories, like stomach aches or blisters, to see which categories replicate the wave pattern and which do not. This will likely be a crucial component to understanding infections. It will also likely provide other useful insights related to those other ailments, which can be investigated in other studies. Other questions worth studying, for example, are how bug bites change over backpacking trips, or how many injuries are associated with certain programs.
