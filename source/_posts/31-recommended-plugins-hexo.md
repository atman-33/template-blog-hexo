---
title: 【Hexo】おすすめプラグイン
date: 2023-03-05 13:31:20
tag: 
- Hexo
categories: Hexo
---

Hexoのおすすめプラグインをまとめておきます。

### ファイルの変更をブラウザに即時反映

```
npm install -g browser-sync

npm install hexo-browsersync --save
```

### GitHub page にデプロイ

```
npm install hexo-deployer-git --save
```

### サイトマップ作成

```
npm install hexo-generator-seo-friendly-sitemap --save
```

_config.ymlに追記

▼Hexoディレクトリ\\_config.yml（設定例）
```
sitemap:
  path: sitemap.xml
  tag: true
  category: true
```

### robots.txt生成

```
pm install hexo-generator-robotstxt --save
```

▼Hexoディレクトリ\\_config.yml（設定例）
```
robotstxt:
  useragent: "*"
  disallow:
    - /about/
    - /gallery/
    - /privacy-policy/
    - /archives/
    - /css/
    - /img/
    - /js/
  allow:
  sitemap: /sitemap.xml
```
