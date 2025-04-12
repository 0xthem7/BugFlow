# SSRF leaking internal google cloud data through upload function [SSH Keys, etc..]

https://hackerone.com/reports/549882  
https://dphoeniixx.medium.com/vimeo-upload-function-ssrf-7466d8630437


# Steps taken 
1. Send file url of google drive where you can find out that authorization token is being sent from the header to access the file
2. Check how the file is actually being fetched by the server, 
  - Setup wireshark on vps and capture the requests while the websites backend server is trying to access the file form the VPS 
3. Found *Range*, and *Content-Range* from which we can deduce that server is only fetching the full file if it's size is small and partial file when it's a big file
4. Tricking the server - Not sending the full file, Let's say the server requests for 500B and but we send only 200B what will happen ? 
5. Scripted a Web Server in python The full file length is 554231B, when Vimeo requests the full file, my server will respond by 8228B
6. Now the attacker got idea about the SSRF, what if we add an redirection ? 
  - The server will follow the redirection ?
  - The server will store the response ? 
  ![Image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*JEp41OYpAlxn_RF8f8h1Vg.png)
7. Attack worked, Time to exploit 
8. Done some recon and found out about the google cloud instance 
  - Sent redirection to the following endpoint http://metadata.google.internal/computeMetadata/v1beta1/instance/service-accounts/default/token
    - Able to get compute enging API access 
  

Official write up author [Twitter \| dPhoeniixx](https://x.com/dPhoeniixx)
[Medium \| vimeo upload function ssrf](https://dphoeniixx.medium.com/vimeo-upload-function-ssrf-7466d8630437)







