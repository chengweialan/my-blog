<script lang="ts">
import { onDestroy, onMount } from "svelte";

let canvas: HTMLCanvasElement;
let ctx: CanvasRenderingContext2D | null = null;
let animationId: number | null = null;
let isDark = $state(false);

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

	constructor(type: "cherry" | "leaf", width: number, height: number) {
		this.type = type;
		this.x = Math.random() * width;
		this.y = Math.random() * height - height; // 从上方开始
		this.vx = (Math.random() - 0.5) * 0.5; // 轻微的水平漂移
		this.vy = Math.random() * 0.5 + 0.3; // 下落速度
		this.size = Math.random() * 8 + 4; // 大小 4-12px
		this.rotation = Math.random() * Math.PI * 2;
		this.rotationSpeed = (Math.random() - 0.5) * 0.02;
		this.opacity = Math.random() * 0.5 + 0.5; // 透明度 0.5-1.0

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
			this.drawCherry(ctx);
		} else {
			// 绘制落叶（棕色/黄色叶子）
			this.drawLeaf(ctx);
		}

		ctx.restore();
	}

	drawCherry(ctx: CanvasRenderingContext2D) {
		// 绘制樱花花瓣（五瓣花瓣）
		const petalCount = 5;
		const angleStep = (Math.PI * 2) / petalCount;

		ctx.fillStyle = "#ffb7d5"; // 淡粉色
		ctx.beginPath();
		for (let i = 0; i < petalCount; i++) {
			const angle = i * angleStep;
			const x = Math.cos(angle) * this.size;
			const y = Math.sin(angle) * this.size * 0.6;
			if (i === 0) {
				ctx.moveTo(x, y);
			} else {
				ctx.lineTo(x, y);
			}
		}
		ctx.closePath();
		ctx.fill();

		// 添加中心点
		ctx.fillStyle = "#ff9ec7"; // 稍深的粉色
		ctx.beginPath();
		ctx.arc(0, 0, this.size * 0.2, 0, Math.PI * 2);
		ctx.fill();
	}

	drawLeaf(ctx: CanvasRenderingContext2D) {
		// 绘制落叶（椭圆形叶子）
		ctx.fillStyle = this.color;
		ctx.beginPath();
		// 绘制椭圆形叶子
		ctx.ellipse(
			0,
			0,
			this.size,
			this.size * 0.7,
			this.rotation,
			0,
			Math.PI * 2,
		);
		ctx.fill();

		// 添加叶脉（主脉）
		ctx.strokeStyle = this.color;
		ctx.lineWidth = 0.5;
		ctx.beginPath();
		ctx.moveTo(0, -this.size * 0.7);
		ctx.lineTo(0, this.size * 0.7);
		ctx.stroke();

		// 添加侧脉
		ctx.beginPath();
		for (let i = -2; i <= 2; i++) {
			if (i === 0) continue;
			const y = (i / 3) * this.size * 0.7;
			ctx.moveTo(-this.size * 0.3, y);
			ctx.lineTo(this.size * 0.3, y);
		}
		ctx.stroke();
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
	if (animationId !== null) return;
	if (!ctx && canvas) {
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
}

onMount(() => {
	// 初始化主题状态
	updateTheme();

	// 等待 canvas 绑定完成
	setTimeout(() => {
		if (canvas) {
			initCanvas();
			start();
		}
	}, 0);

	window.addEventListener("resize", handleResize);

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

