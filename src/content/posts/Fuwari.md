---
category: 关于Fuwari框架博客魔改博客
description: 感谢afoim开源的部分代码跟各位大佬教程
published: 2025-09-01
tags:
- 博客
title: 博客
---

## 主题自适应Giscus评论
:::在src\pages\posts[…slug].astro开头引入:::
```
import Giscus from "../../components/misc/Giscus.astro";
```

:::创建写入修改参数:::
```ts title="src\components\misc/Giscus.astro"
---
  repo = "123456" // 在此输入用户名/仓库名
  repoId = "123456" // 在此输入仓库 ID
  category = "123456" // 在此输入分类名
  categoryId = "123456" //在此输入分类 ID
  mapping = 'pathname',
  reactionsEnabled = true,
  emitMetadata = false,
  inputPosition = 'bottom',
  lang = 'zh-CN'
---

<div id="giscus-container"></div>

<script define:vars={{ repo, repoId, category, categoryId, mapping, reactionsEnabled, emitMetadata, inputPosition, lang }}>
  function loadGiscus() {
    const container = document.getElementById('giscus-container');
    if (!container) return;

    // 获取当前主题
    const isDark = document.documentElement.classList.contains('dark');
    const theme = isDark ? 'dark' : 'light';

    // 创建Giscus脚本
    const script = document.createElement('script');
    script.src = 'https://giscus.app/client.js';
    script.setAttribute('data-repo', repo);
    script.setAttribute('data-repo-id', repoId);
    script.setAttribute('data-category', category);
    script.setAttribute('data-category-id', categoryId);
    script.setAttribute('data-mapping', mapping);
    script.setAttribute('data-strict', '0');
    script.setAttribute('data-reactions-enabled', reactionsEnabled ? '1' : '0');
    script.setAttribute('data-emit-metadata', emitMetadata ? '1' : '0');
    script.setAttribute('data-input-position', inputPosition);
    script.setAttribute('data-theme', theme);
    script.setAttribute('data-lang', lang);
    script.setAttribute('data-loading', 'lazy');
    script.crossOrigin = 'anonymous';
    script.async = true;

    container.appendChild(script);
  }

  // 监听主题变化
  function updateGiscusTheme() {
    const giscusFrame = document.querySelector('iframe[src*="giscus"]');
    if (giscusFrame) {
      const isDark = document.documentElement.classList.contains('dark');
      const theme = isDark ? 'dark' : 'light';

      giscusFrame.contentWindow.postMessage({
        giscus: {
          setConfig: {
            theme: theme
          }
        }
      }, 'https://giscus.app');
    }
  }

  // 监听DOM变化来检测主题切换
  const observer = new MutationObserver((mutations) => {
    mutations.forEach((mutation) => {
      if (mutation.type === 'attributes' && mutation.attributeName === 'class') {
        updateGiscusTheme();
      }
    });
  });

  // 页面加载时初始化
  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', loadGiscus);
  } else {
    loadGiscus();
  }

  // 开始观察主题变化
  observer.observe(document.documentElement, {
    attributes: true,
    attributeFilter: ['class']
  });
</script>
```

# 背景图及透明卡片
>在SiteConfig类型中添加background配置选项
>新增透明背景颜色变量—card-bg-transparent和—float-panel-bg-transparent
>实现背景图片加载检测及卡片透明效果切换
>添加背景图片样式配置,包括位置、大小、重复方式等
```ts title="src\pages\posts[…slug].astro"
			offset = offset - offset % 4;
			document.documentElement.style.setProperty('--banner-height-extend', `${offset}px`);
{/* =========== */}
      const bgUrl = getComputedStyle(document.documentElement).getPropertyValue('--bg-url').trim();
      const bgEnable = getComputedStyle(document.documentElement).getPropertyValue('--bg-enable').trim();
      if (bgUrl && bgUrl !== 'none' && bgEnable === '1') {
        const img = new Image();
        const urlMatch = bgUrl.match(/url\(["']?([^"')]+)["']?\)/);
        if (urlMatch) {
          img.onload = function() {
            document.body.classList.add('bg-loaded');
            document.documentElement.style.setProperty('--card-bg', 'var(--card-bg-transparent)');
            document.documentElement.style.setProperty('--float-panel-bg', 'var(--float-panel-bg-transparent)');
          };
          img.onerror = function() {
            console.warn('Background image failed to load, keeping cards opaque');
          };
          img.src = urlMatch[1];
        }
      }
{/* =========== */}
 </script>
		<style define:vars={{
			configHue,
			'page-width': `${PAGE_WIDTH}rem`,
{/* =========== */}
			'bg-url': siteConfig.background.src ? `url(${siteConfig.background.src})` : 'none',
			'bg-enable': siteConfig.background.enable ? '1' : '0',
			'bg-position': siteConfig.background.position || 'center',
			'bg-size': siteConfig.background.size || 'cover',
			'bg-repeat': siteConfig.background.repeat || 'no-repeat',
			'bg-attachment': siteConfig.background.attachment || 'fixed',
			'bg-opacity': (siteConfig.background.opacity || 0.3).toString()
		}}>
			:root {
				--bg-url: var(--bg-url);
				--bg-enable: var(--bg-enable);
				--bg-position: var(--bg-position);
				--bg-size: var(--bg-size);
				--bg-repeat: var(--bg-repeat);
				--bg-attachment: var(--bg-attachment);
				--bg-opacity: var(--bg-opacity);
			}
			
			/* Background image configuration */
			body {
				--bg-url: var(--bg-url);
				--bg-enable: var(--bg-enable);
				--bg-position: var(--bg-position);
				--bg-size: var(--bg-size);
				--bg-repeat: var(--bg-repeat);
				--bg-attachment: var(--bg-attachment);
				--bg-opacity: var(--bg-opacity);
			}
			
			body::before {
				content: '' !important;
				position: fixed !important;
				top: 0 !important;
				left: 0 !important;
				width: 100% !important;
				height: 100% !important;
				background-image: var(--bg-url) !important;
				background-position: var(--bg-position) !important;
				background-size: var(--bg-size) !important;
				background-repeat: var(--bg-repeat) !important;
				background-attachment: var(--bg-attachment) !important;
				opacity: 0 !important;
				pointer-events: none !important;
				z-index: -1 !important;
				display: block !important;
				transition: opacity 0.3s ease-in-out !important;
			}
			
			body.bg-loaded::before {
				opacity: calc(var(--bg-opacity) * var(--bg-enable)) !important;
			}
{/* =========== */}
			</style>  <!-- defines global css variables. This will be applied to <html> <body> and some other elements idk why -->
```

```ts title="src/types/config.ts"
 background: {
   enable: boolean;
   src: string;
   position?: "top" | "center" | "bottom";
   size?: "cover" | "contain" | "auto";
   repeat?: "no-repeat" | "repeat" | "repeat-x" | "repeat-y";
   attachment?: "fixed" | "scroll" | "local";
   opacity?: number;
};
```

```ts title="src/styles/variables.styl"
--card-bg-transparent: hsl(var(--hue) 10% 10% / 0.6);
--float-panel-bg-transparent: hsl(var(--hue) 10% 10% / 0.6);
```

```ts title="src/config.ts"
    background: {
    enable: true, // Enable background image
    src: "https://pic.2x.nz/?img=h", // Background image URL (supports HTTPS)
    position: "center", // Background position: 'top', 'center', 'bottom'
    size: "cover", // Background size: 'cover', 'contain', 'auto'
    repeat: "no-repeat", // Background repeat: 'no-repeat', 'repeat', 'repeat-x', 'repeat-y'
    attachment: "fixed", // Background attachment: 'fixed', 'scroll', 'local'
    opacity: 0.5, // Background opacity (0-1)
  },
```

## 加文章置顶
```ts title="src/utils/content-utils.ts"
{/* =========== */}
const sorted = allBlogPosts.sort((a, b) => {
		if (a.data.pinned !== b.data.pinned) {
			return a.data.pinned ? -1 : 1;
		}
{/* =========== */}
		const dateA = new Date(a.data.published);
		const dateB = new Date(b.data.published);
		return dateA > dateB ? -1 : 1;
	});
	return sorted;
}
```

```text title="src/components/PostCard.astro"
#38行左右
const isPinned = entry.data.pinned === true;
```

```astro title="src/components/PostCard.astro"
before:absolute before:top-[35px] before:left-[18px] before:hidden md:before:block
        ">
{/* =========== */}
    {isPinned && (
                <span class="inline-flex items-center mr-2 px-2 py-0.5 text-sm font-medium bg-[oklch(95%_0.2_var(--hue))] dark:bg-[oklch(25%_0.2_var(--hue))] text-[oklch(55%_0.2_var(--hue))] dark:text-[oklch(85%_0.2_var(--hue))] rounded">
                    <Icon name="material-symbols:push-pin" class="mr-1 text-base" /> 置顶
                </span>
            )}
{/* =========== */}
    {title}
```

```astro title="src/pages/posts/[...slug].astro"
<!-- title -->
<div class="relative onload-animation">
    <div
        data-pagefind-body data-pagefind-weight="10" data-pagefind-meta="title"
        class="transition w-full block font-bold mb-3
        text-3xl md:text-[2.25rem]/[2.75rem]
        text-black/90 dark:text-white/90
        md:before:w-1 before:h-5 before:rounded-md before:bg-[var(--primary)]
        before:absolute before:top-[0.75rem] before:left-[-1.125rem]
    ">
{/* =========== */}
        {entry.data.pinned && (
        <span class="inline-flex items-center mr-3 px-2.5 py-1 text-sm font-medium bg-[oklch(95%_0.2_var(--hue))] dark:bg-[oklch(25%_0.2_var(--hue))] text-[oklch(55%_0.2_var(--hue))] dark:text-[oklch(85%_0.2_var(--hue))] rounded">
            <Icon name="material-symbols:push-pin" class="mr-1.5 text-base" /> 置顶
        </span>
    )}
{/* =========== */}
        <span>{entry.data.title}</span>
    </div>
</div>
```

```text title="src/content/config.ts"
pinned: z.boolean().optional().default(false),
```