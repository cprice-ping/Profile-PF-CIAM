This Solution provides a CIAM configuration for PingDirectory \ PingFederate

It is based on a layered set of Use Cases:

API Collections (Required): 
* Use Case: PD - Baseline
  * [Documentation]()
  * [Collection](https://www.getpostman.com/collections/251528ba1c88b823da85)
* Use Case: PF - Initial
  * [Collection](https://www.getpostman.com/collections/f8e24e4e53f7059beb10)
* Use Case: PF - CIAM
  * [Collection](https://www.getpostman.com/collections/b17a3494b4f4d54de628)
* [Optional] Use Case: PF - Add Sample Apps
  * [Collection](https://www.getpostman.com/collections/9bd0b2aa44487c0204f0)
* [Optional] Use Case: PF - Add CIBA to CIAM
  * [Collection](https://www.getpostman.com/collections/246ba03433c2ffe26de0)

Server Profiles:
[PD-Base Profile](https://github.com/cprice-ping/Profile-PD-Base)
[PF-Base Profile](https://github.com/cprice-ping/Profile-PF-Base)
[PF-CIAM Profile](https://github.com/cprice-ping/Profile-PF-CIAM)

**Note:** These collections are injected as services in the `docker-compose.yaml`.  

If desired, they can be run manually in either Postman Collection Runner, or with Newman: `run {{collection url}} -e postman_vars.json --insecure --ignore-redirects`

---
## Deployment
* Copy the `docker-compose.yaml`, `env_vars.sample` and `postman_vars.json.sample` files to a folder
* Rename files to `env_vars` and `postman_vars.json`
* Modify the `env_vars` file to match your environment
* Modify the `postman.json` file to match your environment
* Launch the stack with `docker-compose up -d`
* Logs for the stack can be watched with `docker-compose logs -f`
* Logs for individual services can be watched with `docker-compose logs -f {service}`

## Configuration

To access the Admin UI for PF go to:  
https://{{PF_HOSTNAME}}:9999/pingfederate

Credentials (LDAP):  
`Administrator` / `2FederateM0re`

This configuration includes:

### Adapters
* HTML Form with LIP
* Identifier-First (Passwordless)
* PingID SDK
* iOvation IK (not configured)
* ID Data Web IK (not configured)

### Connections
* PingID SDK Connector
  * Auto-enrollment of Email \ SMS -- `(mail=*)`

### PingID SDK - Special Considerations
The PingID adapter uses the secrets from your PingID tenant to create the proper calls to the service. As such, storing those values in a public location, such as GitHub, should be considered **risky**.

For this Profile, you can place the text from a `pingidsdk.properties` file into `postman_vars.json`. The API calls will base64 encode and inject into the PingIDSDK Adapter and HTML Form (for Self-Service Password Reset)

### AuthN Policy - Default AuthN Experiences
Extended Property Selector
* `Basic` (HTML Form with LIP)
* `MFA` (HTML Form with LIP --> PingID SDK)
* `Passwordless` (ID-First --> PingID SDK)
* [Planned] QR-Code (PingID SDK)

### AuthN Policy - Default AuthN API
Extended Property Selector
* `API` (ID-First --> HTML Form with LIP)

### AuthN Policy - Failback
Used for anything without an Ext Prop -- i.e. LIP Profile Management
* HTML Form with LIP

### AuthN Policy - Forgot Password
Used to allow PID SDK for SSPR
* PID SDK Adapter

The Authentication Experience is controlled by setting the `Extended Properties` on the Application.   

### Extended Properties
* `Enhanced` (HTML Form with LIP --  Facebook [not configured] & QR Code buttons) *default*
* `MFA` (HTML Form with LIP --> PingID SDK adapter)
* `Passwordless` (ID-First --> PingID SDK)

### Authentication API
The AuthN API is enabled -- a value in the Extended Property of `API` will trigger it.
* ID-First --> HTML Form with LIP --> AuthN API Explorer 

### [Optional] Applications ([Collection](https://www.getpostman.com/collections/9bd0b2aa44487c0204f0))
Two applications are pre-wired:

**SAML:**  
https://`${PF_BASE_URL}`/idp/startSSO.ping?PartnerSpId=Dummy-SAML

**OAuth \ OIDC:**  
`Issuer` == `${PF_BASE_URL}`  

**OIDC Logon**
`client_id` == PingLogon  

**Introspect**  
`client_id` == PingIntrospect  
`client_secret` == 2FederateM0re

### CIBA Authenticators
* Email
* PingID SDK
  * **Note:** Configuration is done with  [Use Case: Add CIBA to CIAM](https://www.getpostman.com/collections/246ba03433c2ffe26de0)

### Users
`user.[0-4]` / `2FederateM0re`
