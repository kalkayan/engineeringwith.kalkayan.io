---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
author: {{ .Author | default "Manish Sahani" }}
subtitle: ""
description: ""
about: ""
banner: banner.png
layout: "pages/series"
---
