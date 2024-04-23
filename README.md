# SCOUTING ANALYSIS FOR MANCHESTER UNITED FC

## Table of contents
- [Project Overview](#project-overview)
- [Data source](#Data-source)
- [Analysis Tools used](#Analysis-Tools-used)
- [Methodology](#Project-Methodology)
- [Conclusion](#Conclusion)

## Project Overview
Manchester United's underwhelming performance in the 2023-24 Premier League season, particularly in terms of goal scoring, necessitates the identification of potential striker targets to bolster their attack and increase their competitiveness.

## Data source
- Players performance data was obtained from  FBREF (https://fbref.com/en/comps/Big5/stats/players/Big-5-European-Leagues-Stats)
- Players market values and contract expiry dates were obtained from transfer market (https://www.transfermarkt.co.in/)

## Analysis Tools used
- Python (BeautifulSoup, pandas, numpy)
- SQL
- Power BI

##  Project Methodology
### Data scraping - Python
-  The Python library BeautifulSoup was employed to efficiently extract the desired table containing player information.
-  The HTML content of the relevant FBREF page was retrieved and parsed using BeautifulSoup to identify and extract the specific table containing player data for further analysis.
``` python
# Importing the required libraries
from bs4 import BeautifulSoup
import requests
import pandas as pd

# Retrieving the table from the website 
URL = 'https://fbref.com/en/comps/Big5/stats/players/Big-5-European-Leagues-Stats'
page = requests.get(URL)
soup = BeautifulSoup(page.text,'html')
table = soup.find('table')

# Required Column headings
title = table.find_all('th')
headers = [titles.text.strip() for titles in title ]
Column_title = []
for i in headers:
    if (i != ''):
        Column_title.append(i)
        
Column_head = Column_title[6:43]

# Creating a Data frame
column_data = table.find_all('tr')
headers = pd.DataFrame(columns = Column_head)

for row in column_data:
    row_data  = row.find_all('td')
    individual_row_data = [data.text.strip() for data in row_data]
    if(individual_row_data != []) :
        datum = individual_row_data
        length = len(headers)
        headers.loc[length] = datum
stats = pd.DataFrame(data = headers)

```
### Data cleaning/preparation - python
1) Removing the duplicate values
   
  ```python
  stats = stats.drop_duplicates(subset = ['Player'])
  ```
2) Changing the duplicate columns names
   
  ```python
  stats.columns.values[26:36] = ['Gls per 90','Ast per 90','G+A per 90','G-PK per 90','G+A-PK per 90','xG per 90','xAG per 90','xG+xAG per 90','npxG per 90','npxG+xAG per 90']
```

3) Altering the 'Nation' and 'Age' columns
   
      ```python
      stats['Nation'] = Stats['Nation'].apply(lambda x: x.split(' ')[1] if len(x.split(' ')) > 1 else x)
      stats['Age'] = Stats['Age'].apply(lambda x:x.split('-')[0])
      ```
5) Extracting the required stats and changing the datatypes
   
   ```python
   # extracting
   required_stat =  pd.DataFrame(stats[['Player','Age','Pos','Nation','Squad','Comp','MP','Starts','Min','90s','Gls','Ast','G+A','G-PK','xG','xAG','Gls per 90','Ast per 90','xG per 90','xAG per 90']])
   # changing datatypes
   datatypes = {'Player':str,'Age' : int,'Pos':str,'Nation':str,'Squad':str,'Comp':str,'MP':int,'Starts':int,'Min':int,'90s':float,'Gls':int,'Ast':int,'G+A':int,'G-PK':int,'xG':float,'xAG':float,'Gls per 90':float,'Ast per 90' : float,'xG per 90':float,'xAG per 90':float}
   required_stat = required_stat.astype(datatypes)
   ```
### Data analysis/exploration - Python, SQL
The analysis was done according to the following rules :
- The Player should be a FW
- Age should be below 30
- The G/xG ratio should be greater than or equal to 1
- The starts ratio should be greater than 0.60
- The player should have scored more than 10 goals

```python

striker = required_stat[required_stat['Pos'] == 'FW']

striker = striker[striker['Age']<30]

striker['G/xG'] = striker['Gls']/striker['xG']
striker = striker[striker['G/xG'] >= 1]

striker['Start%'] = striker['Starts']/striker['MP']
striker = striker[striker['Start%']>=0.60]

mean_gls = striker['Gls'].mean()
striker = striker[striker['Gls'] > mean_gls]
```

Then the market value and the contract expiry date of the players were found and added to a seperate table using SQL.
```sql
CREATE TABLE strikers(
Player char(30) PRIMARY KEY NOT NULL,
market_value varchar(10) NOT NULL,
contract_end int NOT NULL);

INSERT INTO final_value(Player,market_value,contract_end)
VALUES('Serhou Guirassy','₹320 Cr',2026),
('Kylian MbappÃ©','₹1440 Cr',2024),
('Lautaro MartÃ­nez','₹880 Cr',2026),
('LoÃ¯s Openda','₹480 Cr',2028),
('Ollie Watkins','₹520 Cr',2028),
('Alexander Isak','₹560 Cr',2028),
('Artem Dovbyk','₹240 Cr',2028),
('Jonathan David','₹400 Cr',2025),
('Borja Mayoral','₹120 Cr',2027),
('Jarrod Bowen','₹400 Cr',2030),
('Alexander SÃ¸rloth','₹120 Cr',2028),
('Bukayo Saka','₹1,040 Cr',2027),
('Gorka Guruzeta','₹120 Cr',2028),
('Victor Osimhen','₹880 Cr',2026),
('Hugo Duro','₹128 Cr',2026),
('Vinicius JÃºnior','₹1,200 Cr',2027),
('Youssef En-Nesyri','₹144 Cr',2025);
```
Both the tables were joined 
```sql
 SELECT * from final_strikers as fs INNER JOIN final_value as fv on fs.Player = fv.Player;
```
### Data visualization
The data such as Player's Goals, G-PK, assists, Goals + assists, market value, contract expiry year, current club's location was visulalized.

<img width="630" alt="goals_assists" src="https://github.com/Guruprasad712/DATA_PRJECT_2/assets/160844022/4bdd76d8-851e-49a6-bffc-1faaed082474"><img width="599" alt="player_location" src="https://github.com/Guruprasad712/DATA_PRJECT_2/assets/160844022/7e452df9-b4a2-46a9-a81e-8cbeda1d52ab">

<img width="631" alt="market_value and contract" src="https://github.com/Guruprasad712/DATA_PRJECT_2/assets/160844022/d548612d-24ab-4bcd-872c-a3f3a1788308">

<img width="630" alt="goals_assists" 
src="https://github.com/Guruprasad712/DATA_PRJECT_2/assets/160844022/26ff9f37-b699-4eeb-9f0a-73952bc631de">

## Conclusion

Leveraging the data analysis techniques, we were able to identify 17 promising striker targets from a pool of over 2600 players across Europe's top 5 leagues. This shortlist provides a valuable starting point for Manchester United FC to explore further, allowing them to focus their scouting efforts on strikers with the skills and performance history that could significantly contribute to their goal scoring and overall competitiveness in the upcoming seasons.











