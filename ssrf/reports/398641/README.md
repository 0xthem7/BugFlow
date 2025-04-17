## SSRF on duckduckgo.com/iu/
https://hackerone.com/reports/398641

1. Checked the domain pattern and found out that it's working based on the string match case.
2.  Send the folloowing request
  `https://duckduckgo.com/iu/?u=http://127.0.0.1:6868%2fstatus%2f?q=http://yimg.com/ returns the following: 
