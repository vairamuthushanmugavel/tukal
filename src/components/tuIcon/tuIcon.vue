<template>
  <i
    :style="iconStyle"
    :class="[getIconPack, icon, iconClass, getBg, getBgSize, {'round':round}, {'outlined': outlined}]"
    v-bind="$attrs"
    class="tu-icon notranslate icon-scale unselectable"
  >
	{{ (getIconPack.includes("material-icons") && icon !== null) ? icon : '' }}
    <slot v-if="$slots.default"></slot>
  </i>
</template>
<script lang="ts">
import { computed, defineComponent } from "vue";
import _color from "../../utils/color";
import tuComponent, { ComponentConstants } from "../tuComponent";

export class IconPackConstants {
	public static iconPackGlobal = "fa";
}

export default defineComponent({
	name: "TuIcon",
	extends: tuComponent,
	props: {
		icon: {
			default: null,
			type: String
		},
		iconPack: {
			default: null,
			type: String
		},
		color: {
			default: null,
			type: String
		},
		bg: {
			default: null,
			type: String
		},
		size: {
			default: null,
			type: String
		},
		round: {
			default: false,
			type: Boolean
		},
		outlined: {
			default: false,
			type: Boolean
		}

	},
	setup (props, context) {
		let iconPackGlobal: string | null = null;
		if (iconPackGlobal === null)
			iconPackGlobal = ComponentConstants.iconPackGlobal;

		const getIconPack = computed(() => {
			if (props.iconPack)
				return props.iconPack;

			else if (iconPackGlobal)
				return iconPackGlobal;

			else
				return "material-icons";
		});

		const iconClass = computed(() => {
			const classes = {};
			classes[props.size] = true;
			if (_color.isColor(props.color))
				classes[`tu-icon-${props.color}`] = true;
			if (getIconPack.value.includes("material-icons") && props.outlined)
				classes["material-icons-outlined"] = true;
			return classes;
		});
		const iconStyle = computed(() => {
			const style = {
				width: /(px)/.test(props.size) ? props.size : /(em)/.test(props.size) ? props.size : null,
				height: /(px)/.test(props.size) ? props.size : /(em)/.test(props.size) ? props.size : null,
				"font-size": /(px)/.test(props.size) ? props.size : /(em)/.test(props.size) ? props.size : null,
				color: getColor.value,
				background: getBgColor.value,
				cursor: "pointer"
			};
			return style;
		});
		const getBg = computed(() => {
			const classes = {};

			if (_color.isColor(props.bg))
				classes[`con-tu-icon-bg-${props.bg}`] = true;

			return classes;
		});
		const getBgSize = computed(() => {
			const classes = {};
			if (["small", "medium", "large"].includes(props.size)) {
				classes[`bg-${props.size}`] = true;
				classes["tu-icon-bg"] = true;
			}

			return classes;
		});

		const getColor = computed(() => {
			return _color.isColor(props.color) ? props.color : props.color;
		});

		const getBgColor = computed(() => {
			return _color.isColor(props.bg) ? props.bg : props.bg;
		});

		return {
			getIconPack,
			iconClass,
			iconStyle,
			getBg,
			getColor,
			getBgColor,
			getBgSize
		};
	}
});
</script>
