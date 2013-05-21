---
layout: post
title: "css at the end of page delays page rendering"
date: 2013-05-22 00:29
comments: true
categories: 
---

{% codeblock css_at_the_end lang:html %}
<html>
<head><title>css at the end</title> </head>
<body>
<div class="class">
	Lorem ipsum dolor sit amet, consectetuer adipiscing elit, 
</div>
<b> more elements ... </b>

<link rel="stylesheet" type="text/css" href="app.css">
</body>
</html>
{% endcodeblock %}

would take more time to load than 

{% codeblock css_in_the_head lang:html %}
<html>
<head>
	<title>css_in_the_head</title> 
	<link rel="stylesheet" type="text/css" href="app.css">
</head>
<body>
<div class="class">
	Lorem ipsum dolor sit amet, consectetuer adipiscing elit, 
</div>
<b> more elements ... </b>
</body>
</html>
{% endcodeblock %}

as the browser has to repaint/rerender the elements with the css rules from `app.css` when placed at the end of doc.
