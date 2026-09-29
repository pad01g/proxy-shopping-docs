---
layout: page
title: proxy-shopping
permalink: /
---

**proxy-shopping** — a P2P network for having a *proxy shopper* buy for you with crypto (BTC signet / USDC)
at shops that accept only cash or specific payment methods. Choose your language:

<ul class="langchooser" style="list-style:none;margin-left:0">
{%- for l in site.data.languages %}
  <li style="margin-bottom:.7em" lang="{{ l.code }}" dir="{{ l.dir }}"><a href="{{ '/' | append: l.path | append: '/' | relative_url }}" hreflang="{{ l.code }}"><strong>{{ l.name }}</strong></a><br>{{ l.tagline }}</li>
{%- endfor %}
</ul>

Missing your language, or found a mistake? [Pull requests are welcome](https://github.com/pad01g/proxy-shopping-docs/blob/main/CONTRIBUTING.md).
