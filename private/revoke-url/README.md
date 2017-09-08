# This shows how to use revocation url on IBM APIc #

This demo how the revocation url works

## Pre-requsite ##

IBM APIConnect 5.0.8.2
DataPower 7.6.0 (previous release should work)
Postman
  make sure you set up OAuth redirect_uri as following
  ```
~/GitHub/openid/revoke-url (💃 ) apic config:set oauth-redirect-uri=https://www.getpostman.com/oauth2/callback
oauth-redirect-uri: https://www.getpostman.com/oauth2/callback
  
  ```

## OAuth dance dissect ##

OAuth has multiple grant type, and certain token cannot be reused.  In APIc/DP, there are the following token :
- consent/authorization form
- temporary authorization code
- refresh_token

With the above in mind, the high level flow is such :
- during oauth process, under any of the following circumstance
  - consent/authorization form is submitted (as part of authorization/implicit grant type)
  - authorization code is verified (as part of the authorization grant type)
  - refresh_token is verified (as part of the refresh_token grant type)
  ** note : the revocation endpoint should take this opportunity to record the use of the above token 
- APIc makes a GET call to the authentication url to check if the token is revoked, revoke information can be returned as
  - token itself is revoked
  - owner revoke
  - revoke before certain issued date
- If the token is valid, and APIc does generate an access_token, APIc will issue POST to the revocation url, this allows the 3rd party to record the issuing of the access_token 

## breaking down the payload ##

When an Oauth request which contains one of HTTP header :
- consent : consent/authorization form 
- code : temporary authorization code 
- refresh-token : refresh_token
- access-token : access_token

request :

     GET /revoke-url HTTP/1.1
     Host: third-party.revoke.service.com
     Accept: application/xml
     X-Global-Transaction-ID: ...........
     X-Transaction-ID: ..........
     client-id: 760d75a2-44b1-4485-8c6f-0d264fcf7398
     resource-owner: cn=spoon,o=IBM
     access-token: AAIHZGVmYXVsdLEqlu
     refresh-token: gONoZxH1d3wOneJdEc9yT............
     code: m6bCIq7rGgdnb6IRfDlOjg..............
     consent: IfjG8JcpxUkWu6..........

response :
     HTTP/1.1 200 OK
     Content-Type: application/xml;charset=UTF-8

    <?xml version="1.0" encoding="UTF-8"?>
    <oauth-revocation>
      <token type="access">AAIHZGVmYXVsdLEql...</token>
      <token type="refresh">AALkyQP5RNgTq1jZ...</token>
      <token type="code">AAJWegrjLvUDBV1vKAf......</token>
      <token type="consent">c-oCMAYl09PAfeHHibnnOg..</token>
      <!-- If a resource owner has revoked all tokens issued to a given application, please
           list them as shown here. 
      -->
      <resource-owner client-id="760d75a2-44b1-4485-8c6f-0d264fcf7398">cn=spoon,o=ibm</resource-owner>
      <resource-owner client-id="760d75a2-44b1-4485-8c6f-0d264fcf7398">cn=simon,o=hal</resource-owner>
    </oauth-revocation>

Customer can revoke all the tokens issued before a certain time, using a payload like the following :

HTTP/1.1 200 OK
Content-Type: application/xml;charset=UTF-8

<?xml version="1.0" encoding="UTF-8"?>
<oauth-revocation>
  <everytoken before="2018-08-08T09:38:28Z" />
<oauth-revocation>


Once APIc issues an access_token, it will post the information to the 3rd party entity..

request :

     if token is created because of refresh_token or authorization code, the information will be part of the payload

     POST /revoke-url HTTP/1.1
     Host: third-party.revoke.service.com
     Content-Type: application/xml
     X-Global-Transaction-ID: ...........
     X-Transaction-ID: ..........

     <token code="AAJWegrjLvUDBV1vKAf......">
       <token_type>bearer</token_type>
       <client_id>760d75a2-44b1-4485-8c6f-0d264fcf7398</client_id>
       <access_token>AAIHZGVmYXVsdLEql...</access_token>
       <expires_in>3600</expires_in>
       <scope>checking saving</scope>
       <resource_owner>cn=spoon,o=ibm</resource_owner>
       <refresh_token>AAIHZGVmYXV....</refresh_token>
       <miscinfo><xsl:value-of select="/input/result/miscinfo"/></miscinfo>
     </token>


     If token is created because of refresh_token

     POST /revoke-url HTTP/1.1
     Host: third-party.revoke.service.com
     Content-Type: application/xml
     X-Global-Transaction-ID: ...........
     X-Transaction-ID: ..........

     <token refresh_token="AAJWegrjLvUDBV1vKAf......">
       <token_type>bearer</token_type>
       <client_id>760d75a2-44b1-4485-8c6f-0d264fcf7398</client_id>
       <access_token>AAIHZGVmYXVsdLEql...</access_token>
       <expires_in>3600</expires_in>
       <scope>checking saving</scope>
       <resource_owner>cn=spoon,o=ibm</resource_owner>
       <refresh_token>AAIHZGVmYXV....</refresh_token>
       <miscinfo><xsl:value-of select="/input/result/miscinfo"/></miscinfo>
     </token>

    
     response:

    HTTP/1.1 200 OK

