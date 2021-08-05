## Overview

<img src="https://github.com/SESA545913/SE-EnergyManagement-Team/raw/main/RPC.png" style="zoom:67%;" /> 


***Note: We will create an image for RP/MP apis***


## API overview and usage

This document explain the usage of the **RP-C** API.

This Web API allows you to obtain BACnet objects and properties from the RP-C.

Please note that you need to configure Web API feature on RP-C to use this API. Contact the Schneider Electric Product Support Services (PSS) team for more information.

***Note: need to know PSS is right place to contact more information about Web API***

This document provides a general tutorial for users who want to consume the RP-C API.

## How it works

By using this API, a Schneider Electric partner can remotely access BACnet objects and properties on the RP-C. 

<img src="https://github.com/SESA545913/SE-EnergyManagement-Team/raw/main/WebServiceUserScenario.png" style="zoom:50%;" /> 

In case of Single RP-C, 3rd party client application can access to BACnet objects and properties if user credencial is valid. User will be authenticated based on User DB in RP-C.

In case of EcoStructure System with RP-C's, User will be authenticated based on centralized User DB on AS/ES.

# Developer Guide

## How to enable Web API on RP-C

***Note: need to explain how to enable Web API on RP-C. Currently, we are working on configuration menu for Web API. This section will be updated when it is ready.***

1.	Need to access to RP-C via WorkStation Building Operation

2.	Need to configure Web API feature on RP-C. For example, enabling Web API, IP & port Settings, Security, admin user and password, and so on

	<img src="https://github.com/SESA545913/SE-EnergyManagement-Team/raw/main/RPC.png" style="zoom:67%;" /> 

	***Note: need to create an image or screen shot. It will be updated when it is ready.***

3.	Need to import certificates and keys to RP-C and client application for authentication

	<img src="https://github.com/SESA545913/SE-EnergyManagement-Team/raw/main/RPC.png" style="zoom:67%;" /> 

	***Note: need to create an image or screen shot. It will be updated when it is ready.***


## Limitations

The number of calls to the API in the SANDBOX are limited.

To fully experience and extend the thresholds, please enable the Web API on the RP-C and use the environment.

## Authentication guide

There are two layers of authentication to create secure connection between server and client.

- Server Certificate Validation and Client Certificate Authentication
	
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
		
		Generate configuration file (server.ext). If you have domain name server, you can add address of your RP-C. Otherwise, you can add ip address of your RP-C.
		
		```
		authorityKeyIdentifier=keyid,issuer
		keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
		basicConstraints=CA:FALSE
		subjectAltName = @alt_names
		[alt_names]
		IP.1 = 192.168.0.100
		or
		DNS.1 = www.your-domain.com
		```
		
		Generate server Certificate.
		
		>openssl x509 -req -in ./server.csr -CA ./rootCA.pem -CAkey ./rootCA.key -CAcreateserial -out ./server.crt -days 825 -sha256 -extfile ./server.ext
		
		Convert server key and certificate to DER format to import to RP-C
		
		>openssl rsa -inform PEM -outform DER -in ./server.key -out ./server.key.der \
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
		
		```
		authorityKeyIdentifier = keyid,issuer
		keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
		basicConstraints = CA:FALSE
		nsCertType = client, email
		nsComment = "Client Certificate"
		subjectKeyIdentifier = hash
		extendedKeyUsage = clientAuth, emailProtection
		```
		
		Generate client Certificate.
		
		>openssl x509 -req -in ./client.csr -CA ./rootCA.pem -CAkey ./rootCA.key -CAcreateserial -out ./client.crt -days 365 -sha256 -extfile ./client.ext
		
		Convert root CA certificate to DER format to import to RP-C
		
		>openssl x509 -inform PEM -in rootCA.pem -outform DER -out rootCA.cer
		
		If you are using soap ui for development, you need to merge client key and certificate to pfx format.

		>openssl pkcs12 -export -out client.pfx -inkey client.key -in client.crt -certfile rootCA.pem

- Token based user authentication 

	Token-based authentication is a process where the client application first send a request to server with a valid credencials. The server send an Access Token back to the client as response. The client application then use the token to acess the restricted resources in the next request until the token is valid.
	
	RP-C returns a Web Token which contains enough data to identity a particular user and it has expiry time. If the token is expired, the client application can request for a new access token.
	
	<img src="https://github.com/SESA545913/SE-EnergyManagement-Team/raw/main/TokenBasedAuthentication_v2.png" style="zoom:40%;" /> 
	
- Centralized user authentication 

	If you have EcoStructure System and use user authentication from there, you need to enable EWS Server (Eco Structure Web Services -> EWS Server Configuration). After that, you need to configure authentication on RP Web Server.
	
	<img src="https://github.com/SESA545913/SE-EnergyManagement-Team/raw/main/RPC.png" style="zoom:67%;" /> 
	
	***Note: need to create an image or screen shot. It will be updated when it is ready.***
	
## Response Codes

The RP Series controller Web API follows the error response format proposed in [RFC 7807](https://tools.ietf.org/html/rfc7807)also known as Problem Details for HTTP APIs.  As with our normal API responses, your client must be prepared to gracefully handle additional members of the response.

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
This error lets you know RP-C have experienced a problem. Although this is rare, please contact exchange.support@se.com to let us know that you've encountered this error. **// who to contact more information

### 503
ComplianceRelated
This method has been disabled.

## BACnet Property Identifier
acked-transitions (0)

ack-required (1)

action (2)

action-text (3)

active-text (4)

active-vt-sessions (5)

alarm-value (6)

alarm-values (7)

all (8)

all-writes-successful (9)

apdu-segment-timeout (10)

apdu-timeout (11)

application-software-version (12)

archive (13)

bias (14)

change-of-state-count (15)

change-of-state-time (16)

notification-class (17)

controlled-variable-reference (19)

controlled-variable-units (20)

controlled-variable-value (21)

cov-increment (22)

date-list (23)

daylight-savings-status (24)

deadband (25)

derivative-constant (26)

derivative-constant-units (27)

description (28)

description-of-halt (29)

device-address-binding (30)

device-type (31)

effective-period (32)

elapsed-active-time (33)

error-limit (34)

event-enable (35)

event-state (36)

event-type (37)

exception-schedule (38)

fault-values (39)

feedback-value (40)

file-access-method (41)

file-size (42)

file-type (43)

firmware-revision (44)

high-limit (45)

inactive-text (46)

in-process (47)

instance-of (48)

integral-constant (49)

integral-constant-units (50)

limit-enable (52)

list-of-group-members (53)

list-of-object-property-references (54)

local-date (56)

local-time (57)

location (58)

low-limit (59)

manipulated-variable-reference (60)

maximum-output (61)

max-apdu-length-accepted (62)

max-info-frames (63)

max-master (64)

max-pres-value (65)

minimum-off-time (66)

minimum-on-time (67)

minimum-output (68)

min-pres-value (69)

model-name (70)

modification-date (71)

notify-type (72)

number-of-apdu-retries (73)

number-of-states (74)

object identifier (75)

object-list (76)

object-name (77)

object-property-reference (78)

object type (79)

optional (80)

out-of-service (81)

output-units (82)

event-parameters (83)

polarity (84)

present value (85)

priority (86)

priority-array (87)

priority-for-writing (88)

process-identifier (89)

program-change (90)

program-location (91)

program-state (92)

proportional-constant (93)

proportional-constant-units (94)

protocol-object-types-supported (96)

protocol-services-supported (97)

protocol-version (98)

read-only (99)

reason-for-halt (100)

recipient-list (102)

reliability (103)

relinquish-default (104)

required (105)

resolution (106)

segmentation-supported (107)

setpoint (108)

setpoint-reference (109)

state-text (110)

status-flags (111)

system-status (112)

time-delay (113)

time-of-active-time-reset (114)

time-of-state-count-reset (115)

time-synchronization-recipients (116)

units (117)

update-interval (118)

utc-offset (119)

vendor-identifier (120)

vendor-name (121)

vt-classes-supported (122)

weekly-schedule (123)

attempted-samples (124)

average-value (125)

buffer-size (126)

client-cov-increment (127)

cov-resubscription-interval (128)

event-time-stamps (130)

log-buffer (131)

log-device-object-property (132)

enable (133)
 log-enable was renamed to enable in version 1 revision 5
log-interval (134)

maximum-value (135)

minimum-value (136)

notification-threshold (137)

protocol-revision (139)

records-since-notification (140)

record-count (141)

start-time (142)

stop-time (143)

stop-when-full (144)

total-record-count (145)

valid-samples (146)

window-interval (147)

window-samples (148)

maximum-value-timestamp (149)

minimum-value-timestamp (150)

variance-value (151)

active-cov-subscriptions (152)

backup-failure-timeout (153)

configuration-files (154)

database-revision (155)

direct-reading (156)

last-restore-time
 (157)

maintenance-required (158)

member-of (159)

mode (160)

operation-expected (161)

setting (162)

silenced (163)

tracking-value (164)

zone-members (165)

life-safety-alarm-values (166)

max-segments-accepted (167)

profile-name (168)

auto-slave-discovery (169)

manual-slave-address-binding (170)

slave-address-binding (171)

slave-proxy-enable (172)

last-notify-record (173)

schedule-default (174)

accepted-modes (175)

adjust-value (176)

count (177)

count-before-change (178)

count-change-time (179)

cov-period (180)

input-reference (181)

limit-monitoring-interval (182)

logging-object (183)

logging-record (184)

prescale (185)

pulse-rate (186)

scale (187)

scale-factor (188)

update-time (189)

value-before-change (190)

value-set (191)

value-change-time (192)

align-intervals (193)

interval-offset (195)

last-restart-reason (196)

logging-type (197)

restart-notification-recipients (202)

time-of-device-restart (203)

time-synchronization-interval (204)

trigger (205)

utc-time-synchronization-recipients (206)

node-subtype (207)

node-type (208)

structured-object-list (209)

subordinate-annotations (210)

subordinate-list (211)

actual-shed-level (212)

duty-window (213)

expected-shed-level (214)

full-duty-baseline (215)

requested-shed-level (218)

shed-duration (219)

shed-level-descriptions (220)

shed-levels (221)

state-description (222)

door-alarm-state (226)

door-extended-pulse-time (227)

door-members (228)

door-open-too-long-time (229)

door-pulse-time (230)

door-status (231)

door-unlock-delay-time (232)

lock-status (233)

masked-alarm-values (234)

secured-status (235)

absentee-limit (244)

access-alarm-events (245)

access-doors (246)

access-event (247)

access-event-authentication-factor (248)

access-event-credential (249)

access-event-time (250)

access-transaction-events (251)

accompaniment (252)

accompaniment-time (253)

activation-time (254)

active-authentication-policy (255)

assigned-access-rights (256)

authentication-factors (257)

authentication-policy-list (258)

authentication-policy-names (259)

authentication-status (260)

authorization-mode (261)

belongs-to (262)

credential-disable (263)

credential-status (264)

credentials (265)

credentials-in-zone (266)

days-remaining (267)

entry-points (268)

exit-points (269)

expiration-time (270)

extended-time-enable (271)

failed-attempt-events (272)

failed-attempts (273)

failed-attempts-time (274)

last-access-event (275)

last-access-point (276)

last-credential-added (277)

last-credential-added-time (278)

last-credential-removed (279)

last-credential-removed-time (280)

last-use-time (281)

lockout (282)

lockout-relinquish-time (283)

max-failed-attempts (285)

members (286)

muster-point (287)

negative-access-rules (288)

number-of-authentication-policies (289)

occupancy-count (290)

occupancy-count-adjust (291)

occupancy-count-enable (292)

occupancy-lower-limit (294)

occupancy-lower-limit-enforced (295)

occupancy-state (296)

occupancy-upper-limit (297)

occupancy-upper-limit-enforced (298)

passback-mode (300)

passback-timeout (301)

positive-access-rules (302)

reason-for-disable (303)

supported-formats (304)

supported-format-classes (305)

threat-authority (306)

threat-level (307)

trace-flag (308)

transaction-notification-class (309)

user-external-identifier (310)

user-information-reference (311)

user-name (317)

user-type (318)

uses-remaining (319)

zone-from (320)

zone-to (321)

access-event-tag (322)

global-identifier (323)

verification-time (326)

base-device-security-policy (327)

distribution-key-revision (328)

do-not-hide (329)

key-sets (330)

last-key-server (331)

network-access-security-policies (332)

packet-reorder-time (333)

security-pdu-timeout (334)

security-time-window (335)

supported-security-algorithms (336)

update-key-set-timeout (337)

backup-and-restore-state (338)

backup-preparation-time (339)

restore-completion-time (340)

restore-preparation-time (341)

bit-mask (342)

bit-text (343)

is-utc (344)

group-members (345)

group-member-names (346)

member-status-flags (347)

requested-update-interval (348)

covu-period (349)

covu-recipients (350)

event-message-texts (351)

event-message-texts-config (352)

event-detection-enable (353)

event-algorithm-inhibit (354)

event-algorithm-inhibit-ref (355)

time-delay-normal (356)

reliability-evaluation-inhibit (357)

fault-parameters (358)

fault-type (359)

local-forwarding-only (360)

process-identifier-filter (361)

subscribed-recipients (362)

port-filter (363)

authorization-exemptions (364)

allow-group-delay-inhibit (365)

channel-number (366)

control-groups (367)

execution-delay (368)

last-priority (369)

write-status (370)

property-list (371)

serial-number (372)

blink-warn-enable (373)

default-fade-time (374)

default-ramp-rate (375)

default-step-increment (376)

egress-time (377)

in-progress (378)

instantaneous-power (379)

lighting-command (380)

lighting-command-default-priority (381)

max-actual-value (382)

min-actual-value (383)

power (384)

transition (385)

egress-active (386)

interface-value (387)

fault-high-limit (388)

fault-low-limit (389)

low-diff-limit (390)

strike-count (391)

time-of-strike-count-reset (392)

default-timeout (393)

initial-timeout (394)

last-state-change (395)

state-change-values (396)

timer-running (397)

timer-state (398)

apdu-length (399)

ip-address (400)

ip-default-gateway (401)

ip-dhcp-enable (402)

ip-dhcp-lease-time (403)

ip-dhcp-lease-time-remaining (404)

ip-dhcp-server (405)

ip-dns-server (406)

bacnet-ip-global-address (407)

bacnet-ip-mode (408)

bacnet-ip-multicast-address (409)

bacnet-ip-nat-traversal (410)

ip-subnet-mask (411)

bacnet-ip-udp-port (412)

bbmd-accept-fd-registrations (413)

bbmd-broadcast-distribution-table (414)

bbmd-foreign-device-table (415)

changes-pending (416)

command (417)

fd-bbmd-address (418)

fd-subscription-lifetime (419)

link-speed (420)

link-speeds (421)

link-speed-autonegotiate (422)

mac-address (423)

network-interface-name (424)

network-number (425)

network-number-quality (426)

network-type (427)

routing-table (428)

virtual-mac-address-table (429)

command-time-array (430)

current-command-priority (431)

last-command-time (432)

value-source (433)

value-source-array (434)

bacnet-ipv6-mode (435)

ipv6-address (436)

ipv6-prefix-length (437)

bacnet-ipv6-udp-port (438)

ipv6-default-gateway (439)

bacnet-ipv6-multicast-address (440)

ipv6-dns-server (441)

ipv6-auto-addressing-enable (442)

ipv6-dhcp-lease-time (443)

ipv6-dhcp-lease-time-remaining (444)

ipv6-dhcp-server (445)

ipv6-zone-index (446)

assigned-landing-calls (447)

car-assigned-direction (448)

car-door-command (449)

car-door-status (450)

car-door-text (451)

car-door-zone (452)

car-drive-status (453)

car-load (454)

car-load-units (455)

car-mode (456)

car-moving-direction (457)

car-position (458)

elevator-group (459)

energy-meter (460)

energy-meter-ref (461)

escalator-mode (462)

fault-signals (463)

floor-text (464)

group-id (465)

group-mode (467)

higher-deck (468)

installation-id (469)

landing-calls (470)

landing-call-control (471)

landing-door-status (472)

lower-deck (473)

machine-room-id (474)

making-car-call (475)

next-stopping-floor (476)

operation-direction (477)

passenger-alarm (478)

power-mode (479)

registered-car-call (480)

active-cov-multiple-subscriptions (481)

protocol-level (482)

reference-port (483)

deployed-profile-location (484)

profile-location (485)

tags (486)

subordinate-node-types (487)

subordinate-tags (488)

subordinate-relationships (489)

default-subordinate-relationship (490)

represents (491)


# Support

---

# Blogs

---

# Authentication

---
