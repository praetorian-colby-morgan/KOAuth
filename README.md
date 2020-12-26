# KOAuth
![alt text](https://github.com/morganc3/KOAuth/blob/master/docs/KOAuth.png)

OAuth 2.0 automated security scanner
Work in progress!

OAuth 2.0 is difficult to create a re-usable dynamic scanner because the OAuth 2.0 specification allows redirects to be implemented using any method available to the user agent. This means you cannot simply look for a 3XX status code. 
For this reason, this scanner uses `chromedp` to run the checks in a browser environment. 

Additionally, OAuth authorization servers often diverge from the spec in small but 
frustrating ways. For example, requiring consent to be obtained each time a user 
attempts to perform an OAuth 2.0 flow, even if they have already consented and authenticated 
previously. 

This scanner attempts to provide configuration/CLI options to address these common scenarios. 

The scanner works by:

1. Running `chromedp` and having the user authenticate at an OAuth 2.0 Auth URL
2. Once a user's session is established, opens new tabs in the Chrome browser and runs 
various checks

Usage:
- Place OAuth 2.0 credentials/information in config JSON file. An example is in config-template.json
- `go build`
- `./KOAuth --config configfile.json --checks ./config/resources/checks.json --timeout 4`

The timeout option defines how long each tab will wait to be redirected to the redirect_uri 
before assuming the request failed. 


## Checks
Custom checks can be added by placing the checks into a JSON file and passing with the `--checks` flag.
By default, the checks in `./config/resources/checks.json` will be used. An example check is shown 
below:

```
{
    "name":"redirect-uri-total-change",
    "risk":"high",
    "description":"Completely alters the redirect URI",
    "flowType":"implicit",
    "references":"",
    "authURLParams":{"redirect_uri":["https://maliciousdomain.h0.gs"]},
    "deleteURLParams":["redirect_uri"]
}
```

Mustache templating can be used in these checks to take values from the OAuth config. The 
following fields are supported: REDIRECT_URI, REDIRECT_SCHEME, REDIRECT_DOMAIN, REDIRECT_PATH,
CLIENT_ID, CLIENT_SECRET, SCOPES, AUTH_URL, TOKEN_URL. Example below shows using 
templating to add a redirect_uri parameter that adds a malicious subdomain to the _valid_ 
redirect URI.

```"authURLParams":{"redirect_uri":["{{{REDIRECT_SCHEME}}}://maliciousdomain.{{{REDIRECT_DOMAIN}}}{{{REDIRECT_PATH}}}"]},```

The "deleteURLParams" field is used to delete "required" oauth URL params in the 
authorization request. The "authURLParams" field adds the provided params to the 
authorization URL request, and these parameters will always be added _after_ 
the params specified by "deleteURLParams" are deleted. 

In the previous example, the proper "redirect_uri" from the OAuth 2.0 config is replaced 
with the value of "https://maliciousdomain.h0.gs".

For various checks, you may wish to provide malformed "redirect_uri"
parameters or more than one. In this case where it's not obvious which 
"redirect_uri" should be waited to be redirected to, provide the 
"waitForRedirectTo" key to specify which URL the scanner should wait 
to be redirected to during the flow.

```"waitForRedirectTo":"https://malicious.h0.gs"```

If the check JSON format does not work to automate a check, a custom check function can be added, 
mapping the name of a check to a custom function. An example of this is in ./checks/state.go, 
and the mapping is added in ./checks/mapping.go.