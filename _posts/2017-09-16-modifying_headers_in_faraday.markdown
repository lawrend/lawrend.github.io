---
layout: post
title:  "Modifying Headers in Faraday"
date:   2017-09-16 21:05:47 +0000
---


I searched for ***way*** too long to find this information so if you need this, I hope it helps.

Faraday can be used to get a url:

```
  Faraday.get 'www.example.com'
```
	
And that works fine but usually we want to add params. Most documentation looks something like this:

```
conn = Faraday.new
  response = conn.get 'http://example.com'
  response.status
  response.body
conn.params  = {'tesla' => 'coil'}
conn.headers = {'Accept' => 'vnd.github-v3+json'}
```

Great! instantiate an instance of Faraday and then set params and headers. But what if we are setting up our Faraday like this?

```
resp = Faraday.get('www.example.com') do |req|


end
```

I found plenty of info about setting params, but it took me a long time to find info on setting headers. But here is what it looks like:

```
resp = Faraday.get('www.example.com') do |req|

  req.params['client_id'] = 1234
	req.headers['Accept'] = 'application/json'
	
end
```

It is obvious once I found it but it was frustrating bit of time. I was specifically trying to modify the response from github when going through the authorization process using Oauth. The authorization token is returned in a string and I wanted it returned in JSON. Changing the Accept header was all I had to do but I couldn't figure out how to do it :) 

Hope this helps someone else! 
