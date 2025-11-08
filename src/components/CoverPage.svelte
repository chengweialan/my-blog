<script lang="ts">
import { onMount } from "svelte";
import { profileConfig, siteConfig } from "../config";

let isCoverVisible = $state(true); // 封面是否可见（分立状态），默认显示封面
let coverScroll = $state(0); // 累积滚动距离，用于触发切换

const SWITCH_THRESHOLD = 150; // 切换阈值：需要累积150px的滚动距离才能触发切换

function handleWheel(event: WheelEvent) {
	const scrollTop = window.scrollY || document.documentElement.scrollTop;

	if (isCoverVisible) {
		// 在封面状态时，需要累积向下滚动距离才能切换到内容页
		if (event.deltaY > 0) {
			event.preventDefault(); // 阻止默认滚动行为
			// 累积向下滚动距离
			coverScroll = Math.min(coverScroll + event.deltaY, SWITCH_THRESHOLD * 2);
			// 如果累积距离超过阈值，切换到内容页
			if (coverScroll >= SWITCH_THRESHOLD) {
				resetCover();
				// 滚动到内容页顶部
				window.scrollTo({ top: 0, behavior: "smooth" });
			}
		} else if (event.deltaY < 0) {
			// 如果向上滚动，重置累积距离
			coverScroll = Math.max(coverScroll + event.deltaY, 0);
		}
	} else {
		// 在内容页状态时，必须同时满足两个条件才能切换到封面页：
		// 1. 滑动开始时右侧滑动条在当前页面顶部（scrollTop === 0）
		// 2. 滑动力度够大（累积距离 >= SWITCH_THRESHOLD）

		// 严格检查：如果不在顶部，立即重置累积距离，不进行任何处理
		if (scrollTop !== 0) {
			coverScroll = 0;
			return;
		}

		// 只有在顶部时才开始累积
		if (event.deltaY < 0) {
			// 向上滚动：累积向上滚动距离（deltaY 是负数，所以用绝对值）
			event.preventDefault(); // 阻止默认滚动行为
			coverScroll = coverScroll + Math.abs(event.deltaY);
			// 如果累积距离超过阈值，切换到封面页
			if (coverScroll >= SWITCH_THRESHOLD) {
				isCoverVisible = true;
				coverScroll = 0; // 重置累积距离
				updateCoverState();
			}
		} else if (event.deltaY > 0) {
			// 如果向下滚动，重置累积距离
			coverScroll = 0;
		}
	}
}

function updateCoverState() {
	// 根据封面状态更新样式和触发事件
	const progress = isCoverVisible ? 1 : 0;
	window.dispatchEvent(
		new CustomEvent("coverScroll", { detail: { progress } }),
	);
}

function resetCover() {
	coverScroll = 0;
	isCoverVisible = false;
	updateCoverState();
}

function handleScroll() {
	const scrollTop = window.scrollY || document.documentElement.scrollTop;

	// 如果页面滚动离开顶部，重置封面状态和累积距离
	if (scrollTop > 0) {
		// 如果不在顶部，重置封面状态和累积距离
		if (isCoverVisible) {
			resetCover();
		} else {
			// 不在封面状态时，重置累积距离
			coverScroll = 0;
		}
	} else if (scrollTop === 0 && !isCoverVisible) {
		// 如果回到页面顶部且不在封面状态，重置累积距离
		// 这样可以确保从顶部重新开始累积
		coverScroll = 0;
	}
}

function scrollToContent() {
	// 重置封面并滚动到内容区域
	resetCover();
	// 稍微滚动一下，确保离开顶部
	window.scrollTo({ top: 1, behavior: "smooth" });
}

onMount(() => {
	// 初始化封面状态
	updateCoverState();

	window.addEventListener("wheel", handleWheel, { passive: false });
	window.addEventListener("scroll", handleScroll, { passive: true });

	// 监听链接点击事件（排除封面页的点击）
	const handleLinkClick = (e: MouseEvent) => {
		// 如果点击的是封面页，不处理链接点击
		const target = e.target as HTMLElement;
		const coverPage = target.closest(
			'[role="button"][aria-label="点击跳转到内容"]',
		);
		if (coverPage) {
			return;
		}

		const link = target.closest("a");
		if (link?.href) {
			// 如果是内部链接，重置封面
			const url = new URL(link.href, window.location.href);
			if (url.origin === window.location.origin) {
				resetCover();
			}
		}
	};
	document.addEventListener("click", handleLinkClick);

	// 监听 swup 页面切换事件
	const setupSwup = () => {
		const swup = window.swup as
			| { hooks?: { on: (event: string, callback: () => void) => void } }
			| undefined;
		if (swup?.hooks) {
			swup.hooks.on("visit:start", resetCover);
		}
	};

	const swup = window.swup as
		| { hooks?: { on: (event: string, callback: () => void) => void } }
		| undefined;
	if (swup?.hooks) {
		setupSwup();
	} else {
		window.addEventListener("swup:enable", setupSwup);
	}

	return () => {
		window.removeEventListener("wheel", handleWheel);
		window.removeEventListener("scroll", handleScroll);
		document.removeEventListener("click", handleLinkClick);
	};
});
</script>

<div
	class="fixed inset-0 z-20 flex flex-col items-center justify-center"
	class:opacity-0={!isCoverVisible}
	class:opacity-100={isCoverVisible}
	class:pointer-events-none={!isCoverVisible}
	class:pointer-events-auto={isCoverVisible}
	class:cursor-pointer={isCoverVisible}
	style="transition: opacity 0.3s ease-out;"
	onclick={isCoverVisible ? scrollToContent : undefined}
	role={isCoverVisible ? "button" : undefined}
	aria-label={isCoverVisible ? "点击跳转到内容" : undefined}
	tabindex={isCoverVisible ? 0 : undefined}
>
	<div class="text-center relative">
		<!-- 电磁光圈效果 -->
		<div class="absolute inset-0 flex items-center justify-center">
			<div class="electromagnetic-ring"></div>
		</div>
		
		<h1
			class="text-6xl md:text-8xl font-bold mb-4 text-[var(--primary)] relative z-10"
			style="font-family: 'STXingkai', 'Xingkai SC', 'KaiTi', '楷体', serif; text-shadow: 2px 2px 8px rgba(0,0,0,0.5), 0 0 20px rgba(0,0,0,0.3); letter-spacing: 0.05em;"
		>
			{profileConfig.name}
		</h1>
		<p
			class="text-2xl md:text-3xl text-black/80 dark:text-white/80 italic relative z-10"
			style="font-family: 'STXingkai', 'Xingkai SC', 'KaiTi', '楷体', serif; text-shadow: 1px 1px 4px rgba(0,0,0,0.3);"
		>
			{siteConfig.subtitle}
		</p>
	</div>
</div>

<style>
	@keyframes electromagnetic-pulse {
		0% {
			box-shadow: 0 0 0 0 rgba(59, 130, 246, 0.7),
				0 0 0 0 rgba(59, 130, 246, 0.5),
				0 0 0 0 rgba(59, 130, 246, 0.3);
		}
		50% {
			box-shadow: 0 0 0 20px rgba(59, 130, 246, 0),
				0 0 0 40px rgba(59, 130, 246, 0),
				0 0 0 60px rgba(59, 130, 246, 0);
		}
		100% {
			box-shadow: 0 0 0 0 rgba(59, 130, 246, 0),
				0 0 0 0 rgba(59, 130, 246, 0),
				0 0 0 0 rgba(59, 130, 246, 0);
		}
	}

	@keyframes electromagnetic-rotate {
		0% {
			transform: rotate(0deg);
		}
		100% {
			transform: rotate(360deg);
		}
	}

	.electromagnetic-ring {
		width: 300px;
		height: 300px;
		border: 2px solid rgba(59, 130, 246, 0.6);
		border-radius: 50%;
		position: relative;
		animation: electromagnetic-pulse 2s ease-in-out infinite,
			electromagnetic-rotate 20s linear infinite;
	}

	.electromagnetic-ring::before {
		content: "";
		position: absolute;
		top: -2px;
		left: -2px;
		right: -2px;
		bottom: -2px;
		border: 2px solid rgba(59, 130, 246, 0.4);
		border-radius: 50%;
		animation: electromagnetic-pulse 2s ease-in-out infinite 0.3s,
			electromagnetic-rotate 15s linear infinite reverse;
	}

	.electromagnetic-ring::after {
		content: "";
		position: absolute;
		top: -4px;
		left: -4px;
		right: -4px;
		bottom: -4px;
		border: 1px solid rgba(59, 130, 246, 0.3);
		border-radius: 50%;
		animation: electromagnetic-pulse 2s ease-in-out infinite 0.6s,
			electromagnetic-rotate 25s linear infinite;
	}

	:global(.dark) .electromagnetic-ring {
		border-color: rgba(96, 165, 250, 0.6);
	}

	:global(.dark) .electromagnetic-ring::before {
		border-color: rgba(96, 165, 250, 0.4);
	}

	:global(.dark) .electromagnetic-ring::after {
		border-color: rgba(96, 165, 250, 0.3);
	}

	:global(.dark) .electromagnetic-ring {
		animation: electromagnetic-pulse-dark 2s ease-in-out infinite,
			electromagnetic-rotate 20s linear infinite;
	}

	:global(.dark) .electromagnetic-ring::before {
		animation: electromagnetic-pulse-dark 2s ease-in-out infinite 0.3s,
			electromagnetic-rotate 15s linear infinite reverse;
	}

	:global(.dark) .electromagnetic-ring::after {
		animation: electromagnetic-pulse-dark 2s ease-in-out infinite 0.6s,
			electromagnetic-rotate 25s linear infinite;
	}

	@keyframes electromagnetic-pulse-dark {
		0% {
			box-shadow: 0 0 0 0 rgba(96, 165, 250, 0.7),
				0 0 0 0 rgba(96, 165, 250, 0.5),
				0 0 0 0 rgba(96, 165, 250, 0.3);
		}
		50% {
			box-shadow: 0 0 0 20px rgba(96, 165, 250, 0),
				0 0 0 40px rgba(96, 165, 250, 0),
				0 0 0 60px rgba(96, 165, 250, 0);
		}
		100% {
			box-shadow: 0 0 0 0 rgba(96, 165, 250, 0),
				0 0 0 0 rgba(96, 165, 250, 0),
				0 0 0 0 rgba(96, 165, 250, 0);
		}
	}
</style>

