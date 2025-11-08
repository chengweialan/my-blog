<script lang="ts">
import { onDestroy, onMount } from "svelte";

let canvas: HTMLCanvasElement;
let ctx: CanvasRenderingContext2D | null = null;
let animationId: number | null = null;
let isDark = $state(false);
let codeRainEnabled = $state(false); // 代码雨是否启用

// 粒子类
class Particle {
	x: number;
	y: number;
	vx: number; // 水平速度
	vy: number; // 垂直速度
	size: number;
	rotation: number;
	rotationSpeed: number;
	opacity: number;
	type: "cherry" | "leaf"; // 樱花或落叶
	color: string; // 粒子颜色（用于落叶）
	variant: "normal" | "bent" | "torn" | "serrated" | "withStem"; // 形状变体

	constructor(type: "cherry" | "leaf", width: number, height: number) {
		this.type = type;
		this.x = Math.random() * width;
		this.y = Math.random() * height - height; // 从上方开始
		this.vx = (Math.random() - 0.5) * 0.5; // 轻微的水平漂移
		this.vy = Math.random() * 0.5 + 0.3; // 下落速度
		this.size = Math.random() * 8 + 6; // 大小 6-14px（稍微大一点）
		this.rotation = Math.random() * Math.PI * 2;
		this.rotationSpeed = (Math.random() - 0.5) * 0.02;
		this.opacity = Math.random() * 0.3 + 0.7; // 透明度 0.7-1.0（稍微不透明一点）

		// 随机选择形状变体
		const variants: Array<
			"normal" | "bent" | "torn" | "serrated" | "withStem"
		> = ["normal", "bent", "torn", "serrated", "withStem"];
		this.variant = variants[Math.floor(Math.random() * variants.length)];

		// 如果是落叶，随机选择一个颜色
		if (type === "leaf") {
			const colors = ["#8b4513", "#a0522d", "#cd853f", "#daa520", "#b8860b"];
			this.color = colors[Math.floor(Math.random() * colors.length)];
		} else {
			this.color = "";
		}
	}

	update(width: number, height: number) {
		this.x += this.vx;
		this.y += this.vy;
		this.rotation += this.rotationSpeed;

		// 如果粒子超出屏幕，重新从顶部生成
		if (this.y > height) {
			this.y = -this.size;
			this.x = Math.random() * width;
		}
		if (this.x < -this.size) {
			this.x = width + this.size;
		}
		if (this.x > width + this.size) {
			this.x = -this.size;
		}
	}

	draw(ctx: CanvasRenderingContext2D) {
		ctx.save();
		ctx.translate(this.x, this.y);
		ctx.rotate(this.rotation);
		ctx.globalAlpha = this.opacity;

		if (this.type === "cherry") {
			// 绘制樱花（粉色花瓣）
			this.drawCherry(ctx, this.variant);
		} else {
			// 绘制落叶（棕色/黄色叶子）
			this.drawLeaf(ctx, this.variant);
		}

		ctx.restore();
	}

	drawCherry(
		ctx: CanvasRenderingContext2D,
		variant: "normal" | "bent" | "torn" | "serrated" | "withStem",
	) {
		const petalCount = 5;
		const angleStep = (Math.PI * 2) / petalCount;
		const baseRadius = this.size;
		let petalLength = baseRadius * 0.8;

		// 根据变体调整参数
		if (variant === "bent") {
			// 弯曲的花瓣
			petalLength = baseRadius * 0.7;
		} else if (variant === "torn") {
			// 撕碎的花瓣
			petalLength = baseRadius * 0.6;
		}

		// 创建渐变
		const gradient = ctx.createRadialGradient(0, 0, 0, 0, 0, baseRadius);
		gradient.addColorStop(0, "#ffc0d9");
		gradient.addColorStop(0.5, "#ffb7d5");
		gradient.addColorStop(1, "#ff9ec7");

		ctx.fillStyle = gradient;
		ctx.beginPath();

		// 绘制五瓣花瓣
		for (let i = 0; i < petalCount; i++) {
			const angle = i * angleStep;
			const nextAngle = (i + 1) * angleStep;

			const startAngle = angle + angleStep * 0.15;
			const startX = Math.cos(startAngle) * baseRadius * 0.3;
			const startY = Math.sin(startAngle) * baseRadius * 0.3;

			let tipX = Math.cos(angle) * petalLength;
			let tipY = Math.sin(angle) * petalLength * 0.7;

			// 根据变体调整花瓣形状
			if (variant === "bent") {
				// 弯曲：花瓣向一侧弯曲
				const bendOffset = Math.sin(angle * 2) * baseRadius * 0.2;
				tipX += bendOffset;
			} else if (variant === "torn") {
				// 撕碎：花瓣边缘不规则
				const tearOffset = (Math.random() - 0.5) * baseRadius * 0.3;
				tipX += tearOffset;
				tipY += tearOffset * 0.5;
			}

			const endAngle = nextAngle - angleStep * 0.15;
			const endX = Math.cos(endAngle) * baseRadius * 0.3;
			const endY = Math.sin(endAngle) * baseRadius * 0.3;

			if (i === 0) {
				ctx.moveTo(startX, startY);
			} else {
				ctx.lineTo(startX, startY);
			}

			const controlX = Math.cos(angle) * baseRadius * 0.6;
			const controlY = Math.sin(angle) * baseRadius * 0.5;
			ctx.quadraticCurveTo(controlX, controlY, tipX, tipY);

			const controlX2 = Math.cos(nextAngle) * baseRadius * 0.6;
			const controlY2 = Math.sin(nextAngle) * baseRadius * 0.5;
			ctx.quadraticCurveTo(controlX2, controlY2, endX, endY);
		}

		ctx.closePath();
		ctx.fill();

		// 添加锯齿边缘（serrated变体）
		if (variant === "serrated") {
			ctx.strokeStyle = "#ff9ec7";
			ctx.lineWidth = 0.5;
			ctx.beginPath();
			for (let i = 0; i < petalCount; i++) {
				const angle = i * angleStep;
				const x = Math.cos(angle) * petalLength;
				const y = Math.sin(angle) * petalLength * 0.7;
				if (i === 0) {
					ctx.moveTo(x, y);
				} else {
					ctx.lineTo(x, y);
				}
			}
			ctx.closePath();
			ctx.stroke();
		}

		// 添加中心花蕊
		ctx.fillStyle = "#ff9ec7";
		ctx.beginPath();
		ctx.arc(0, 0, this.size * 0.15, 0, Math.PI * 2);
		ctx.fill();

		// 添加根茎（withStem变体）
		if (variant === "withStem") {
			ctx.strokeStyle = "#ff9ec7";
			ctx.lineWidth = 1;
			ctx.beginPath();
			ctx.moveTo(0, this.size * 0.2);
			ctx.lineTo(0, this.size * 0.5);
			ctx.stroke();
		}

		// 添加中心点高光
		ctx.fillStyle = "#ffc0d9";
		ctx.beginPath();
		ctx.arc(0, 0, this.size * 0.08, 0, Math.PI * 2);
		ctx.fill();
	}

	drawLeaf(
		ctx: CanvasRenderingContext2D,
		variant: "normal" | "bent" | "torn" | "serrated" | "withStem",
	) {
		const width = this.size;
		let height = this.size * 0.8;
		const darkColor = this.color;
		const lightColor = this.adjustColorBrightness(darkColor, 1.2);

		// 根据变体调整高度
		if (variant === "bent") {
			height = this.size * 0.7; // 弯曲的叶子更窄
		} else if (variant === "torn") {
			height = this.size * 0.6; // 撕碎的叶子更小
		}

		// 创建渐变
		const gradient = ctx.createLinearGradient(
			-width * 0.5,
			-height * 0.5,
			width * 0.5,
			height * 0.5,
		);
		gradient.addColorStop(0, lightColor);
		gradient.addColorStop(0.5, darkColor);
		gradient.addColorStop(1, lightColor);

		ctx.fillStyle = gradient;
		ctx.beginPath();

		// 绘制叶子形状
		if (variant === "bent") {
			// 弯曲的叶子：向一侧弯曲
			ctx.moveTo(0, -height * 0.5);
			ctx.bezierCurveTo(
				-width * 0.3,
				-height * 0.2,
				-width * 0.4,
				height * 0.2,
				-width * 0.2,
				height * 0.4,
			);
			ctx.lineTo(0, height * 0.5);
			ctx.bezierCurveTo(
				width * 0.2,
				height * 0.4,
				width * 0.4,
				height * 0.2,
				width * 0.3,
				-height * 0.2,
			);
		} else if (variant === "torn") {
			// 撕碎的叶子：边缘不规则
			ctx.moveTo(0, -height * 0.5);
			ctx.bezierCurveTo(
				-width * 0.4 + Math.random() * width * 0.2 - width * 0.1,
				-height * 0.3,
				-width * 0.5 + Math.random() * width * 0.2 - width * 0.1,
				height * 0.1,
				-width * 0.3 + Math.random() * width * 0.2 - width * 0.1,
				height * 0.4,
			);
			ctx.lineTo(0, height * 0.5);
			ctx.bezierCurveTo(
				width * 0.3 + Math.random() * width * 0.2 - width * 0.1,
				height * 0.4,
				width * 0.5 + Math.random() * width * 0.2 - width * 0.1,
				height * 0.1,
				width * 0.4 + Math.random() * width * 0.2 - width * 0.1,
				-height * 0.3,
			);
		} else {
			// 正常的心形叶子
			ctx.moveTo(0, -height * 0.5);
			ctx.bezierCurveTo(
				-width * 0.4,
				-height * 0.3,
				-width * 0.5,
				height * 0.1,
				-width * 0.3,
				height * 0.4,
			);
			ctx.lineTo(0, height * 0.5);
			ctx.bezierCurveTo(
				width * 0.3,
				height * 0.4,
				width * 0.5,
				height * 0.1,
				width * 0.4,
				-height * 0.3,
			);
		}

		ctx.closePath();
		ctx.fill();

		// 添加锯齿边缘（serrated变体）
		if (variant === "serrated") {
			ctx.strokeStyle = darkColor;
			ctx.lineWidth = 0.5;
			ctx.beginPath();
			const serrationCount = 12;
			for (let i = 0; i < serrationCount; i++) {
				const angle = (i / serrationCount) * Math.PI * 2;
				const x = Math.cos(angle) * width * 0.45;
				const y = Math.sin(angle) * height * 0.45;
				if (i === 0) {
					ctx.moveTo(x, y);
				} else {
					ctx.lineTo(x, y);
				}
			}
			ctx.closePath();
			ctx.stroke();
		}

		// 添加主叶脉
		ctx.strokeStyle = darkColor;
		ctx.lineWidth = 0.8;
		ctx.beginPath();
		ctx.moveTo(0, -height * 0.5);
		ctx.lineTo(0, height * 0.5);
		ctx.stroke();

		// 添加侧脉
		ctx.lineWidth = 0.4;
		ctx.beginPath();
		for (let i = -2; i <= 2; i++) {
			if (i === 0) continue;
			const y = (i / 3) * height * 0.5;
			const sideLength = width * 0.3 * (1 - Math.abs(i) / 3);
			ctx.moveTo(0, y);
			ctx.lineTo(-sideLength, y - height * 0.1);
			ctx.moveTo(0, y);
			ctx.lineTo(sideLength, y - height * 0.1);
		}
		ctx.stroke();

		// 添加根茎（withStem变体）
		if (variant === "withStem") {
			ctx.strokeStyle = darkColor;
			ctx.lineWidth = 1;
			ctx.beginPath();
			ctx.moveTo(0, height * 0.5);
			ctx.lineTo(0, height * 0.8);
			ctx.stroke();
		}
	}

	// 辅助函数：调整颜色亮度
	adjustColorBrightness(color: string, factor: number): string {
		// 简单的颜色亮度调整
		const hex = color.replace("#", "");
		const r = Number.parseInt(hex.substring(0, 2), 16);
		const g = Number.parseInt(hex.substring(2, 4), 16);
		const b = Number.parseInt(hex.substring(4, 6), 16);

		const newR = Math.min(255, Math.floor(r * factor));
		const newG = Math.min(255, Math.floor(g * factor));
		const newB = Math.min(255, Math.floor(b * factor));

		return `#${newR.toString(16).padStart(2, "0")}${newG.toString(16).padStart(2, "0")}${newB.toString(16).padStart(2, "0")}`;
	}
}

let particles: Particle[] = [];

function initCanvas() {
	if (!canvas) return;
	ctx = canvas.getContext("2d");
	if (!ctx) return;

	// 设置画布大小
	canvas.width = window.innerWidth;
	canvas.height = window.innerHeight;

	// 创建粒子
	const particleCount = Math.floor((canvas.width * canvas.height) / 15000); // 根据屏幕大小调整粒子数量
	particles = [];
	for (let i = 0; i < particleCount; i++) {
		const type = isDark ? "leaf" : "cherry";
		particles.push(new Particle(type, canvas.width, canvas.height));
	}
}

function draw() {
	if (!canvas || !ctx) return;

	// 清空画布（使用透明背景，让背景图片显示）
	ctx.clearRect(0, 0, canvas.width, canvas.height);

	// 更新并绘制所有粒子
	for (const particle of particles) {
		particle.update(canvas.width, canvas.height);
		particle.draw(ctx);
	}

	animationId = requestAnimationFrame(draw);
}

function updateTheme() {
	const newIsDark = document.documentElement.classList.contains("dark");
	if (newIsDark !== isDark) {
		isDark = newIsDark;
		// 主题改变时，重新创建粒子
		if (canvas) {
			initCanvas();
		}
	}
}

function handleResize() {
	if (!canvas) return;
	initCanvas();
}

function start() {
	// 如果代码雨启用，不启动粒子特效
	if (codeRainEnabled) {
		stop();
		return;
	}
	if (animationId !== null) return;
	if (!ctx && canvas) {
		initCanvas();
	}
	if (ctx) {
		draw();
	}
}

function checkCodeRainStatus() {
	const stored = localStorage.getItem("codeRainEnabled");
	const newEnabled = stored === null ? false : stored === "true";
	if (newEnabled !== codeRainEnabled) {
		codeRainEnabled = newEnabled;
		if (codeRainEnabled) {
			stop();
		} else {
			start();
		}
	}
}

function stop() {
	if (animationId !== null) {
		cancelAnimationFrame(animationId);
		animationId = null;
	}
}

onMount(() => {
	// 初始化主题状态
	updateTheme();

	// 检查代码雨状态
	checkCodeRainStatus();

	// 等待 canvas 绑定完成
	setTimeout(() => {
		if (canvas) {
			initCanvas();
			start();
		}
	}, 0);

	window.addEventListener("resize", handleResize);

	// 监听代码雨状态变化
	window.addEventListener("codeRainToggle", checkCodeRainStatus);
	window.addEventListener("storage", checkCodeRainStatus);

	// 监听主题变化
	const themeObserver = new MutationObserver(() => {
		updateTheme();
	});

	themeObserver.observe(document.documentElement, {
		attributes: true,
		attributeFilter: ["class"],
	});

	return () => {
		window.removeEventListener("resize", handleResize);
		window.removeEventListener("codeRainToggle", checkCodeRainStatus);
		window.removeEventListener("storage", checkCodeRainStatus);
		themeObserver.disconnect();
	};
});

onDestroy(() => {
	stop();
});

// 响应式更新
$effect(() => {
	if (canvas) {
		// 确保 canvas 已初始化
		if (!ctx && canvas) {
			initCanvas();
		}
		start();
	}
});
</script>

<!-- 粒子特效画布 -->
<canvas
	bind:this={canvas}
	class="fixed top-0 left-0 w-full h-full pointer-events-none z-[1]"
></canvas>

<style>
	canvas {
		opacity: 0.6;
	}
</style>

