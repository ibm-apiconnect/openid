## Protect access to APIs using OpenID Connect

In this tutorial, you will protect access to your APIs using [OpenID Connect](http://openid.net/connect/). This tutorial will also help address Open Banking / PSD2 requirements.

**What is OpenID Connect?**

OpenID Connect (OIDC) is built on top of the OAuth 2.0 protocol and focuses on identity assertion and exchanging of verifiable claims. OIDC provides a flexible framework for identity providers to validate and assert user identities for Single Sign-On (SSO) to web, mobile, and API workloads. This capability also helps address authentication and authorization requirements for Payment Services Directive 2 (PSD2) and Open Banking. 

**Authors** 
* [Shiu-Fun Poon](https://github.com/shiup)
* [Ozair Sheikh](https://github.com/ozairs)

**Duration**: 10 minutes

**Skill level**: Intermediate

**Prerequisites:** 

* [API Connect Developer Toolkit 5.0.8.0+](https://www.ibm.com/support/knowledgecenter/SSMNED_5.0.0/com.ibm.apic.toolkit.doc/tapim_cli_install.html)
* For testing, you will need [Postman](https://www.getpostman.com/).
* Download the Postman collection [here](https://www.getpostman.com/collections/9ab248322bd2f0a75eea)

In this tutorial, you will learn how to setup API Connect with the OpenID Connect hybrid code flow to protect access to backend API resources. If you run this tutorial without using the API Developer Toolkit, you will need to adjust the endpoints appropriately.

**Instructions:** 

These instructions assume you are familiar with the basic steps of the API designer. You will import the OAuth provider YAML file which provides support for OIDC. We will review it first to understand the core functions.

1. Import API definitions file: oauth, utility and Weather. Click the **Add (+)** button and select **Import API from a file or URL**. 
	* [https://raw.githubusercontent.com/ibm-apiconnect/openid/master/weather-provider-api_1.0.0.yaml](https://raw.githubusercontent.com/ibm-apiconnect/openid/master/weather-provider-api_1.0.0.yaml) 
	* [https://raw.githubusercontent.com/ibm-apiconnect/openid/master/oidc_1.0.0.yaml](https://raw.githubusercontent.com/ibm-apiconnect/openid/master/oidc_1.0.0.yaml). 
	* [https://raw.githubusercontent.com/ibm-apiconnect/openid/master/utility/utility_1.0.0.yaml](https://raw.githubusercontent.com/ibm-apiconnect/openid/master/utility/utility_1.0.0.yaml).

	In this tutorial, we will simulate the authentication service (leveraging HTTP Basic AUth) using the `utility` API, which is an API Connect hosted Assembly that will examine the input request and return back the appropriate response. You could also use alternative approaches to validate the resource owner (such as `redirect`), but for simplicity, we will use a simple service to demonstrate the input and output parameters of the authentication service call.

2. Navigate to the folder [https://github.com/ibm-apiconnect/openid/tree/master/utility/src/authentication](https://github.com/ibm-apiconnect/openid/tree/master/utility/src/authentication) directory and open the `basic-auth.js` file. This file will simulate the authentication service during the OIDC handshake. A few points about the code:
	* Authentication is simulated by validating the username/password passed in via HTTP Basic Auth header against the query params. For example, the input request will contain a HTTP Basic Auth header with value `spoon:spoon` (base 64 encoded) and will compare that against the Authentication URL call in the OIDC provider (ie `https://$(api.endpoint.address):9443/utility/spoon/spoon`).
	* If you need access to the original query parameters passed into the request during authentication, you can access the `message` context to parse them. An example is shown that passes in a `request` query parameter which is a JWT token containing information about the transaction. This token can be passed to the service performing the authentication
	```
	var requrl = querystring.parse(apim.getvariable('message.headers.x-uri-in').split('?')[1]);
	console.error('parsed query param: request (jwt) %s', requrl['request']);
	```
	* Optionally, after authentication is done, you can pass various response headers if you want to perform any of the following:
	 * Modify scope values
	 * Inject metadata into the access token
	 * Inject metadata into the payload 
3. Open the API designer and select the `utility` API. This API will simulate the authentication service. 
4. Click the **Assemble** tab and select the `switch` statement with the condition `/basic-auth/{username}/{password}` and its corresponding GatewayScript. You can modify the code from `basic-auth.js` and copy it here or leave it as-is.
5. Click the **OAuth 2 OIDC Provider 1.0.0** API. Scroll down to Authentication section and make a note of the **Authentication URL**: `https://$(api.endpoint.address):9443/utility/basic-auth/spoon/spoon`. 
6. Scroll down to the Properties section. You will notice three properties:
 * OIDCIssuer: string to represent the issuer of the token (default value is IBM APIc)
 * JWSSignPrivateKey: string to represent the private key used to sign the JWT token
 * JWSAlgorithm: algorithm used during the signing process
7. Click the **Assemble** tab at the top. You will notice several policies that control the generation of the JWT token for OIDC flows. The `set-variable` uses the variables defined in the **Properties** section. 
8. When using the API Connect developer toolkit with OAuth Access code flow, you will need to redirect the application to an OAuth client to exchange the authorization code for an access code. This is typically done in an OAuth application, but we can use a couple of techniques to streamline testing.
9. Configure environment for OAuth Access Code 
	* Open a command prompt within the project directory (ie same directory as the project yaml files). Enter the command `apic config:set oauth-redirect-uri=https://www.getpostman.com/oauth2/callback`. 
	* Open Postman Preferences and disable **Automatically follow redirects**. This allow us to capture the code and id_token sent back from API Connect.
9. Protect Weather API with OpenID Connect Access Code flow. You can also use your own API definition and follow the same steps. The steps below use a sample API definition to demonstrate the integration.
	* Open the **Weather Provider API** and scroll down to **Security Definitions**. Click the + button and select **OAuth**.
	* Enter the name `openid-accesscode` and select the **Access Code** flow. Enter the **Authorize URL** value `https://$(api.endpoint.address):9443/oauth2/authorize` and **Token URL** value `https://$(api.endpoint.address):9443/oauth2/token`.
	* Scroll down to the scopes section and enter the scopes `weather` and `openid`.
	* In the **Security** section, click the + button to create a new option and select **openid-accesscode (OAuth)** and the two scopes. Move the security definition at the top.
	* Save the API definition.	
	**Note:** Multiple security definitions allow you provide multiple options to satisfy consumer security requirements.
10. Using Postman, open the request called `OIDC Access Code (OpenBanking)`. Adjust the values if your endpoint is different than `https://127.0.0.1:4001`.
	* Submit the request and make sure you get the following response:
	```
	<?xml version="1.0" encoding="UTF-8"?>
	<html>
		<body>Go ahead</body>
	</html>
	```
	The Access Code flow requires an additional step to obtain an access token. You will need to exchange the code for an access token. This is typically done by an OAuth application but you will simulate one for simplicity.
	* Click the **Headers** tab and note the value after `code=` and before `state=`. Notice that the response contains both the authorization code and id_token.
	* Open the `OAuth AC to Token` request and click the **Body** tab. The code field uses a variable to populate the `code` paramter. Click **Send** and verify you receive an access token
11. Open the Weather request and select the **Headers** tab. Click **Send** and validate that the request is successful.
	```
	{
		"zip": 10504,
		"temperature": 78,
		"humidity": 46,
		"city": "Armonk, North Castle",
		"state": "New York",
		"message": "Sample Randomly Generated",
		"platform": "Powered by IBM API Connect"
	}
	```
	All the test cases till now have focused on accessing the API using an OAuth access token although an JWT token (via `id_token` field) is also returned. The JWT token allows the OAuth application access to information about the user identity.

12. Use the Web site [jwt.io](https://jwt.io) and enter the following URL: https://jwt.io?value=<id_token>. It should then display the decoded token.

The fields within the JWT token can be customized based on your environment. The OAuth provider Assembly provides the flexibility to generate a JWT token and optionally sign and encrypt it. Let's review how the fields are generated:
 * jti: automatically generated unique id
 * iss: OIDCIssuer property value in the OAuth provider YAML 
 * sub: authenticated credential from the **basic-auth.js** file
 * aud: application clientId (ie `default`)
 * exp/iat: validity periods for the token
 * c_hash/at_hash: hash values of the tokens in the OpenID connect flow
 * disclaimer: custom claim

 The last three fields (exp/iat/disclaimer) are hardcoded within the jwt-generate action. Let's review the jwt-generate policy to understand how these fields can be customized.

13. Switch back to the API designer and select the **Assemble** tab. Scroll to the right and select `jwt-generate`. You will see the validity period is 3600 seconds. You can modify these value based on your environment. You should also see another variable called `added.claim` under the **Private claims** field. You can also create your own custom claims and include them in the jwt-generate action. 
14. Change the value of `added.claim` to `myclaims`.
14. Select the `set-variable` policy and create new action called `myclaims` with the value `{ "Statement" : "API Connect is great" }`
15. Run the tests again and view the JWT token again using [jwt.io](https://jwt.io). You should see the custom claim in the token.

In this tutorial, you learned how to protect an API using OpenID Connect hybrid flow and support PSD2/Open Banking requirements. You learned how to customize the OIDC provider YAML file to support different provider requirements.