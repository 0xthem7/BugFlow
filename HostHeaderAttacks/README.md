## Host Header Attacks 

The purpose of the HTTP Host header is to help identify which back-end component the client wants to communicate with. If requests didn't contain Host headers, or if the Host header was malformed in some way, this could lead to issues when routing incoming requests to the intended application. 

It's used to guide to request to the specific severs specific part, Let's say multiple sites are hosted under same IP address / Same server then virtual host helps to distinguish the site which virtual host help to get the data or sometimes it's due to intermediary routining through either CDN or load balancer.
