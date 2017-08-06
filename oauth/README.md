# Demo how OAuth works #

Dependency :
APIC 5.0.8.0 (+)
DataPower 7.8.0.0 (+)

You can also use older release, most of the features will work with previous releases.

## What are all the files ##

Files contributes to 5 `Products`, 6 API swaggers

Product list:
1. BasicOAuthWithAPI (basicoauthwithapi_1.0.0.yaml)

  This showcases basic OAuth functionalities

  It uses :
  - For OAuth, end user
    - HTML form for end user authentication
      - username : spoon
      - password : spoon
    - Uses authenticate-url to 5 below to verify user credential
    - Uses OOTB HTML consent form
  - Contain 2 yamls
    - oauth_1.0.0.yaml
      - OAuth provider api
    - weather-provider-api_1.0.0.yaml
      - consumer api

2. OAuthWithCustomForm (oauth-customform_1.0.0.yaml)

  This showcases OAuth functionalities with customer hosted form
  It uses :
  - For OAuth, end user, it uses
    - Customer hosted HTML form (see 5 below) for end user authentication
      - username : spoon
      - password : spoon
    - Uses authenticate-url to 5 below to verify user credential
    - Uses customer hosted form (see 5 below)
  - Contain 1 yaml
    - oauth-customform_1.0.0.yaml
      - OAuth provider api

3. OAuthWithoutConsent (oauth-noconsent_1.0.0.yaml)

  This showcases OAuth functionalities in which successful authentication, provide the consent
  It uses :
  - For OAuth, end user, it uses
    - OOTB HTML form (see 5 below) for end user authentication
      - username : spoon
      - password : spoon
    - Uses authenticate-url to 5 below to verify user credential
    - Uses `authenticated` as option
  - Contain 1 yaml
    - oauth-noconsent_1.0.0.yaml
      - OAuth provider api

4. OAuthWithRedirect (oauth-redirect_1.0.0.yaml)

  This showcases OAuth functionalities in which a [redirect to a 3rd party Identity Provider](https://www.ibm.com/support/knowledgecenter/en/SSFS6T/com.ibm.apic.toolkit.doc/task_apionprem_redirect_form_.html).
  - Uses 5 below as pseudo 3rd party identity provider
  - Contain 1 yaml
    - oauth-redirect_1.0.0.yaml
      - OAuth provider api

5. UtilityPlan (utilityplan_1.0.0.yaml)

  This covers mock backend, mock 3rd party identity provider, and other apis to support the above.


## How to test ##

Import the postman collection, and enjoy (remember to setup the environment variable properly to your environment)
