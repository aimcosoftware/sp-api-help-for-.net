# Amazon SP API tips for .Net
Sample code and tips for connecting to Amazon Selling Partner API from .Net

## Some of the things we have learned while setting up SP API

#### We have only used pseudo code here, except where a specific library is needed


## 1. Get the LWA AccessToken
### Required to get STS token to sign SP API requests

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
## 2. Get the AWS STS token from your IAM User
### Required to make SP API calls
User your IAM user role credentials to get a security token
```
TimeStamp = UTC.ToString("yyyyMMddTHHmmssZ")
DateStamp = UTC.ToString("yyyyMMdd")

' Uri encoded form body
Body = $"Action=AssumeRole&Version=2011-06-15&RoleArn=arn%3Aaws%3Aiam%3A%3Axxxxxxxxxxxx%3Arole%2FSellingPartnerAPIRole&RoleSessionName={Now.ToUnixTime}"

' Uri details to hash for signing, region is us-east-1, eu-west-1, etc
Post = "POST" & vbLf
Post &= "/" & vbLf & vbLf

Post &= $"host:sts.{Region}.amazonaws.com" & vbLf
Post &= $"x-amz-date:{TimeStamp}" & vbLf & vbLf

Post &= "host;x-amz-date" & vbLf
Post &= Hash(Body)

' AWS4 string to sign
Sign = "AWS4-HMAC-SHA256" & vbLf
Sign &= TimeStamp & vbLf
Sign &= $"{DateStamp}/{Region}/sts/aws4_request" & vbLf
Sign &= Hash(Post)

SecretKey = "AWS IAM user secret key"
AccessKey = "AWS IAM user access key"

' Authorization header (Signature function is below)
Req.Header = $"Authorization:AWS4-HMAC-SHA256 Credential={AccessKey}/{DateStamp}/{Region}/sts/aws4_request, SignedHeaders=host;x-amz-date, Signature={Signature(Sign, "sts")}"

' Add other headers and make request
Req.Header = "content-type:application/x-www-form-urlencoded"
Req.Header = $"host:sts.{Region}.amazonaws.com"
Req.Header = $"x-amz-date:{TimeStamp}"
Dim Res = Req.PostData($"https://sts.{Region}.amazonaws.com", Body)

' Credentials to sign SP API calls (valid for 1 hour)
AccessKey = XMLValue(Res, "AccessKeyId")
SecretKey = XMLValue(Res, "SecretAccessKey")
SecureToken = XMLValue(Res, "SessionToken")
```
## 3. Signing and hashing support
```
' Create signature for each request
Function Signature(StringToSign As String, Service As String) As String
	kSecret = $"AWS4{SecretKey}".ToUTF
	kDate = Hmac(UTC.ToString("yyyyMMdd"), kSecret)
	kRegion = Hmac(Region, kDate)
	kService = Hmac(Service, kRegion)
	kSigning = Hmac("aws4_request", kService)
	Return Hmac(StringToSign, kSigning).ToHex
End Function

' Use this specific class from System.Security.Cryptography
Function Hmac(Data As String, Key As Byte()) As Byte()
	Using Hma = KeyedHashAlgorithm.Create("HmacSHA256")
		Hma.Key = Key
		Return Hma.ComputeHash(Encoding.UTF8.GetBytes(Data))
	End Using
End Function

Function Hash(Data As String) As String
	Using Sha = HashAlgorithm.Create("SHA256")
		Return Sha.ComputeHash(Encoding.UTF8.GetBytes(Data)).ToHex
	End Using
End Function
```
