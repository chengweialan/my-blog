<script lang="ts">
import { onMount } from "svelte";
import { siteConfig } from "../config";
import { url } from "../utils/url-utils";

let isCoverVisible = $state(true); // 封面是否可见（分立状态），默认显示封面
let coverScroll = $state(0); // 累积滚动距离，用于触发切换
let isDark = $state(false);

// 获取字体文件 URL（包含 base 路径）
const fontUrl = url("/fonts/YeZiGongChangZuiHanJiangXingCao-2.ttf");
const subtitleFontUrl = url("/fonts/AnJingChenXingShuFanTi-2.ttf");

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

function updateTheme() {
	const newIsDark = document.documentElement.classList.contains("dark");
	if (newIsDark !== isDark) {
		isDark = newIsDark;
	}
}

function showCover() {
	// 显示封面页
	isCoverVisible = true;
	coverScroll = 0;
	updateCoverState();
	// 滚动到页面顶部
	window.scrollTo({ top: 0, behavior: "smooth" });

	// 确保字体样式存在（处理页面切换后字体丢失的情况）
	const fontStyleId = "cover-page-fonts";
	let fontStyle = document.getElementById(fontStyleId) as HTMLStyleElement;
	if (!fontStyle) {
		fontStyle = document.createElement("style");
		fontStyle.id = fontStyleId;
		fontStyle.textContent = `
			@font-face {
				font-family: "Xingkai";
				src: url("${fontUrl}") format("truetype");
				font-weight: normal;
				font-style: normal;
				font-display: swap;
			}
			@font-face {
				font-family: "AnJingChenXingShuFanTi";
				src: url("${subtitleFontUrl}") format("truetype");
				font-weight: normal;
				font-style: normal;
				font-display: swap;
			}
		`;
		document.head.appendChild(fontStyle);
	} else {
		// 如果字体样式已存在，更新URL（处理页面切换的情况）
		fontStyle.textContent = `
			@font-face {
				font-family: "Xingkai";
				src: url("${fontUrl}") format("truetype");
				font-weight: normal;
				font-style: normal;
				font-display: swap;
			}
			@font-face {
				font-family: "AnJingChenXingShuFanTi";
				src: url("${subtitleFontUrl}") format("truetype");
				font-weight: normal;
				font-style: normal;
				font-display: swap;
			}
		`;
	}
}

onMount(() => {
	// 动态创建字体样式 - 使用ID确保不会重复创建
	const fontStyleId = "cover-page-fonts";
	let fontStyle = document.getElementById(fontStyleId) as HTMLStyleElement;

	if (!fontStyle) {
		fontStyle = document.createElement("style");
		fontStyle.id = fontStyleId;
		fontStyle.textContent = `
			@font-face {
				font-family: "Xingkai";
				src: url("${fontUrl}") format("truetype");
				font-weight: normal;
				font-style: normal;
				font-display: swap;
			}
			@font-face {
				font-family: "AnJingChenXingShuFanTi";
				src: url("${subtitleFontUrl}") format("truetype");
				font-weight: normal;
				font-style: normal;
				font-display: swap;
			}
		`;
		document.head.appendChild(fontStyle);
	} else {
		// 如果字体样式已存在，更新URL（处理页面切换的情况）
		fontStyle.textContent = `
			@font-face {
				font-family: "Xingkai";
				src: url("${fontUrl}") format("truetype");
				font-weight: normal;
				font-style: normal;
				font-display: swap;
			}
			@font-face {
				font-family: "AnJingChenXingShuFanTi";
				src: url("${subtitleFontUrl}") format("truetype");
				font-weight: normal;
				font-style: normal;
				font-display: swap;
			}
		`;
	}

	// 初始化封面状态 - 检查页面是否在顶部
	const initializeCoverState = () => {
		const scrollTop = window.scrollY || document.documentElement.scrollTop;
		// 如果页面在顶部，确保封面可见；如果不在顶部，确保封面不可见
		if (scrollTop === 0) {
			isCoverVisible = true;
			coverScroll = 0;
		} else {
			isCoverVisible = false;
			coverScroll = 0;
		}
		updateCoverState();
	};

	// 立即初始化封面状态
	initializeCoverState();

	// 延迟检查一次，确保状态正确（处理浏览器恢复滚动位置的情况）
	setTimeout(() => {
		initializeCoverState();
	}, 0);

	// 再延迟检查一次，确保完全加载后状态正确
	setTimeout(() => {
		initializeCoverState();
	}, 100);

	// 初始化主题状态
	updateTheme();

	// 监听显示封面事件
	const handleShowCover = (event: CustomEvent<{ show: boolean }>) => {
		if (event.detail.show) {
			showCover();
		}
	};
	window.addEventListener("showCover", handleShowCover as EventListener);

	// 监听跳转到内容事件
	const handleScrollToContent = () => {
		scrollToContent();
	};
	window.addEventListener(
		"scrollToContent",
		handleScrollToContent as EventListener,
	);

	window.addEventListener("wheel", handleWheel, { passive: false });
	window.addEventListener("scroll", handleScroll, { passive: true });

	// 监听主题变化
	const themeObserver = new MutationObserver(() => {
		updateTheme();
	});
	themeObserver.observe(document.documentElement, {
		attributes: true,
		attributeFilter: ["class"],
	});

	// 封面页点击事件：跳转到内容页
	const handleCoverClick = (e: MouseEvent) => {
		if (isCoverVisible) {
			// 检查点击的目标是否在导航栏或其他允许点击的区域
			const target = e.target as HTMLElement;
			const navbar = target.closest("#navbar");
			const navMenuPanel = target.closest("#nav-menu-panel");
			const displaySetting = target.closest("#display-setting");
			const codeRainPanel = target.closest("#code-rain-panel");
			const coverPageDownPanel = target.closest("#cover-page-down-panel");
			const coverPagePanel = target.closest("#cover-page-panel");
			const artisticArrow =
				target.closest(".artistic-arrow-container") ||
				target.closest(".artistic-arrow-btn") ||
				target.closest(".artistic-arrow");

			// 如果点击的是导航栏、相关面板或艺术箭头，允许点击（不阻止默认行为）
			if (
				navbar ||
				navMenuPanel ||
				displaySetting ||
				codeRainPanel ||
				coverPageDownPanel ||
				coverPagePanel ||
				artisticArrow
			) {
				return;
			}

			// 否则跳转到内容页
			scrollToContent();
			e.preventDefault();
			e.stopPropagation();
			e.stopImmediatePropagation();
		}
	};
	document.addEventListener("click", handleCoverClick, true); // 使用捕获阶段

	// 监听链接点击事件（排除封面页的点击）
	const handleLinkClick = (e: MouseEvent) => {
		// 如果点击的是封面页，不处理链接点击
		if (isCoverVisible) {
			const target = e.target as HTMLElement;
			const navbar = target.closest("#navbar");
			const navMenuPanel = target.closest("#nav-menu-panel");
			const displaySetting = target.closest("#display-setting");
			const codeRainPanel = target.closest("#code-rain-panel");
			const coverPageDownPanel = target.closest("#cover-page-down-panel");
			const coverPagePanel = target.closest("#cover-page-panel");
			const artisticArrow =
				target.closest(".artistic-arrow-container") ||
				target.closest(".artistic-arrow-btn") ||
				target.closest(".artistic-arrow");

			// 如果点击的是导航栏、相关面板或艺术箭头，允许点击
			if (
				navbar ||
				navMenuPanel ||
				displaySetting ||
				codeRainPanel ||
				coverPageDownPanel ||
				coverPagePanel ||
				artisticArrow
			) {
				// 检查是否是主页链接，如果是主页链接且当前在封面页，应该重置封面并跳转到内容
				const link = target.closest("a");
				if (link?.href) {
					const url = new URL(link.href, window.location.href);
					// 如果是主页链接（/），重置封面并跳转到内容
					const currentPath = window.location.pathname;
					const linkPath = url.pathname.replace(/\/$/, "") || "/";
					if (
						url.origin === window.location.origin &&
						(linkPath === "/" || linkPath === currentPath.replace(/\/$/, ""))
					) {
						resetCover();
						// 稍微滚动一下，确保离开顶部
						window.scrollTo({ top: 1, behavior: "smooth" });
					}
				}
				return;
			}

			// 否则阻止链接点击
			return;
		}

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
		window.removeEventListener("showCover", handleShowCover as EventListener);
		window.removeEventListener(
			"scrollToContent",
			handleScrollToContent as EventListener,
		);
		window.removeEventListener("wheel", handleWheel);
		window.removeEventListener("scroll", handleScroll);
		document.removeEventListener("click", handleCoverClick, true);
		document.removeEventListener("click", handleLinkClick);
		themeObserver.disconnect();
	};
});
</script>

<div
	class="fixed inset-0 z-20 flex flex-col items-center justify-center pointer-events-none"
	class:opacity-0={!isCoverVisible}
	class:opacity-100={isCoverVisible}
	class:pointer-events-auto={isCoverVisible}
	class:cursor-pointer={isCoverVisible}
	style="transition: opacity 0.3s ease-out;"
>
	<div class="text-center relative">
		<!-- 墨迹装饰元素 -->
		<div class="ink-blot ink-blot-1"></div>
		<div class="ink-blot ink-blot-2"></div>
		<div class="ink-blot ink-blot-3"></div>
		<div class="ink-blot ink-blot-4"></div>
		
		<!-- SVG 墨迹装饰 -->
		<svg class="ink-svg ink-svg-1" viewBox="0 0 200 150">
			<path
				d="M50,75 Q30,50 20,30 Q10,20 15,10 Q20,5 30,15 Q40,25 50,40 Q60,55 70,65 Q80,75 90,80 Q100,85 110,90 Q120,95 130,100 Q140,105 150,110 Q160,115 170,120 Q180,125 185,130 Q190,135 195,140 Q200,145 195,150 Q190,155 180,150 Q170,145 160,140 Q150,135 140,130 Q130,125 120,120 Q110,115 100,110 Q90,105 80,100 Q70,95 60,90 Q50,85 45,80 Q40,75 50,75 Z"
				fill="currentColor"
				opacity="0.1"
			/>
		</svg>
		<svg class="ink-svg ink-svg-2" viewBox="0 0 200 150">
			<path
				d="M100,20 Q80,30 60,40 Q40,50 30,60 Q20,70 25,80 Q30,90 40,100 Q50,110 60,115 Q70,120 80,125 Q90,130 100,135 Q110,140 120,145 Q130,150 140,145 Q150,140 160,135 Q170,130 180,125 Q190,120 195,115 Q200,110 195,100 Q190,90 180,80 Q170,70 160,60 Q150,50 140,40 Q130,30 120,25 Q110,20 100,20 Z"
				fill="currentColor"
				opacity="0.08"
			/>
		</svg>
		
		<!-- 印章装饰 -->
		<div class="seal seal-left"></div>
		<div class="seal seal-right"></div>
		
		<!-- 主标题：玮指导的个人博客 -->
		<h1
			class="calligraphy-title relative z-10"
		>
			玮指导的个人博客
		</h1>
		
		<!-- 副标题：两行文字，部分重叠 -->
		<div class="subtitle-container relative z-10 mt-6">
			<div class="subtitle-line subtitle-line-1">胸中无一字</div>
			<div class="subtitle-line subtitle-line-2">好作理塘诗</div>
		</div>
	</div>
	
	<!-- 艺术下箭头 -->
	<div class="artistic-arrow-container absolute bottom-8 left-1/2 transform -translate-x-1/2 z-10">
		<button
			aria-label="跳转到内容"
			class="artistic-arrow-btn"
			onclick={(e) => {
				e.preventDefault();
				e.stopPropagation();
				scrollToContent();
			}}
		>
			<svg class="artistic-arrow" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
				<!-- 箭头主体 - 金色描边 -->
				<path
					d="M50 20 L50 60 M30 50 L50 70 L70 50"
					stroke="#ffd700"
					stroke-width="4"
					fill="none"
					stroke-linecap="round"
					stroke-linejoin="round"
					class="arrow-path-gold"
				/>
				<!-- 箭头主体 - 白色填充 -->
				<path
					d="M50 20 L50 60 M30 50 L50 70 L70 50"
					stroke="#ffffff"
					stroke-width="3"
					fill="none"
					stroke-linecap="round"
					stroke-linejoin="round"
					class="arrow-path"
				/>
				<!-- 装饰线条 -->
				<path
					d="M20 50 Q50 30 50 50 Q50 70 80 50"
					stroke="#ffd700"
					stroke-width="1.5"
					fill="none"
					stroke-linecap="round"
					opacity="0.6"
					class="arrow-decoration"
				/>
			</svg>
		</button>
	</div>
</div>



<style>
	/* 自定义字体声明 - 通过 JavaScript 动态创建 */

	/* 行草字体样式 - 黑边白字 */
	.calligraphy-title {
		font-size: 5rem;
		font-weight: 900;
		color: #ffffff;
		font-family: "Xingkai", "STXingkai", "Xingkai SC", "KaiTi", "楷体", "STKaiti", "STCaiyun", "STHupo", "STLiti", "STSong", "STFangsong", "FangSong", "FangSong SC", "SimSun", "SimHei", "Microsoft YaHei", "Microsoft JhengHei", "PingFang SC", "Hiragino Sans GB", "WenQuanYi Micro Hei", sans-serif;
		letter-spacing: 0.1em;
		line-height: 1.2;
		position: relative;
		display: inline-block;
		padding: 0.5em 1em;
		/* 黑色描边效果 - 使用多层阴影模拟描边 */
		text-shadow: 
			/* 描边效果 - 8个方向 */
			-2px -2px 0 #000000,
			2px -2px 0 #000000,
			-2px 2px 0 #000000,
			2px 2px 0 #000000,
			-1px -1px 0 #000000,
			1px -1px 0 #000000,
			-1px 1px 0 #000000,
			1px 1px 0 #000000,
			/* 外层描边 */
			-3px -3px 0 #000000,
			3px -3px 0 #000000,
			-3px 3px 0 #000000,
			3px 3px 0 #000000,
			/* 深度阴影 */
			4px 4px 8px rgba(0, 0, 0, 0.5),
			6px 6px 12px rgba(0, 0, 0, 0.4);
		/* 3D 透视效果 */
		transform: perspective(1000px) rotateX(2deg);
		/* 动画 */
		animation: titleFadeIn 1.5s ease-out, titleFloat 4s ease-in-out infinite 1.5s, titleBreathe 3s ease-in-out infinite 2s;
		/* 增强发光 */
		filter: drop-shadow(0 0 4px rgba(0, 0, 0, 0.5)) drop-shadow(0 0 8px rgba(0, 0, 0, 0.3));
	}

	:global(.dark) .calligraphy-title {
		color: #ffffff;
		text-shadow: 
			/* 描边效果 - 8个方向 */
			-2px -2px 0 #000000,
			2px -2px 0 #000000,
			-2px 2px 0 #000000,
			2px 2px 0 #000000,
			-1px -1px 0 #000000,
			1px -1px 0 #000000,
			-1px 1px 0 #000000,
			1px 1px 0 #000000,
			/* 外层描边 */
			-3px -3px 0 #000000,
			3px -3px 0 #000000,
			-3px 3px 0 #000000,
			3px 3px 0 #000000,
			/* 深度阴影 */
			4px 4px 8px rgba(0, 0, 0, 0.5),
			6px 6px 12px rgba(0, 0, 0, 0.4);
		filter: drop-shadow(0 0 4px rgba(0, 0, 0, 0.5)) drop-shadow(0 0 8px rgba(0, 0, 0, 0.3));
	}

	/* 文字淡入动画 */
	@keyframes titleFadeIn {
		from {
			opacity: 0;
			transform: perspective(1000px) rotateX(2deg) translateY(30px);
		}
		to {
			opacity: 1;
			transform: perspective(1000px) rotateX(2deg) translateY(0);
		}
	}

	/* 文字浮动动画 */
	@keyframes titleFloat {
		0%, 100% {
			transform: perspective(1000px) rotateX(2deg) translateY(0);
		}
		50% {
			transform: perspective(1000px) rotateX(2deg) translateY(-8px);
		}
	}

	/* 文字呼吸动画 */
	@keyframes titleBreathe {
		0%, 100% {
			transform: perspective(1000px) rotateX(2deg) scale(1);
		}
		50% {
			transform: perspective(1000px) rotateX(2deg) scale(1.02);
		}
	}

	/* 墨迹装饰元素 */
	.ink-blot {
		position: absolute;
		opacity: 0.12;
		pointer-events: none;
		background: radial-gradient(ellipse at 30% 30%, #1a1a1a 0%, #1a1a1a 40%, transparent 70%);
		filter: blur(2px);
		animation: ink-blot-fade 10s ease-in-out infinite;
	}

	:global(.dark) .ink-blot {
		background: radial-gradient(ellipse at 30% 30%, #f5f5f5 0%, #f5f5f5 40%, transparent 70%);
		opacity: 0.08;
	}

	.ink-blot-1 {
		width: 150px;
		height: 100px;
		top: 8%;
		left: 12%;
		border-radius: 50% 40% 60% 30%;
		transform: rotate(-25deg);
		animation-delay: 0s;
	}

	.ink-blot-2 {
		width: 120px;
		height: 90px;
		top: 18%;
		right: 18%;
		border-radius: 40% 50% 30% 60%;
		transform: rotate(35deg);
		animation-delay: 2.5s;
	}

	.ink-blot-3 {
		width: 110px;
		height: 80px;
		bottom: 22%;
		left: 8%;
		border-radius: 60% 30% 50% 40%;
		transform: rotate(-15deg);
		animation-delay: 5s;
	}

	.ink-blot-4 {
		width: 130px;
		height: 95px;
		bottom: 12%;
		right: 12%;
		border-radius: 30% 60% 40% 50%;
		transform: rotate(45deg);
		animation-delay: 7.5s;
	}

	@keyframes ink-blot-fade {
		0%, 100% {
			opacity: 0.08;
			transform: scale(1) rotate(var(--rotation, 0deg));
		}
		50% {
			opacity: 0.18;
			transform: scale(1.05) rotate(calc(var(--rotation, 0deg) + 5deg));
		}
	}

	/* SVG 墨迹装饰 */
	.ink-svg {
		position: absolute;
		pointer-events: none;
		width: 200px;
		height: 150px;
		opacity: 0.1;
		animation: inkSvgFlow 12s ease-in-out infinite;
		color: #1a1a1a;
		filter: blur(1px);
	}

	:global(.dark) .ink-svg {
		color: #f5f5f5;
		opacity: 0.08;
	}

	.ink-svg-1 {
		top: 5%;
		left: 5%;
		transform: rotate(-20deg);
		animation-delay: 0s;
	}

	.ink-svg-2 {
		bottom: 10%;
		right: 8%;
		transform: rotate(25deg);
		animation-delay: 6s;
	}

	@keyframes inkSvgFlow {
		0% {
			opacity: 0.05;
			transform: scale(0.9) rotate(var(--rotation, 0deg));
		}
		50% {
			opacity: 0.15;
			transform: scale(1.1) rotate(calc(var(--rotation, 0deg) + 10deg));
		}
		100% {
			opacity: 0.05;
			transform: scale(0.9) rotate(var(--rotation, 0deg));
		}
	}

	/* 印章装饰 */
	.seal {
		position: absolute;
		width: 80px;
		height: 80px;
		border: 3px solid;
		border-color: #8b0000;
		border-radius: 8px;
		opacity: 0.3;
		pointer-events: none;
		transform: rotate(-15deg);
		animation: sealPulse 4s ease-in-out infinite;
	}

	:global(.dark) .seal {
		border-color: #ff6b6b;
		opacity: 0.2;
	}

	.seal-left {
		top: 15%;
		left: 8%;
		animation-delay: 0s;
	}

	.seal-right {
		bottom: 20%;
		right: 10%;
		transform: rotate(15deg);
		animation-delay: 2s;
	}

	.seal::before {
		content: "玮";
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		font-size: 2.5rem;
		font-weight: 900;
		font-family: "STKaiti", "KaiTi", "楷体", serif;
		color: inherit;
		opacity: 0.5;
	}

	@keyframes sealPulse {
		0%, 100% {
			opacity: 0.2;
			transform: scale(1) rotate(var(--rotation, -15deg));
		}
		50% {
			opacity: 0.4;
			transform: scale(1.05) rotate(calc(var(--rotation, -15deg) + 3deg));
		}
	}

	/* 宣纸纹理背景效果 */
	.calligraphy-title::before {
		content: "";
		position: absolute;
		top: -20px;
		left: -20px;
		right: -20px;
		bottom: -20px;
		background: 
			radial-gradient(circle at 20% 30%, rgba(0, 0, 0, 0.02) 0%, transparent 50%),
			radial-gradient(circle at 80% 70%, rgba(0, 0, 0, 0.02) 0%, transparent 50%),
			radial-gradient(circle at 50% 50%, rgba(0, 0, 0, 0.01) 0%, transparent 50%);
		pointer-events: none;
		z-index: -1;
		border-radius: 10px;
	}

	:global(.dark) .calligraphy-title::before {
		background: 
			radial-gradient(circle at 20% 30%, rgba(255, 255, 255, 0.01) 0%, transparent 50%),
			radial-gradient(circle at 80% 70%, rgba(255, 255, 255, 0.01) 0%, transparent 50%),
			radial-gradient(circle at 50% 50%, rgba(255, 255, 255, 0.005) 0%, transparent 50%);
	}

	/* 副标题样式 */
	.subtitle-container {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		position: relative;
	}

	.subtitle-line {
		font-family: "AnJingChenXingShuFanTi", "STXingkai", "Xingkai SC", "KaiTi", "楷体", serif;
		font-size: 4rem;
		font-weight: normal;
		color: #ffffff; /* 白色 */
		line-height: 1.2;
		position: relative;
		display: inline-block;
		/* 黑色描边效果 */
		text-shadow: 
			-2px -2px 0 #000000,
			2px -2px 0 #000000,
			-2px 2px 0 #000000,
			2px 2px 0 #000000,
			-1px -1px 0 #000000,
			1px -1px 0 #000000,
			-1px 1px 0 #000000,
			1px 1px 0 #000000,
			-3px -3px 0 #000000,
			3px -3px 0 #000000,
			-3px 3px 0 #000000,
			3px 3px 0 #000000;
		white-space: nowrap;
	}

	.subtitle-line-1 {
		/* 第一行稍微靠左 */
		transform: translateX(-22%);
		z-index: 2;
	}

	.subtitle-line-2 {
		/* 第二行稍微靠右，与第一行部分重叠 */
		transform: translateX(22%);
		margin-top: -0.3em; /* 让两行更靠近，形成重叠效果 */
		z-index: 1;
	}

	:global(.dark) .subtitle-line {
		color: #ffffff; /* 白色 */
		text-shadow: 
			-2px -2px 0 #000000,
			2px -2px 0 #000000,
			-2px 2px 0 #000000,
			2px 2px 0 #000000,
			-1px -1px 0 #000000,
			1px -1px 0 #000000,
			-1px 1px 0 #000000,
			1px 1px 0 #000000,
			-3px -3px 0 #000000,
			3px -3px 0 #000000,
			-3px 3px 0 #000000,
			3px 3px 0 #000000;
	}

	/* 艺术下箭头样式 */
	.artistic-arrow-container {
		pointer-events: auto;
		cursor: pointer;
	}

	.artistic-arrow-btn {
		background: none;
		border: none;
		padding: 0;
		cursor: pointer;
		display: flex;
		align-items: center;
		justify-content: center;
		transition: transform 0.3s ease;
		-webkit-tap-highlight-color: transparent;
	}

	.artistic-arrow-btn:hover {
		transform: scale(1.1);
	}

	.artistic-arrow-btn:active {
		transform: scale(0.95);
	}

	.artistic-arrow {
		width: 60px;
		height: 60px;
		color: #ffffff;
		filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.3));
		animation: arrowFloat 2s ease-in-out infinite;
	}

	.arrow-path-gold {
		stroke: #ffd700; /* 金色描边 */
		stroke-width: 4;
		filter: drop-shadow(0 0 3px rgba(255, 215, 0, 0.8)) drop-shadow(0 0 6px rgba(255, 215, 0, 0.4));
		animation: arrowPulse 2s ease-in-out infinite;
	}

	.arrow-path {
		stroke: #ffffff; /* 白色主体 */
		stroke-width: 3;
		filter: drop-shadow(0 0 2px rgba(0, 0, 0, 0.3));
		animation: arrowPulse 2s ease-in-out infinite;
	}

	.arrow-decoration {
		stroke: #ffd700; /* 金色 */
		stroke-width: 1.5;
		opacity: 0.6;
		animation: arrowFade 2s ease-in-out infinite;
		filter: drop-shadow(0 0 2px rgba(255, 215, 0, 0.6));
	}

	@keyframes arrowFloat {
		0%, 100% {
			transform: translateY(0);
		}
		50% {
			transform: translateY(-10px);
		}
	}

	@keyframes arrowPulse {
		0%, 100% {
			opacity: 1;
		}
		50% {
			opacity: 0.7;
		}
	}

	@keyframes arrowFade {
		0%, 100% {
			opacity: 0.6;
		}
		50% {
			opacity: 0.3;
		}
	}

	:global(.dark) .artistic-arrow {
		color: #ffffff;
		filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.5));
	}

	:global(.dark) .arrow-path-gold {
		stroke: #ffd700; /* 金色描边 */
		filter: drop-shadow(0 0 3px rgba(255, 215, 0, 0.8)) drop-shadow(0 0 6px rgba(255, 215, 0, 0.4));
	}

	:global(.dark) .arrow-path {
		stroke: #ffffff; /* 白色主体 */
		filter: drop-shadow(0 0 2px rgba(0, 0, 0, 0.5));
	}

	:global(.dark) .arrow-decoration {
		stroke: #ffd700; /* 金色 */
		filter: drop-shadow(0 0 2px rgba(255, 215, 0, 0.6));
	}

	/* 响应式设计 */
	@media (max-width: 768px) {
		.calligraphy-title {
			font-size: 3rem;
			letter-spacing: 0.05em;
			transform: perspective(800px) rotateX(1deg);
		}

		.subtitle-line {
			font-size: 2.2rem;
		}

		.subtitle-line-1 {
			transform: translateX(-18%);
		}

		.subtitle-line-2 {
			transform: translateX(18%);
		}

		.ink-blot {
			width: 60px;
			height: 40px;
		}

		.ink-svg {
			width: 100px;
			height: 75px;
		}

		.seal {
			width: 50px;
			height: 50px;
		}

		.seal::before {
			font-size: 1.5rem;
		}

		.artistic-arrow {
			width: 50px;
			height: 50px;
		}
	}

	@media (min-width: 769px) and (max-width: 1024px) {
		.calligraphy-title {
			font-size: 4rem;
		}

		.subtitle-line {
			font-size: 3rem;
		}

		.ink-svg {
			width: 150px;
			height: 112px;
		}

		.seal {
			width: 65px;
			height: 65px;
		}

		.seal::before {
			font-size: 2rem;
		}

		.artistic-arrow {
			width: 55px;
			height: 55px;
		}
	}

	@media (min-width: 1025px) {
		.calligraphy-title {
			font-size: 6rem;
		}

		.subtitle-line {
			font-size: 4.5rem;
		}
	}
</style>

