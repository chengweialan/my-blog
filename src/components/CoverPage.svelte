<script lang="ts">
import Icon from "@iconify/svelte";
import { onMount } from "svelte";
import { profileConfig, siteConfig } from "../config";

let isCoverVisible = $state(false); // 封面是否可见（分立状态）
let coverScroll = $state(0); // 累积滚动距离，用于触发切换

const SWITCH_THRESHOLD = 150; // 切换阈值：需要累积150px的滚动距离才能触发切换

function handleWheel(event: WheelEvent) {
	const scrollTop = window.scrollY || document.documentElement.scrollTop;

	// 严格检查：只有在页面顶部时才能触发封面切换
	if (scrollTop !== 0 && !isCoverVisible) {
		// 如果不在顶部且不在封面状态，重置累积距离，不进行任何处理
		coverScroll = 0;
		return;
	}

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
	} else if (scrollTop === 0) {
		// 在内容页顶部时，需要累积向上滚动距离才能切换到封面页
		// 严格检查：只有在顶部时才开始累积
		if (event.deltaY < 0) {
			event.preventDefault(); // 阻止默认滚动行为
			// 累积向上滚动距离（deltaY 是负数，所以用绝对值）
			coverScroll = Math.min(
				coverScroll + Math.abs(event.deltaY),
				SWITCH_THRESHOLD * 2,
			);
			// 如果累积距离超过阈值，切换到封面页
			if (coverScroll >= SWITCH_THRESHOLD) {
				isCoverVisible = true;
				coverScroll = 0; // 重置累积距离
				updateCoverState();
			}
		} else if (event.deltaY > 0) {
			// 如果向下滚动，重置累积距离
			coverScroll = Math.max(coverScroll - event.deltaY, 0);
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
	window.addEventListener("wheel", handleWheel, { passive: false });
	window.addEventListener("scroll", handleScroll, { passive: true });

	// 监听链接点击事件
	const handleLinkClick = (e: MouseEvent) => {
		const target = e.target as HTMLElement;
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
	class="fixed inset-0 z-20 flex flex-col items-center justify-center pointer-events-none"
	class:opacity-0={!isCoverVisible}
	class:opacity-100={isCoverVisible}
	style="transition: opacity 0.3s ease-out;"
>
	<div class="text-center">
		<h1
			class="text-6xl md:text-8xl font-bold mb-4 text-[var(--primary)]"
			style="font-family: 'Georgia', 'Times New Roman', serif; text-shadow: 2px 2px 8px rgba(0,0,0,0.5), 0 0 20px rgba(0,0,0,0.3); letter-spacing: 0.05em;"
		>
			{profileConfig.name}
		</h1>
		<p
			class="text-2xl md:text-3xl text-black/80 dark:text-white/80 italic"
			style="font-family: 'Georgia', 'Times New Roman', serif; text-shadow: 1px 1px 4px rgba(0,0,0,0.3);"
		>
			{siteConfig.subtitle}
		</p>
	</div>
	
	<!-- 下箭头按钮 -->
	<button
		class="absolute bottom-8 pointer-events-auto cursor-pointer transition-opacity duration-300 hover:opacity-70 active:scale-95"
		class:opacity-0={!isCoverVisible}
		class:opacity-100={isCoverVisible}
		onclick={scrollToContent}
		aria-label="跳转到内容"
		style="transition: opacity 0.3s ease-out, transform 0.2s ease-out;"
	>
		<Icon
			icon="material-symbols:keyboard-arrow-down"
			class="text-5xl text-[var(--primary)] drop-shadow-lg"
		></Icon>
	</button>
</div>

