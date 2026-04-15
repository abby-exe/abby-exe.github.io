---
layout: default
title: "Welcome to AbbySec"
---

# ☠️ Abbhilash Simanchalam ☠️

Cybersecurity graduate from UNITEN | CTF player | OSINT enthusiast | Bug bounty enthusiast

I’m a cybersecurity enthusiast who focuses on thinking like an attacker to build stronger defenses.

I actively practice through:

> 🔍 CTF challenges
> 🕵️ OSINT investigations
> 🌐 Web application security testing (in near future!)

I focus on understanding how vulnerabilities actually work, not just using tools.

Welcome to my portfolio – here you'll find:

<p>
  <a href="/ctf.html">▶️ CTF Writeups</a><br>
  <p> </p>
  <a href="/bugbounty.html">🐞 Bug Bounty Reports</a><br>
  <p></p>
  <a href="/events.html">🦹 Events</a><br>
  <p></p>
  <a href="/osint.html">💀 OSINT Investigations (Real Cases)</a><br>
  <p></p>
  <a href="/simulatedosint.html">🕵🏽‍♂️ Simulated OSINT Investigations</a><br>
  <p></p>
  <a href="/experiments.html">🔬 Experiments (Projects)</a><br>
</p>

---
<p><strong>Latest Post:</strong></p>
{% for post in site.posts %}
### 🔗 [{{ post.title }}]({{ post.url }})
📅 {{ post.date | date: "%B %d, %Y" }}
<hr>
{% endfor %}
