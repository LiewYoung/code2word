<template>
  <div class="preview-container">
    <!-- 代码高亮显示区域 -->
    <div class="title">
      <div class="font-size-controls">
        <button @click="decreaseFontSize">-</button>
        <span>{{ fontSize }}px</span>
        <button @click="increaseFontSize">+</button>
      </div>
      高亮输出区
      <div class="detected-language">语言: {{ codeStore.detectedLanguage }}</div>
    </div>
    <pre><code ref="codeBlock">{{ code }}</code></pre>
  </div>
</template>

<script setup>
import {useCodeStore} from '@/global-data-store/codeStore'
import {ref, watch, onMounted} from 'vue';
import {hljs} from '@/assets/highlight/highlight.min.js';

const codeStore = useCodeStore()

const code = ref('');
const codeBlock = ref(null);
const fontSize = ref(12); // 初始字体大小

const olId="renderedHighlightCode";

// 定义处理高亮代码并添加 ol 列表的方法
const renderHighlightCodeAndCreateOl = () => {
  if(codeStore.emitCode) {
    // 使用 hljs 进行高亮处理
    let result;
    if (codeStore.selectedLanguage === 'auto') {
      result = hljs.highlightAuto(codeStore.emitCode); // 自动检测
    } else {
      result = hljs.highlight(codeStore.emitCode, {language: codeStore.selectedLanguage}); // 指定语言
    }
    const highlightedCode = result.value;
    codeStore.detectedLanguage = (codeStore.selectedLanguage === 'auto' ? result.language : codeStore.selectedLanguage) || "未识别";


    // 判断是否存在旧的渲染代码。
    let codeElement = document.getElementById(olId);
    if (codeElement) {
      codeElement.remove();
    }

    const ol = document.createElement('ol');
    ol.id = 'renderedHighlightCode';
    if(codeStore.outside) {
      ol.style="list-style: decimal;font-family: Consolas, 'Courier New', monospace;margin: 10px 10px 10px 4em !important;";
    }else {
      ol.style="list-style: decimal;font-family: Consolas, 'Courier New', monospace;margin: 10px 0 !important;";
    }
    ol.style.fontSize = fontSize.value + "px";
    // 将每一行代码分割成数组
    const lines = highlightedCode.split('\n');
    // 遍历每一行，创建 li 标签并添加到 ol 中
    let index = 1;
    lines.forEach(line => {
      const li = document.createElement('li');
      if(codeStore.outside) {
        li.style = "font-family: Consolas, 'Courier New', monospace;white-space: pre-wrap;word-wrap: break-word;border-right: 4px solid #87CEEB;border-left: 4px solid #87CEEB;list-style: decimal-leading-zero;list-style-position: outside !important;padding: 0 4px 0 4px !important;" + (index++ % 2 == 0 ? "background-color: #F6F9FB;" : "background-color: #FDFDFD;");
      }else {
        li.style = "font-family: Consolas, 'Courier New', monospace;white-space: pre-wrap;word-wrap: break-word;border-right: 4px solid #87CEEB;border-left: 4px solid #87CEEB;list-style: decimal-leading-zero;list-style-position: inside !important;padding: 0 4px 0 4px !important;" + (index++ % 2 == 0 ? "background-color: #F6F9FB;" : "background-color: #FDFDFD;");
      }
      li.innerHTML = line;  // 设置高亮代码为 li 的内容
      ol.appendChild(li);    // 将 li 添加到 ol 中
    });

    // 清空原有的 codeBlock 内容并插入新的 ol
    codeBlock.value.innerHTML = ''; // 清空原本的内容
    codeBlock.value.parentNode.appendChild(ol);

    updateHighlightCode();
  }else {
    codeStore.detectedLanguage = "未识别";


    // 判断是否存在旧的渲染代码。
    let codeElement = document.getElementById(olId);
    if (codeElement) {
      codeElement.remove();
    }
  }
};

const updateHighlightCode = () => {
  codeStore.highlightedCode = document.getElementById(olId).outerHTML;
}

onMounted(() => {

});

// 监听 highlightedCode 的变化，触发 applyHighlight 方法
// 监听 highlightedCode 的变化，触发 applyHighlight 方法
watch(() => [codeStore.emitCode, codeStore.selectedLanguage], () => {
  renderHighlightCodeAndCreateOl()
})
// 字体大小调整方法
const increaseFontSize = () => {
  if (fontSize.value < 40) {
    fontSize.value += 2; // 最大为30px
    document.getElementById(olId).style.fontSize = (fontSize.value + "px");
    updateHighlightCode();
  }
};

const decreaseFontSize = () => {
  if (fontSize.value > 12) {
    fontSize.value -= 2; // 最小为10px
    document.getElementById(olId).style.fontSize = (fontSize.value + "px")
    updateHighlightCode();
  }
};
</script>

<style>
@import '../assets/highlight/hightlight-theme-github.min.css';
</style>

<style scoped>
.preview-container {
  flex: 1;
  padding: 16px;
  background-color: var(--md-sys-color-background);
  color: var(--md-sys-color-on-background);
  display: flex;
  flex-direction: column;
}

.title {
  font: var(--md-sys-typescale-title-medium);
  color: var(--md-sys-color-on-surface-variant);
  margin-bottom: 8px;
  padding-left: 4px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 32px; /* Fixed height for alignment */
  position: relative;
}

pre {
  flex: 1;
  width: 100%;
  color: var(--md-sys-color-on-surface);
  font-family: 'Roboto Mono', monospace;
  background-color: var(--md-sys-color-surface);
  border-radius: 8px;
  border: 1px solid var(--md-sys-color-outline-variant); 
  /* No heavy border, just outline variant */
  font-size: 12px;
  overflow: auto;
  padding: 0; /* Let ol handle padding */
  box-shadow: var(--md-sys-elevation-1);
}

.detected-language {
  font: var(--md-sys-typescale-label-medium);
  color: var(--md-sys-color-on-surface-variant);
}

.font-size-controls {
  display: flex;
  align-items: center;
  gap: 8px;
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
}

.font-size-controls span {
  font: var(--md-sys-typescale-label-medium);
  min-width: 32px;
  text-align: center;
}

.font-size-controls button {
  background-color: var(--md-sys-color-secondary-container);
  color: var(--md-sys-color-on-secondary-container);
  border: none;
  width: 24px;
  height: 24px;
  border-radius: 12px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
  line-height: 1;
  transition: background-color 0.2s;
}

.font-size-controls button:hover {
  background-color: var(--md-sys-color-secondary);
  color: var(--md-sys-color-on-secondary);
}
</style>


<!--

/*
以下ol、li相关的样式，都转换成内联样式。方便代码复制内容时，直接复制到内联的代码。
即使不使用内联样式写法，使用浏览器手动复制时，也会自动转换为内联样式。之所以这么做只是为了方便代码复制。
*/
<style>
ol {
  list-style: decimal;
  font-family: Consolas, Monaco, 'Andale Mono', 'Ubuntu Mono', monospace;
  margin: 10px 10px 10px 4em !important;
}

ol li {
  font-family: monospace;
  white-space: pre-wrap; /* 让文本换行 */
  word-wrap: break-word; /* 自动换行，防止溢出 */
  border-right: 4px solid #87CEEB;
  border-left: 4px solid #87CEEB;
  list-style: decimal-leading-zero;
  list-style-position: outside !important;
  padding: 0 4px 0 4px !important;
}


/* 设置偶数行样式 */
li:nth-child(even) {
  background-color: #F6F9FB;
}

/* 设置奇数行样式 */
li:nth-child(odd) {
  background-color: #FDFDFD;
}

</style>
-->
