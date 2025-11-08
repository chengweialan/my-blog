<script lang="ts">
import Icon from "@iconify/svelte";
import { onMount } from "svelte";

let enabled = $state(false); // 默认关闭

function toggle() {
	enabled = !enabled;
	localStorage.setItem("codeRainEnabled", enabled.toString());

	// 触发自定义事件，通知其他组件
	window.dispatchEvent(new CustomEvent("codeRainToggle"));
}

function showPanel() {
	const panel = document.querySelector("#code-rain-panel");
	if (panel) {
		panel.classList.remove("float-panel-closed");
	}
}

function hidePanel() {
	const panel = document.querySelector("#code-rain-panel");
	if (panel) {
		panel.classList.add("float-panel-closed");
	}
}

onMount(() => {
	const stored = localStorage.getItem("codeRainEnabled");
	// 如果 localStorage 中没有值，默认为 false（关闭）
	enabled = stored === null ? false : stored === "true";
	// 确保 localStorage 中有值
	if (stored === null) {
		localStorage.setItem("codeRainEnabled", "false");
	}
});
</script>

<!-- z-50 make the panel higher than other float panels -->
<div class="relative z-50" role="menu" tabindex="-1" onmouseleave={hidePanel}>
	<button
		aria-label="代码雨开关"
		role="button"
		class="relative btn-plain scale-animation rounded-lg h-11 w-11 active:scale-90"
		id="code-rain-switch"
		onclick={toggle}
		onmouseenter={showPanel}
	>
		<div class="absolute" class:opacity-0={enabled}>
			<Icon
				icon="line-md:switch-off"
				class="text-[1.25rem]"
			></Icon>
		</div>
		<div class="absolute" class:opacity-0={!enabled}>
			<Icon
				icon="line-md:switch-filled"
				class="text-[1.25rem]"
			></Icon>
		</div>
	</button>

	<div id="code-rain-panel" class="hidden lg:block absolute transition float-panel-closed top-11 -right-2 pt-5">
		<div class="card-base float-panel p-2">
			<div class="text-sm text-black/75 dark:text-white/100 whitespace-nowrap px-2 py-1">
				代码雨
			</div>
		</div>
	</div>
</div>
