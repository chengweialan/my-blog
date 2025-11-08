<script lang="ts">
import { onDestroy, onMount } from "svelte";

// 直接导入图片 - Vite/Astro 会自动处理为 URL
import lightBgImageSrc from "../assets/images/joy4.jpg";
import darkBgImageSrc from "../assets/images/joy4_2.jpeg";

// 确保是字符串类型
const lightBgImage: string =
	typeof lightBgImageSrc === "string"
		? lightBgImageSrc
		: lightBgImageSrc.src || "";
const darkBgImage: string =
	typeof darkBgImageSrc === "string"
		? darkBgImageSrc
		: darkBgImageSrc.src || "";

let canvas: HTMLCanvasElement;
let ctx: CanvasRenderingContext2D | null = null;
let animationId: number | null = null;
let enabled = $state(false);
let isDark = $state(false);
let backgroundOpacity = $state(0);

// 字符集：字母和数字
const chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
const fontSize = 14;
let columns: number[] = [];
let drops: number[] = [];

function initCanvas() {
	if (!canvas) return;
	ctx = canvas.getContext("2d");
	if (!ctx) return;

	// 设置画布大小
	canvas.width = window.innerWidth;
	canvas.height = window.innerHeight;

	// 计算列数
	const columnCount = Math.floor(canvas.width / fontSize);
	columns = Array(columnCount)
		.fill(0)
		.map(() => Math.random() * -100);
	drops = Array(columnCount)
		.fill(0)
		.map(() => Math.random() * -100);
}

function draw() {
	if (!canvas || !ctx || !enabled) return;

	// 半透明背景，创造拖尾效果
	ctx.fillStyle = isDark ? "rgba(0, 0, 0, 0.05)" : "rgba(255, 255, 255, 0.05)";
	ctx.fillRect(0, 0, canvas.width, canvas.height);

	// 设置文字样式 - 根据主题调整颜色
	const textColor = isDark ? "#0f0" : "#006400"; // 深色模式用亮绿色，浅色模式用深绿色
	ctx.fillStyle = textColor;
	ctx.font = `${fontSize}px monospace`;

	// 绘制每一列
	for (let i = 0; i < columns.length; i++) {
		// 随机选择一个字符
		const text = chars[Math.floor(Math.random() * chars.length)];

		// 绘制字符
		ctx.fillText(text, i * fontSize, drops[i] * fontSize);

		// 如果到达底部或随机重置
		if (drops[i] * fontSize > canvas.height && Math.random() > 0.975) {
			drops[i] = 0;
		}

		// 移动下一个位置
		drops[i]++;
	}

	animationId = requestAnimationFrame(draw);
}

function updateTheme() {
	const newIsDark = document.documentElement.classList.contains("dark");
	if (newIsDark !== isDark) {
		isDark = newIsDark;
	}
}

function animateOpacity(targetOpacity: number) {
	const startOpacity = backgroundOpacity;
	const startTime = performance.now();
	const duration = 500; // 500ms 淡入淡出动画

	function animate(currentTime: number) {
		const elapsed = currentTime - startTime;
		const progress = Math.min(elapsed / duration, 1);

		// 使用 ease-in-out 缓动函数
		const easeProgress =
			progress < 0.5
				? 2 * progress * progress
				: 1 - (-2 * progress + 2) ** 2 / 2;

		backgroundOpacity =
			startOpacity + (targetOpacity - startOpacity) * easeProgress;

		if (progress < 1) {
			requestAnimationFrame(animate);
		} else {
			backgroundOpacity = targetOpacity;
		}
	}

	requestAnimationFrame(animate);
}

function start() {
	if (!enabled || animationId !== null) return;
	if (!ctx) {
		initCanvas();
	}
	if (ctx) {
		draw();
	}
}

function stop() {
	if (animationId !== null) {
		cancelAnimationFrame(animationId);
		animationId = null;
	}
	if (canvas && ctx) {
		ctx.clearRect(0, 0, canvas.width, canvas.height);
	}
}

function handleResize() {
	if (!canvas || !enabled) return;
	initCanvas();
}

// 监听 localStorage 变化
function checkEnabled() {
	const stored = localStorage.getItem("codeRainEnabled");
	// 如果 localStorage 中没有值，默认为 true（开启）
	const newEnabled = stored === null ? true : stored === "true";
	if (newEnabled !== enabled) {
		enabled = newEnabled;
		// 代码雨状态改变时，淡入淡出背景图片
		if (enabled) {
			animateOpacity(0); // 代码雨打开，隐藏背景图片
		} else {
			animateOpacity(0.3); // 代码雨关闭，显示背景图片（半透明）
		}
	}
}

onMount(() => {
	// 初始化主题状态
	updateTheme();

	// 等待 canvas 绑定完成
	setTimeout(() => {
		if (canvas) {
			initCanvas();
		}
	}, 0);

	// 检查初始状态
	checkEnabled();

	// 根据初始状态设置背景图片透明度
	if (enabled) {
		backgroundOpacity = 0;
	} else {
		backgroundOpacity = 0.3;
	}

	window.addEventListener("resize", handleResize);
	window.addEventListener("storage", checkEnabled);

	// 监听自定义事件
	window.addEventListener("codeRainToggle", checkEnabled);

	// 监听主题变化
	const themeObserver = new MutationObserver(() => {
		const wasDark = isDark;
		updateTheme();
		// 如果主题改变且代码雨关闭，触发淡入淡出
		if (wasDark !== isDark && !enabled && backgroundOpacity > 0) {
			// 淡出当前图片，然后淡入新图片
			animateOpacity(0);
			setTimeout(() => {
				animateOpacity(0.3);
			}, 250);
		}
	});

	themeObserver.observe(document.documentElement, {
		attributes: true,
		attributeFilter: ["class"],
	});

	return () => {
		window.removeEventListener("resize", handleResize);
		window.removeEventListener("storage", checkEnabled);
		window.removeEventListener("codeRainToggle", checkEnabled);
		themeObserver.disconnect();
	};
});

onDestroy(() => {
	stop();
});

// 响应式更新
$effect(() => {
	if (enabled) {
		// 确保 canvas 已初始化
		if (!ctx && canvas) {
			initCanvas();
		}
		start();
	} else {
		stop();
	}
});
</script>

<!-- 背景图片 -->
<div
	class="fixed top-0 left-0 w-full h-full pointer-events-none z-0"
	style="opacity: {backgroundOpacity}; transition: opacity 500ms ease-in-out;"
>
	<img
		src={lightBgImage}
		alt="Background"
		class="w-full h-full object-cover transition-opacity duration-500"
		class:hidden={isDark}
		style="opacity: {isDark ? 0 : 1};"
	/>
	<img
		src={darkBgImage}
		alt="Background"
		class="w-full h-full object-cover transition-opacity duration-500"
		class:hidden={!isDark}
		style="opacity: {isDark ? 1 : 0};"
	/>
</div>

<!-- 代码雨画布 -->
<canvas
	bind:this={canvas}
	class="fixed top-0 left-0 w-full h-full pointer-events-none z-0"
	class:hidden={!enabled}
></canvas>

<style>
	canvas {
		opacity: 0.4;
	}
</style>
