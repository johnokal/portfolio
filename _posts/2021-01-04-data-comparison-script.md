### **Background**
This project helped me answer the frequent business question, "What was X value yesterday, last week, last quarter, last year?"

Reasons for why data changes are numerous yet managers and executives always wonder what is the cause? Arriving at the root cause for why values change within financial data is important since financial data is time series based. For example, if a past $10 transaction now shows $11, that could pose big problems. The case for most things in financial reporting is that many smaller components add to one final reporting metric. Risks to FSLI, auditing support, among many other concerns are all reasons that tracking changes is an important tool. This technique is step one in the problem-solving process. How are you supposed to determine a root cause if you don't know where to start or which underlying number changed?

### **Code**
It is important to note that in order for this code to work, the data structure must remain unchanged from one time period to the next. Column name changes will cause the script to be uncomparable.

```python
import pandas as pd
import numpy as np
import datetime as dt

# Read current day's data
one = pd.read_csv('filepath/name.csv')
# Read yesterday's data
yesterday = pd.read_csv('filepath/name - yesterday.csv')

# Identify any columns that are datetime since we want them to display in a certain way later
columns = [
column1,
column2,
column3
]

# Creating a column called 'time' so that we can determine which dataset each row came from
# This also creates a level of our multi-index columns later
yesterday['time'] = 'Yesterday'
yesterday = yesterday.set_index(['column1','column2'])
yesterday[columns] = yesterday[columns].apply(pd.to_datetime)
yesterday[columns] = yesterday[columns].apply(lambda x: x.dt.strftime('%m/%d/%y'))

today['time'] = 'Today'
today = today.set_index(['column1','column2'])
today[columns] = today[columns].apply(pd.to_datetime)
today[columns] = today[columns].apply(lambda x: x.dt.strftime('%m/%d/%y'))

# Stacking vertically to index later
variable = pd.concat([yesterday, today])
```
At this point, we have Yesterday's data and Today's data stacked with a new column called 'time' that indicates which dataset each row came from. We formatted the columns in our preferred format (mm/dd/yyyy) and can now continue.

```python
# Fill in any nan values with the word 'None'
variable = variable.replace(np.nan, 'None', regex=True)
variable = variable.sort_values(by='column1').reset_index().set_index(['column1', 'time']).unstack(level=1)
```
The above step is important since we just took our 'time' column and placed it under each of the columns we're comparing. For example, column1 may be named AccountName and we have that as the top-level hierarchy with 'Yesterday' and 'Today' as columns below so comparison is easier to see.

```python
# Function to highlight in yellow each change from Yesterday until Today so that it is digestible to anyone familiar with excel and not necessarily coders
# We are increasing the amount of people this can be useful for
def highlight_diff(data, color='yellow')
    attr = 'background-color: {}'.format(color)
    other = data.xs('Yesterday', axis='columns', level=-1)
    return pd.DataFrame(np.where(data.ne(other, level=0), attr, ''), index=data.index, columns=data.columns)
    
print(len(variable))
new_variable = variable.style.apply(highlight_diff, axis=None)
new_variable
```
Since the function returns every row in our combined dataset, the next step is to eliminate any noise by filtering for only the values that changed between one time period and another.

```python
x = []
for cell, color in new_variable.ctx.items():
    x.append(variable.iloc[cell[0]])
    
y = pd.DataFrame(x)
y = y.drop_duplicates()
y1 = y.style.apply(highlight_diff, axis=None)
print(len(y1))

# Get today's date so that we can save each daily file with a timestamp to easily reference later
timestr = time.strftime('%Y%m%d')

# Export to excel so that highlighting and usability remains
y1.to_excel('filepath/name_' + timestr + '.xlsx')
```

### **Output**
The resulting output will look like the below screenshot. I have erased any sensitive data and only included the first 10 rows. A blank value under Yesterday but an active value Today means that data was uploaded yesterday and we can see it reflected in our system of record today.
![image1]({{site.baseurl}}/assets/img/ComparisonOutputClose.PNG)

The second image shows what the entire file looks like zoomed out. Yellow highlights are data changes.
![image2]({{site.baseurl}}/assets/img/ComparisonOutputFar.PNG)

### **Conclusion**
This is an important first step in identifying changes to data and can be used in a multitude of situations. I've used this to track many different changes over many different time periods. If you can identify what changed then it is easier to implement processes to eliminate changes altogether or to report up and answer the ultimate question that X change because of y, z reasons.
