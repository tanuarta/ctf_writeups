# Jester

Kind of like ssh session of solving problems but through webpages instead.

Kinda of got stuck because python was sometimes too slow but used sessions to speed it up.


```python
import requests
import numpy
import re


s = requests.Session()

# The API endpoint
url = "https://jester.acmcyber.com/validate"

# A GET request to the API
response = s.get(url)

# Grab the numbers
strArray = re.findall(r'\d+', response.text)

num1 = strArray[5]
num2 = strArray[6]

answer = int(num1) + int(num2)
myobj = {'answer': str(answer)}

response = s.post(url, data=myobj)

# Grab the numbers
strArray = re.findall(r'\d+', response.text)

p = [strArray[5], strArray[7], strArray[8]]

gfg = numpy.roots(p)

myobj = {'answer1': round(gfg[0]), 'answer2': round(gfg[1])}

response = s.post(url, data=myobj)

print(response.text)
```