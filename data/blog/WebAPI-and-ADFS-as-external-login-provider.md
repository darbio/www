---
layout: post
title: 'WebAPI and ADFS as external login provider'
date: 2014-06-30 12:19
comments: true
categories:
---

# Setting up ADFS

For these steps, I presume that you are running ADFS 3 on Windows Server 2012 R2 and that it has been already installed and configured.

## Setting up the Relying Party Trust

1. Open ADFS Managememt
2. Expand 'Trust Relationships' and click on 'Relying Party Trusts'.

![http://i.imgur.com/HeGm5yl.png](http://i.imgur.com/HeGm5yl.png)

3. Click on 'Add Relying Party Trust' in the right hand panel.

![http://i.imgur.com/o22tVfW.png](http://i.imgur.com/o22tVfW.png)

4. Click 'Start' and on the next page select 'Enter data about the relying party manually' before clicking 'Next'.
5. Enter a display name for the relying party (e.g. the name of the application) and click 'Next'.
6. Ensure the 'AD FS profile' is selected and click 'Next'.
7. Ignore the optional token encryption certificate, and click 'Next'.
8. On the next page check the box for WS-Federation, and enter your WS-Federation endpoint. Click 'Next'.

![http://i.imgur.com/vYOB74r.png](http://i.imgur.com/vYOB74r.png)

9. Add your identifier for the relying party. This is usually the web address for the application (e.g. https://app.example.com). Click 'Next'.
10. Ensure 'I do not want to configure multi-factor authentication' is selected and click 'Next'.
11. Ensure 'Permit all users to access this relying party' is checked and click 'Next'.
12. On the next page, click 'Next', followed by 'Close' on the next step.

## Adding claims

1. Right click on the relying party in the trusts dialog, and click 'Edit claim rules'.

![http://i.imgur.com/ammrQrI.png](http://i.imgur.com/ammrQrI.png)

2. Click on 'Add rule', and ensure trhat 'Send LDAP Attributes as Claims' is selected in the drop down. Click 'Next'.
3. Add a name for the rule, for example 'Email address' and select 'Active Directory' as the attribute store.
4. In the mapping panel, select 'Email Addresses' from the LDAP Attribute selector, and 'Email Address' in the claim type.

![http://i.imgur.com/1NYUmG6.png](http://i.imgur.com/1NYUmG6.png)

5. Click 'Finish'.
6. Repeat steps 13 to 17 for each claim type you would like to send to the relying party (e.g. Given name and Surname).
7. Close the relying party dialog by clicking 'OK'.

**N.B.** For WebAPI to work out of the box, you will need to add a 'Name ID' claim type for the user. For example:

![http://i.imgur.com/vENaDlh.png](http://i.imgur.com/vENaDlh.png)

## Adding ADFS as a authentication provider in Web API

1.  Open your `Startup.Auth.cs` file and add the following:

        // Add ADFS to our list
        var adfsOptions = new WsFederationAuthenticationOptions()
        {
            Wtrealm = "{application-uri}",
            MetadataAddress = "https://{adfs-server-uri}/federationmetadata/2007-06/federationmetadata.xml",
            Caption = "ADFS for Darb.io",
            AuthenticationType = "DARBIO.Federation",
            AuthenticationMode = Microsoft.Owin.Security.AuthenticationMode.Passive
        };
        app.UseWsFederationAuthentication(adfsOptions);

## Linking up to Web API external login

TODO
