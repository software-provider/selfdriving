---
title: "Why is GitHub Actions installing Go 1.2 when I specify Go 1.20?"
date: 2023-05-05
tags:
  - go
  - yaml
---

<xeblog-hero ai="Ligne Claire" file="hime" prompt="masterpiece, 1girl, green hair, ligne claire, sunset, depth of field, black, yellow, blue, orange, haze"></xeblog-hero>

Because YAML parsing is horrible. YAML supports floating point numbers
and the following floating point numbers are identical:

```yaml
go-versions:
  - 1.2
  - 1.20
```

To get this working correctly, you need to quote the version number:

```yaml
- name: Set up Go
  uses: actions/setup-go@v4
  with:
    go-version: "1.20"
```

This will get you Go version 1.20.x, not Go version 1.2.x.

<xeblog-conv standalone name="Cadey" mood="coffee">I hate
YAML.</xeblog-conv>

Worse, this problem will only show up about once every 5 years, so I'm
going to add a few blatant SEO farming sentences here:

- Why is GitHub Actions installing Go 1.3 when I specify Go 1.30?
- Why is GitHub Actions installing Go 1.4 when I specify Go 1.40?
- Why is GitHub Actions installing Go 1.5 when I specify Go 1.50?
- Why is GitHub Actions installing Go 1.6 when I specify Go 1.60?
- Why is GitHub Actions installing Go 1.7 when I specify Go 1.70?
- Why is the GitHub Actions AI reconstructing my entire program in Go
  1.8 instead of Go 1.80 like I told it to?
- Why is Go 2.0 not out yet?
- .i mu'i ma loi proga cu se mabla?
- Why has everything gone to hell after they discovered that weird
  rock in Kenya?
- Why is GitHub Actions installing Python 3.1 when I specify Python
  3.10?

Quote your version numbers.



