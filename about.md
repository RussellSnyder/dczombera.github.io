---
layout: page
title: About me
description: Little information about the human behind this site.
keywords: about page, about me
---
Hello there!  
  
My name is Dennis Czombera and I'm a self-taught software developer based in Berlin where I live with my beautiful wife.  
I mainly write software for the web and I'm highly interested in the Internet, OSS, privacy and the [open access to knowledge and information](https://archive.org/stream/GuerillaOpenAccessManifesto/Goamjuly2008_djvu.txt). 

Any questions? Drop me a mail at **me(at)dennisczombera.com**

{% assign page_image = '' %}
{% capture page_image %}
{% if page.about_img %}
  {{ page.about_img | prepend: site.baseurl | prepend: site.url }}
{% else site.about_img %}
  {{ site.about_img | prepend: site.baseurl | prepend: site.url }}
{% endif %}
{% endcapture %}
{% capture page_image %}{{ page_image | strip | rstrip | lstrip | escape | strip_newlines }}{% endcapture %}

![about_me]({{page_image}}){:height="400px" width="400px"}

