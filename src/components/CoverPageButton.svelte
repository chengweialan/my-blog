<script lang="ts">
import Icon from "@iconify/svelte";
import { onMount } from "svelte";

let isCoverVisible = $state(true); // 默认封面可见

function scrollToCover(e: MouseEvent) {
	e.preventDefault();
	e.stopPropagation();
	// 滚动到页面顶部
	window.scrollTo({ top: 0, behavior: "smooth" });
	// 触发封面页显示
	window.dispatchEvent(
		new CustomEvent("showCover", { detail: { show: true } }),
	);
}

function showPanel() {
	const panel = document.querySelector("#cover-page-panel");
	if (panel) {
		panel.classList.remove("float-panel-closed");
	}
}

function hidePanel() {
	const panel = document.querySelector("#cover-page-panel");
	if (panel) {
		panel.classList.add("float-panel-closed");
	}
}

onMount(() => {
	// 监听封面状态变化
	const handleCoverScroll = (event: CustomEvent<{ progress: number }>) => {
		isCoverVisible = event.detail.progress === 1;
	};

	window.addEventListener("coverScroll", handleCoverScroll as EventListener);

	// 检查初始状态
	const checkInitialState = () => {
		const scrollTop = window.scrollY || document.documentElement.scrollTop;
		// 如果不在顶部，封面应该不可见
		if (scrollTop > 0) {
			isCoverVisible = false;
		} else {
			// 在顶部时，默认封面是可见的
			isCoverVisible = true;
		}
	};

	// 立即检查一次，然后延迟再检查一次确保同步
	checkInitialState();
	setTimeout(checkInitialState, 100);
	setTimeout(checkInitialState, 500);

	return () => {
		window.removeEventListener(
			"coverScroll",
			handleCoverScroll as EventListener,
		);
	};
});
</script>

<!-- 上箭头按钮：在调整颜色按钮的左边，只在非封面页时显示 -->
{#if !isCoverVisible}
	<div class="relative z-50" role="menu" tabindex="-1" onmouseleave={hidePanel}>
		<button
			aria-label="返回封面"
			role="button"
			class="btn-plain scale-animation rounded-lg h-11 w-11 active:scale-90"
			onclick={scrollToCover}
			onmouseenter={showPanel}
		>
			<Icon
				icon="material-symbols:arrow-circle-up-outline-rounded"
				class="text-[1.25rem]"
			></Icon>
		</button>

		<div id="cover-page-panel" class="hidden lg:block absolute transition float-panel-closed top-11 -right-2 pt-5">
			<div class="card-base float-panel p-2">
				<div class="text-sm text-black/75 dark:text-white/100 whitespace-nowrap px-2 py-1">
					返回封面
				</div>
			</div>
		</div>
	</div>
{/if}

