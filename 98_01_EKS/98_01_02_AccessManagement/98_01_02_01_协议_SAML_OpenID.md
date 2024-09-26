
SAML (SAML 1.0 and 2.0) and OpenID Connect (OIDC) are identity protocols, designed to authenticate users, and provide identity data for access control and as a communication method for a user’s identity. Either protocol may be the basis for Identity Providers (IdPs) that offer a range of user identity management and services and may be used for single sign-on (SSO) applications.


这连个协议用于 Identity Providers (IdPs) 和 single sign-on (SSO) applications


# 1 SAML

Mainly used for Enterprise and Government applications, [SAML 2.0](https://auth0.com/intro-to-iam/saml-vs-openid-connect-oidc/) is a mature technology dating from 2005 and supports a wide range of identity functionality. SAML uses XML for its identity data format and simple HTTP or SOAP for data transport mechanisms. 
- Relying Party (RP) : The service requesting and receiving data from the IdP is known as the Relying Party (RP). 
- SAML Assertion: The user identity data, encapsulated in an XML document called the SAML Assertion, is in the form of attributes, e.g., email address, name, phone, etc.

SAML can provide a mechanism for federated identity.





