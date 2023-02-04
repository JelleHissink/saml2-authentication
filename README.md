[![Build](https://github.com/dina-heidar/saml2-authentication/actions/workflows/builld.yml/badge.svg)](https://github.com/dina-heidar/saml2-authentication/actions/workflows/builld.yml) [![License: CC0-1.0](https://img.shields.io/badge/License-CC0_1.0-lightgrey.svg)](https://github.com/dina-heidar/saml2-authentication/blob/main/LICENSE) [![Release](https://img.shields.io/github/release/dina-heidar/saml2-authentication.svg)](https://github.com/dina-heidar/saml2-authentication/releases/latest)

authRequest
-subject
-consent
-conditions
-scoping
-nametypeId not NameIDPolicy
-providername
-signaturetype

saml logout bindings https://www.oasis-open.org/committees/download.php/35387/sstc-saml-bindings-errata-2.0-wd-05-diff.pdf

http://docs.oasis-open.org/security/saml/Post2.0/sstc-saml-tech-overview-2.0.html
http://docs.oasis-open.org/security/saml/v2.0/saml-conformance-2.0-os.pdf

https://www.oasis-open.org/committees/download.php/35711/sstc-saml-core-errata-2.0-wd-06-diff.pdf

- found fun facts:
1. Authn requests either can have the assertionURl And protocol binding request OR just assertionConsumerIndex
2. ECDSA certificates are only for signing not encryptian

## Todo
OpenTelemtry
ability to sign authnRequest with ecdsa ???
ability to set issuer value for token validatio or defaults to Idp entityId
ability to set issuer endpoint index for signin, and logout or defaults to first Idp endpoint
~~SamlCookieName~~
~~ReqAuthn type as forms or windows~~
Dynamic provider set up
~~artifcate binding~~
Idp initiated logout
Idp initiated login "UnsolicitedLogins" set timout https://www.scottbrady91.com/saml/dangers-of-idp-initiated-sso
cake build?
choco?
dockerfile for otel collector?
This iFrame should then be displayed on your logged out view like so:

@if (Model.SamlSignOutIframeUrl != null)
{
    <iframe width="0" height="0" class="samlsignout" src="@Model.SamlSignOutIframeUrl"></iframe>
}

new 
    SignedOutRedirectUri
    RemoteSignOutPath


    currently=> authn is redirect binding
            =>logout is redirect binding


    authnRequest => redirect or post(ispassive?) + only front channel
        -The IDP can generally use any method it wants (Username/Password, Digital Certs, two-factor auth, etc). However, if you are sending an "IsPassive" in the AuthnRequest the IDP must not prompt the end user for any authentication information. Typically, something like IWA/Kerberos performs this way. If the IDP cannot determine the identity of the user passively, then you must return the appropriate error response to the SP. BTW - using HTTP Post the AuthnRequest is sent via the User-Agent.
        - isPassive=true.....There are a few common ways to re-authenticate a user with IsPassive=true. For example, Integrated Windows Auth (Kerberos) and x509 Cert Based Auth can both be done w/out visibly working with the user's experience.
        If you combine a ForceAuthn=true and IsPassive=true in your AuthnRequest, it should force the IDP to re-authenticate the user if both conditions can be met.
    
        
        logoutRequest=>redirect or post + can be a backchannel




# SAML2Core Authentication

This is a Saml2 middleware that can be used with any .Net Core applications to authenticate via SAML. This middleware is opensource (free to use) and is not dependent on any .NET Framework libraries. It has been tested with ADFS and IdentityServer4 as well.

##### The middleware:
1. Creates AuthnRequests to the Idp.
2. Decrypts encrypted SAML ssertions (if encrypted).
3. Checks and validates Idp signatures (if required, see options).
4. Converts the Saml Assertions into User claims.
5. Self generates the SP (application) metadata.xml file if needed (see options).  

## Nuget Installation 

[Link to Nuget page](https://www.nuget.org/packages/SAML2Core/)
```cs

PM> Install-Package SAML2Core

```
## Options

### 1. Required

| Name                      | Example value                                                                 | Datatype          |    Description    |
| -------------             |-------------                                                                  | -----             |    -----         |
| SignOutPath               | "/signedout"                                                                  | string            | The endpoint for the IDP to perform its signout action |
| ServiceProvider.EntityId  | "https://my.la.gov.local"                                                     | string            | The Relying Party Identifier |
| MetadataAddress           | "https://dev.adfs.la.gov/federationmetadata/2007-06/FederationMetadata.xml"  | string            | The IDP FederationMetadata. This can be either a URL or a file at the root of your project e.g '@"FederationMetadata.xml"' |



##### The following section is if your application (SP) has a certificate and is signing the Authn Request with it.


| Name                                                  | Example value                                     | Datatype          |    Description    |
| -------------                                         |-------------                                      | -----             |    -----          |
| ServiceProvider.X509Certificate2                      | new X509Certificate2(@"democert.pfx", "1234");    | string, string    | The SP certificate file location, password       |

###### OR using an X509Certificate2 extension to search the Certificate store

####### PRE-REQUISITE: Install your certificate in your server/local certificate store under the Trusted Root folder. [Click here](https://blogs.technet.microsoft.com/sbs/2008/05/08/installing-a-self-signed-certificate-as-a-trusted-root-ca-in-windows-vista)
                                                                                   
ServiceProvider.X509Certificate2 = new Cryptography.X509Certificates.Extension.X509Certificate2(
                                                            Configuration["AppConfiguration:ServiceProvider:CertificateSerialNumber"],
                                                            StoreName.My,
                                                            StoreLocation.LocalMachine,
                                                            X509FindType.FindBySerialNumber);  


 
### 2. Optional

| Name                      | Example value                                                                 | Datatype          |    Description    |
| -------------             |-------------                                                                  | -----             |    -----         |
| WantAssertionsSigned | true | boolean | Require the IDP to sign assertions. The default is 'false' |
| RequireMessageSigned | false | boolean | Require the IDP to sign assertions. The default is 'false'. This must be set as well on IDP side. |  
| CreateMetadataFile              | true                                                                 | boolean           | Have the middleware create the metadata file for you. The default is false.|
| DefaultMetadataFileName  | "MyMetadataFilename"                                                  | string            | the default is "Metadata" |
| DefaultMetadataFolderLocation           | "MyPath" | string            | the default is "wwwroot" so it can be accessible via "https://[host name]/MyMetadataFilename.xml". |
| VerifySignatureOnly          | true | boolean           | the default is 'true', this means only the signing signature will be validated.. If set to 'false', both the signing signature and its certificate will be validated. so it can be accessible via "https://[host name]/MyMetadataFilename.xml". | 
| ForceAuthn| true| boolean| if you are requiring users to enter credentials into the IDP every time. Default is set to true. |
| ServiceProvider.ServiceName |"My Test Site" | string | 
| ServiceProvider.Language | "en-US" | string |


## Usage

1. Modify `ConfigureServices()` in Startup.cs
```cs
services.AddAuthentication(sharedOptions =>
{
    sharedOptions.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    sharedOptions.DefaultSignInScheme = CookieAuthenticationDefaults.AuthenticationScheme;
})
   .AddSamlCore(options =>
            {

            // SignOutPath- The endpoint for the idp to perform its signout action. default is "/signedout"
                //options.SignOutPath = "/signmeout";

            // EntityId (REQUIRED) - The Relying Party Identifier e.g. https://my.la.gov.local
                options.ServiceProvider.EntityId = Configuration["AppConfiguration:ServiceProvider:EntityId"];

            // There are two ways to provide FederationMetadata:
                // Option 1 - A FederationMetadata.xml already exists for your application
                // options.MetadataAddress = @"FederationMetadata.xml";

                // Option 2 - Have the middleware generate the FederationMetadata.xml file for you
                options.MetadataAddress = Configuration["AppConfiguration:IdentityProvider:MetadataAddress"];

                options.RequireMessageSigned = false;
                options.WantAssertionsSigned = true;

            // Have the middleware create the metadata file for you
                // The default is false. If you don't want a file generated by the middleware, comment the line below.
                options.CreateMetadataFile = true;

                // If you want to specify the filename and path for the generated metadata file do so below:
                //options.DefaultMetadataFileName = "MyMetadataFilename.xml";
                //options.DefaultMetadataFolderLocation = "MyPath";

                // (REQUIRED IF) signing AuthnRequest with Sp certificate to Idp. The value here is the certifcate serial number.
                //if the certificate is in the project. make sure the path to ti is correct. 
                //password value is needed to access private keys for signature and decryption.
                options.ServiceProvider.X509Certificate2 = new X509Certificate2(@"democert.pfx", "1234");

                //if you want to search in cert store - can be used for production
                options.ServiceProvider.X509Certificate2 = new Cryptography.X509Certificates.Extension.X509Certificate2(
                    Configuration["AppConfiguration:ServiceProvider:CertificateSerialNumber"],
                    StoreName.My,
                    StoreLocation.LocalMachine,
                    X509FindType.FindBySerialNumber);

                // Force Authentication (optional) - Is authentication required?
                options.ForceAuthn = true;


                options.WantAssertionsSigned = false;
                options.RequireMessageSigned = true;

           // Service Provider Properties (optional) - These set the appropriate tags in the metadata.xml file
                options.ServiceProvider.ServiceName = "My Test Site";
                options.ServiceProvider.Language = "en-US";
                options.ServiceProvider.OrganizationDisplayName = "Louisiana State Government";
                options.ServiceProvider.OrganizationName = "Louisiana State Government";
                options.ServiceProvider.OrganizationURL = "https://my.test.site.gov";
                options.ServiceProvider.ContactPerson = new ContactType()
                {
                    Company = "Louisiana State Government - OTS",
                    GivenName = "Dina Heidar",
                    EmailAddress = new[] { "dina.heidar@la.gov" },
                    contactType = ContactTypeType.technical,
                    TelephoneNumber = new[] { "+1 234 5678" }
                };
                //Events - Modify events below if you want to log errors, add custom claims, etc.
                options.Events.OnRemoteFailure = context =>
                {
                    return Task.FromResult(0);
                };
                options.Events.OnTicketReceived = context =>
                {  //if you need to add custom claims here
                    
                    //example:
                    //var identity = context.Principal.Identity as ClaimsIdentity;
                    //var claims = context.Principal.Claims;                 
                    //if (claims.Any(c => c.Type == "userID"))
                    //{
                    //    var userId = claims.FirstOrDefault(c => c.Type == "userID").Value;
                    //    var name = claims.FirstOrDefault(c => c.Type == ClaimTypes.Name);
                    //    identity.TryRemoveClaim(name);
                    //    identity.AddClaim(new Claim(ClaimTypes.Name, userId));
                    //}
                    return Task.FromResult(0);
                };
            })

.AddCookie();
```

2. Don't forget to add the following line in `Configure()`

```cs
app.UseAuthentication();
```