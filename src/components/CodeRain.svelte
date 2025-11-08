<script lang="ts">
import { onDestroy, onMount } from "svelte";

let canvas: HTMLCanvasElement;
let ctx: CanvasRenderingContext2D | null = null;
let animationId: number | null = null;
let enabled = $state(false);

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
	const isDark = document.documentElement.classList.contains("dark");
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
	}
}

onMount(() => {
	// 等待 canvas 绑定完成
	setTimeout(() => {
		if (canvas) {
			initCanvas();
		}
	}, 0);

	// 检查初始状态
	checkEnabled();

	window.addEventListener("resize", handleResize);
	window.addEventListener("storage", checkEnabled);

	// 监听自定义事件
	window.addEventListener("codeRainToggle", checkEnabled);

	return () => {
		window.removeEventListener("resize", handleResize);
		window.removeEventListener("storage", checkEnabled);
		window.removeEventListener("codeRainToggle", checkEnabled);
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

<canvas
	bind:this={canvas}
	class="fixed top-0 left-0 w-full h-full pointer-events-none z-0"
></canvas>

<style>
	canvas {
		opacity: 0.4;
	}
</style>
