---
title: 使用Github Actions定制Github个人主页
date: 2025-4-5 23:18:47
top_img: /img/CoverImage1.png
cover: /img/CoverImage1.png
type: "post"
categories:
  - Github
tags:
  - Github-Actions
---
# 先放一张效果图
![](https://cdn.jsdelivr.net/gh/admibrill/admibrill@master/metrics.svg)
# 教程

其实就是利用这个lowlighter/metrics的gh-actions来每天定时生成一张svg。
详细教程见[lowlighter/metrics: 📊 An infographics generator with 30+ plugins and 300+ options to display stats about your GitHub account and render them as SVG, Markdown, PDF or JSON!](https://github.com/lowlighter/metrics)

# `metrics.yml`

以下就是我的同款workflow配置
```yml
name: Metrics
on:
  schedule: [{cron: "0 0 * * *"}]
  workflow_dispatch:
  push: {branches: ["main"]}
jobs:
  github-metrics:
    runs-on: ubuntu-latest
    environment: 
      name: production
    permissions:
      contents: write
    steps:
      - uses: lowlighter/metrics@latest
        with:
          filename: metrics.svg
          token: ${{ secrets.METRICS_TOKEN }}
          config_timezone: Asia/Shanghai
          base: header, activity, community, repositories, metadata
          
          plugin_isocalendar: yes
          plugin_isocalendar_duration: full-year
          
          plugin_lines: yes
          plugin_lines_history_limit: 1
          plugin_lines_sections: base
          plugin_lines_delay: 30
          
          plugin_languages: yes
          plugin_languages_skipped: dotfiles
          plugin_languages_details: percentage, bytes-size
          
          plugin_followup: yes
          plugin_followup_sections: user
          
          plugin_code: yes
          plugin_code_languages: javascript, typescript, python, cpp
          plugin_code_load: 400

          plugin_wakatime: yes
          plugin_wakatime_sections: time, projects, projects-graphs, languages, languages-graphs, editors, os
          plugin_wakatime_token: ${{ secrets.WAKATIME_TOKEN }}
          
          plugin_rss: yes
          plugin_rss_source: https://blog.qyadbr.top/atom.xml
          plugin_rss_limit: 10

          plugin_gists: yes

          plugin_achievements: yes

          plugin_activity: yes
```