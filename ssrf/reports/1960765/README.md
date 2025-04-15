## Blind SSRF to internal services in matrix preview_link API

### Steps To Reproduce:

1. Visit the https://matrix.redditspace.com/_matrix/media/r0/preview_url/?url=*
2. Replaced * with the internal url I guess, something like localhost.
