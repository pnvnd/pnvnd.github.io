---
layout: post
title: "YorkU GPA to Percent"
subtitle: "Based on Cumulative GPA"
background: '/img/posts/numerical_methods.jpg'
---

# Formula
Based on the Grade Point Average to Percentages given in the Faculty of Science and Engineering Calendar in 2011, we use polynomial regression in Excel to get the following equation:

`f(x) = 0.0076*(x**5)-0.1541*(x**4)+1.154*(x**3)-3.9327*(x**2)+11.017*(x)+38.958`

## Python
Here's a quick and easy way to implement this in Python 3:

```python
def yorku_percent(cpga):
    """
    Calculates grade percentage from CGPA based on Faculty of Science and Engineering Calendar.
    """
    if cpga < 0 or cpga > 9:
        print("Invalid GPA")
    else:
        percent = 0.0076*(cpga**5)-0.1541*(cpga**4)+1.154*(cpga**3)-3.9327*(cpga**2)+11.017*(cpga)+38.958
        print("A cumulative GPA of " + str(cpga) + " is approximately " + str(int(percent)) + "%.")
```