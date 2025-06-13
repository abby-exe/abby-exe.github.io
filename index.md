---
layout: default
title: "Welcome to AbbySec"
---

# 👨🏽‍💻 Abbhilash Simanchalam

Cybersecurity student at UNITEN | CTF player | Bug bounty enthusiast  
Welcome to my portfolio – here you'll find:

- 🧠 CTF writeups
- 🐞 Bug bounty reports
- 📄 Projects and resume

---

{% for post in site.posts %}
### 🔗 [{{ post.title }}]({{ post.url }})
📅 {{ post.date | date: "%B %d, %Y" }}
<hr>
{% endfor %}
