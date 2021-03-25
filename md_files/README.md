## Overview

<img src="https://www.se.com/us/en/assets/739/media/176145/1200/SpaceLogic-IP-Controllers-IC-1360x775.jpg" style="zoom:67%;" /> 
**//Note: We will create an image for RP/MP apis

## API overview and usage

This document explain the usage of the `RP Controller` API.

This API allows to get BACnet objects and properties from RP Controller.

Please note that you need to configure web service feature on RP Controller to use this API. Contact your Schneider Electric ?? for more information. 

**// need to know who to contact more information

This document provides a general tutorial for users who want to consume the RP Controller API.

## How it works

Thanks to this API, a Schneider Electric partner can remotely have access to BACnet objects and properties on RP Controller. 

Of course, this partner need to enable web service feature on RP controller. 

<img src="https://www.se.com/us/en/assets/739/media/176145/1200/SpaceLogic-IP-Controllers-IC-1360x775.jpg" style="zoom:50%;" /> 

**// Need to create an image to show how it works (how connections look like)

**// Need to describe the image

# Developer Guide

## How to enable WEB API on RP Controller

**// need to explain how to enable WEB API on RP controller. Currently, we are working on configuration menu for WEB Service. This section will be updated when it is ready.

1.	**// need to access to RP controller via WorkStation Building Operation

2.	**// need to configure web servoce feature on RP controller. For example, enabling feature, IP & port Settings, Security, admin user and password, and so on
<img src="https://www.se.com/us/en/assets/739/media/176145/1200/SpaceLogic-IP-Controllers-IC-1360x775.jpg" style="zoom:67%;" /> 
**// need to create an image or screen shot

3.	**// user need to create Self Signed certificate and import this to RP controller and client application
<img src="https://www.se.com/us/en/assets/739/media/176145/1200/SpaceLogic-IP-Controllers-IC-1360x775.jpg" style="zoom:67%;" /> 
**// need to create an image or screen shot


## Limitations

Amount of calls to the API in the SANDBOX are limited.

To get a full experience and extend the thresholds, please enable WEB API on RP controller and use the production environment.

## Authentication guide
There are two layers of authentication to authenticate and allow access to the API.
1. Server and Client Certificate Authentication
	Server Certificate Authentication
	**// User can create self signed certificate for server authentication. User need to creat certificate chain to generate CA Certificate for applications or application users.
	
	Client Certificate Authentication
	**// User can create self signed certificate for client authentication. User need to creat certificate chain to generate entity certificate(leaf certificate) for applications or application users.

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
