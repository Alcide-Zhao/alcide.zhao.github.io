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

In preparation
======
------
**Zhao A.** et, The East-West divide in global air pollution: drivers and impacts, aiming at **Nature**.  <br/><br/>
Pandey, A.K., Stevenson, D.S., **Zhao, A**., et al., Evaluating Nitrogen dioxide in UKCA using OMI satellite retrievals over South and East Asia, aiming at **Atmospheric Chemistry and Physics**. <br/><br/>

In review
======
------
Stevenson, D.S., **Zhao, A.**., et al., Trends in global tropospheric hydroxyl radical and methane lifetime since 1850 from AerChemMIP, **Atmospheric Chemistry and Physics**. [in revision](https://doi.org/10.5194/acp-2019-1219). <br/><br/>



Published
======
------
{% for post in site.publications reversed %}
  {% include archive-single.html %}
{% endfor %}
