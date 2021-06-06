---
layout: post
title: "Ontario COVID-19"
subtitle: "Example with matplotlib"
background: '/img/ontario-covid/01.png'
---

# Data Visualization
Start by getting copying the following template to get a chart with labels.

<script src="https://gist.github.com/pnvnd/1a29b507baf8f07d427f7cf3be932b6d.js"></script>

For the data processing part, I'm using the dataset from the Ontario Government for COVID testing:  
`filename = "https://data.ontario.ca/dataset/f4f86e54-872d-43f8-8a86-3892fd3cb5e6/resource/ed270bb8-340b-41f9-a7c6-e8ef587e6d11/download/covidtesting.csv"`

To process the data, I used columns 1, 5, and 6 (Reported Date, Confirmed Positive, Deaths):
```python
# Get dates, case and death counts from CSV file.
dates, casesOnt, deaths = [], [], []
for row in reader:
    current_date = datetime.strptime(row[0], "%Y-%m-%d")

    if row[4] == "":
        caseOnt = 0
    else:
        caseOnt = float(row[4])

    if row[6] =="":
        death = 0
    else:
        death = float(row[6])
    
    dates.append(current_date)
    casesOnt.append(caseOnt)
    deaths.append(death)
```
The result is a chart that looks like this:
![Ontario COVID-19](https://github.com/pnvnd/Ontario-COVID-19/raw/master/covid_cases_deaths.png)

Check out [the repository on Github](https://github.com/pnvnd/Ontario-COVID-19.git).
