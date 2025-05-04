This Python script is a job listing scraper and database uploader, specifically designed to:

Scrape job data from a UAE-based job listing website (likely Naukrigulf) using Selenium WebDriver.

Save the data in a CSV file.

Open each job link from that CSV and extract detailed job information.

Store that information into a MySQL database table.

Let's go section by section to understand it in detail:

1. Imports and Setup
python
Copy
Edit
from selenium import webdriver
import pandas as pd
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
import time
These lines import all necessary libraries:

selenium: for browser automation.

pandas: to create and manage dataframes.

webdriver_manager: automatically downloads the correct Chrome driver version.

time: to introduce delays (though not used here explicitly).

2. Job Listing Scraping (Page-wise)
python
Copy
Edit
job_title1 = []
job_exp1 = []
job_loc1 = []
job_link1 = []
These lists store scraped data: job title, experience, location, and job link.

Loop Over Pages
python
Copy
Edit
for i in range(1, 24):
    web = webdriver.Chrome(service=Service(ChromeDriverManager().install()))
    url = 'please mention URL of web page'
    if(i == 1): 
        web.get(url)
    else:
        url = url + (str(i))
        web.get(url)
Loops through 23 pages of the job listing website.

For the first page, uses the base URL.

For subsequent pages, appends the page number.

Note: 'please mention URL of web page' is a placeholder – you need to insert the actual job search results URL.

Job Info Scraping
python
Copy
Edit
products = web.find_elements(by='xpath', value='//div[contains(@class, "ng-box srp-tuple")]')
Grabs all job listings (tuples) on the page.

Then inside the loop:

python
Copy
Edit
job_title1.append(product.find_element(...).text)
job_exp1.append(product.find_element(...).text)
job_loc1.append(product.find_element(...).text)
Grabs title, experience, location for each job.

Job Link Extraction
Tries two XPaths for link buttons – one for listings with logo and one without logo:

python
Copy
Edit
link_element = product.find_element(...logo-false)
job_link1.append(link_element.get_attribute('href'))
If neither is found, stores "N/A".

Finally, the browser is closed:

python
Copy
Edit
web.quit()
3. Save to CSV
python
Copy
Edit
jobs1 = pd.DataFrame({"Title": job_title1, "Experience": job_exp1, "Location": job_loc1, "Link": job_link1})
jobs1.to_csv('UAE_Jobs.csv', index='False')
Creates a DataFrame and exports the scraped job list to a CSV file named UAE_Jobs.csv.

4. MySQL Table Setup
python
Copy
Edit
import mysql.connector
mydb = mysql.connector.connect(
  host="localhost",
  user="root",
  password="***********",
  database="UAEJOBS_SQL"
)
mycursor = mydb.cursor()
mycursor.execute(
    "CREATE TABLE Link_info(...)")
Connects to a local MySQL database named UAEJOBS_SQL.

Creates a table Link_info with relevant columns (link, organization, date, experience, location, etc.)

5. Link-wise Detailed Scraping and SQL Insert
python
Copy
Edit
df = pd.read_csv('UAE_Jobs.csv')
df = df['Link']
Reads the CSV and isolates the Link column.

Loop Through Each Link:
python
Copy
Edit
for link in df:
    web = webdriver.Chrome(...) 
    web.get(link)
For each job link, opens the webpage and scrapes detailed job data.

Extract Company & Date
python
Copy
Edit
organization = web.find_element(...).text
Date = web.find_element(...).text
Extract Job Attributes (Experience, Salary, etc.)
python
Copy
Edit
details = web.find_element(...)
heads = details.find_elements(by='xpath', value='.//p[contains(@class, "head")]')
values = details.find_elements(by='xpath', value='.//p[contains(@class, "value")]')
It iterates over heads and values to match data:

Example: if head = "Experience", get corresponding value.

If none are matched, assigns "NA" to each field.

Insert Into MySQL Table
python
Copy
Edit
mycursor.execute("INSERT INTO Link_info ...", (...))
mydb.commit()
Each scraped job is inserted into the MySQL table.

Handling Failures
python
Copy
Edit
except:
    not_working_link_Number.append(c)
finally:
    web.quit()
If a link fails to load or parse, it stores the index in not_working_link_Number.

Summary of What This Code Does
Step	Description
1	Loops through job listing pages (23 pages)
2	Extracts title, experience, location, and job link
3	Saves this data to a CSV file
4	Creates a MySQL table for storing job details
5	For each job link: opens page, extracts company name, date, experience, location, etc.
6	Saves these detailed job records into MySQL
7	Tracks and stores failed link indexes for review

Security Tips
Do not hardcode passwords (password="***********").

Consider using .env files for credentials.

Use try-except blocks wisely to catch specific exceptions.
