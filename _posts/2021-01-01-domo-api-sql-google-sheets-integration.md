---
title: Python, Domo API, SQL, Google Sheets Integration
permalink: /:Python, Domo API, SQL, Google Sheets Integration/
---

The versatility of Python allows for incredible data transformations across systems that seemed disparate. I use Python as the driver to accomplish this task. Below, I will show pieces of different scripts that can combine to create a powerful, automated ETL.

### **Connect to Domo API**
According to Wikipedia, "Domo, Inc. is a cloud software company based in American Fork, Utah, United States. It specializes in business intelligence tools and data visualization." It can also act as a data warehouse.
```python
import pandas as pd
import logging
import json
from pydomo import Domo

domo_creds = {"client_id": "UniqueID1",
              "client_secret": "SECRET1",
              "api_host": "api.domo.com"}

domo_dev_creds = {"client_id": "UniqueID2",
              "client_secret": "SECRET2",
              "api_host": "api.domo.com"}

# Create functions that work together to access dataflows from Domo
# Domo has a production and development side that we can access with different credentials as shown above

def domo_login():
    domo = Domo(domo_creds['client_id'], domo_creds['client_secret'], logger_name='foo', log_level=logging.INFO, api_host=domo_creds['api_host'])
    return domo

def domo_dev_login():
    domo = Domo(domo_dev_creds['client_id'], domo_dev_creds['client_secret'], api_host=domo_dev_creds['api_host'])
    return domo

def domo_query(session, dataset_id, query):
    response = session.transport.post("/v1/datasets/query/execute/" + dataset_id, {"sql":query}, None)
    json_to_parse = json.loads(response.text)
    df = pd.DataFrame(json_to_parse['rows'], columns=json_to_parse['columns'])
    return df

def download(dataset, query):
    df = domo_query(domo_login(), dataset, query)
    return df

def dev_download(dataset, query):
    df = domo_query(domo_dev_login(), dataset, query)
    return df 
```
Now that we have successfully connected to the Domo API, how do we extract information? Domo uses SQL to download data. Luckily with Python (with one quirk that there must be a space at the end of each line), we can use an f string to write SQL queries to extract any amount of information. Below is an example of a simple query:

### **SQL in Python**
```python
# SQL Query to pull data
dataset_id = 'DOMO DATAFLOW ID HERE'
data_query = f"""
SELECT      DATEDIFF(Month,'1899-12-30') AS Month 
,           BillableAccountName 
,           Case When ... Then ... 
            Else ... End AS Column2
,           Sum(`Column4`) as Column4
FROM        `DATAFLOW ID HERE`
WHERE       Month BETWEEN '2021-02-01' AND '2023-01-31' AND Column4 != 0 
GROUP BY    Column1,Column2,Column3
ORDER BY    Column1,Column2,Column3
"""

# Download data from Domo and assign to a variable
# 'd' below is the alias on the import not shown here that we use the above code block to access
df1 = d.download(dataset_id, data_query)
```
Once data is downloaded, we can transform it as much as needed before exporting to a Google Sheet for further modeling. Common things that I do at this step is merge with another dataset from a different system that doesn't have an API. Adding columns, pivoting, filtering, or altering any informational fields is easier to manage in a Jupyter Notebook so that the entire dataflow is automated for future use.

### **Send to Google Sheets**
```python
import pygsheets

# Note: must set up a google developer account and get your unique credentials
gc = pygsheets.authorize(service_file='C:/python_credentials.json')
sh = gc.open('GOOGLE SHEET NAME')

# I commonly delete a sheet and then add it back. Formulas remain intact on other tabs and don't #ref out.
wks1 = sh.worksheet_by_title('WORKSHEET1 NAME')
wks1 = sh.del_worksheet(wks1)
wks1 = sh.add_worksheet('WORKSHEET1 NAME')
wks1.set_dataframe(df1.reset_index(), start=(2,1), extend=True)
wks1.set_dataframe(df1.reset_index(), start=(x+14,1), extend=True)

wks2 = sh.worksheet_by_title('WORKSHEET2 NAME')
wks2 = sh.del_worksheet(wks2)
wks2 = sh.add_worksheet('WORKSHEET2 NAME')
wks2.set_dataframe(df2.reset_index(), start=(2,1), extend=True)
```

### **Conclusion**
Using multiple systems is useful as each has pros/cons. When combined, a whole new set of possibilities opens up. We can create real-time, automated ETL's that simplify processes and limit human errors.
