---
layout: post
title: "Divisibility Rules"
subtitle: "Find Factors Faster"
background: '/img/posts/numerical_methods.jpg'
---

# Divisibility Rules
A number is divisible by:

- 2 if it ends in 0, 2, 4, 6, or 8  
- 3 if the sum of the digits is divisible by 3  
- 4 if the last two digits are divisible by 4  
- 5 if the number ends in 0 or 5  
- 6 if the number us divisible by 2 and 3  
- 8 if the last three digits are divisible by 8  
- 9 if the sum of the digits are divisible by 9  
- 10 if the numbber ends in 0  

## Python Code
To check divisibility for a positive number with Python, try this function:  It checks the input number and performs the modulus operation to see if it's 0.  If there is no remainder, it is divisible by that number.  
<script src="https://gist.github.com/pnvnd/934aac4db95773dbbb53bf6249ee0823.js"></script>
