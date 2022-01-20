---
title: "CSS中内联SVG"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"其他"
]
tags : [
"Other"
]
series : [
"Manual"
]
images : [

]
---

CSS中内联SVG

内联SVG比内联图像的base64效果更好，图片更加保真：

```css
.icon-arrow-down {
    width: 20px; height: 20px;
    background: url('data:image/svg+xml;utf8,<svg version="1.1" xmlns="http://www.w3.org/2000/svg" width="200" height="200" viewBox="0 0 200 200"><path fill="#00A5E0" d="M145.659,68.949c-5.101-5.208-13.372-5.208-18.473,0L99.479,97.233 L71.772,68.949c-5.1-5.208-13.371-5.208-18.473,0c-5.099,5.208-5.099,13.648,0,18.857l46.18,47.14l46.181-47.14 C150.759,82.598,150.759,74.157,145.659,68.949z"/></svg>') no-repeat center;
    background-size: 100%;
}
```

 

