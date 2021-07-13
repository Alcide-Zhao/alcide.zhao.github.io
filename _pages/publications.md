---
layout: archive
permalink: /publications/
author_profile: false
sidebar:
  nav: "sidenav"
---

{% if author.googlescholar %}
  You can also find my articles on <u><a href="{{author.googlescholar}}">my Google Scholar profile</a>.</u>
{% endif %}

{% include base_path %}

Under Preparation & Review
======
------

**Zhao A.** et al., How well do the CMIP6 models simulate dust aerosols? (submitted to ACP) <br/><br/>
**Zhao A.** et al., The key role of atmospheric absorption in shaping summertime Asian dust climate impacts (under preparation) <br/><br/>
**Zhao A.** et al., The North-South divide in global air pollution: drivers and impacts (under preparation)<br/><br/>

Pandey, A.K., Stevenson, D.S., **Zhao, A**., et al., Evaluating Nitrogen dioxide in UKCA using OMI satellite retrievals over South and East Asia (under preparation) <br/><br/>

<!-- In Press
======
------

 -->


Published
======
------
{% for post in site.publications reversed %}
  {% include archive-single.html %}
{% endfor %}
