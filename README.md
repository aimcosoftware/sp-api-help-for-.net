# sp-api-help-for-.Net
Sample code and tips for connecting to Amazon Selling Partner API from .Net

## Some of the things we have learned while setting up SP API

#### We have only used pseudo code here, except where a specific library is needed


## 1. Getting the LWA AccessToken

Use your ClientId and ClientSecret from your Amazon App to get an Access Token
```
'Create JSON request (we use our own library for this)

Json.OpenBrace()
Json.AddPair("grant_type", "refresh_token")
Json.AddPair("client_id", "amzn1.application-oa2-client.xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx")
Json.AddPair("client_secret", "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx")
Json.AddPair("refresh_token", RefreshToken)
Json.CloseBrace()

'Make the request (we use our own library based on WebClient)

Req.Header = "content:application/json"
Req.Header = "content-type:application/json"

Dim Res = Req.PostData("https://api.amazon.com/auth/o2/token", Json.ToString)

If Res.Contains("access_token")
  AccessToken = JsonValue(Res, "access_token")
End If

```
