# Case-Study-1

This is a Case study that I've got from a recruitment of a Fintech company. The objective is mainly about the effectiveness of brand-new program which had has just launched. All the information about this company was ommited or replaced. I thought this case was very interesting so I decided to give it a try to see how far I could go. So, my work is kept updating every time I have more ideas or find a effective solutions for the questions.
I have attached the case study (raw data and requirements) file and my answers for you to have a better comparison. You can open the Excel file while reading this. Also, you can have a look for Python version of my answers (Part 1, Question 1 and 2 and Part 3, question 2 Part 3) with full step-by-step explanations.
So, let's dive in!

**Data cleansing**

First, I created a copy of the workbook to avoid losing the original data. I formatted all the data under Excel structured data types: number, date, and so on. E.g. in the Data. Loyalty Points worksheet, the Point Mechanism should only be in one type of value, which is number, and other information should be left for the column’ title.

This dataset is already a square dataset without missing values. According to the scheme, the Order_id included unique values (even one user can not have Order_id). ```Order_id``` is the primary key for the ```Transactions``` table. Then I only omitted the duplicate rows depending on the Order_id column, using ```Data``` > ```Remove Duplicates``` > tick on ```Order_id```. I leveraged ```Filter``` in ```Excel``` to check if there were any grammar misspellings. It appeared that everything was alright.

**Data analysing**

There are three parts with 7 questions. We will go through each question.

**Part 1 Question 1**

The requirement of this question is that the audience wants to see the Ranking of every user on a daily basis and asks for the number of Gold users on March 31st , 2022. 

The idea is to let the audience a specific date in which they want to see how many users in each rank name. I devided my work into two part: </br>
+ Calculating loyalty point for the users on 03-31-2022: 

First, I added a column named ```Loyalty Point``` to Transactions sheet. This step is really important, because loyalty point for each transaction is the base values to calculate other numbers (accumulated loyalty points, cashback cost, etc.) and you will see later. Perhaps, that' why it's hinted right at the first question. </br>
From the ```Loyalty_Point``` table in the ```Data. Loyalty Points``` worksheet, the loyalty points depends on the Group Service and GMV and is limited by Maximum Point. In other words, for each 1000 GMV, the user will have different loyalty point depending on their used service as long as the total loyalty point is less or equal to maximun point. First, I created a column ```Maximum Point``` in ```Transactions``` table using ```VLOOKUP```, then added ```Loyaly Point``` using:

```=IF(QUOTIENT([@GMV],1000)*[@[Point Mechanism per 1000 GMV]]<= [@[Maximum points]],QUOTIENT([@GMV],1000)*[@[Point Mechanism per 1000 GMV]],[@[Maximum points]])```

The ranking of users on a specific day depends on the accumulated loyalty points that they had. But there is one thing needs be noticed: loyalty points would expire after 30 days since the day transaction is made. I needed to know on the picked day (in this case is 03-31-2022), which transaction had their loyalty points usable/available. First, I created a new column named ```End date``` , which equals ```Start date``` + 30 days, to have the usable time length of all transactions. If 03-31-2022 lays between the usable time length of any transactions, that transaction will say "YES" to the quesion "is your loyalty point usable on 03-31-2022?", otherwise their loyalty point has expired or the transaction hasn't happened yet:

```=IF(AND([@[Start date]]<=Dashboard!$C$3,Dashboard!$C$3<[@[End date]]),"YES","NOT YET/NO")```

After that I used a filter to extract all the rows that included the “YES” value. At this point, I had a list of users who have usable loyalty points``` on 03-31-2022. Then, I summed loyalty point by user_id (```Classes_of_users``` table in ```Part1.Q1``` worksheet):

```SUMIFS(Transactions[Loyalty Points],Transactions[User_id],'Part 1.Q1'!A3,Transactions[Usable points?],"YES")```
+ Identifying Rank name 

I created ```Loyalty_Ranking``` table (```Loyalty Ranking``` worksheet) from the rules defined in the question requirement. Next, find the Rank name for each user by reverse ```VLOOKUP```:

```VLOOKUP(B2,CHOOSE({2,1},Loyalty_Ranking[Rank_name],Loyalty_Ranking[Loyalty Points]),2,TRUE)```

Finally, I counted the number of users for each ranking:

```COUNTIF(Classes_of_users[Class],"STANDARD")```
**Number of GOLD users on 03-31-2022 was 61**

**Part 1 Question 2**
This question asked about the total Cashback Cost in Feb, 2022. 

On 01-01-2022, a programm named "Money cashback" has been launched to give rewards to the user for their money spending. Casback Cost actually is the money that pay back to the users, therefore it is nothing but a spending of the fintech company to retain customer loyalty. </br>
According to the given information, Cashback cost can be calculated by multiplying %cashback with GMV. From the ```Transactions``` table (```Data. Transactions``` worksheet), we already have information of GMV, then we need to find the %cash back. Let's take a look at the Loyalty_benefits table (Data. Loyalty Benefits worksheet). %cashback is identified by service that users has spent and the Class_ID which is in ```Loyalty Ranking``` worksheet. </br>
First, I filted all transactions in Feb (```Calculated_cashback_cost``` table, ```Part 1.Q2``` worksheet) and their information of ```Order_id, User_id, GMV```, and ```Service Group```. Silimarly to the Question 1, I calculated loyalty point accumulated 30 days before the transaction day. I added a column named ```30 days before start date```. Here's the idea: summing all the loyalty point in the transactions that happend between ```30 days before start date``` and the ```Start date``` by for each user_id information in each row. You might find the duplicated values because they have take the same data entry. That's okay, if a user did more than two transaction in a day, that makes sense when their transactions had the same accumulated loyalty points:
```SUMIFS(Transactions[Loyalty Points],Transactions[User_id],[@[User_id]],Transactions[Start date],">="&[@[30 days before start day]],Transactions[Start date],"<"&[@[Start date]])```
Next, combine with ```Loyalty Ranking table``` to find Class ID:
```=VLOOKUP(G3,CHOOSE({3,2,1},Loyalty_Ranking[Class ID],Loyalty_Ranking[Rank_name],Loyalty_Ranking[Loyalty Points]),3,TRUE)```
To find %cashback, there are 2 conditions: ```Service Group``` and ```Class ID```. This means that we will do a VLOOKUP with two reference keys. The best way is to concatenate ```Service Group``` and ```Class ID``` in ```Loyalty_Ranking``` table to create a new only one key. Then combine with ```Calculated_cashback_cost``` table to find %cashback:
```IFERROR(VLOOKUP(F2&H2,Loyalty_benefits[[Cashback name]:[Cashback]],2,FALSE),0)```

Finnaly, I multiplied %cashback with GMV while ensuring the results always less or equal to 10000.
```IF(I2/100*E2<10000,I2/100*E2,10000)```
*** The total Cashback Cost in Feburary was VND 3,009,699.94 VND**

**Part 1 Question 3**

Since the program was launched, there were 13 weeks recorded. To measure the retention of the users, I used the users using the app the first time in week 1 and tracked them for the other next weeks to see if they had at least a transaction for the entire week. The same track was repeated for every week in the given period of time. However, if I have created a track lasting for 13 weeks, it could have led to blank values in the table. For example, a person who commited the app the first time at the 12th week would only have a chance to be tracked in the 2-week period of time. This person should not be included. For the fields of finance, it was suggested that an 8-week period of tracking customers’ retention would be suitable.

After having the list of unique customers in the first week, I checked if they used the app for the first time (new user) by searching if there was any transaction from the same person in the past.

```IF(COUNTIF('Data. Transactions'!$F$18815:$F$49176,[@[User_id]])=0,"YES","NO")```

I filtered out the user with NO result and tracked people with the YES for the next 8 weeks. Every week, I counted the number of users who used the services at least once. The same procedure was repeated until the last week which was also the end of March.

I gathered all the data and put it in a summarized table. Then, I calculated the % of users committing the app compared to the first week. Finally, I found the average of those percentages.

Additionally, to have a better understanding of the effectiveness of the Money cashback  program, I made the same procedures for the 13 weeks before the Loyalty program was launched, and compared it to the data after then. You can see the comparison on the chart in Part2. Q1 worksheet.

**Part 3 Question 2**

According to the rules, the winners have to be in DIAMOND rank for at least 20 consecutive days. Therefore, it's easy to spot out one trick: he last date for any user who wanted to win the game was March 12, 2022. This means on this day, the winners had already to be at DIAMOND. So here's the solution: filtering all the users who are DIAMOND on March 12, 2022 and then seeing how long their DIAMOND streaks will last. </br>
Similarly to what we have done one the first question, picking a date (in this case is 03-12-2022) and finding the rank name of the users on that day. 
```SUMIFS(Transactions[Loyalty Points],Transactions[User_id],F7,Transactions[Start date],"<44632",Transactions[Start date],">=44602")``` (44632 = 03-12-2022)
Using filter, we have a list of potential winners/. Next, we need to observe the streak these users. I created a table with column is user id  and row header is days in Mar. To be easier to observe, I let the cell to display "DIAMOND" or blank:
```IF(SUMIFS(Transactions[Loyalty Points],Transactions[User_id],$F2,Transactions[Start date],"<"&G$1,Transactions[Start date],">="&G$1-30)>=5000,"DIAMOND","")```
From the Gamification table, Part 3.Q2 worksheet: </br>
** There were 4 winners and 47662326 is the winners that had the special reward**

**Part 2 Question 2**

Marketplace4:

Let's compare marketplace 1 and marketplace 2, they have the highest frequencies and ```%GMV``` in the marketplace group, even the ```Cashback Cost``` is 0. The one thing is ```Marketplace``` service has the same low ```Loyalty Points``` as other groups.  Accordingly, putting a high percentage of cashback in these groups does not pay off, compared with the high cost that we have to pay. Marketplace group was the highest of frequency and ```GMV``` before the Money Cashback program was launched. However, we should not turn the percentages to 0. That is because after the launch of the cashback program, the marketplace contributed a significant influence, at 68%, in the growth of ```GMV``` and 34% in the increase of customer retention.

Proposed plan: set the ```marketplace4``` down to 3% and ```marketplace3``` down to 1%.

Data2: this type of cash back accounted for 9,4% of cost while it only made up 1,7% of ```GMV```. That might be because of the high number of transactions. The Cashback program made the users head for paying in the marketplace, rather than data as they used to. The attribution of data payment frequency decreased from 42% to 30%. However, in general, this area is still doing great to keep customers in 2022. We can still keep a cashback plan for ```data2```.
