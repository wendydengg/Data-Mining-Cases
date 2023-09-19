# EDA

## Case study 1: Audience Size
**Goal**: estimate the audience size of Wharton Talk Show Business Radio Powerd by the Wharton School <br>
**Approach**: calculate the proportion of the Wharton listeners to that of the Sirius listeners <br>
**Result**: The goal of the study is to estimate the Wharton Radio audience size by calculating the proportion of Wharton listeners to Sirius listeners times the total Sirius listener base. Data was collected via a survey on Amazon Mechanical Turk and ran for a length of 6 days. A total of 1764 responses were collected, which was reduced to 1725 after removal of incomplete/inaccurate responses. Given the total number of Sirius users (51.6 million), and the proportion of Sirius listeners to Wharton listeners in our sample (5.01%), it is estimated that there are 2.58 million listeners of Wharton Radio. A summary of the participants’ demographic information shows that people who took the survey are likely representative of the MTurk population, but they are not representative of the US population since people who take MTurk surveys are generally younger and more educated than the US population. Thus, the main limitation of this study is that we cannot extrapolate the results of the data to the general US population, but we can say that among the MTurk population, those who listen to Wharton radio is about 5.01% (p) of the population. <br>

## Case study 2: Women in Science
**Goal**: answer the following questions: <br>
1. Are women underrepresented in science in general? 
2. How does gender relate to the type of educational degree pursued? 
3. Does the number of higher degrees increase over the years? <br>
**Approach**: perform EDA <br>
**Results**: In general, we observe evidence that more males than females obtain science and engineering degrees, and this is most notable at the PhD level. Although the total number of science and engineering degrees appear to be increasing over time, the proportion of degrees conferred to females in these fields has remained constant or slightly decreased. One exception to this trend is the field of data science, where we observe increasing numbers of MS degrees being conferred to females over time. <br>

One potential concern with the data set is that it is limited to BS, MS, and PhD degrees - it is possible that inclusion of other degree types (such as BA and MA) could impact the results. It could also be of interest to incorporate career information, as receipt of a science/engineering degree does not necessarily mean the individual ended up working in that field (and vice-versa). Finally, it would be helpful to further specify the research question to specific fields, or better define what fields count as “science”, as we can see when looking at individual fields that trends are not consistent across fields. <br>

## Case study 3: Major League Baseball <br>
**Goal**: explore how payroll affects performance among Major Leadue Baseball teams <br>
**Approach**: perform EDA to find the relationship between payroll changes and performance <br>
**Result**: R-squared in the linear model for average win percentage and average log increases in payroll is relatively small, showing the linear regression model did not fit the data well and there was no strong linear relationship between log increases in payroll and win percentage. Same for the linear regression model when considering year and interaction between year and log increases in payroll. <br>

The correlation coefficient is -0.0946, indicating a low correlation.
