from selenium import webdriver
import pandas as pd
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
import time

-----------------------------------------------------------------------------------------------------------------------------

job_title1=[]
job_exp1=[]
job_loc1=[]
job_link1=[]
for i in range(1,24):
    web = webdriver.Chrome(service=Service(ChromeDriverManager().install()))
    url='please mention URL of web page'
    if(i==1): 
        web.get(url)
    else:
        url=url+(str(i))
        web.get(url)
    products=web.find_elements(by='xpath',value='//div[contains(@class, "ng-box srp-tuple")]')
    for product in products:
        job_title1.append(product.find_element(by='xpath',value='.//p[contains(@class, "designation-title")]').text)
        job_exp1.append(product.find_element(by='xpath',value='.//li[contains(@class, "info-exp")]').text)
        job_loc1.append(product.find_element(by='xpath',value='.//li[contains(@class, "info-loc")]').text)   
        try:
            link_element=product.find_element(by='xpath',value='.//a[contains(@class, "info-position logo-false ")]')
            job_link1.append(link_element.get_attribute('href'))
        except:
            try:
                link_element=product.find_element(by='xpath',value='.//a[contains(@class, "info-position logo-true ")]')
                job_link1.append(link_element.get_attribute('href'))
            except:
                job_link1.append("N/A") 
    web.quit()

-----------------------------------------------------------------------------------------------------------------------------

jobs1=pd.DataFrame({"Title":job_title1, "Experience":job_exp1, "Location":job_loc1, "Link":job_link1})
jobs1.to_csv('UAE_Jobs.csv',index='False')

-----------------------------------------------------------------------------------------------------------------------------

#pip install mysql-connector-python
import mysql.connector
mydb = mysql.connector.connect(
  host="localhost",
  user="root",
  password="***********",
  database="UAEJOBS_SQL"
)
mycursor = mydb.cursor()
mycursor.execute(
    "CREATE TABLE Link_info(link VARCHAR(255), organization VARCHAR(255), Date VARCHAR(255), Experience VARCHAR(255), Job_Location VARCHAR(255), Education VARCHAR(255), Nationality VARCHAR(255), Gender VARCHAR(255), Vacancy VARCHAR(255), Monthly_Salary VARCHAR(255), Benefits VARCHAR(255))")


-----------------------------------------------------------------------------------------------------------------------------

df = pd.read_csv('UAE_Jobs.csv')
df=df['Link']
not_working_link_Number=[]
c=0
for link in df:
    try:
        web = webdriver.Chrome(service=Service(ChromeDriverManager().install())) 
        web.get(link)
        try:
            organization=web.find_element(by='xpath', value='.//a[contains(@class, "info-org")]').text
        except:
            organization=web.find_element(by='xpath', value='.//p[contains(@class, "info-org")]').text
        Date=web.find_element(by='xpath', value='//p[contains(@class, "time-stamp")]').text
        details=web.find_element(by='xpath',value='//div[contains(@class, "candidate-profile")]')
        heads=details.find_elements(by='xpath',value='.//p[contains(@class, "head")]')
        values=details.find_elements(by='xpath',value='.//p[contains(@class, "value")]')
        for i in range(0,len(heads)):
            if(heads[i].text=='Experience'):
                Experience=values[i].text
            elif(heads[i].text=='Job Location'):
                Location=values[i].text
            elif(heads[i].text=='Education'):
                Education=values[i].text 
            elif(heads[i].text=='Nationality'):
                Nationality=values[i].text
            elif(heads[i].text=='Gender'):
                Gender=values[i].text
            elif(heads[i].text=='Vacancy'):
                Vacancy=values[i].text
            elif(heads[i].text=='Monthly Salary'):
                Salary=values[i].text
            elif(heads[i].text=='Benefits'):
                Benefits=values[i].text
            else:
                Experience='NA'
                Location='NA'
                Education='NA'
                Nationality='NA'
                Gender='NA'
                Vacancy='NA'
                Salary='NA'
                Benefits='NA'
        mycursor.execute("INSERT INTO Link_info (link, organization, Date, Experience, Job_Location, Education, Nationality, Gender, Vacancy, Monthly_Salary, Benefits) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)", (link, organization, Date, Experience, Location, Education, Nationality, Gender, Vacancy, Salary, Benefits))
        mydb.commit()
    except:
        not_working_link_Number.append(c)
    finally:
        web.quit()
    c=c+1

-----------------------------------------------------------------------------------------------------------------------------
