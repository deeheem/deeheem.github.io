---
title: "OAuth for Dummies [Part 2] OAuth in Detail"
date: 2021-08-22 18:57:15
category:
- Security
tags:
- "#oauth"
---

_:grey_exclamation: This is Part 2 of a two-part blog series on OAuth._
_&nbsp;&nbsp;&nbsp;&nbsp;{% post_link why-oauth 'Part 1: Why OAuth'%} focuses on the need for OAuth._
_&nbsp;&nbsp;&nbsp;&nbsp;{% post_link oauth-explained 'Part 2: OAuth in Detail'%} goes deep into the workings of OAuth._

## Protocol Endpoints

There are two types of endpoints in an Authorization Server:
1. Authorization endpoints - They handle all user interaction via the user agent (browser).
2. Token endpoints - They are meant for machines only, for interacting away from the browser via a secure API call.

Both these layers must use TLS.
The location of these endpoints must be known to the client applications before they can be used as the authorization server.

<div style="width: 550px; margin: auto;">
    {% zoom /blog/oauth-explained/oauth-2-protocol-endpoints.jpg %}
</div>

## OAuth Scope

OAuth Scope refers to the permission to do something from within a protected resource on behalf of the resource owner.

Examples of scopes:
* bitterbrew_api
* bitterbrew_api.read
* bitterbrew_api.feed

## Authorization Code Grant Type

### Flow

The Authorization Code Grant Type is:
* designed for "confidential access"
* best for websites with a server backend
* explicit user & client authentication

Let's take a look at how a typical Authorization Request would look like:

{% codeblock "Authorization Request" lang:text %}
https://authserver.example.com/authorize
    ?response_type=code
    &client_id=s6Bhdjk3H
    &redirect_uri=https://client.example.com/callback
    &state=xyz
    &scope api1 api2.read
{% endcodeblock %}

* `response_type=code` signals the Authorization Server to use the authorization code flow
* `client_id` is a unique ID for the client app
* `redirect_uri` is where the auth endpoint will redirect to once it has finished
interacting with the resource owner. This endpoint must have been previously
registered within the Authorization Server.
* `state`'s value will be echoed back to us in the response. This offers us a couple of things:
    * A way to round trip information
    * A simple way of verifying the response we receive is actually intended for us, giving us some basic defence against XSRF and unsolicited tokens.
The value should not be guessable. It is an optional parameter, but a recommended one.
* `scope` gives the client app a way to explicitly specify what permissions they want, i.e. what they want to do on behalf of the user. It is a space-delimited string. If not included in the request, the authorization server will fall back to a default set of scopes. Could be a global default or default for this particular client application or application type.

Once the resource owner authenticates and consents, the Authorization Server will then redirect to the client app’s redirect URI along with the following query params:

{% codeblock "Authorization Response" lang:text %}
https://client.example.com/callback
    ?code=KHSjdesjd38dufhkjszhH
    &state=xyz
{% endcodeblock %}

* `code` is the authorization code that is synonymous with the grant type, and represents that the user consents to the authorization (physical representation of the Authorization Grant)
    * Has a very short lifetime
    * Is bound to client id, redirect URI, resource owner and the scopes the application has been delegated
* `state` must match the value we used in the Authorization Request

We now have the Authorization Code. Let’s move away from the browser and swap the code for an Access Token using the Authorization Server’s token endpoint.

The token request is made via a POST HTTP request to facilitate the transfer of confidential data using TLS.

{% codeblock "Token Request" lang:text %}
POST /token HTTP 1.1
    Host: server.example.com
    Content-Type: application/x-www-form-urlencoded
    
    grant_type=authorization_code
    &code=KHSjdesjd38dufhkjszhH
    &redirect_uri=https://client.example.com/cb
    &client_id=kKDJ38zdnksnja
    &client_secret=dskf8dshKJf
{% endcodeblock %}

Note that there's a slight difference between the encoding style for Basic Authentication and OAuth:

**Basic Authentication Style (RFC 7617)**
`Base64(client_id + ":" + client_secret)`

**OAuth Style (RFC 6749)**
`Base64(urlformencode(client_id) + ":" urlformencode(client_secret))`

The token response received from the Authorization Server looks something like this:

{% codeblock "Token Response" lang:text %}
HTTP/1.1 200 OK
    Content-Type: application/json
{    
    "access_token": "jshdas7JKAshDKYsd893",
    "token_type": "Bearer",
    "expired_in": 3600,
    "scope": "api2.read"
}
{% endcodeblock %}

Token type and the access token are used to access the protected resource.

### Example

Let's trigger a request for fetching the authorization code:

{% zoom /blog/oauth-explained/oauth-2-auth-code-1.jpg %}

We are redirected to the Bitter Brew site, do note the request details below:

{% zoom /blog/oauth-explained/oauth-2-auth-code-2.png %}

After authenticating, we are taken to the consent screen:

{% zoom /blog/oauth-explained/oauth-2-auth-code-3.jpg %}

Accepting the permissions takes us back to the redirect URL:

{% zoom /blog/oauth-explained/oauth-2-auth-code-4.jpg %}

We can see the code has been sent and the state matches the state we issued. We then swap the code with the token by calling the token endpoint:

{% zoom /blog/oauth-explained/oauth-2-auth-code-5.jpg %}

Note the expiry of the token is in 1 hour's time. Also note that we normally don’t expose the tokens on the browser in this way, instead keep them safe and secret using some mechanism driven by our backend server.

Finally using this token we call the rewards API on behalf of the user.

{% zoom /blog/oauth-explained/oauth-2-auth-code-6.jpg %}

## The Client Credentials Grant Type

What if there is no clear resource owner? What if we are dealing with a client application that has no users involved? Can we still protect an API using the OAuth Authorization Server? Or do we need to fall back to something like API Keys or even HTTP basic authentication?

This is where the Client Credentials Grant Type can be used.

Client Credentials Grant Type is:
* designed for client applications who are the resource owner
* best for machine-to-machine communication
* requires client authentication

### Flow

{% zoom /blog/oauth-explained/oauth-2-cc-flow.jpg %}

{% codeblock "Token Request" lang:text %}
POST /token HTTP 1.1
    Host: server.example.com
    Content-Type: application/x-www-form-urlencoded
    Authorization: Basic skhf7Jbsdf3ofsdfdflkjn
    
    grant_type=client_credentials
    &scope=api1 api2.read
{% endcodeblock %}

{% codeblock "Token Response" lang:text %}
HTTP/1.1 200 OK
    Content-Type: application/json
{    
    "access_token": "jshdas7JKAshDKYsd893",
    "token_type": "Bearer",
    "expired_in": 3600,
    "scope": "api2.read"
}
{% endcodeblock %}

For those paying attention, the benefits of OAuth over API keys or HTTP Basic Auth become a little less obvious in this case. However, we do have some advantages:
1. Application credentials are not sent with every single request to the protected resource. Instead, they’re only sent on the token request. This significantly reduces the attack surface.
2. By using short-lived access tokens, we gain the ability to remove the credentials from the request.
3. It also reduces the amount of time any stolen tokens might be useful. (compared to an API key, which would probably live for months). It also does not require any manual intervention to update.

## Long-lived access with Refresh Tokens

What happens when the tokens expire? This is where the Refresh Token comes into the picture. 

A refresh token is:
* Long-lived token
* Used by the client app and swapped for new access tokens
* Allows performing long-running background tasks without the user being present / or to simply save the user from reauthorizing the application every half hour
* Must be kept highly confidential – if it gets stolen, the thief can get access to the access token to access the APIs
* User should be made aware that refresh tokens are being requested

{% codeblock "Authorization Request" lang:text %}
https://authserver.example.com/authorize
    ?response_type=code
    &client_id=s6Bhdjk3H
    &redirect_uri=https://client.example.com/callback
    &state=xyz
    &scope api1 api2.read offline_access
{% endcodeblock %}

Note that `offline_access` scope has been added in the request.

{% codeblock "Token Response" lang:text %}
HTTP/1.1 200 OK
    Content-Type: application/json
{    
    "access_token": "jshdas7JKAshDKYsd893",
    "token_type": "Bearer",
    "expired_in": 3600,
    "refresh_token": "skfjhiw84jkezSKJ84"
    "scope": "api2.read"
}
{% endcodeblock %}

Now the swapping of refresh token with new access token can be triggered either when one unauthorized response is received, or based on the expiration time returned in the token response.

{% codeblock "Refresh Token Request" lang:text %}
POST /token HTTP 1.1
    Host: server.example.com
    Content-Type: application/x-www-form-urlencoded
    Authorization: Basic skhf7Jbsdf3ofsdfdflkjn
    
    grant_type=refresh_token
    &scope=api1
{% endcodeblock %}

Note that the scope should only be included when requesting access to fewer scopes than what was initially delegated by the client application. Since the user is no longer present, it's not possible to ask for more or different scopes than what we were originally delegated.

{% codeblock "Refresh Token Response" lang:text %}
HTTP/1.1 200 OK
    Content-Type: application/json
{    
    "access_token": "jk6gcdHdjy76kljSDG",
    "token_type": "Bearer",
    "expired_in": 3600,
    "refresh_token": "oih897GSG34khjb9"
    "scope": "api2.read offline_access"
}
{% endcodeblock %}

In the response, we receive the usual access token, and if requested, the refresh token too.

Refresh tokens could be reusable or for one-time use. They could have set expiry or sliding expiry based on usage. It is entirely up to the Authorization Server to decide the Refresh Token policy for the client application.

### Example

Let's start with our usual Authorization flow from where we land on to the Bitter Brew website:

{% zoom /blog/oauth-explained/oauth-2-refresh-token-1.jpg %}

<p>

{% zoom /blog/oauth-explained/oauth-2-refresh-token-2.png %}

Note the scope has an `offline_access` value as well.

{% zoom /blog/oauth-explained/oauth-2-refresh-token-3.jpg %}

After allowing we get the refresh token in the response:

{% zoom /blog/oauth-explained/oauth-2-refresh-token-4.jpg %}

Now, this Refresh Token can be used to get the Access Tokens as and when needed.

## Extending OAuth

With that, I'll wind up my two-part series on OAuth. We have just touched the tip of the iceberg and there is certainly a lot more to explore within OAuth.

Leaving with some more resources to explore further:

* [OAuth + Identity with Open ID Connect](https://auth0.com/docs/protocols/openid-connect-protocol)
* [RFC 8414](https://datatracker.ietf.org/doc/html/rfc8414) Automatically Configuring the Client with OAuth Metadata 
* [RFC 8628](https://datatracker.ietf.org/doc/html/rfc8628) Securing Authorizing the IOT with OAuth Device Flow
* [RFC 8414](https://datatracker.ietf.org/doc/html/rfc8414) Combing SAML and OAuth with SAML Assertion Grant
* [RFC 8693](https://datatracker.ietf.org/doc/html/rfc8693) Securing Microservices with Token Exchange
