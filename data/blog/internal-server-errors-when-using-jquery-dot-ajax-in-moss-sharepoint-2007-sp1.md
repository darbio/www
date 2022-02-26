---
title: '500 Internal Server Errors when using jQuery.ajax in MOSS SharePoint 2007 SP1'
date: 2012-02-24 23:22
tags: ['jquery', 'sharepoint']
draft: false
summary: Errors when using jQuery with SharePoint
---

I have recently been architecting a SharePoint forms solution which uses jQuery.ajax To communicate, using JSON, with a .Net 2.0 web service - hosted in the `\_vti\_bin` / ISAPI folder of a SharePoint host.

For various reasons we have been developing in SharePoint 2010, and it has worked fine in our dev environments. Today we did our first preliminary deployment of the solution into the client's development environment...

After some fun and games replacing the SharePoint 14 dll's with the MOSS 12 dll's we were ready to deploy the solution.

The application pages worked, however, we are using AJAX calls to communicate with a web service. MOSS did not like this, returning a 500 Internal Server error.

After some head scratching, numerous coffees and some less than savory exchanges we realized that the web service worked when we requested it using SOAP, but that when the content-type was set to JSON the server baulked and threw a not so helpful exception:

`"{"Message":"There was an error processing the request.","StackTrace":"","ExceptionType":""}"`

![""][1]

After some googling, I found out that MOSS does not support AJAX at all in the RTM. Indeed even in SP1 AJAX is not out of the box functionality.

Mike Ammerlaan has written some <a href="http://msdn.microsoft.com/en-us/library/bb861898.aspx" target="_blank">instructions on MSDN on how to enable AJAX extensions in WSS</a> - they are pretty long winded and perhaps a little daunting for a MOSS admin to get implemented overnight... (Especially in a large corporate). The solution is due very soon, in short - I need these installed yesterday!

To cut a long story short, you have to have MOSS SP1, then you have to install the ASP.Net AJAX Extensions, and then you have to edit copious web.config files, before putting your head between your legs and praying!..

Enter the dragon...

I found an Open Source project named <a title="AJAXify moss" href="http://ajaxifymoss.codeplex.com/" target="_blank">AJAXify MOSS</a> on CodePlex.

> stsadm commands for adding/removing needed web.config entries to MOSS Web Applications for ASP.NET AJAX 1.0 extensions.

As it says on the tin, the solution adds the web.config values required to get the [ASP.Net AJAX extensions][2] installed into MOSS - doing all of the hard work for us.

Clever hey?! Kudos to [Rich Finn][3]

[1]: http://i.imgur.com/r1WBB.png
[2]: http://www.microsoft.com/download/en/details.aspx?displaylang=en&id=883 'ASP.Net AJAX'
[3]: http://blog.richfinn.net/blog/
