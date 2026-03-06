---
layout: default
title: "Welcome to AbbySec"
---

# ☠️ Abbhilash Simanchalam ☠️

Cybersecurity student at UNITEN | CTF player | Bug bounty enthusiast  
Welcome to my portfolio – here you'll find:

<p>
  <a href="/ctf.html">▶️ CTF Writeups</a><br>
  <p> </p>
  <a href="/bugbounty.html">🐞 Bug Bounty Reports</a><br>
  <p></p>
  <a href="/events.html">🦹 Events</a><br>
  <p></p>
  <a href="/osint.html">💀 OSINT Investigations (Real Cases)</a><br>
</p>

---
<p><strong>Latest Post:</strong></p>
{% for post in site.posts %}
### 🔗 [{{ post.title }}]({{ post.url }})
📅 {{ post.date | date: "%B %d, %Y" }}
<hr>
{% endfor %}
