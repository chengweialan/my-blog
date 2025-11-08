<script lang="ts">
import { onMount } from "svelte";

let contentOpacity = $state(1);
let contentTranslateY = $state(0);

function handleCoverScroll(event: CustomEvent<{ progress: number }>) {
	const progress = event.detail.progress;

	// 根据封面滚动进度，内容向下滑出
	contentOpacity = 1 - progress; // 从1到0
	contentTranslateY = progress * 150; // 从0到150px向下移动

	// 更新所有页面内容的样式
	const mainGrid = document.getElementById("main-grid");
	const contentWrapper = document.getElementById("content-wrapper");
	const swupContainer = document.getElementById("swup-container");

	if (mainGrid) {
		mainGrid.style.opacity = String(contentOpacity);
		mainGrid.style.transform = `translateY(${contentTranslateY}px)`;
		mainGrid.style.transition =
			"opacity 0.3s ease-out, transform 0.3s ease-out";
	}
	if (contentWrapper) {
		contentWrapper.style.opacity = String(contentOpacity);
		contentWrapper.style.transform = `translateY(${contentTranslateY}px)`;
		contentWrapper.style.transition =
			"opacity 0.3s ease-out, transform 0.3s ease-out";
	}
	if (swupContainer) {
		swupContainer.style.opacity = String(contentOpacity);
		swupContainer.style.transform = `translateY(${contentTranslateY}px)`;
		swupContainer.style.transition =
			"opacity 0.3s ease-out, transform 0.3s ease-out";
	}
}

function handleScroll() {
	const scrollTop = window.scrollY || document.documentElement.scrollTop;

	// 如果页面滚动离开顶部，重置内容状态
	if (scrollTop > 0) {
		contentOpacity = 1;
		contentTranslateY = 0;

		const mainGrid = document.getElementById("main-grid");
		const contentWrapper = document.getElementById("content-wrapper");
		const swupContainer = document.getElementById("swup-container");

		if (mainGrid) {
			mainGrid.style.opacity = "1";
			mainGrid.style.transform = "translateY(0px)";
		}
		if (contentWrapper) {
			contentWrapper.style.opacity = "1";
			contentWrapper.style.transform = "translateY(0px)";
		}
		if (swupContainer) {
			swupContainer.style.opacity = "1";
			swupContainer.style.transform = "translateY(0px)";
		}
	}
}

onMount(() => {
	window.addEventListener("coverScroll", handleCoverScroll as EventListener);
	window.addEventListener("scroll", handleScroll, { passive: true });
	return () => {
		window.removeEventListener(
			"coverScroll",
			handleCoverScroll as EventListener,
		);
		window.removeEventListener("scroll", handleScroll);
	};
});
</script>

