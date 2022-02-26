---
title: 'Sending an email with an attachment from Workflow Foundation'
date: 2012-02-24 23:32
draft: false
tags: ['Sharepoint', 'DotNet']
summary: 'Send an email from a SharePoint WorkFlow, using the SP Outbound Mail Service and adding some attachments to the message.'
---

## The problem:

The `SendEmailActivity` in WF sucks... because you can't send an email with attachments from it.

I want to be able to send an email from a SharePoint WorkFlow, using the SP Outbound Mail Service and adding some attachments to the message.

## The workaround:

Use the .Net `MailMessage` class (`System.Net.Mail`) and a Code activity to send the email:

Non-SharePoint example:

```csharp
// Set the variables
string smtpAddress = "mail.example.com";
string fromAddress = "from@example.com";

// Create the SMTPClient
var smtp = new SmtpClient(smtpAddress);
smtp.Credentials = new NetworkCredential("username", "password");

// Construct the MailMessage
var mail = new MailMessage();
mail.From = new MailAddress(fromAddress);
mail.To.Add("to@example.com");
mail.Subject = "Email subject here";
mail.Body = "Body";

// Attach the files
mail.Attachments.Add(new Attachment("filename"));
mail.Attachments.Add(new Attachment("filename"));

// Send the MailMessage
smtp.Send(mail);
```

SharePoint example:

```csharp
using (var web = new SPSite("http://localhost").OpenWeb())
{
    // Get the variables from SP
    string smtpAddress = web.Site.WebApplication.OutboundMailServiceInstance.Server.Address;
    string fromAddress = web.Site.WebApplication.OutboundMailSenderAddress;

    // Create the SMTPClient
    var smtp = new SmtpClient(smtpAddress);
    smtp.Credentials = CredentialCache.DefaultNetworkCredentials;

    // Construct the MailMessage
    var mail = new MailMessage();
    mail.From = new MailAddress(fromAddress);
    mail.To.Add("to@example.com");
    mail.Subject = "Email subject here";
    mail.Body = "Body";

    // Attach the files
    mail.Attachments.Add(new Attachment("filename"));
    mail.Attachments.Add(new Attachment("filename"));

    // Send the MailMessage
    smtp.Send(mail);
}
```

## SharePoint Gotcha's

SharePoint's smtp server must be configured in Central Admin, otherwise you will get a `NullReferenceException` at `web.Site.WebApplication.OutboundMailServiceInstance`

Restart the SharePoint Timer Service to refresh it's assembly cache if the workflow does not kick in

`SPContext` is not available in the Timer Job Context (under which the WF will run)

Get the source code here
