---
layout: post
title: 'Compile mono from source on OSX'
date: 2012-07-25 00:12
draft: false
tags: ['Mono']
summary: 'How to compile Mono from source on OSX'
---

The following is how I compiled mono latest from source:

<ol>
	<li>Install MacPorts from http://macports.org</li>
	<li>Install gettext
<pre>sudo port install gettext</pre>
</li>
	<li>Locate your current mono environment. It is usually at:
<pre>/Library/Framework/Mono.framework/Versions/Current</pre>
</li>
	<li>Download the mono source from GitHub master branch
<pre>git clone https://github.com/mono/mono.git</pre>
</li>
	<li>Run autogen with your OSX version prefix
<pre>./autogen.sh --prefix=/Library/Framework/Mono.framework/Versions/Current</pre>
</li>
	<li>Run make
<pre>sudo make</pre>
</li>
	<li>Optionally, you can run the tests
<pre>sudo make check</pre>
</li>
	<li>Run make install
<pre>sudo make install</pre>
</li>
</ol>
