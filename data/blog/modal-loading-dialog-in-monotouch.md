---
title: 'Modal loading dialog in MonoTouch'
date: 2012-05-23 23:57
tags: ['MonoTouch', 'dotnet']
draft: false
summary: How to implement a modal loading dialog in MonoTouch
---

**Update:** I was using this in a project which called the <code>UILoadingView</code> class from an anonymous delegate and kept coming up against the dreaded SIGSEGV error... Well it was a silly mistake on my behalf. I forgot to wrap the `DismissWithClickedButtonIndex(0, true);` in `BeginInvokeOnMain (...)`. Code updated below.

As some of you may know, I work as a Solutions Architect at <a href="http://www.iqxbusiness.com">IQX</a> in Sydney - a company specialising in SAP integration with, well, pretty much everything.

Part of our offering is <a href="http://www.iqxbusiness.com/docs/datasheets/iqmobility2-1.pdf?Status=Master">mobility</a>, and of late we have been refactoring our apps to make use of (Read: DogFood) one of our products, <a href="http://www.iqxbusiness.com/docs/datasheets/iq-foundation_datasheet.pdf">IQfoundation</a>. In short, IQfoundation is an accelerator to access SAP data from anything. For those of you who are interested, the datasheet is <a href="http://www.iqxbusiness.com/docs/datasheets/iq-foundation_datasheet.pdf">here</a>.

Shameless plug over...

The point of all of this is that, accessing SAP can be slow, especially if you are on an iPhone, connected by 3G into your network.

In the past we have put up a little spinner in the top right corner of the app, which tells the user when data is being accessed, and when that request has finished. All a little bit too subtle for some... Which is fine for interactions where you don't want to block user input, but for areas of the app which rely on the data being there before the user can continue it was a flaw in the design of our apps, making them less user friendly than they could be.

![](http://i.imgur.com/4Ljzo.png)

In case you missed it (don't worry, you are not alone) it's that little, subtle, light-footed white spinner up the top. How very quaint.

In the example screenshot above, we don't want to block the user interaction with the app, so a spinner is great! But... What about areas of the app which we don't want the user to continue with until they have a full set of data?

Within the same app, <a href="http://developer.apple.com/library/ios/#documentation/UserExperience/Conceptual/MobileHIG/UEBestPractices/UEBestPractices.html#//apple_ref/doc/uid/TP40006556-CH20-SW12">against Apple's infinite wisdom</a>, we have some in-app settings which require communication with SAP before the user should be able to continue on interacting with the app.

![](http://i.imgur.com/Ar92k.png)

Obviously (or not) these are selections from data available in SAP. What is the user tried to select the Sales Organisation, before it had been loaded from SAP? What is the user tried to select the Distribution Channel before the Sales Organisation had filtered them down? Would the world end?

Luckily, the world doesn't end - but those not possessing the eyesight of an eagle may not have spotted the little spinner in the top right corner - leading to a bug report of "It doesn't work".

So, how to fix the problem? As always, Apple's iOS User Experience guidelines are a good start. But I also like to look at how others have done it.

A quick Google returned a blog by <a href="http://www.yetanotherchris.me/home/2010/10/10/monotouch-tips-and-snippets.html">Chris Small</a> who found an <a href="http://mobiledevelopertips.com/user-interface/uialertview-without-buttons-please-wait-dialog.html">interesting ObjectiveC link</a> and noted that:

<blockquote>On the iPhone there is an example of a modal "loading" dialog when you set the wallpaper from one of your images.</blockquote>
Good find Chris! I would have checked this, but... Well my cat would be very disappointed if I changed my wallpaper, and she already scratched me this morning... I'll take your word for it.

Chris details some code which works, however when I used it didn't work quite how I wanted it to. If I had a shared UILoadingView upon which .Show () was called multipel times, I would get the spinner redrawn every time - looked a bit ugly when this happened. I also played with the numbers slightly, though these really do depend upon the text in your message (Maybe I will update it in the future to do some funky auto-calculations)...

I refactored it slightly, making it behave more like the UIAlertView I was used to. Giving me:

![](http://i.imgur.com/X8F5L.png)

Why is this good? Well it solves my problem... The user knows that the data is loading, and the `UIAlertView` stops them from interacting with the application until all of the data is loaded. Magic! Just remember to put in some timeout handling around your longer running tasks... We don't want to leave the user in an endless wait... Not yet anyway...

Here is the class:

```csharp
using System;
using MonoTouch.UIKit;
using System.Drawing;
using System.Linq;

// Original idea:	http://mymonotouch.wordpress.com/2011/01/27/modal-loading-dialog/
// Based on:		http://mobiledevelopertips.com/user-interface/uialertview-without-buttons-please-wait-dialog.html

public class UILoadingView : UIAlertView
{
    private UIActivityIndicatorView activityIndicatorView;

    public new bool Visible {
        get;
        set;
    }

    public UILoadingView (string title, string message) : base (title, message, null, null, null)
    {
        this.Visible = false;
        activityIndicatorView = new UIActivityIndicatorView (UIActivityIndicatorViewStyle.WhiteLarge);
        AddSubview (activityIndicatorView);
    }

    public new void Show ()
    {
        base.Show ();

        activityIndicatorView.Frame = new RectangleF ((Bounds.Width / 2) - 15, Bounds.Height - 60, 30, 30);
        activityIndicatorView.StartAnimating ();
        this.Visible = true;
    }

    public void Hide ()
    {
        this.Visible = false;
        activityIndicatorView.StopAnimating ();

        BeginInvokeOnMainThread (delegate () {
            DismissWithClickedButtonIndex(0, true);
        });
    }
}
```

Enjoy! Let me know if you use it.
