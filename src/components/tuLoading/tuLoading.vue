<template>
	<transition name="tu-loading">
		<div
			v-if="isVisible"
			class="tu-loading"
			:class="[
				`tu-loading--${type || 'default'}`,
				{ 'tu-loading--target': !!target },
				{ 'tu-loading--background': !!background },
			]"
		>
			<div
				class="tu-loading__load"
				:style="{
					transform: `scale(${scale})`,
				}"
			>
				<div class="tu-loading__load__animation">
					<div class="tu-loading__load__percent">
						{{ percent }}
					</div>
					<div class="tu-loading__load__animation__1" />
					<div class="tu-loading__load__animation__2" />
					<div class="tu-loading__load__animation__3" />
				</div>
				<div class="tu-loading__load__text">
					{{ text }}
				</div>
			</div>
			<div v-if="progress" class="tu-loading__progress">
				<div
					class="tu-loading__progress__bar"
					:style="{
						width: `${progress}%`,
					}"
				></div>
			</div>
		</div>
	</transition>
</template>

<script lang="ts">
import { setColor, setVar } from "@/utils";
import { defineComponent, getCurrentInstance, watch } from "vue";
import tuComponent from "../tuComponent";

export default defineComponent({
	name: "TuLoading",	
	props: {
		text: { type: String, default: null },
		type: { type: String, default: "default" },
		color: { type: String, default: null },
		background: { type: String, default: null },
		opacity: { type: String, default: null },
		percent: { type: String, default: null },
		progress: { type: Number, default: null },
		scale: { type: String, default: null },
		target: {},
		isVisible: { type: Boolean, default: true },
	},
	setup(props, context) {
		let instance = getCurrentInstance();

		watch(
			() => props.isVisible,
			() => {
				if (instance?.vnode.el) {
					setColor("color", props.color, instance.vnode.el);
					setColor("background", props.background, instance.vnode.el);
					if (props.opacity) {
						setVar("opacity", props.opacity, instance.vnode.el);
					}
				}
			}
		);
	},
});
</script>

<style lang="scss" scoped>
@import "../../style/sass/_mixins";

.tu-loading-enter-active,
.tu-loading-leave-active {
	transition: opacity 0.5s;
}

.tu-loading-enter,
.tu-loading-leave-to {
	opacity: 0;
}

.tu-loading {
	--tu-color: var(--tu-primary);
	--tu-background: 255, 255, 255;
	--tu-opacity: 0.6;

	top: 0px;
	left: 0px;
	width: 100%;
	height: 100%;
	z-index: 100000;
	display: flex;
	align-items: center;
	justify-content: center;
	background: -getColor("background", -var("opacity"));
	transition: all 0.25s ease;
	padding: 20px;
	border-radius: inherit;

	&--target {
		position: absolute;
	}

    &:not(.tu-loading--target) {
        position: fixed;
    }

	&__load {
		display: flex;
		align-items: center;
		justify-content: center;
		flex-direction: column;

		&__text {
			font-size: 0.75em;
			margin: 7px;
			font-weight: bold;
			color: -getColor("color");
		}

		&__percent {
			position: relative;
			font-size: 0.65rem;
			font-weight: bold;
			color: -getColor("color");
			margin-top: 1px;
			z-index: 200;
		}

		&__animation {
			width: 40px;
			height: 40px;
			position: relative;
			display: flex;
			align-items: center;
			justify-content: center;
		}
	}
}

.tu-loading__progress {
	width: 100%;
	position: absolute;
	top: 0px;
	left: 0px;
	height: 4px;
	background: -getColor("color", 0.2);

	&__bar {
		background: -getColor("color");
		height: 100%;
		position: relative;
		border-radius: 0px 10px 10px 0px;
	}
}

.tu-loading--default {
	.tu-loading__load__animation {
		border-radius: 50%;

		&__1 {
			position: absolute;
			width: 100%;
			height: 100%;
			border: 3px solid -getColor("color");
			border-radius: inherit;
			border-top: 3px solid transparent;
			border-left: 3px solid transparent;
			border-right: 3px solid transparent;
			animation: rotate 0.8s ease infinite;
			top: 0px;
		}

		&__2 {
			top: 0px;
			position: absolute;
			width: 100%;
			height: 100%;
			border: 3px dashed -getColor("color");
			border-radius: inherit;
			border-top: 3px solid transparent;
			border-left: 3px solid transparent;
			border-right: 3px solid transparent;
			animation: rotate 0.8s linear infinite;
			opacity: 0.2;
		}

		&__3 {
			display: none;
		}
	}
}

@keyframes rotate {
	0% {
		transform: rotate(0deg);
	}

	100% {
		transform: rotate(360deg);
	}
}

.tu-loading--waves {
	.tu-loading__load__animation {
		display: flex;
		align-items: center;
		justify-content: center;

		&__1 {
			width: 0px;
			background: transparent;
			height: 0px;
			position: absolute;
			animation: waves 0.7s ease infinite;
			box-shadow: 0px 0px 10px 0px -getColor("color", 0.5);
			border-radius: 50%;
		}

		&__2 {
			width: 0px;
			background: transparent;
			height: 0px;
			position: absolute;
			animation: waves 1.4s linear infinite;
			box-shadow: 0px 0px 10px 0px -getColor("color", 0.5);
			border-radius: 50%;
		}

		&__3 {
			width: 0px;
			background: transparent;
			height: 0px;
			position: absolute;
			animation: waves 1.75s ease infinite;
			box-shadow: 0px 0px 10px 0px -getColor("color", 0.5);
			border-radius: 50%;
		}
	}
}

@keyframes waves {
	0% {
		width: 0px;
		height: 0px;
	}

	100% {
		height: 50px;
		width: 50px;
		opacity: 0;
	}
}

.tu-loading--corners {
	.tu-loading__load__animation {
		display: flex;
		align-items: center;
		justify-content: center;

		&__1 {
			width: 100%;
			background: transparent;
			height: 100%;
			position: absolute;
			animation: corners 1s ease infinite;
			border-radius: 50%;
			border: 2px solid -getColor("color", 1);
		}

		&__2 {
			display: none;
		}

		&__3 {
			display: none;
		}
	}

	// corners
}

@keyframes corners {
	0% {
		border-radius: 50%;
		transform: rotate(0deg);
	}

	25% {
		border-radius: 50% 50% 50% 15%;
	}

	50% {
		border-radius: 50% 50% 15% 30%;
	}

	75% {
		border-radius: 50% 15% 30% 30%;
	}

	100% {
		border-radius: 50%;
		transform: rotate(-180deg);
	}
}

.tu-loading--border {
	.tu-loading__load__animation {
		display: flex;
		align-items: center;
		justify-content: center;
		border-radius: 50%;

		&__1 {
			position: absolute;
			width: 100%;
			height: 100%;
			border: 1px solid -getColor("color");
			border-radius: inherit;
			border-top: 1px solid transparent;
			border-left: 1px solid transparent;
			border-right: 1px solid transparent;
			animation: rotate 1s linear infinite;
			top: 0px;
		}

		&__2 {
			top: 0px;
			position: absolute;
			width: 100%;
			height: 100%;
			border: 1px dashed -getColor("color");
			border-radius: inherit;
			border-top: 1px solid transparent;
			border-left: 1px solid transparent;
			border-right: 1px solid transparent;
			animation: rotate 1s linear infinite 0.2s;
		}

		&__3 {
			top: 0px;
			position: absolute;
			width: 100%;
			height: 100%;
			border: 1px dashed -getColor("color");
			border-radius: inherit;
			animation: rotate 1s linear infinite 0.4s;
			opacity: 0.2;
		}
	}
}

.tu-loading--points {
	.tu-loading__load__percent {
		position: absolute;
		top: -10px;
	}

	.tu-loading__load__animation {
		display: flex;
		align-items: flex-end;
		justify-content: center;
		width: auto;

		&__1 {
			width: 8px;
			height: 8px;
			background: -getColor("color");
			border-radius: 50%;
			margin: 3px;
			animation: points 0.75s ease infinite;
		}

		&__2 {
			width: 8px;
			height: 8px;
			background: -getColor("color");
			border-radius: 50%;
			margin: 3px;
			animation: points 0.75s ease infinite 0.25s;
		}

		&__3 {
			width: 8px;
			height: 8px;
			background: -getColor("color");
			border-radius: 50%;
			margin: 3px;
			animation: points 0.75s ease infinite 0.5s;
		}
	}
}

@keyframes points {
	0% {
		transform: translate(0px, 0px);
	}

	50% {
		transform: translate(0, -15px);
	}

	100% {
		transform: translate(0px, 0px);
	}
}

.tu-loading--square {
	.tu-loading__load__animation {
		display: flex;
		align-items: center;
		justify-content: center;

		&__1 {
			position: absolute;
			width: 100%;
			height: 100%;
			border: 2px solid -getColor("color");
			border-radius: inherit;
			animation: rotateSquare 4s ease infinite;
			top: 0px;
		}

		&__2 {
			top: 0px;
			position: absolute;
			width: 100%;
			height: 100%;
			border: 2px solid -getColor("color");
			border-radius: inherit;
			animation: rotateSquare 4s ease infinite reverse;
			background: -getColor("background", 1);
		}

		&__3 {
			display: none;
		}
	}
}

@keyframes rotateSquare {
	0% {
		transform: rotate(0deg);
	}

	50% {
		transform: rotate(360deg);
	}

	100% {
		transform: rotate(0deg);
	}
}

.tu-loading--gradient {
	.tu-loading__load__animation {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 60px;
		height: 60px;

		&__1 {
			position: absolute;
			width: 100%;
			height: 100%;
			border: 0px;
			border-radius: inherit;
			animation: rotate 1s linear infinite;
			top: 0px;
			background: linear-gradient(
				0deg,
				-getColor("background", 0) 33%,
				-getColor("color", 1) 100%
			);
			border-radius: 50%;
		}

		&__2 {
			top: 2px;
			position: absolute;
			width: calc(100% - 4px);
			height: calc(100% - 4px);
			border: 0px;
			border-radius: inherit;
			background: -getColor("background", 1);
			border-radius: 50%;
		}

		&__3 {
			display: none;
		}
	}
}

.tu-loading--rectangle {
	.tu-loading__load__percent {
		position: absolute;
		bottom: -2px;
	}

	.tu-loading__load__animation {
		display: flex;
		align-items: flex-end;
		justify-content: center;

		&__1 {
			position: absolute;
			width: 15px;
			height: 15px;
			border: 0px;
			border-radius: inherit;
			animation: rectangle 1s ease infinite;
			background: -getColor("color", 1);
		}

		&__2 {
			position: absolute;
			width: 15px;
			height: 15px;
			border: 0px;
			border-radius: inherit;
			animation: rectangle 1s ease-out infinite;
			background: -getColor("color", 0.2);
		}

		&__3 {
			display: none;
		}
	}
}

@keyframes rectangle {
	0% {
		transform: translate(-50px);
	}

	50% {
		transform: translate(50px);
	}

	100% {
		transform: translate(-50px);
	}
}

.tu-loading--circles {
	.tu-loading__load__animation {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 60px;
		height: 60px;

		&__1 {
			position: absolute;
			width: 40px;
			height: 40px;
			animation: rotate 1s ease infinite;
			border-radius: 50%;
			border: 2px solid -getColor("color", 1);
			border-top: 3px solid transparent;
			border-left: 3px solid transparent;
			border-right: 3px solid transparent;
		}

		&__2 {
			position: absolute;
			width: 50px;
			height: 50px;
			animation: rotate 1s ease-in-out infinite;
			border-radius: 50%;
			border: 2px dashed -getColor("color", 1);
			border-top: 3px solid transparent;
			border-left: 3px solid transparent;
			border-right: 3px solid transparent;
		}

		&__3 {
			position: absolute;
			width: 60px;
			height: 60px;
			animation: rotate 1s linear infinite reverse;
			border-radius: 50%;
			border: 2px solid -getColor("color", 1);
			border-top: 3px solid transparent;
			border-left: 3px solid transparent;
			border-right: 3px solid transparent;
		}
	}
}

.tu-loading--square-rotate {
	.tu-loading__load__percent {
		position: absolute;
		bottom: 30px;
	}

	.tu-loading__load__animation {
		display: flex;
		align-items: flex-end;
		justify-content: center;
		width: 60px;
		height: 60px;

		&__1 {
			position: absolute;
			width: 25px;
			height: 25px;
			animation: squareRotate 3s ease infinite;
			background: -getColor("color", 1);
		}

		&__2 {
			display: none;
		}

		&__3 {
			display: none;
		}
	}
}

@keyframes squareRotate {
	0% {
		transform: rotateX(0deg) rotateY(0deg);
	}

	25% {
		transform: rotateX(180deg) rotateY(0deg);
	}

	50% {
		transform: rotateX(180deg) rotateY(180deg);
	}

	75% {
		transform: rotateX(0deg) rotateY(180deg);
	}

	100% {
		transform: rotateX(0deg) rotateY(0deg);
	}
}

.tu-loading--scale {
	.tu-loading__load__percent {
		position: absolute;
		bottom: 40px;
	}

	.tu-loading__load__animation {
		display: flex;
		align-items: cneter;
		justify-content: center;
		width: 40px;
		height: 40px;

		&__1 {
			position: relative;
			width: 5px;
			height: 5px;
			background: -getColor("color", 1);
			margin: 3px;
			animation: scale 0.8s ease infinite;
			border-radius: 5px;
		}

		&__2 {
			position: relative;
			width: 5px;
			height: 5px;
			background: -getColor("color", 1);
			margin: 3px;
			animation: scale 0.8s ease infinite 0.2s;
			border-radius: 5px;
		}

		&__3 {
			position: relative;
			width: 5px;
			height: 5px;
			background: -getColor("color", 1);
			margin: 3px;
			animation: scale 0.8s ease infinite 0.4s;
			border-radius: 5px;
		}
	}
}

@keyframes scale {
	0% {
		height: 5px;
	}

	50% {
		height: 25px;
	}

	0% {
		height: 5px;
	}
}
</style>