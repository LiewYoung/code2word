<template>
  <nav class="navbar">
    <div class="logo-container">
      <img :src="logo" alt="Logo" class="logo"/>
      <h1>Code2Word - 让高亮的代码完美呈现至 Word</h1>
    </div>
    <div class="nav-links">
      <a :href="Utils.home" class="link">主页</a>
      <a :href="Utils.blog" class="link">博客</a>
      <a :href="Utils.github" target="_blank" class="link">GitHub</a>
    </div>
  </nav>
</template>

<script setup>
import logo from '@/assets/logo.svg'; // 导入 SVG 文件
import {Utils} from "@/utils/utils";
</script>

<style scoped>
.navbar {
  height: 64px; /* MD3 Small Top App Bar height */
  background-color: var(--md-sys-color-surface);
  color: var(--md-sys-color-on-surface);
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 16px; /* MD3 horizontal padding */
  position: fixed;
  top: 0;
  width: 100vw;
  z-index: 100;
  box-shadow: var(--md-sys-elevation-2); /* MD3 Elevation */
}

.logo-container {
  display: flex;
  align-items: center;
}

.logo {
  width: 40px;
  height: 40px;
  margin-right: 12px;
}

h1 {
  font: var(--md-sys-typescale-title-large);
  margin: 0;
}

.nav-links {
  display: flex;
  gap: 24px;
}

.link {
  color: var(--md-sys-color-on-surface-variant);
  text-decoration: none;
  font: var(--md-sys-typescale-label-large);
  transition: color 0.2s ease-in-out;
  position: relative;
  padding: 8px 12px;
  border-radius: 20px;
}

.link:hover {
  color: var(--md-sys-color-on-surface);
  background-color: rgba(255, 255, 255, 0.08); /* Hover state */
}
</style>
