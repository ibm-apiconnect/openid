# utility.yaml #

## Background ##
When working with IBM APIc OAuth Provider or with Consumer API, there is need to demo how certain features work, **utility.yaml** is the result of that effort.

utility.yaml is provided as-is.

## Context variables ##
Once you import the utility.yaml, you can modify the context variable to the value to drive the behavior of the api calls defined in this yaml.

The following are defined in the first action, **set-variable**, in the assemble.
```
- set-variable:
    title: set-variable
    actions:
      - set: demo.api-authenticated-credential
        value: 'cn=spoon,ou=ozair,o=ibm'
      - set: demo.application.x-selected-scope
        value: accountinfo
      - set: demo.owner.x-selected-scope
        value: 'read:8888-8888'
      - set: demo.authenticate-url.x-selected-scope
        value: mutual     
      - set: demo.identity.redirect.username
        value: spoon
      - set: demo.identity.redirect.confirmation
        value: ozair
      - set: demo.authenticate-url.metainfo.4.token
        value: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI4ODgtODgtODg4OCIsIm5hbWUiOiJTcG9vbiIsImFkbWluIjp0cnVlfQ.FUzbH2_0OTbK4zwWOjzz-ivop1hetZkBpldGF77bJwM
      - set: demo.authenticate-url.metainfo.4.payload
        value: For your eyes only
      - set: demo.introspect.response.scope
        value: spoon fork knief
```
- demo.api-authenticated-credential
  - use in OAuth Provider -> authenticate-url
  - for setting the value for the optional http response header, api-authenticated-credential
  - assert the owner's identity in an OAuth processing
  - Unset this variable by either removing it, or set the value to space
- demo.application.x-selected-scope
  - For testing `OAuth Provider API`, Advanced Scope Check -> Enable Application Scope Check.
  - For setting the value for http response header, x-selected-scope
  - Unset this variable by either removing it, or set the value to space
- demo.owner.x-selected-scope
  - For testing `OAuth Provider API`, Advanced Scope Check -> Enable Owner Scope Check.
  - For setting the value for http response header, x-selected-scope
  - Unset this variable by either removing it, or set the value to space
- demo.authenticate-url.x-selected-scope
  - For testing `OAuth Provider API`, authenticate-url
  - setting the optional http response header, x-selected-scope
  - Unset this variable by either removing it, or set the value to space
- demo.authenticate-url.x-selected-scope
  - For testing `OAuth Provider API`, authenticate-url
  - setting the optional http response header, x-selected-scope
  - Unset this variable by either removing it, or set the value to space
- demo.identity.redirect.username
  - For testing `OAuth Provider API`, Extract Identity -> redirect
  - This setups the `username` the demo 3rd party identity provider will return to IBM APIc
  - This is a `must` variable for testing the above feature
- demo.identity.redirect.confirmation
  - For testing `OAuth Provider API`, Extract Identity -> redirect
  - This setups the `confirmation` the demo 3rd party identity provider will return to IBM APIc
  - This is a `must` variable for testing the above feature
- demo.authenticate-url.metainfo.4.token
  - For testing `OAuth Provider API`, Authenticate -> authenticate-url
  - This set up the optional HTTP response to inject metadata into the access_token
  - Unset this variable by either removing it, or set the value to space
- demo.authenticate-url.metainfo.4.payload
  - For testing `OAuth Provider API`, Authenticate -> authenticate-url
  - This set up the optional HTTP response to inject metadata into the access_token's payload
  - Unset this variable by either removing it, or set the value to space
- demo.introspect.response.scope
  - For testing consumer api, when 3rd party OAuth Provider introspection endpoint is used
  - This will set the scope response for the demo `3rd party OAuth Provider introspection`
  - Unset this variable by either removing it, or set the value to space

To test the utility.yaml
1. ping
  - make sure it is hosted properly
