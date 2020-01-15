This profile provides a CIAM configuration for PingFederate - it starts with [PF-Base](https://github.com/cprice-ping/Profile-PF-Base) and makes the following modifications:

* Replaces PingID with PingID SDK
 * Creates PID SDK Email Template for PF AuthN
 * Uses PID SDK in MFA Policy
* Adds CIBA to the OIDC AS
 * Adds the PingID SDK CIBA Authenticator
* [Planned] PA configuration (PA-Base \ PA-CIAM)


It uses Postman to do an Admin API collection set to fully configure PF from a Ping Docker image.

The Postman collections are documented here:  
* [PF Admin API - Base](https://documenter.getpostman.com/view/1239082/SWLh4RQB)
* [PF Admin API - CIAM](https://documenter.getpostman.com/view/1239082/SWLk4kLF)

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

For this Profile, you can place the text from a `pingidsdk.properties` file into `postman_vars.json`. The API calls will base64 encode and inject into the PingID Adapter and HTML Form (for Self-Service Password Reset)

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
