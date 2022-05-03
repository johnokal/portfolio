---
title: Interactive Reporting with Plotly and HTML
permalink: /:Interactive Reporting with Plotly and HTML/
---

### **Situation**
Imagine that you are a financial analyst that codes in a department of non-coders. How do you share a script when no one knows how to open it? Although what you built may be a game-changing solution your work will go unnoticed unless time and effort is spent on training your co-workers. My solution was to create an interactive HTML file of clickable graphs and charts thanks to the help of the Plotly library. Below, I will go over the code that makes the charts and then show the finished product with altered data so that no sensitive information is shared.

### **Disclaimer**
I am using **altered data** that reflects a similar but not exact report that was used for C-Suite reporting on a monthly basis. Prior to these charts being created, I needed to download, transform, and manipulate millions of rows of data in order to aggregate it in a way that was useful to show in a chart/graph. This automated report saved days of work over the months that it was used since the only manual input was downloading the new files each month and clicking run to generate the HTML file.

### **Code**
```python
import numpy as np
import pandas as pd
import plotly.graph_objects as go
import plotly.express as px
from datetime import datetime, timedelta

# Enter Months to update columns header names below
month0 = 'March'
month1 = 'February'
```
Note: there were many other inputs needed but I simplified the above to only compare two months of revenue.

```python
fig1 = px.bar(df1, x='BillableAccountName', y=[month1+'Rev',month0+'Rev'],
              title=f"Revenue swings between {month0} and {month1}",
              barmode='group',
              color_discrete_sequence=["#2F73DA", "#6BCDDB","#704C9F"],
              hover_name='BillableAccountName',
              hover_data={'BillableAccountName':False})

fig1.update_yaxes(title_text='Revenue Amount (USD)')
fig1.update_xaxes(title_text='Billable Account')

fig1.update_traces(yhoverformat="$,.2f")

fig1.update_layout(legend_title_text='Month')
fig1.update_yaxes(showspikes=True, spikethickness=1, spikemode='across')

fig1.show()
```
fig1's task was to select accounts on an automated basis that met certain criteria such as greater than or less than X value for the accounting department to investigate variances. The chart is sorted based on largest difference between one month and another. Months and chart titling are dynamic due to use of f strings.

```python
def revbyclient(df):
    df1 = df.groupby(['Month','BillableAccountName'])['Revenue'].sum().reset_index().sort_values(by=['BillableAccountName','Month'], ascending=[True,True])
    df1['Month'] = pd.to_datetime(df1['Month'])
    df1['Month'] = df1.apply(lambda x: x['Month'] - pd.offsets.MonthBegin(1), axis=1)

    df1['PercentChange'] = np.where(df1['BillableAccountName'] == df1['BillableAccountName'].shift(),
                                    round(((df1['Revenue'] - df1['Revenue'].shift()) / df1['Revenue'].shift()*100),2),
                                    0)

    df1['Diff'] = np.where(df1['BillableAccountName'] == df1['BillableAccountName'].shift(),
                           df1['Revenue'] - df1['Revenue'].shift(),
                           0)

    df1['PercentChange'] = df1['PercentChange'].replace(np.nan, 0)
    df1['PercentChange'] = df1['PercentChange'].replace(np.inf, 0)
    df1['PercentChange'] = df1['PercentChange'].astype(str) + '%'
    
    df1['Revenue1'] = df1['Revenue'].apply(lambda x: "${:,.2f}".format(x))
    df1['Diff1'] = df1['Diff'].apply(lambda x: "${:,.2f}".format(x))
    df1['Abs'] = abs(df1['Diff'])
    
    df1['Desc'] = df1['Revenue1'].astype(str) + "<br>" + "Var: " + df1['Diff1'].astype(str) + "<br>" + df1['PercentChange'].astype(str)
    df1 = df1.sort_values(by=['Month','Abs'], ascending=False)
    
    return df1
    
fig2 = px.bar(revbyclient(df2), x='Month', y='Revenue', color='BillableAccountName',
              title="Revenue by Customer <br> click the legend to view the chart by account<br> sorted by absolute change",
              color_discrete_sequence=["#704C9F"],
              hover_data={'PercentChange':True,
                          'Desc':False,
                          'BillableAccountName':False},
              hover_name='BillableAccountName',
              text='Desc')

fig2.update_yaxes(title_text='Revenue Amount (USD)')
fig2.update_xaxes(title_text='Month')

fig2.update_traces(yhoverformat="$,.2f")

fig2.update_layout(legend_title_text='Billable Account',
                   legend_itemclick='toggleothers',
                   uniformtext_mode='hide')

fig2.show()
```
fig2 creates a drillable chart sorted by largest absolute change per customer. It is a monthly time series that you can select by customer on the right-hand side to see their yearly revenue trends. This chart was especially important since it shows the trend, and dollar/percentage differences MoM. It gave the team a pulse into underlying data issues or client up/down-ticks in activity.

```python
fig3 = px.sunburst(df3,
                  path=[px.Constant('Comparison total'),'Year','BillableAccountName'],
                  values='Revenue',
                  title='Top 25 clients by revenue from start of year until current month for each year listed',
                  color='Revenue',
                  color_continuous_scale='blues',
                  hover_name='BillableAccountName')

fig3.update_traces(root_color="lightgrey", textinfo='label+value')

fig3.show()

fig4 = px.treemap(df3,
                  path=[px.Constant('Comparison total'),'Year','BillableAccountName'],
                  values='Revenue',
                  title='Top 25 clients by revenue from start of year until current month for each year listed',
                  color='Revenue',
                  color_continuous_scale='blues')

fig4.update_traces(root_color="lightgrey", textinfo='label+value')

fig4.show()
```
fig3 and fig4 are the same data but displayed differently. fig4 is more dynamic than fig3 in that fig4 has an animation component to it. I liked these types of ranking charts so that stakeholders could see how certain customers compare against each other.

```python
# Last step is to aggregate all charts into an HTML file so that it can be shared
# Note: the 'img src' tag can be replaced with any image file and stored in a google drive folder shared with those who will access the report
# The google drive folder must be shared with the stakeholders who wish to see the image - usually I would place a company logo so that the header looked formal
header = f"""<body style="margin-top:25px; margin-left:100px; margin-right:100px"></body>
<div>
        <img src="YOUR FILEPATH" style="width:100px;height:100px;"/>
        <h1 style="text-align:center;font-family: Arial, Helvetica, sans-serif;">{str.upper(month0)} CLOSE REVIEW</h1>
        <hr>
</div>"""

# Section headers broke up the report with a line divider
section1 = """<div>
        <h2 style="text-align:left;font-family: Arial, Helvetica, sans-serif;">Revenue Review</h2>
        <hr>
</div>"""

section2 = """<div>
        <h2 style="text-align:left;font-family: Arial, Helvetica, sans-serif;">Customer Ranking</h2>
        <hr>
</div>"""

# Write above HTML to a file and compile charts. Save anywhere.
with open(f"YOUR FILEPATH", 'a') as f:
    f.write(header)
    f.write(section1)
    f.write(fig1.to_html(full_html=False, include_plotlyjs='cdn'))
    f.write(fig2.to_html(full_html=False, include_plotlyjs='cdn'))
    f.write(section2)
    f.write(fig3.to_html(full_html=False, include_plotlyjs='cdn'))
    f.write(fig4.to_html(full_html=False, include_plotlyjs='cdn'))
```

### **Example Output File**
[LINK TO EXAMPLE FILE]({{ site.baseurl }}{% link /assets/Portfolio_Graphs.html %})
