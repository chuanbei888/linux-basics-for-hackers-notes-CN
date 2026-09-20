---
layout: default
title: 黑客 Linux 基础
description: 面向安全学习者的中文 Linux 基础与实验笔记
permalink: /
---

<section class="hero">
  <p class="eyebrow">Linux · Security · Practice</p>
  <h1>黑客 Linux 基础</h1>
  <p>一套面向安全学习者的中文 Linux 学习笔记。从终端和文件系统开始，逐步掌握网络、权限、脚本、服务与内核基础。</p>
  <div class="hero-actions">
    <a class="button" href="{{ '/Module_00_Getting_Started.html' | relative_url }}">从模块 0 开始</a>
    <a class="button secondary" href="https://github.com/{{ site.repository }}">查看 GitHub 仓库</a>
  </div>
</section>

<section aria-labelledby="modules-title">
  <p class="section-label" id="modules-title">17 个模块 · 循序学习</p>
  <div class="module-grid">
    {% for module in site.data.modules %}
      <a class="module-card" href="{{ module.url | relative_url }}">
        <span class="module-card-number">{{ module.number }}</span>
        <span>
          <h2>{{ module.title }}</h2>
          <p>{{ module.description }}</p>
        </span>
      </a>
    {% endfor %}
  </div>
</section>

<section class="reading-note" aria-labelledby="note-title">
  <h2 id="note-title">学习说明</h2>
  <p>内容基于 OccupyTheWeb 的 <em>Linux Basics for Hackers</em> 整理。所有网络、安全和无线实验都应在自己的虚拟机或明确授权的环境中完成。</p>
</section>
