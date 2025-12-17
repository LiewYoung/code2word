<template>
  <div class="control-panel">
    <button @click="highlightCode">高亮</button>
    <select v-model="codeStore.selectedLanguage" class="language-select">
      <option value="auto">自动识别</option>
      <option v-for="lang in languageList" :key="lang" :value="lang">
        {{ lang }}
      </option>
    </select>
    <button @click="copyCode">复制</button>
    <button @click="jsObfuscator">JS混淆</button>
    <button @click="toggleOutside">{{ codeStore.outside ? '外部序号' : '内部序号' }}</button>
    <button @click="copyDesc">复制说明</button>
  </div>
</template>

<script setup>
import {useCodeStore} from '@/global-data-store/codeStore'
import {Utils} from "@/utils/utils";
import {obfuscate} from 'javascript-obfuscator';
import {hljs} from '@/assets/highlight/highlight.min.js';
import githubThemeCss from '@/assets/highlight/hightlight-theme-github.min.css?raw'; // 导入样式文件内容

const languageList = hljs.listLanguages();

const codeStore = useCodeStore()

function jsObfuscator() {
  let rawCode = codeStore.rawCode;
  if (rawCode) {
    codeStore.rawCode = obfuscate(
        rawCode,
        {
          compact: true,                      // 紧凑模式，移除多余空格和换行符
          controlFlowFlattening: true,       // 启用控制流平坦化（极大增强混淆程度）
          controlFlowFlatteningThreshold: 1, // 100% 启用控制流平坦化
          numbersToExpressions: true,        // 将数字转换为复杂表达式
          simplify: false,                    // 关闭代码简化优化
          stringArrayShuffle: true,          // 乱序字符串数组，提高混淆效果
          splitStrings: true,                 // 拆分字符串
          stringArrayThreshold: 1            // 100% 的字符串都进行混淆
        }
    ).getObfuscatedCode();
  } else {
    alert("请先输入内容");
  }
}

const highlightCode = () => {
  if (!codeStore.rawCode) {
    alert("请先输入内容");
  } else {
    codeStore.emitCode = codeStore.rawCode;
  }
};
const toggleOutside = () => {
  codeStore.outside = !codeStore.outside;
  codeStore.emitCode = '';
  alert("已切换为" + (codeStore.outside ? "外部序号" : "内部序号") + ", 点击高亮按钮，即可重新渲染");
}
const copyDesc = async () => {
  Utils.usage();
};
const copyCode = async () => {
  try {
    let html = codeStore.highlightedCode;
    if (!html) {
      alert("请先高亮代码");
      return;
    }
    
    // 移除所有 border-left 和 border-right 内联样式，防止 Word 中出现蓝色方框
    html = html.replace(/border-left:\s*[^;]+;/gi, '');
    html = html.replace(/border-right:\s*[^;]+;/gi, '');
    
    // GitHub 主题的颜色映射 - 将 CSS 类转换为内联样式
    const colorMap = {
      'hljs-keyword': 'color:#d73a49',
      'hljs-type': 'color:#d73a49',
      'hljs-doctag': 'color:#d73a49',
      'hljs-meta': 'color:#005cc5',
      'hljs-template-tag': 'color:#d73a49',
      'hljs-template-variable': 'color:#d73a49',
      'hljs-variable': 'color:#005cc5',
      'hljs-title': 'color:#6f42c1',
      'hljs-function': 'color:#6f42c1',
      'hljs-attr': 'color:#005cc5',
      'hljs-attribute': 'color:#005cc5',
      'hljs-literal': 'color:#005cc5',
      'hljs-number': 'color:#005cc5',
      'hljs-operator': 'color:#005cc5',
      'hljs-selector-attr': 'color:#005cc5',
      'hljs-selector-class': 'color:#005cc5',
      'hljs-selector-id': 'color:#005cc5',
      'hljs-string': 'color:#032f62',
      'hljs-regexp': 'color:#032f62',
      'hljs-built_in': 'color:#e36209',
      'hljs-symbol': 'color:#e36209',
      'hljs-comment': 'color:#6a737d',
      'hljs-code': 'color:#6a737d',
      'hljs-formula': 'color:#6a737d',
      'hljs-name': 'color:#22863a',
      'hljs-quote': 'color:#22863a',
      'hljs-selector-tag': 'color:#22863a',
      'hljs-selector-pseudo': 'color:#22863a',
      'hljs-subst': 'color:#24292e',
      'hljs-section': 'color:#005cc5;font-weight:700',
      'hljs-bullet': 'color:#735c0f',
      'hljs-emphasis': 'color:#24292e;font-style:italic',
      'hljs-strong': 'color:#24292e;font-weight:700',
      'hljs-addition': 'color:#22863a;background-color:#f0fff4',
      'hljs-deletion': 'color:#b31d28;background-color:#ffeef0',
    };
    
    // 将 class="hljs-xxx" 转换为 style="color:xxx"
    for (const [className, style] of Object.entries(colorMap)) {
      const regex = new RegExp(`class="${className}"`, 'gi');
      html = html.replace(regex, `style="${style}"`);
    }
    
    // 处理多个类名的情况，如 class="hljs-title function_"
    html = html.replace(/class="hljs-title[^"]*"/gi, 'style="color:#6f42c1"');
    html = html.replace(/class="hljs-variable[^"]*"/gi, 'style="color:#005cc5"');
    html = html.replace(/class="hljs-meta[^"]*"/gi, 'style="color:#005cc5"');
    
    // 将每行开头的空格转换为 &nbsp;，Word 才能正确显示缩进
    // 只转换 <li> 标签内容开头的连续空格
    html = html.replace(/<li([^>]*)>(\s*)/g, (match, attrs, spaces) => {
      if (spaces) {
        // 只转换空格，保留换行等其他空白字符
        const nbspSpaces = spaces.replace(/ /g, '&nbsp;').replace(/\t/g, '&nbsp;&nbsp;&nbsp;&nbsp;');
        return `<li${attrs}>${nbspSpaces}`;
      }
      return match;
    });
    
    // 包装 HTML，确保字体大小
    let wrappedHtml = `
      <div style="font-family: Consolas, 'Courier New', monospace; font-size: 14px;">
        ${html}
      </div>
    `;

    await navigator.clipboard.write([
      new ClipboardItem({
        "text/html": new Blob([wrappedHtml], {type: "text/html"}),
        "text/plain": new Blob([html.replace(/<[^>]+>/g, '')], {type: "text/plain"}),
      }),
    ]);
    alert("复制成功！可直接粘贴到 Word 中。");
  } catch (err) {
    alert("复制失败！请手动复制！" + err);
  }
};

</script>

<style scoped>
.control-panel {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 24px 16px;
  background-color: var(--md-sys-color-surface-variant);
  color: var(--md-sys-color-on-surface-variant);
  width: 200px; /* Slightly wider */
  gap: 12px;
  box-shadow: var(--md-sys-elevation-1);
}

button {
  width: 100%;
  height: 40px;
  border-radius: 20px;
  border: none;
  background-color: var(--md-sys-color-primary);
  color: var(--md-sys-color-on-primary);
  font: var(--md-sys-typescale-label-large);
  cursor: pointer;
  transition: box-shadow 0.2s, background-color 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
}

button:hover {
  background-color: var(--md-sys-color-primary-container);
  color: var(--md-sys-color-on-primary-container);
  box-shadow: var(--md-sys-elevation-1);
}

/* Secondary/Tonal buttons for less important actions if needed, 
   but for now uniform look is clean. Let's make 'Highlight' distinct? 
   Actually, 'Highlight' is the primary action. Others could be secondary. 
   Let's keep them uniform for simplicity or use partial styling.
*/

.language-select {
  width: 100%;
  height: 56px; /* MD3 text field height */
  border-radius: 4px 4px 0 0;
  border: none;
  border-bottom: 2px solid var(--md-sys-color-outline);
  background-color: var(--md-sys-color-surface);
  color: var(--md-sys-color-on-surface);
  font: var(--md-sys-typescale-body-large);
  padding: 0 16px;
  margin: 5px 0;
  cursor: pointer;
  appearance: none; /* Remove default arrow */
  background-image: url("data:image/svg+xml;charset=UTF-8,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='%23E6E1E5'%3e%3cpath d='M7 10l5 5 5-5z'/%3e%3c/svg%3e");
  background-repeat: no-repeat;
  background-position: right 10px center;
  background-size: 24px;
}

.language-select:focus {
  border-bottom-color: var(--md-sys-color-primary);
  outline: none;
  background-color: var(--md-sys-color-surface-variant);
}

/* Specific button styles if we want hierarchy */
button:nth-child(n+3) { /* Actions other than Highlight (and Select) */
  background-color: var(--md-sys-color-secondary-container);
  color: var(--md-sys-color-on-secondary-container);
}

button:nth-child(n+3):hover {
  background-color: var(--md-sys-color-secondary);
  color: var(--md-sys-color-on-secondary);
}
</style>
