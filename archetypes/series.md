---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
author: {{ $.Site.Params.Author | default "Manish Sahani" }}
subtitle: ""
description: ""
about: ""
banner: ""
layout: "pages/series"
---
