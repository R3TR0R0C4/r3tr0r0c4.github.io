---
layout: page
title: Project Hub
hide_side_bar: false
---

Welcome to my security and networking repository. This site serves as a centralized knowledge base for my technical journey.

| Category | Description |
| :--- | :--- |
| [**CTFs**](/categories/ctfs/) | Capture The Flag writeups and challenge solutions. |
| [**CTF Tools**](/ctf-tools/) | Tools comonly used in CTFs. |
| [**General Wiki**](/wiki/) | Miscellaneous IT notes and security research. |
| [**Cisco Wiki**](/cisco-wiki/) | IOS commands, and infrastructure guides. |

---

### Capture The Flag
I focus primarily on **Web Exploitation** and **Privilege Escalation**. You can find my latest machine walkthroughs in the [CTF section](/categories/ctfs/).

---

## Recent Activity
{% for post in site.posts limit:5 %}
- **[{{ post.title }}]({{ post.url | relative_url }})** *{% if post.categories %}{{ post.categories | join: ', ' }}{% endif %}* — {{ post.date | date: "%b %d, %Y" }}
{% endfor %}

---

### Quick Links
[GitHub](https://github.com/r3tr0r0c4) | [TryHackMe](https://tryhackme.com/p/rocanico5) | [LinkedIn](https://www.linkedin.com/in/nicola-roca-mühlemann-408419222/)