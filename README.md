This profile provides a CIAM configuration for PingFederate - it starts with [PF-Base](https://github.com/cprice-ping/Profile-PF-Base) and makes the following modifications:

* Adds Local Identity Profile
  * Adds Default Identity Profile
  * Adds HTML Form with LIP
* Adds Authentication API Application
* Adds PingID SDK Components
  * Creates PID SDK Email Template for PF AuthN
  * Adds SDK Connector to auto-enroll email \ SMS
  * SSPR configured to use PID SDK
* AuthN Policy - Default (Extended Property -- `authnExp`)
  * `Basic` -- HTML Form with LIP (including User Registration)
  * `MFA` -- HTML Form with LIP --> PingID SDK Adapter
  * `Passwordless` -- ID-First --> PingID SDK Adapter
* AuthN Policy - API Application (Extended Property -- `authnExp`)
  * `API` -- ID-First --> HTML Form with LIP (with AuthN API Application)
* AuthN Policy -- Fallback (used for anything without an Ext Prop -- i.e. Profile Management)
* AuthN Policy -- Forgot Password (Used for SSPR)
  * PID SDK Adapter
* Adds 3rd Party Risk IKs (not configured)
  * iOvation
  * ID Data Web
* Adds CIBA Authenticators
  * Email
  * PID SDK
  * **Note:** Requires - [Use Case: Add CIBA to CIAM](https://www.getpostman.com/collections/246ba03433c2ffe26de0) to configure
* [Planned] PA configuration (PA-Base \ PA-CIAM)

This solution is based on a layered set of Use Cases:

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

**Note:** Since the Admin account is in LDAP, you *will* see the First Run Wizard in the Admin UI. You can just click `Next` through it to get to the configured server.

(This does not affect the Runtime operations)

This configuration includes:

### Adapters
* HTML Form (Not Used)
* HTML Form with LIP
* Identifier-First (Passwordless)
* PingID (Not Used)
* PingID SDK

### PingID SDK - Special Considerations
The PingID adapter uses the secrets from your PingID tenant to create the proper calls to the service. As such, storing those values in a public location, such as GitHub, should be considered **risky**.

For this Profile, you can place the text from a `pingidsdk.properties` file into `postman_vars.json`. The API calls will base64 encode and inject into the PingIDSDK Adapter and HTML Form (for Self-Service Password Reset)

### Authentication Policy
Extended Property Selector
  * Enhanced (HTML Form with LIP)
  * MFA (Enhanced --> PingID SDK)
  * Passwordless (ID-First --> PingID SDK)
  * [Planned] QR-Code (PingID SDK)

The Authentication Experience is controlled by setting the `Extended Properties` on the Application.   

### Extended Properties
* `Enhanced` (HTML Form with LIP --  Facebook [not configured] & QR Code buttons) *default*
* `MFA` (HTML Form with LIP --> PingID SDK adapter)
* `Passwordless` (ID-First --> PingID SDK)
* _Anything Else_ (AuthN API Explorer)

### Authentication API
The AuthN API is enabled -- any value in the Extended Property *other* than the above will trigger it.
* ID-First --> HTML Form with LIP --> AuthN API Explorer 

### Applications
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

### Users
If you are using the BASELINE PingDirectory image, the credentials you can use for these applications are:

`user.[0-4]` / `2FederateM0re`
