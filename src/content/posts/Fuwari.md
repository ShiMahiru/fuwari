---
category: 博客
description: 感谢afoim开源的部分代码跟各位大佬教程
published: 2025-09-01
tags:
- 博客
title: 关于我的Fuwari博客修改
---
# 功能实现
- 1.自适应Giscus评论区
- 1.背景图及透明卡片
- 2.文章置顶
- 3.文章帮助反馈
- 4.右上角友链+赞助+统计
> 若修改找不到建议搜索关键词，ps:修改高亮部分
> 
> 关于路径文件在哪里？ps:复制最左边
> 
> 至于什么时候更新这篇文章，ps:跟随本站同步更新

---
## 自适应Giscus评论区
>Giscus是利用 GitHub Discussions 实现的评论系统

1.前往[GitHub](https://github.com/new)创建一个公开仓库，否则访客将无法查看

在仓库设置中`启用 Discussion功能`

- 前往[Giscus](https://giscus.app/zh-CN)进行配置
- 填写用户名/仓库名
- 特性:✅将评论框放在评论上方 ✅懒加载评论
- 其他保持默认即可

配置好之后，下方会自动生成对应JS代码，复制下来保存，接下来我们要修改自适应㳀色/深色，将刚刚复制的JS加上 `data-theme`这行就行了

2.在src/pages/posts目录下创建Giscus.svelte文件
```diff
<section>
    <script src="https://giscus.app/client.js"
        data-repo="[在此输入仓库]"
        data-repo-id="[在此输入仓库 ID]"
        data-category="[在此输入分类名]"
        data-category-id="[在此输入分类 ID]"
        data-mapping="pathname"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
+       data-theme={$mode === DARK_MODE ? 'dark' : 'light'}
        data-theme="preferred_color_scheme"
        data-lang="zh-CN"
        crossorigin="anonymous"
        async>
    </script>
</section>

<script>
import { AUTO_MODE, DARK_MODE } from '@constants/constants.ts'
import { onMount } from 'svelte'
import { writable } from 'svelte/store';
import { getStoredTheme } from '@utils/setting-utils.ts'
const mode = writable(AUTO_MODE)
onMount(() => {
  mode.set(getStoredTheme())
})

function updateGiscusTheme() {
  const theme = document.documentElement.classList.contains('dark') ? 'dark' : 'light'
  const iframe = document.querySelector('iframe.giscus-frame')
  if (!iframe) return
  iframe.contentWindow.postMessage({ giscus: { setConfig: { theme } } }, 'https://giscus.app')
}

const observer = new MutationObserver(updateGiscusTheme)
observer.observe(document.documentElement, { attributes: true, attributeFilter: ['class'] })

window.onload = () => {
  updateGiscusTheme()
}
</script>
```
3.在 `src\pages\posts\[...slug].astro`中引入 Giscus 组件，在前5行插入既可
```diff
+ import Giscus from "../../pages/Giscus.svelte"
```
在<!-- 版权信息 -->下引入组件就完成了，不知道在哪里可以搜索关键词
```diff
    {licenseConfig.enable && <License title={entry.data.title} slug={entry.slug} pubDate={entry.data.published} class="mb-6 rounded-xl license-container onload-animation"></License>}

+   <Giscus client:only="svelte"></Giscus>

```
---
## 2.背景图及透明卡片
- 检测背景图片加载状态，成功加载后启用透明效果
### 修改点 : `/src/layouts/Layout.astro`
```diff
	document.documentElement.style.setProperty('--banner-height-extend', `${offset}px`);

+    const bgUrl = getComputedStyle(document.documentElement).getPropertyValue('--bg-url').trim();
+    const bgEnable = getComputedStyle(document.documentElement).getPropertyValue('--bg-enable').trim();
+    if (bgUrl && bgUrl !== 'none' && bgEnable === '1') {
+    const img = new Image();
+    const urlMatch = bgUrl.match(/url\(["']?([^"')]+)["']?\)/);
+    if (urlMatch) {
+      img.onload = function() {
+        document.body.classList.add('bg-loaded');
+        document.documentElement.style.setProperty('--card-bg', 'var(--card-bg-transparent)');
+        document.documentElement.style.setProperty('--float-panel-bg', 'var(--float-panel-bg-transparent)');
+      };
+      img.onerror = function() {
+        console.warn('Background image failed to load, keeping cards opaque');
+      };
+      img.src = urlMatch[1];
+    }
+    }

</script>
<style define:vars={{
	configHue,
	'page-width': `${PAGE_WIDTH}rem`,

+	'bg-url': siteConfig.background.src ? `url(${siteConfig.background.src})` : 'none',
+	'bg-enable': siteConfig.background.enable ? '1' : '0',
+	'bg-position': siteConfig.background.position || 'center',
+	'bg-size': siteConfig.background.size || 'cover',
+	'bg-repeat': siteConfig.background.repeat || 'no-repeat',
+	'bg-attachment': siteConfig.background.attachment || 'fixed',
+	'bg-opacity': (siteConfig.background.opacity || 0.3).toString()
+ }}>
+	:root {
+		--bg-url: var(--bg-url);
+		--bg-enable: var(--bg-enable);
+		--bg-position: var(--bg-position);
+		--bg-size: var(--bg-size);
+		--bg-repeat: var(--bg-repeat);
+		--bg-attachment: var(--bg-attachment);
+		--bg-opacity: var(--bg-opacity);
+	}
	
+	/* Background image configuration */
+	body {
+		--bg-url: var(--bg-url);
+		--bg-enable: var(--bg-enable);
+		--bg-position: var(--bg-position);
+		--bg-size: var(--bg-size);
+		--bg-repeat: var(--bg-repeat);
+		--bg-attachment: var(--bg-attachment);
+		--bg-opacity: var(--bg-opacity);
+	}
+	
+	body::before {
+		content: '' !important;
+		position: fixed !important;
+		top: 0 !important;
+		left: 0 !important;
+		width: 100% !important;
+		height: 100% !important;
+		background-image: var(--bg-url) !important;
+		background-position: var(--bg-position) !important;
+		background-size: var(--bg-size) !important;
+		background-repeat: var(--bg-repeat) !important;
+		background-attachment: var(--bg-attachment) !important;
+		opacity: 0 !important;
+		pointer-events: none !important;
+		z-index: -1 !important;
+		display: block !important;
+		transition: opacity 0.3s ease-in-out !important;
+	}
+	
+	body.bg-loaded::before {
+		opacity: calc(var(--bg-opacity) * var(--bg-enable)) !important;
+	}
+
</style>  <!-- defines global css variables. This will be applied to <html> <body> and some other elements idk why -->
```
### 修改点 : `/src/types/config.ts`
```diff
+ background: {
+   enable: boolean;++   src: string;
+   position?: "top" | "center" | "bottom";
+   size?: "cover" | "contain" | "auto";
+   repeat?: "no-repeat" | "repeat" | "repeat-x" | "repeat-y";
+   attachment?: "fixed" | "scroll" | "local";
+   opacity?: number;
+ };
```

### 修改点 : `/src/styles/variables.styl`
```diff
  --card-bg-transparent: hsl(var(--hue) 10% 10% / 0.6);
  --float-panel-bg-transparent: hsl(var(--hue) 10% 10% / 0.6);
```
### 修改点 : `/src/config.ts`
```diff
+    background: {
+    enable: true, // Enable background image
+    src: "https://pic.2x.nz/?img=h", // Background image URL (supports HTTPS)
+    position: "center", // Background position: 'top', 'center', 'bottom'
+    size: "cover", // Background size: 'cover', 'contain', 'auto'
+    repeat: "no-repeat", // Background repeat: 'no-repeat', 'repeat', 'repeat-x', 'repeat-y'
+    attachment: "fixed", // Background attachment: 'fixed', 'scroll', 'local'
+    opacity: 0.5, // Background opacity (0-1)
+  },
```
---
## 3.加文章置顶
### 修改点 : `/src/utils/content-utils.ts`
```diff
const sorted = allBlogPosts.sort((a, b) => {

+		// 如果一个是置顶一个不是置顶，置顶的排在前面
+		if (a.data.pinned !== b.data.pinned) {
+			return a.data.pinned ? -1 : 1;
+		}
+		// 都是置顶或都不是置顶，按发布日期时间排序（包含小时分钟秒）
		const dateA = new Date(a.data.published);
		const dateB = new Date(b.data.published);
		return dateA > dateB ? -1 : 1;
	});
	return sorted;
}
```
### 修改点 : `/src/components/PostCard.astro`
```diff
  const isPinned = entry.data.pinned === true;
```
### 修改点 : `/src/components/PostCard.astro`
```diff
    before:absolute before:top-[35px] before:left-[18px] before:hidden md:before:block
    ">
    
+    {isPinned && (
+    <span class="inline-flex items-center mr-2 px-2 py-0.5 text-sm font-medium bg-[oklch(95%_0.2_var(--hue))] dark:bg-[oklch(25%_0.2_var(--hue))] text-[oklch(55%_0.2_var(--hue))] dark:text-[oklch(85%_0.2_var(--hue))] rounded">
+        <Icon name="material-symbols:push-pin" class="mr-1 text-base" /> 置顶
+    </span>
+    )}
```
### 修改点 : `/src/pages/posts/[...slug].astro`
```diff
        before:absolute before:top-[0.75rem] before:left-[-1.125rem]
    ">

+            {entry.data.pinned && (
+            <span class="inline-flex items-center mr-3 px-2.5 py-1 text-sm font-medium bg-[oklch(95%_0.2_var(--hue))] dark:bg-[oklch(25%_0.2_var(--hue))] text-[oklch(55%_0.2_var(--hue))] dark:text-[oklch(85%_0.2_var(--hue))] rounded">
+                <Icon name="material-symbols:push-pin" class="mr-1.5 text-base" /> 置顶
+            </span>
+        )}
```
### 修改点 : `/src/content/config.ts`
```diff
pinned: z.boolean().optional().default(false),
```
---
## 4.文章帮助反馈
### 修改点 : `/src/pages/posts/[...slug].astro`
```diff
        {licenseConfig.enable && <License title={entry.data.title} slug={entry.slug} pubDate={entry.data.published} class="mb-6 rounded-xl license-container onload-animation"></License>}

+        <!-- 文章帮助反馈区域 -->
+        <div class="mb-4 p-3 rounded-lg bg-[var(--license-block-bg)] border border-[var(--line-divider)] onload-animation">
+            <div class="flex items-center justify-between">
+                <div class="flex items-center gap-2">
+                    <div class="h-4 w-4 rounded bg-[var(--primary)] flex items-center justify-center">
+                        <Icon name="material-symbols:help-outline" class="text-[0.75rem] text-white dark:text-black/70"></Icon>
+                    </div>
+                   <p class="text-black/80 dark:text-white/80 text-sm">这篇文章是否对你有帮助？</p>
+                </div>
+                <div class="flex gap-2">
+                    <a href="路径" class="group flex items-center gap-1 px-3 py-1.5 rounded bg-[var(--btn-regular-bg)] hover:bg-[var(--btn-regular-bg-hover)] active:bg-[var(--btn-regular-bg-active)] text-[var(--btn-content)] text-xs font-medium transition-all active:scale-95">
+                        <Icon name="material-symbols:contact-mail-outline" class="text-[0.875rem] group-hover:scale-110 transition-transform"></Icon>
                        联系
+                    </a>
+                    <a href="路径" class="group flex items-center gap-1 px-3 py-1.5 rounded bg-[var(--primary)] hover:bg-[oklch(0.65_0.16_var(--hue))] active:bg-[oklch(0.60_0.18_var(--hue))] text-white dark:text-black/80 text-xs font-medium transition-all active:scale-95">
+                        <Icon name="material-symbols:favorite-outline" class="text-[0.875rem] group-hover:scale-110 transition-transform"></Icon>
+                        赞助
+                 </a>
+          </div>
+    </div>
+   </div>
```
---
## 5.右上角友链+赞助+统计
### 修改点 : `/src\config.ts`
```diff
export const navBarConfig: NavBarConfig = {
	links: [
		LinkPreset.Home,
		LinkPreset.Archive,
		LinkPreset.About,
+		{
+			name: "友链",
+			url: "/friends/",
+			external: false,
+		},
+		{
+			name: "赞助",
+			url: "/donate/",
+			external: false,
+		},
+		{
+			name: "统计",
+			url: "https://us.umami.is/share/xxxxxxxxx",
+			external: true,
+		},
+		{
+			name: "GitHub",
+			url: "https://github.com/saicaca/fuwari",
+			external: true,
+		},
	],
};
```
## 创建赞助文件写入修改参数
赞助命名alipay.svg跟wechat.svg二维码svg格式放在/donate/会改的也可以其他地方随便
```ts title="/pages/friends.astro"
---
import MainGridLayout from "@layouts/MainGridLayout.astro";
import { Icon } from "astro-icon/components";
---

<MainGridLayout title="赞助支持">
    <div class="card-base p-6 md:p-8">
        <div class="flex items-center gap-2 mb-6">
            <div class="h-8 w-8 rounded-lg bg-[var(--primary)] flex items-center justify-center text-white dark:text-black/70">
                <Icon name="material-symbols:favorite" class="text-[1.5rem]">
            </div>
            <h1 class="text-2xl font-bold text-black dark:text-white">赞助支持</h1>
        </div>

        <div class="mb-6">
            <p class="text-lg text-black/80 dark:text-white/80 mb-3">
                如果您觉得我的内容对您有帮助，欢迎通过以下方式支持我的创作。您的每一份支持都是我持续创作的动力！
            </p>
            <p class="text-sm text-black/60 dark:text-white/60">
                所有赞助将用于网站维护、服务器费用以及内容创作。
            </p>
        </div>

        <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-6 max-w-4xl mx-auto">
            <!-- 支付宝 -->
            <div class="donate-card">
                <div class="flex items-center gap-3 mb-4">
                    <div class="h-10 w-10 rounded-lg bg-blue-500 flex items-center justify-center">
                        <Icon name="fa6-brands:alipay" class="text-[1.5rem] text-white">
                    </div>
                    <div>
                        <h3 class="text-lg font-bold text-black dark:text-white">支付宝</h3>
                        <p class="text-sm text-black/60 dark:text-white/60">扫码支付</p>
                    </div>
                </div>
                <div class="qr-code-placeholder">
                    <img src="/donate/alipay.svg" alt="支付宝二维码" class="w-48 h-48 mx-auto rounded-lg" />
                </div>
            </div>

            <!-- 微信支付 -->
            <div class="donate-card">
                <div class="flex items-center gap-3 mb-4">
                    <div class="h-10 w-10 rounded-lg bg-green-500 flex items-center justify-center">
                        <Icon name="fa6-brands:weixin" class="text-[1.5rem] text-white">
                    </div>
                    <div>
                        <h3 class="text-lg font-bold text-black dark:text-white">微信支付</h3>
                        <p class="text-sm text-black/60 dark:text-white/60">扫码支付</p>
                    </div>
                </div>
                <div class="qr-code-placeholder">
                    <img src="/donate/wechat.svg" alt="微信支付二维码" class="w-48 h-48 mx-auto rounded-lg" />
                </div>
            </div>
        </div>

        <br>

        <!-- 其他支持方式 -->
        <div class="card-base p-6 mb-6">
            <h2 class="text-xl font-bold text-black dark:text-white mb-3">其他支持方式</h2>
            <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                <div class="support-card">
                    <div class="flex items-center gap-2 mb-2">
                        <Icon name="material-symbols:share" class="text-[1.2rem] text-[var(--primary)]">
                        <span class="font-semibold text-black dark:text-white">分享推荐</span>
                    </div>
                    <p class="text-sm text-black/60 dark:text-white/60">
                        将我的博客分享给更多朋友
                    </p>
                </div>

                <div class="support-card">
                    <div class="flex items-center gap-2 mb-2">
                        <Icon name="material-symbols:comment" class="text-[1.2rem] text-[var(--primary)]">
                        <span class="font-semibold text-black dark:text-white">留言互动</span>
                    </div>
                    <p class="text-sm text-black/60 dark:text-white/60">
                        在文章下方留下您的想法
                    </p>
                </div>

                <div class="support-card">
                    <div class="flex items-center gap-2 mb-2">
                        <Icon name="material-symbols:star" class="text-[1.2rem] text-[var(--primary)]">
                        <span class="font-semibold text-black dark:text-white">关注订阅</span>
                    </div>
                    <p class="text-sm text-black/60 dark:text-white/60">
                        订阅RSS或关注社交媒体
                    </p>
                </div>
            </div>
        </div>
        <div class="sponsors-section card-base p-6">
            <h2 class="text-xl font-bold text-black dark:text-white mb-3 flex items-center gap-2">
                <Icon name="material-symbols:group" class="text-[1.5rem] text-[var(--primary)]"></Icon>
                <span>已赞助的小伙伴</span>
            </h2>
            <p class="text-sm text-black/60 dark:text-white/60 mb-4">
            如果您已赞助，并且想加入赞助名单，请自行提交 <a target="_blank" href="mailto:242531778@qq.com" class="transition link text-[var(--primary)] font-medium underline">点击这里提交</a>
            </p>
            <div class="sponsors-grid">

                <div class="sponsor-card">
                    <div class="flex items-center gap-3 p-4 rounded-lg bg-[var(--card-bg)] border border-black/10 dark:border-white/10">
                        <div class="w-10 h-10 rounded-full bg-gray-300 dark:bg-gray-600 flex items-center justify-center overflow-hidden">
                        	<img src="https://q2.qlogo.cn/headimg_dl?dst_uin=242531778&spec=0" alt="" class="w-full h-full object-cover">
                        </div>
                        <div class="flex-1">
                            <h4 class="font-semibold text-black dark:text-white text-sm">ShiMahiru</h4>
                            <p class="text-xs text-black/60 dark:text-white/60">2025-09-1</p>
                        </div>
                        <div class="text-xs text-[var(--primary)] font-medium">100 ￥</div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</MainGridLayout>

<style>
.donate-card {
    @apply flex flex-col p-4 rounded-lg bg-[var(--card-bg)] border border-black/10 dark:border-white/10 hover:border-black/20 dark:hover:border-white/20 transition;
}

.support-card {
    @apply p-4 rounded-lg bg-[var(--card-bg)] hover:bg-black/5 dark:hover:bg-white/5 transition;
}

.qr-code-placeholder {
    @apply text-center;
}

.sponsors-section {
    @apply mt-6;
}

.sponsors-grid {
    @apply grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4;
}

.sponsor-placeholder {
    @apply transition-all duration-200 hover:scale-105;
}

.sponsor-card {
    @apply transition-all duration-200 hover:shadow-md hover:scale-105;
}
.crypto-info {
    @apply space-y-3;
}

.wallet-address, .bybit-uid {
    @apply text-center;
}

.copy-btn {
    @apply transition-colors duration-200;
}

.copy-btn:hover {
    color: var(--primary);
    opacity: 0.8;
}
</style>

<script>
function copyToClipboard(text) {
    navigator.clipboard.writeText(text).then(function() {
        // 可以添加复制成功的提示
        alert('已复制到剪贴板');
    }, function(err) {
        console.error('复制失败: ', err);
    });
}
</script>
```
# 创建友链文件写入修改参数
```ts title="src/pages/donate.astro"
---
import MainGridLayout from "@layouts/MainGridLayout.astro";
import { Icon } from "astro-icon/components";
---

<MainGridLayout title="友链">
    <div class="card-base p-6 md:p-8">
        <div class="flex items-center gap-2 mb-6">
            <div class="h-8 w-8 rounded-lg bg-[var(--primary)] flex items-center justify-center text-white dark:text-black/70">
                <Icon name="material-symbols:diversity-3" class="text-[1.5rem]">
            </div>
            <h1 class="text-2xl font-bold text-black dark:text-white">友链</h1>
        </div>
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
            <a href="https://vercel.com/" target="_blank" class="friend-card">
                <div class="flex items-center gap-2">
                    <img src="https://assets.vercel.com/image/upload/front/favicon/vercel/apple-touch-icon-256x256.png" loading="lazy" class="w-5 h-5 rounded">
                    <div class="font-bold text-black dark:text-white">Vercel</div>
                </div>
                <div class="text-sm text-black/50 dark:text-white/50">该网站由 Vercel 部署并加速。</div>
            </a>
        </div>

        <!-- 申请友链 -->
        <div class="sponsors-section mt-8">
        	<h2 class="text-xl font-bold text-black dark:text-white mb-4 flex items-center gap-2">
        		<Icon name="material-symbols:group" class="text-[1.5rem] text-[var(--primary)]"/>
        		将您的网站加入本站友链
              </h2>
        	<p class="text-sm text-black/60 dark:text-white/60">
        		请自行提交 <a target="_blank" href="mailto:242531778@qq.com" class="transition link text-[var(--primary)] font-medium underline">点击这里提交</a>
        	</p>
        <br>
    </div>

</MainGridLayout>

<style>
.friend-card {
    @apply flex flex-col gap-1 p-4 rounded-lg bg-[var(--card-bg)] hover:bg-black/5 dark:hover:bg-white/5 transition;
}
</style>
```

# 引入umami
### 修改点 : `/src/layout/Layout.astro`
```diff
+ <script defer src="https://us.umami.is//script.js" data-website-id="xxxxxxxxxxxxxxxx"></script>
```
这个去 [umami](https://us.umami.is/) 注册设置添加既可