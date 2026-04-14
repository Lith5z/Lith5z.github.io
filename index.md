---
layout: default
title: 主页
description: 这里是我的极简个人介绍和笔记分享
nav_order: 1
---

# 你好，我是锂盐/Lith5z 👋

> To see a world in a grain of sand.

我打算在这里放各种笔记，从基础的编程语法、正在学习的算法到读书笔记、工具教程等等。 <br>
---

<h2>🚀 最近更新</h2>
<ul>
  {% assign sorted_pages = site.pages | sort: 'date' | reverse %}
  {% for page in sorted_pages limit: 5 %}
    {% if page.date %}
      <li>
        <a href="{{ page.url | relative_url }}">{{ page.title }}</a>
        <small style="color: #666;">({{ page.date | date: "%Y-%m-%d" }})</small>
      </li>
    {% endif %}
  {% endfor %}
</ul>

## 笔记库

~~其实就是为了这点醋包的饺子~~，按主题跳转：

### 编程 Coding
- [一个简单的个人c语言项目](https://github.com/Lith5z/cpl-practice)
- [未完成 滑动窗口](notes/Code/sliding_window.html)
- [从C到C++ 语法学习](notes/Code/C2CPP.html)

### 和智能科学 AI
- [3B1B 线性代数](notes/AI/linear_algebra/main.html)
- [伯克利CS188 人工智能导论](notes/AI/algorithms_intro/CS188_main.html)
- [3B1B 深度学习](notes/AI/deep_learning/3b1b_main.html)

### 读书笔记和随笔 Others
- [存在主义是一种人道主义](notes/Others/existentialism.html)
- [月亮与六便士](notes/Others/moon_and_sixpence.html)

### 使用方法记录 Instructions

事实上，这里不是教程，只是我学的时候记录的笔记。 <br>
比较像教程实则是工具推荐的可以看[飞书文档 百宝箱推荐](https://rcnx330isacs.feishu.cn/drive/folder/PbBUfzbnBlQcGQd3fKncd4MMnOf?from=from_copylink)

- [Latex数学公式](notes/Instructions/latex.html)
- [PS使用记录](notes/Instructions/ps_use.html)



---

## 技能和兴趣

那我最好是真会了：

- C/C++ 正在学习算法
- Git/Latex 正在熟悉
- Ps/Pr 会一点点平面设计
- [其他..](notes/about_me.html)



---

## 联系方式
Source: [这个网页的仓库](https://github.com/Lith5z/Lith5z.github.io)



南京大学 智能科学与技术 大一在读中
- Email: yingxuanhuang@smail.nju.edu.cn
- GitHub: [@Lith5z](https://github.com/Lith5z)
- Bilibili: [@锂盐Lith](https://space.bilibili.com/643955126)


<figure>
  <img src="/assets/images/index_pic.webp" alt="kano" />
</figure>