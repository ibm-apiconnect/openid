# How to integration with google OAuth provider #

The use case is :
APIc is protecting an api, but Google is the oauth provider.  Application will get an access_token from Google, it will then turn around and send the token to IBM apic.

- application has 2 credentials
  - one with google, for the OAuth dance
  - one with APIc for appication identifier

Steps :
- application will contact google, it receives an access_token from google in the following format

```
HTTP/1.1 200 OK
Content-length: 266
X-xss-protection: 1; mode=block
X-content-type-options: nosniff
Transfer-encoding: chunked
Expires: Mon, 01 Jan 1990 00:00:00 GMT
Vary: Origin, X-Origin
Server: GSE
-content-encoding: gzip
Pragma: no-cache
Cache-control: no-cache, no-store, max-age=0, must-revalidate
Date: Sun, 10 Sep 2017 03:34:54 GMT
X-frame-options: SAMEORIGIN
Alt-svc: quic=":443"; ma=2592000; v="39,38,37,35"
Content-type: application/json; charset=UTF-8

{
  "access_token": "ya29.GlvCBB8sp8rTsyIV5THL9hzgKkL-AkDWmvV0jTUomfq68X66WdtcDiHPnOKSgUArSW3bAnE43hjBb5mGpx64RDZSO1DuCl-cCZ069GlPKzAWXC5jpAEQvZAUyJTg",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "1/cJP4B0UF1EIK03So0bjfDD1E-hR7imbgROqOLlXdFeE"
}
```

- application will contact IBM APIc, with the above access_token
  - IBM apic will use IETF https://tools.ietf.org/html/rfc7662 to verify the above token
  - Given google does not support RFC7662, google uses its own endpoint for verification, `https://www.googleapis.com/oauth2/v3/tokeninfo?access_token=y....`
  - APIc will host a microservices (see utility.yaml)

  ```
apim.setvariable('message.status.code', 200);
apim.readInputAsBuffer(function (error, buffer) {
  if (error) {
    var ar = {'active': false};
    apim.setvariable('message.body', ar);
  }
  else {
    // get the information as in buffer, now parse
    var topsplit = (buffer.toString()).split('&');
    var accesstoken = '';
    topsplit.forEach(function(a) {
      if (a.substring(0, 6) === 'token=') {
        accesstoken = a.substring(6);
      }
    });

    var urlOptions = {
      target: 'https://www.googleapis.com/oauth2/v3/tokeninfo?access_token=',
      method: 'GET',
      sslProxyProfile: 'api-sslcli-all'
    }
    urlOptions.target = urlOptions.target + accesstoken;
    try {
      var urlopen=require('urlopen');
      var request = urlopen.open(urlOptions, function (connectError, response) {
        if (connectError) {
          var ar = {'active': false};
          apim.setvariable('message.body', ar);
        }
        else {
          // Note: The back-end may have put an x-dp-response-code header in the
          // response. However, that header is not supported for Gateway script
          // and will be ignored. This does not cause any problems.
          var responseCode = response.statusCode;
          if (responseCode === 200) {
            var ar = {'active': true};
            apim.setvariable('message.body', ar);
          }
          else {
            var ar = {'active': false};
            apim.setvariable('message.body', ar);
          }
        } // end of not connection error
      }); // end of urlopen                          
    } catch (err) {
      // Reject the session so that when the action completes, there
      // will be a processing rule error.
      var ar = {'active': false};
      apim.setvariable('message.body', ar);
    }
  }
});
  ```
  - The actual API is configued to use introspection url to the microservice to verify the access_token provided by Google.  There are 2 security requirements
    1. api-key-1 : this is for identify the IBM APIc credential
    2. GoogleOAuthProvider : this configures the introspect call out to verify the access_token from Google

  ```
securityDefinitions:
  api-key-1:
    type: apiKey
    description: ''
    in: header
    name: X-IBM-Client-Id
  GoogleOAuthProvider:
    type: oauth2
    description: 'Given provider does not support rfc7662, this will call a service which will perform the message format transformation.'
    flow: accessCode
    scopes: {}
    authorizationUrl: 'https://accounts.google.com/signin/oauth/oauthchooseaccount'
    tokenUrl: 'https://accounts.google.com/signin/oauth/oauthchooseaccount'
    x-tokenIntrospect:
      url: 'https://$(api.endpoint.address):9443/utility/third-party-oauth/introspect/google-microservice'
  ```
