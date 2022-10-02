# Case-Study-1

This is a Case study that I've got from a recruitment of a Fintech company. The objective is mainly about the effectiveness of brand-new program which had has just launched. All the information about this company was ommited or replaced. I thought this case was very interesting so I decided to give it a try to see how far I could go. So, my work is kept updating every time I have more ideas or find a effective solutions for the questions.</br>
I have attached the case study (raw data and requirements) file and my answers for you to have a better comparison. You can open the Excel file while reading this. Also, you can have a look for Python version of my answers (includes Section A, Question 1&2 in Part 1 and Question 2 in Part 3 of Section B) with step-by-step explanation.
So, let's dive in!

## A. Data source

These date set are relatively clean and acquire just a little of adjustment: </br>
+ ```Data. Loyalty Points```: ```Point Mechanism``` column should only contain one type of data. Therefore I changed this column's values to the numbers of point and changed its header to ```Point Mechanism per 1000 GMV```. Similarly, values in ```Maximum Point Per Trans" were changed to the numeric type.</br>
+ ```Data. Loyalty benefits```: already clean. </br>
+ ```Data. Transactions```: I removed duplicates based on ```Order_id``` column because Order_id can only include unique values according to the scheme. I set the ```DATE``` column in date format. </br>
+ ```Data. Merchant```: already clean. (This data set actually wasn's touched during out later analysis). </br>

## B. Requirements

There are three parts with 7 questions. Let's go through each question.

### **Part 1: Data processing** </br>
#### **Question 1** </br>
(Part 1.Q1 worksheet) </br>
*"Combined with the 'Loyalty Points' table, add a column 'Loyalty Points' in 'Transactions' table with given rules. Then create another table named 'Loyalty Ranking' which must includes columns named Rank_name and Calculated_points to calculate the Rank of each user on daily basic. At the end of Mar 2022, how many user achived rank Gold?"* </br>
The requirement of this question is that the audience want to see the Ranking of every user on daily basis and ask for the number of Gold users on March 31st , 2022. </br>
My idea is to let the audience pick a specific date in which they want to see how many users in each rank name. I divided my work into two parts: </br>
+ Calculating loyalty point for the users on 03-31-2022: 

First, I added a column named ```Loyalty Point``` to Transactions sheet. This step is really important, because loyalty points for each transaction is the base value to calculate other numbers ( like accumulated loyalty points, cashback cost, etc.) that you will see later. Perhaps, that' why it's hinted right at the first question. </br>
From the ```Loyalty_Point``` table in the ```Data. Loyalty Points``` worksheet, the loyalty points depends on the Group Service and GMV and is limited by Maximum Point. For more specific, for each 1000 GMV, the user will have different loyalty point depending on their used service as long as the total loyalty point is less or equal to maximun point. </br>
Combining with ```Loyalty_Point``` table, I created a column ```Maximum Point``` in ```Transactions``` table using ```VLOOKUP```, then added ```Loyaly Point``` column using:

```IF(QUOTIENT([@GMV],1000)*[@[Point Mechanism per 1000 GMV]]<= [@[Maximum points]],QUOTIENT([@GMV],1000)*[@[Point Mechanism per 1000 GMV]],[@[Maximum points]])```

The ranking of users on a specific day depends on the accumulated loyalty points that they had. But there is one thing needs be noticed: loyalty points would expire after 30 days since the day transaction is made. I needed to know on the picked day (in this case is 03-31-2022), which transaction had their loyalty points usable/available. First, I created a new column named ```End date``` , which equals ```Start date``` adding 30 days, to have the usable time length of all transactions. If 03-31-2022 laid between the usable time length of any transactions, that transaction would return "YES" to the quesion "Is your loyalty point usable on 03-31-2022?", otherwise their loyalty point has expired or the transaction hasn't happened yet:

```IF(AND([@[Start date]]<=Dashboard!$C$3,Dashboard!$C$3<[@[End date]]),"YES","NOT YET/NO")```

After that, I used a filter to extract all the rows including the “YES” value. At this point, I had a list of users who have usable loyalty points on 03-31-2022. Then, I summed loyalty point by user_id (```Classes_of_users``` table in ```Part1.Q1``` worksheet):

```SUMIFS(Transactions[Loyalty Points],Transactions[User_id],'Part 1.Q1'!A3,Transactions[Usable points?],"YES")```
+ Identifying Rank name 

I created ```Loyalty_Ranking``` table (```Loyalty Ranking``` worksheet) from the rules in the question requirement. Next, matched the Rank name for each user by reverse ```VLOOKUP```:

```VLOOKUP(B2,CHOOSE({2,1},Loyalty_Ranking[Rank_name],Loyalty_Ranking[Loyalty Points]),2,TRUE)```

Finally, I counted the number of users for each ranking:

```COUNTIF(Classes_of_users[Class],"STANDARD")```

**Number of GOLD users on 03-31-2022 was 61 (F4 cell)**

#### **Question 2** </br>
(Part 1.Q2 worksheet) </br>
*"Combined with the 'Loyalty benefits' table and 'Loyalty Ranking' table, add columns '%cashback'  in 'Transactions' table and calculate the total cashback cost in Feb 2022."*

On 01-01-2022, a program named "Money cashback" has been launched to give rewards to the user for their money spending. Casback Cost actually is the money that pay back to the users, therefore it is nothing but an cost amount of the fintech company to retain customer loyalty. </br>
According to the given information, Cashback cost can be calculated by multiplying %cashback with GMV. From the ```Transactions``` table (```Data. Transactions``` worksheet), we already have information of GMV, then we need to find the %cash back. Let's take a look at the Loyalty_benefits table (Data. Loyalty Benefits worksheet). %cashback is identified by service that users has spent and the Class_ID which is in ```Loyalty Ranking``` worksheet. </br>
First, I filted all transactions in Feb (```Calculated_cashback_cost``` table, ```Part 1.Q2``` worksheet) and their information of ```Order_id, User_id, GMV```, and ```Service Group```. Silimarly to the Question 1, I calculated loyalty point accumulated 30 days before the transaction day. I added a column named ```30 days before start date```. Here's the idea: summing all the loyalty point in the transactions that happend between ```30 days before start date``` and the ```Start date``` by for each user_id information in each row. You might find the duplicated values because they have take the same data entry. But that's okay, if a user had more than two transaction in a day, that makes sense when their multiple transactions had the same accumulated loyalty points:

```SUMIFS(Transactions[Loyalty Points],Transactions[User_id],[@[User_id]],Transactions[Start date],">="&[@[30 days before start day]],Transactions[Start date],"<"&[@[Start date]])```

Next, I combined with ```Loyalty Ranking table``` to find Class ID:

```=VLOOKUP(G3,CHOOSE({3,2,1},Loyalty_Ranking[Class ID],Loyalty_Ranking[Rank_name],Loyalty_Ranking[Loyalty Points]),3,TRUE)```

To find %cashback, there are 2 conditions needed to meet: ```Service Group``` and ```Class ID```. This means that we will do a VLOOKUP with two reference keys. The best way is to concatenate ```Service Group``` and ```Class ID``` in ```Loyalty_Ranking``` table to create a new only one key (```Cashback name``` column in ```Data. Loyalty benefits``` worksheet) then combine with ```Calculated_cashback_cost``` table to find %cashback:

```IFERROR(VLOOKUP(F2&H2,Loyalty_benefits[[Cashback name]:[Cashback]],2,FALSE),0)```

Finnaly, I multiplied %cashback with GMV while ensuring the results always less or equal to 10000.

```IF(I2/100*E2<10000,I2/100*E2,10000)```

**The total Cashback Cost in Feburary was VND 3,009,699.94 VND (M2 cell)**

#### **Question 3** </br>
(Part 1.Q3 worksheet) </br>
*"Design a weekly retention charts of since the program was lauched to monitor."*

All we need to do is display the retention of the new users over week, start from Week 1. Weekly retention means during a week, if a user used services at least once, the customer's retention in that week would be counted. </br>
From this definition, I first pulled the list of users in Week 1 (Jan 1 to Jan 7), then checked if they had any transaction before Jan 1. If they did, the result would have returned "NO" to the question "New user?", otherwise "Yes". 

```IF(COUNTIF('Data. Transactions'!$F$18815:$F$49176,[@[User_id]])=0,"YES","NO")```

Gather users with "Yes" answer, I had list of new users in Week 1 (D column, Part 2.Q1(data-Q42021) worksheet)  </br>
From this list, I tracked their retention streak over weeks . If they had at least one transaction in any week, their retention would be retained by return "Yes" to the question "Retention?". Then, the same procedure was repeated to the Week 2, Week 3 to the end of Mar (Week 13). 
OK, I know you might wondering when my tracking ends in Week 9. I will make this point clear. </br>
Since the program was launched, there were 13 weeks recorded. However, let's imagine, if a user was new in Week 12, they would only have the retention streak last for 2 weeks. These users should be excluded from tracking. Also, I have done a research saying that a 8-week tracking lenghth was used regularly in weekly retention monitoring. Therfore, I decided to cut the tracking streaks down to 8-week lengths. </br>
Because of that, the next retention tracking length starts from Week 2 to Week 10.  </br>
Finally, I counted the number of "Yes" answer by week, I had the weekly retention streak.

```COUNTIF(E6:E127,"YES")```

All weekly retention streak were garthered and transfered to percentages, that's my weekly customer retention charts (```Retention_chart_2022b``` table). To have a more vibrant summary, I visualize ```Retention_chart_2022b``` to a line chart. 

### **Part 2: Analyze and comment** </br>
#### **Question 1** </br>
(Part 2.Q1 worksheet) </br>
*"User retention and transaction behavior (Is there any trend?) since Loyalty program launched. Do you have any advice for the Marketing department in designing promotion campaigns to increase user retention's performance monthly?"*

To access Customer Retention before and after the launch of Money Cashback program, I created a weekly retention chart for the same period of time in 2021, from Oct 2 to the end of 2021. </br>
This is the chart of comparison of before and after Money Cashback program was launched.
![image](https://user-images.githubusercontent.com/106227875/193440552-42496746-3942-47fa-bff2-2e5496eda409.png) </br>
Since the program was launched, the average customer retention has improved significantly. Even when we compare the retention rate of each week in 2022 with the average retention of Q4 2021, there always shows a dominant for weeks in 2022.

#### **Question 2** </br>
(Part 2.Q2 worksheet) </br>
*"The business is facing increasing amount of cashback cost as well as  GMV since launched. However, we want to optimize the cost but still want to keep growth of GMV and increase the retention rate.  Based on data given, please propose ideas to change the schemes of Loyalty benefits and Loyalty Points to alleviate the cost amount."*

+ Marketplace4:

Let's compare marketplace 1 and marketplace 2, they have the highest frequencies and ```%GMV``` in the marketplace group, even the ```Cashback Cost``` is 0. The one thing is ```Marketplace``` service has the same low ```Loyalty Points``` as other groups.  Accordingly, putting a high percentage of cashback in these groups does not pay off, compared with the high cost that we have to pay. Marketplace group was the highest of frequency and ```GMV``` before the Money Cashback program was launched. However, we should not turn the percentages to 0. That is because after the launch of the cashback program, the marketplace contributed a significant influence, at 68%, in the growth of ```GMV``` and 34% in the increase of customer retention.

Proposed plan: set the ```marketplace4``` down to 3% and ```marketplace3``` down to 1%.

+ Data2: 

This type of cashback accounted for 9,4% of cost while it only made up 1,7% of ```GMV```. That might be because of the high number of transactions. The Cashback program made the users head for paying in the marketplace, rather than data as they used to. The attribution of data payment frequency decreased from 42% to 30%. However, in general, this area is still doing great to keep customers in 2022. We can still keep a cashback plan for ```data2```.

### **Part 3: Extended questions** </br>
#### **Question 2** </br>
(Part 3.Q2 worksheet) </br>
*"Gamification is usually a sensible option for apps to raise users’ stickiness. In our loyalty program development strategy, we also plan to hold a small game for users. The rule is simple: any users who can maintain a 20-day or longer streak of being in the DIAMOND ranking is a winner (in other words, winners are users who have total loyalty points greater than or equal to 5,000 for at least 20 consecutive days). We also want to give a special reward for the user(s) who can maintain the longest streak. Could you help us to calculate how many winners were there during the last thirty days in the given data (March 01 - March 31) and who was/were the one(s) boasting the longest streak during that time?"*

According to the rules, the winner has to be in DIAMOND rank for at least 20 consecutive days. Therefore, it's easy to spot this trick: the last date for any user who wanted to win the game was March 12, 2022. This means on that day, the winners had already to be DIAMOND. So here's the solution: filtering all the users who are DIAMOND on March 12, 2022 and then seeing how long their DIAMOND streaks will last. </br>
Similarly to what we have done one the first question, picking a date (in this case is 03-12-2022) and finding the rank name of the users on that day. 

```SUMIFS(Transactions[Loyalty Points],Transactions[User_id],F7,Transactions[Start date],"<44632",Transactions[Start date],">=44602")``` (44632 = 03-12-2022)

After filtering, we have a list of potential winners. Next, we need to observe the streak these users. I created a table with columns were ids of users and rowd were days in Mar. To be easier to observe, I let the cell to display "DIAMOND" or blank:

```IF(SUMIFS(Transactions[Loyalty Points],Transactions[User_id],$F2,Transactions[Start date],"<"&G$1,Transactions[Start date],">="&G$1-30)>=5000,"DIAMOND","")```

From the Gamification table, Part 3.Q2 worksheet: </br>
**There were 4 winners and 47662326 is the winner that had the special reward (F11:G12 range)**

