## Overview

<img src="https://www.se.com/us/en/assets/739/media/176145/1200/SpaceLogic-IP-Controllers-IC-1360x775.jpg" style="zoom:67%;" /> 
**Note:** We will create an image for RP/MP apis

## API overview and usage

This document explain the usage of the **RP** **Controller** API.

This API allows to get BACnet objects and properties from RP Controller.

Please note that you need to configure web api feature on RP Controller to use this API. Contact your Schneider Electric **??** for more information. 

**Note:** need to know who to contact more information **

This document provides a general tutorial for users who want to consume the RP Controller API.

## How it works

Thanks to this API, a Schneider Electric partner can remotely have access to BACnet objects and properties on RP Controller. 

Of course, this partner need to enable web api feature on RP controller. 

<img src="https://www.se.com/us/en/assets/739/media/176145/1200/SpaceLogic-IP-Controllers-IC-1360x775.jpg" style="zoom:50%;" /> 

**Note:** Need to create an image to show how it works (how connections look like)

**Note:** Need to describe the image

# Developer Guide

## How to enable WEB API on RP Controller

**Note:** need to explain how to enable WEB API on RP controller. Currently, we are working on configuration menu for WEB api. This section will be updated when it is ready.

1.	Need to access to RP controller via WorkStation Building Operation

2.	Need to configure web api feature on RP controller. For example, enabling web api, IP & port Settings, Security, admin user and password, and so on

<img src="https://www.se.com/us/en/assets/739/media/176145/1200/SpaceLogic-IP-Controllers-IC-1360x775.jpg" style="zoom:67%;" /> 

**Note:** need to create an image or screen shot. It will be updated when it is ready.

3.	Need to import certificates and keys to RP controller and client application for authentication

<img src="https://www.se.com/us/en/assets/739/media/176145/1200/SpaceLogic-IP-Controllers-IC-1360x775.jpg" style="zoom:67%;" /> 

**Note:** need to create an image or screen shot. It will be updated when it is ready.


## Limitations

Amount of calls to the API in the SANDBOX are limited.

To get a full experience and extend the thresholds, please enable WEB API on RP controller and use the production environment.

## Authentication guide

There are two layers of authentication to create secure connection between server and clilent.

1. Server Certificate Validation and Client Certificate Authentication
	
	- Server Certificate Validation
	
	To protect https client, server need to return certificate to client. Client should be able to validate identity and ownership of certificate from server using CA Certificate.
	
	You can create self-signed certificates to set this up.
	
	First of all, Generate root key. To protect your key, please set up pass phrase.
	
	>openssl genrsa -des3 -out rootCA.key 2048
	
	Then, generate a root certificate. You need to fill out a list of information.
	
	>openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1825 -out rootCA.pem
	
	Now, you need to create server key and certificate using this root certificate.
	
	Generate server key.
	
	>openssl genrsa -out server.key 2048
	
	Generate CSR (Certificate Signing Request). You need to fill out a list of information.
	
	>openssl req -new -key server.key -out server.csr
	
	Generate configuration file (server.ext). If you have domain name server, you can add address of your RP Controller. Otherwise, you can add ip address of your RP Controller.
	
	>authorityKeyIdentifier=keyid,issuer\
	>keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment\
	>basicConstraints=CA:FALSE\
	>subjectAltName = @alt_names\
	>[alt_names]\
	>IP.1 = 192.168.0.100\
	>or\
	>DNS.1 = www.your-domain.com\
	
	Generate server Certificate.
	
	>openssl x509 -req -in ./server.csr -CA ./rootCA.pem -CAkey ./rootCA.key -CAcreateserial -out ./server.crt -days 825 -sha256 -extfile ./server.ext
	
	Convert server key and certificate to DER format to import to RP Controler
	
	>openssl rsa -inform PEM -outform DER -in ./server.key -out ./server.key.der
	
	>openssl x509 -inform PEM -outform DER -in ./server.crt -out ./server.crt.der
	
	- Client Certificate Authentication
	
	Server can authenticate a client using Certificate. You can also create self-signed certificates to set this up.
	
	First of all, Generate root key. To protect your key, please set up pass phrase.
	
	>openssl genrsa -des3 -out rootCA.key 2048
	
	Then, generate a root certificate. You need to fill out a list of information.
	
	>openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1825 -out rootCA.pem
	
	Now, you need to create client key and certificate using this root certificate.
	
	Generate client key.
	
	>openssl genrsa -out client.key 2048
	
	Generate CSR (Certificate Signing Request). You need to fill out a list of information.
	
	>openssl req -new -key client.key -out client.csr
	
	Generate configuration file (client.ext).
	
	>authorityKeyIdentifier = keyid,issuer\
	>keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment\
	>basicConstraints = CA:FALSE\
	>nsCertType = client, email\
	>nsComment = "Client Certificate"\
	>subjectKeyIdentifier = hash\
	>extendedKeyUsage = clientAuth, emailProtection\
	
	Generate client Certificate.
	
	>openssl x509 -req -in ./client.csr -CA ./rootCA.pem -CAkey ./rootCA.key -CAcreateserial -out ./client.crt -days 365 -sha256 -extfile ./client.ext
	
	Convert root CA certificate to DER format to import to RP Controler
	
	>openssl x509 -inform PEM -in rootCA.pem -outform DER -out rootCA.cer
	
	If you are using soap ui for development, you need to merge client key and certificate to pfx format.

	>openssl pkcs12 -export -out client.pfx -inkey client.key -in client.crt -certfile rootCA.pem

2. Token based user authentication 

	User need to create a list of users on RP controller. Client application first sends a request to RP controller with a valid credencials, then RP controller sends instance access token to the client as a response. The client application use the token to access APIs until the token is valid.
	
	
## Response Codes

We follow the error response format proposed in [RFC 7807](https://tools.ietf.org/html/rfc7807)also known as Problem Details for HTTP APIs.  As with our normal API responses, your client must be prepared to gracefully handle additional members of the response.

### Unauthorized
<!--<RedocResponse pointer={"#/components/responses/Unauthorized"} />-->

### AccessForbidden
<!--<RedocResponse pointer={"#/components/responses/AccessForbidden"} />-->

### 400
BadRequest
Your request could not be processed.
This is a generic error.
InvalidAction
The action requested was not valid for this resource.
This error is returned when you try to access an action that doesn't exist. For example, /campaigns/{id}/actions/send.
InvalidResource
The resource submitted could not be validated.
For field-specific details, see field_warnings or field_errors objects. This error means that the object submitted to a POST or PATCH request failed to validate against JSON schema, and could relate to campaign, interest group, merge field, or any other available object.
JSONParseError
We encountered an unspecified JSON parsing error.
This error means that your JSON was formatted incorrectly or was considered invalid or incomplete.
### 401
APIKeyMissing
Your request did not include an API key.
This error suggests that your API key was missing from your request, or that something was formatted or named improperly in your header. To learn more, check out Get Started with the Mailchimp API.
APIKeyInvalid
Your API key may be invalid, or you've attempted to access the wrong data center.
Check that your API key was input correctly, and verify which data center to access. To learn more, check out Get Started with the Mailchimp API.
### 403
Forbidden
You are not permitted to access this resource.
This is a generic error.
UserDisabled
This account has been disabled.
The Mailchimp account is deactivated. Contact our support team if you need more help.
WrongDatacenter
The API key provided is linked to a different data center.
This error suggests that you tried to contact the wrong data center. It's often associated with misconfigured libraries.
### 404
ResourceNotFound
The requested resource could not be found.
This error tells you a specific resource doesn't exist. It's possible that the resource has been moved or deleted, or that there's a typo in your request.
### 405
MethodNotAllowed
The requested method and resource are not compatible. See the Allow header for this resource's available methods.
This error means that the requested resource does not support the HTTP method you used. Find out which methods are allowed for each resource in the API Reference.
### 414
ResourceNestingTooDeep
The sub-resource requested is nested too deeply.
This uncommon error appears if you've tried to generate a URL with too many resources.
### 422
InvalidMethodOverride
You can only use the X-HTTP-Method-Override header with the POST method.
This error lets you know you've tried to override an incompatible method. The Mailchimp API only permits method override with POST.
RequestedFieldsInvalid
The fields requested from this resource are invalid.
This error suggests there is a typo in your field request or some other type of syntax error or problem that invalidates your request.
### 429
TooManyRequests
You have exceeded the limit of 10 simultaneous connections.
When you reach the connection limit, we'll throttle server response. If any of your requests time out after you've reached the limit, those requests could still be considered open and continue to slow your connection. Contact the Exchange support team at exchange.support@se.com if you think this is the case.

### 500
InternalServerError
An unexpected internal error has occurred. Please contact Support for more information.
This error lets you know RP Controller have experienced a problem. Although this is rare, please contact exchange.support@se.com to let us know that you've encountered this error. **// who to contact more information

### 503
ComplianceRelated
This method has been disabled.

# Support

---

# Blogs

---

# Authentication

---
