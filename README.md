# Amazon SP API tips for .Net
Sample code and tips for connecting to Amazon Selling Partner API from .Net

## Some of the things we have learned while setting up SP API

#### We have only used pseudo code here, except where a specific library is needed


## 1. Getting the LWA AccessToken

Use your ClientId and ClientSecret from your Amazon App and your client's Refresh Token to get an Access Token
```
' Create JSON body (we use our own JSON library)
Json.OpenBrace()
Json.AddPair("grant_type", "refresh_token")
Json.AddPair("client_id", "amzn1.application-oa2-client.xxxxxxx")
Json.AddPair("client_secret", "xxxxxxx")
Json.AddPair("refresh_token", "Atzr|xxxxxxx")
Json.CloseBrace()

' Make the request (we use our own library inherited from WebClient)
Req.AddHeader("content:application/json")
Req.AddHeader("content-type:application/json")
Res = Req.PostData("https://api.amazon.com/auth/o2/token", Json.ToString)

' Retrieve the Access Token (valid for 1 hour)
If Res.Contains("access_token")
  AccessToken = JsonValue(Res, "access_token")
End If

```
