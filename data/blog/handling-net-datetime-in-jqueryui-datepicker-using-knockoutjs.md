---
title: 'Handling .Net DateTime in jQueryUI DatePicker using KnockoutJS'
date: 2012-03-05 23:37
draft: false
tags: ['jQueryUI', 'DotNet', 'KnockoutJS']
summary: 'How to use .Net DateTime type in jQuery UI'
---

In a recent project we were using <a href="http://knockoutjs.com/" target="_blank">KnockoutJS</a> to bind a .Net model to a JavaScript ViewModel to a HTML View.

The View has a number of jQuery UI controls on it, including the infamous the <a href="http://jqueryui.com/demos/datepicker/" target="_blank">DatePicker</a>.

The model had a number of 'complex' types on it, however the type which gave us the most headache was the 'simple' .Net CLR `DateTime` type.

When the JavaScriptSerializer serializes a .Net DateTime it spits out a string which looks like:

```javascript
"\/Date(1000001352100)/\"
```

Which, from my days as a Unix programmer, I recognised the to be the beloved Epoch timestamp (milliseconds since epoch/1st January 1970) surrounded by Date().

First suggestion was to eval the Date and be done with it... But we all know that <a href="http://blogs.msdn.com/b/ericlippert/archive/2003/11/01/53329.aspx" target="_blank">eval === evil</a> as it is sloooooow, and anyone can inject malicious code into the page and run it on our unsuspecting user - this was not an option.

Onwards...

After some messing about, we decided that creating a <a href="http://knockoutjs.com/documentation/custom-bindings.html" target="_blank">Knockout custom binding</a> would be the solution.

I stumbled upon a <a href="http://jsfiddle.net/rniemeyer/NAgNV/" target="_blank">jsFiddle</a> written by <a href="https://github.com/rniemeyer" target="_blank">rniemeyer</a> which implemented a custom jQuery UI DatePicker binding - all we had to do was shoehorn the conversion from the ViewModel DateTime string representation, to a JavaScript Date Object.

As the DateTime format from the JavaScript Serializer always spits out dates in the same format, we can:

1.  Use string.replace to remove all of the crap, leaving us with the Epoch time
2.  Use parseInt(string) to get the string as an int
3.  Use the new Date(epochTime) constructor to create the proper JavaScript Date object Initially I thought that we might need to convert back to Epoch time to send the data back to the server, however the JavaScriptSerializer is capable of reading the Date Object's .toString() method and converting it into the CLR DateTime type. Too easy! Here is the

<a href="http://jsfiddle.net/macropus/An4Jc/" target="_blank">fiddle</a> and the code for completeness sake:

```javascript
ko.bindingHandlers.datepicker = {
  init: function (element, valueAccessor, allBindingsAccessor) {
    //initialize datepicker with some optional options
    var options = allBindingsAccessor().datepickerOptions || {}
    $(element).datepicker(options)

    //handle the field changing
    ko.utils.registerEventHandler(element, 'change', function () {
      var observable = valueAccessor()
      observable($(element).datepicker('getDate'))
    })

    //handle disposal (if KO removes by the template binding)
    ko.utils.domNodeDisposal.addDisposeCallback(element, function () {
      $(element).datepicker('destroy')
    })

    //handle .Net DateTime
    var value = ko.utils.unwrapObservable(valueAccessor())
    var obs = valueAccessor()

    if (obs() !== null && !isNaN(obs().replace(/\/Date\((-?\d+)\)\//, '$1'))) {
      obs(new Date(parseInt(value.replace(/\/Date\((-?\d+)\)\//, '$1'))))
      $(element).datepicker('setDate', obs())
    }
  },
  update: function (element, valueAccessor) {
    var value = ko.utils.unwrapObservable(valueAccessor()),
      current = $(element).datepicker('getDate')

    if (value - current !== 0) {
      $(element).datepicker('setDate', value)
    }
  },
}
```
