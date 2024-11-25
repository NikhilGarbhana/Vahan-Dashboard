# Vahan-Dashboard
A web scraping project on Indian government website, to fetch the data of vehicles sold in India at RTO level.

---

### Step 1 Data Collection
Using python and selenium scrape the data from the website and store it in local server

```python
from selenium import webdriver
from bs4 import BeautifulSoup
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import Select
import time
import pandas as pd
import csv
import re
import warnings
warnings.filterwarnings(action='ignore', category=UserWarning)

url = "https://vahan.parivahan.gov.in/vahan4dashboard/vahan/view/reportview.xhtml"

def table_data():
    rows=[]
    vahan_soup = BeautifulSoup(driver.page_source, 'html5lib')

    table_body = vahan_soup.find('tbody', id='groupingTable_data')
    for i in table_body.find_all('tr'):
        row = []
        for j in i.find_all('td'):
            title = j.text
            row.append(title)
        rows.append(row)

    return rows

def rto_filters():
    filter_open = wait.until(EC.visibility_of_element_located((By.XPATH, '//*[@id="filterLayout-toggler"]/span/a/span')))
    filter_open.click()
    time.sleep(2)

    filter1 = wait.until(EC.visibility_of_element_located((By.XPATH,'//*[@id="VhCatg"]/tbody/tr[1]/td/label')))
    filter1.click()

    filter2 = wait.until(EC.visibility_of_element_located((By.XPATH,'//*[@id="VhCatg"]/tbody/tr[2]/td/label')))
    filter2.click()

    filter3 = wait.until(EC.visibility_of_element_located((By.XPATH,'//*[@id="VhCatg"]/tbody/tr[3]/td/label')))
    filter3.click()

    filter_refresh = wait.until(EC.visibility_of_element_located((By.XPATH, '//*[@id="j_idt74"]/span[2]')))
    filter_refresh.click()
    time.sleep(5)

    filter_close = wait.until(EC.visibility_of_element_located((By.XPATH, '//*[@id="filterLayout"]/div[1]/a/span')))
    filter_close.click()

def primary_filters():
    y_axis = wait.until(EC.visibility_of_element_located((By.XPATH, '//*[@id="yaxisVar"]/div[3]')))
    y_axis.click()
    time.sleep(1)

    y_axis_value = "Maker"
    select_y_axis = wait.until(EC.visibility_of_element_located((By.XPATH,'//li[text()="'+ y_axis_value +'"]')))
    select_y_axis.click()
    time.sleep(1)

    x_axis = wait.until(EC.visibility_of_element_located((By.XPATH, '//*[@id="xaxisVar"]/div[3]')))
    x_axis.click()
    time.sleep(1)

    x_axis_value = "Month Wise"
    select_x_axis = wait.until(EC.visibility_of_element_located((By.XPATH,'//li[text()="'+ x_axis_value +'"]')))
    select_x_axis.click()
    time.sleep(1)

    # year = wait.until(EC.visibility_of_element_located((By.XPATH, '//*[@id="selectedYear"]/div[3]')))
    # year.click()
    # time.sleep(1)

    # year_value = "2022"
    # select_year = wait.until(EC.visibility_of_element_located((By.XPATH,'//li[text()="'+ year_value +'"]')))
    # select_year.click()
    # time.sleep(1)

if __name__ == '__main__':

    # dataframe = pd.DataFrame(columns=["State", "Rto", "S No", "Maker", "JAN", "FEB", "MAR", "APR", "MAY", "JUN", "JUL", "AUG", "SEP", "OCT", "NOV", "Dec","Total"]) 
    # dataframe_ev = pd.DataFrame(columns=["State", "Rto", "S No", "Maker", "JAN", "FEB", "MAR", "APR", "MAY", "JUN", "JUL", "AUG", "SEP", "OCT", "NOV", "Dec","Total"])

    driver = webdriver.Chrome()
    driver.get(url)
    driver.maximize_window()
    wait = WebDriverWait(driver, 5, 1)

    primary_filters()

    for i in range(1,11):

        dataframe = pd.DataFrame(columns=["State", "Rto", "S No", "Maker", "JAN", "FEB", "MAR", "APR", "MAY", "JUN", "JUL"]) #, "FEB", "MAR", "APR", "MAY", "JUN", "JUL", "AUG", "SEP", "OCT", "NOV", "DEC","Total"
        dataframe_ev = pd.DataFrame(columns=["State", "Rto", "S No", "Maker", "JAN", "FEB", "MAR", "APR", "MAY", "JUN","JUL"]) # , "FEB", "MAR", "APR", "MAY", "JUN", "JUL", "AUG", "SEP", "OCT", "NOV", "DEC","Total"

        state = wait.until(EC.visibility_of_element_located((By.XPATH, '//*[@id="j_idt33"]/div[3]/span')))
        state.click()
        time.sleep(3)

        select_state = wait.until(EC.visibility_of_element_located((By.XPATH,'//*[@id="j_idt33_'+str(i)+'"]')))
        select_state.click()
        time.sleep(3)

        state_value = wait.until(EC.visibility_of_element_located((By.XPATH,'//*[@id="j_idt33_label"]'))).text
        print(state_value)
        rtos_count = int(re.findall(r'\b\d+\b', state_value)[0])
        # print(rtos_count)

        # for n in range(1, rtos_count+1):
        n = 1
        while (n < rtos_count+1):

            try:
                rto = wait.until(EC.visibility_of_element_located((By.XPATH, '//*[@id="selectedRto"]/div[3]')))
                rto.click()
                time.sleep(3)

                select_RTO = wait.until(EC.visibility_of_element_located((By.XPATH, '//*[@id="selectedRto_'+str(n)+'"]')))
                select_RTO.click()
                time.sleep(2)
                rto_name = wait.until(EC.visibility_of_element_located((By.XPATH, '//*[@id="selectedRto_label"]'))).text
                print(rto_name)

                refresh = wait.until(EC.visibility_of_element_located((By.XPATH, '//*[@id="j_idt66"]/span[2]')))
                refresh.click()
                time.sleep(3)

                heading = wait.until(EC.visibility_of_element_located((By.XPATH, '//*[@id="groupingTable"]/div[1]/span'))).text
                # print(heading)
                table_result = table_data()

                if "Maker" in heading:
                    if rto_name.split('(')[0] in heading:
                        if table_result[0][0] !="No records found.":
                            rto_filters()
                            k = 0
                            while k<1:

                                new_table_data = table_data()
                                if new_table_data[0][0] !="No records found.":
                                    df = pd.DataFrame(new_table_data, columns=["S No", "Maker", "JAN", "FEB","MAR", "APR", "MAY", "JUN","JUL", "Total"]) # , "FEB", "MAR", "APR", "MAY", "JUN", "JUL", "AUG", "SEP", "OCT", "NOV", "DEC", "Total"
                                    df.insert(0, "Rto", rto_name)
                                    df.insert(0, "State", state_value)
                                    # print(df)
                                    dataframe = dataframe._append(df)

                                    try:
                                        wait.until(EC.visibility_of_element_located((By.XPATH,'//*[@id="groupingTable_paginator_bottom"]/a[3]/span'))).click()
                                        time.sleep(2)
                                    except:
                                        k = 1
                                        n += 1
                                else:
                                    n += 1
                                    k = 1
                            

                        else:
                            # print("passed - " + rto_name)
                            n += 1

                    else:
                        time.sleep(5)

                else:
                    primary_filters()
            except:
                print("retry")
                primary_filters()
                state = wait.until(EC.visibility_of_element_located((By.XPATH, '//*[@id="j_idt33"]/div[3]/span')))
                state.click()
                time.sleep(3)

                select_state = wait.until(EC.visibility_of_element_located((By.XPATH,'//*[@id="j_idt33_'+str(i)+'"]')))
                select_state.click()
                time.sleep(3)

                state_value = wait.until(EC.visibility_of_element_located((By.XPATH,'//*[@id="j_idt33_label"]'))).text
                print(state_value)
                rtos_count = int(re.findall(r'\b\d+\b', state_value)[0])
                pass
                # print(rtos_count)


        dataframe.to_csv("all"+str(i)+".csv", index=False)

```

### Step 2 Data Ingestion

