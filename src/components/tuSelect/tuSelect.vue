<template>
	<div
		class="tu-select-content"
		:style="{
			['--tu-color']: color ? getColor(color) : ''
		}"
		:class="[
			{ block: block },
			{ inline: inline },
			// colors
			{
				[`tu-component--primary`]:
					!danger && !success && !warn && !dark && !color
			},
			{ [`tu-component--danger`]: !!danger },
			{ [`tu-component--warn`]: !!warn },
			{ [`tu-component--success`]: !!success },
			{ [`tu-component--dark`]: !!dark },
			{ [`tu-component--dropdown`]: !!dropdown }
		]"
	>
		<div
			class="tu-select"
			ref="select"
			:class="[
				`tu-select--state-${state}`,
				{
					'tu-select--disabled': disabled,
					activeOptions: activeOptions,
					loading: loading
				}
			]"
			v-on="selectListener"
		>
			<input
				:readonly="!filter && true"
				:id="!multiple && uid"
				class="tu-select__input"
				ref="input"
				:value="activeFilter ? textFilter : getValueLabel"
				:class="[
					{
						multiple: multiple,
						simple: !multiple && !filter
					}
				]"
				v-bind="$attrs"
				v-on="inputListener"
			/>
			<label
				v-if="label || labelPlaceholder"
				class="tu-select__label"
				:for="uid"
				:class="{
					'tu-select__label--placeholder': labelPlaceholder,
					'tu-select__label--label': label,
					'tu-select__label--hidden': isValue,
					'chips-hovered-multiple': chipsHovered && multiple
				}"
			>
				{{ labelPlaceholder || label }}
			</label>
			<label
				v-if="!multiple && !labelPlaceholder"
				ref="placeholder"
				:for="uid"
				class="tu-select__label tu-select__placeholder"
				:class="{ 'tu-select__label--hidden': isValue || textFilter }"
			>
				{{ placeholder }}
				<slot name="icon" />
			</label>
			<button
				v-if="multiple"
				class="tu-select__chips"
				ref="chips"
				v-on="chipsListener"
			>
				<div v-for="item of getChips" :key="item">
					<component :is="item" />
				</div>
				<input
					v-if="filter"
					class="tu-select__chips__input"
					ref="chipsInput"
					:placeholder="placeholder"
					:id="uid"
					:value="textFilter"
					v-on="chipsFilterListener"
				/>
			</button>
			<transition name="tu-select">
				<div
					v-if="activeOptions"
					class="tu-select__options"
					ref="options"
					:style="{
						['--tu-color']: color ? getColor(color) : ''
					}"
					:class="[
						{
							isColorDark: isColorDark
						},
						// colors
						{
							[`tu-component--primary`]:
								!danger && !success && !warn && !dark && !color
						},
						{ [`tu-component--danger`]: !!danger },
						{ [`tu-component--warn`]: !!warn },
						{ [`tu-component--success`]: !!success },
						{ [`tu-component--dark`]: !!dark }
					]"
					@mouseleave="
						() => {
							targetSelect = false;
							targetSelectInput = false;
						}
					"
					@mouseenter="
						() => {
							targetSelect = true;
							targetSelectInput = true;
						}
					"
				>
					<div class="tu-select__options__content" ref="content">
						<div
							v-if="notData"
							class="tu-select__options__content__not-data"
						>
							<slot v-if="$slots.notData" name="notData" />
							<span v-else> No Data Available </span>
						</div>
						<slot />
					</div>
				</div>
			</transition>
			<div v-if="loading" class="tu-select__loading" />
			<tu-icon
				@click="iconClicked"
				class="tu-icon-arrow"
				:class="{
					'tu-select-icon-down': !activeOptions,
					'tu-select-icon-up': activeOptions
				}"
			></tu-icon>
		</div>

		<transition @before-enter="beforeEnter" @enter="enter" @leave="leave">
			<div
				v-if="$slots[`message-success`]"
				class="tu-select__message tu-select__message--success"
			>
				<slot name="message-success" />
			</div>
		</transition>
		<transition @before-enter="beforeEnter" @enter="enter" @leave="leave">
			<div
				v-if="$slots['message-danger']"
				class="tu-select__message tu-select__message--danger"
			>
				<slot name="message-danger" />
			</div>
		</transition>
		<transition @before-enter="beforeEnter" @enter="enter" @leave="leave">
			<div
				v-if="$slots[`message-warn`]"
				class="tu-select__message tu-select__message--warn"
			>
				<slot name="message-warn" />
			</div>
		</transition>
		<transition @before-enter="beforeEnter" @enter="enter" @leave="leave">
			<div
				v-if="$slots[`message-primary`]"
				class="tu-select__message tu-select__message--primary"
			>
				<slot name="message-primary" />
			</div>
		</transition>
	</div>
</template>

<script lang="ts">
import {
	ref,
	h,
	computed,
	nextTick,
	defineComponent,
	watch,
	getCurrentInstance,
	onMounted,
	onBeforeUnmount
} from "vue";
import tuComponent from "../tuComponent";
import { insertBody, removeBody, setCords } from "@/utils";
import tuOption from "../tuSelect/tuSelectOption.vue";
import tuIcon from "../tuIcon/tuIcon.vue";
import * as _ from "lodash";
import { SelectOptionConstants } from ".";

class SelectConstants {
	public static id = 0;
}

export default defineComponent({
	name: "TuSelect",
	extends: tuComponent,
	components: {
		tuOption
	},
	props: {
		modelValue: {},
		multiple: { type: Boolean, default: false },
		dropdown: { type: Boolean, default: false },
		inline: { type: Boolean, default: false },
		filter: { type: Boolean, default: false },
		placeholder: { type: String, default: "" },
		labelPlaceholder: { type: String, default: "" },
		label: { type: String, default: "" },
		disabled: { type: Boolean, default: false },
		collapseChips: { type: Boolean, default: false },
		loading: { type: Boolean, default: false },
		state: { type: String, default: null },
		block: { type: Boolean, default: false },
		selectItems: { type: Array, default: () => [] }
	},
	provide () {
		return {
			dropdown: computed(() => this.dropdown),
			textFilter: computed(() => this.textFilter),
			uids: computed(() => this.uids),
			hoverOption: computed(() => this.hoverOption),
			parentSelect: this,
			renderSelect: computed(() => this.renderSelect),
			isMultiple: computed(() => this.multiple),
			addChildOption: (
				disabled: boolean,
				value: any,
				label: string,
				offsetTop: number
			) => {
				this.childOptions.push({
					disabled,
					value,
					label,
					offsetTop
				});
			},
			addUid: (uid: any) => {
				this.uids.push(uid);
			},
			callSetHover: () => {
				this.setHover();
			},
			onClickOption: (value: any, label: any) => {
				this.onClickOption(value, label);
			},
			updateActiveOptions: (value: boolean) => {
				this.activeOptions = value;
			},
			targetSelect: computed(() => {
				return this.targetSelect;
			}),
			targetClose: computed(() => {
				return this.targetClose;
			}),
			parentValue: computed(() => {
				return this.modelValue;
			})
		};
	},
	setup (props, context) {
		const renderSelect = ref(false);
		const activeOptions = ref(false);
		const valueLabel = ref<any>(null);
		const hoverOption = ref(-1);
		const uids = ref<any[]>([]);
		const childOptions = ref<any[]>([]);
		const targetSelect = ref(false);
		const targetSelectInput = ref(false);
		const targetClose = ref(false);
		const activeFilter = ref(false);
		const chipsHovered = ref(false);
		const textFilter = ref<String>();
		const childVisibles = ref(0);

		// Template refs
		const chips = ref<HTMLButtonElement>();
		const chipsInput = ref<HTMLInputElement>();
		const input = ref<HTMLInputElement>();
		const options = ref<HTMLDivElement>();
		const select = ref<HTMLDivElement>();
		const content = ref<HTMLDivElement>();

		const uid = "select-" + ++SelectConstants.id;
		const instance = getCurrentInstance();

		const setHover = function () {
			let index: number = -1;
			childOptions.value.forEach((item: any, i: number) => {
				if (item.value === props.modelValue)
					index = i;
			});

			hoverOption.value = index;
		};

		const handleWindowClick = function (evt: any) {
			if (!targetSelectInput.value)
				handleBlur();

			if (props.filter && !activeOptions.value)
				activeFilter.value = false;

			if (
				evt.target === input.value &&
				activeOptions.value &&
				!props.filter
			) {
				handleBlur();
				setTimeout(() => {
					input.value?.blur();
				}, 100);
			}
		};

		const handleBlur = function () {
			nextTick(() => {
				activeOptions.value = false;
				childOptions.value = [];
				SelectOptionConstants.id = 0;
			});
			context.emit("blur");
			setHover();
			window.removeEventListener("click", handleWindowClick);
			if (activeOptions.value) {
				textFilter.value = "";
				if (!props.multiple)
					activeFilter.value = false;
			}
		};

		const clickOption = function (value: any, label: any) {
			if (props.multiple) {
				const oldVal = [...(props.modelValue as Array<any>)];
				if (_.indexOf(oldVal, value) === -1)
					oldVal.push(value);
				else
					oldVal.splice(_.indexOf(oldVal, value), 1);

				const labels = _.reduce(
					childOptions.value,
					function (result: any[], item, key) {
						if (_.indexOf(oldVal, item.value) !== -1) {
							result.push({
								value: item.value,
								label: item.label
							});
						}
						return result;
					},
					[]
				);

				context.emit("update:modelValue", oldVal);
				valueLabel.value = labels;
			}
			else {
				context.emit("update:modelValue", value);
				valueLabel.value = label;
			}

			setTimeout(() => {
				if (props.multiple && activeOptions.value)
					chips.value?.focus();
			}, 10);

			if (!props.multiple)
				handleBlur();
		};

		const onClickOption = function (value: any, label: any) {
			clickOption(value, label);
		};

		const isValue = computed(() => {
			if (Array.isArray(props.modelValue)) return props.modelValue.length !== 0; else {
				return !(
					_.isNull(props.modelValue) &&
					!_.isUndefined(props.modelValue)
				);
			}
		});
		// computeds
		const getChips = computed(() => {
			let id = 0;
			const chipIds: number[] = [];
			const chip = function (item: any, isCollapse: boolean) {
				return h(
					"span",
					{
						class: ["tu-select__chips__chip", { isCollapse }],
						id: ++id
					},
					[
						h("span", {}, item.label),
						!isCollapse &&
							h(
								"span",
								{
									class: "tu-select__chips__chip__close",
									onClick: evt => {
										setTimeout(() => {
											targetClose.value = false;
										}, 100);
										if (!activeOptions.value) {
											chips.value?.blur();
											if (props.filter)
												chipsInput.value?.blur();
										}
										clickOption(item.value, item.label);
										evt.stopPropagation();
									},
									onMouseLeave: () => {
										targetClose.value = false;
									},
									onMouseEnter: () => {
										targetClose.value = true;
									},
									onMouseDown: evt => {
										evt.stopPropagation();
									}
								},
								[
									h(
										tuIcon,
										{
											hover: "less",
											style: { "font-size": "0.5rem" }
										},
										() => {
											return "close";
										}
									)
								]
							)
					]
				);
			};

			let chipsarr: any[] = [];
			if (Array.isArray(valueLabel.value)) {
				for (const item of valueLabel.value as any) {
					const chipEl = chip(item, false);
					chipsarr.push(chipEl);
				}
			}

			if (props.collapseChips) {
				chipsarr = [
					chipsarr[0],
					chipsarr.length > 1 &&
						chip(
							{ label: `+ ${chipsarr.length - 1}`, value: null },
							true
						)
				];
			}

			return chipsarr;
		});

		const getValue = function () {
			const options = childOptions.value;
			const filterOptions = options.filter((option: any): boolean => {
				return typeof props.modelValue === "number" ||
					typeof props.modelValue === "string"
					? props.modelValue === option.value
					: _.find(props.modelValue as Array<any>, {
						value: option.value
					}) !== undefined;
			});

			const label: any[] = [];
			filterOptions.forEach((item: any) => {
				label.push({
					label: (item as any).label,
					value: (item as any).value
				});
			});

			valueLabel.value = label;
		};

		const getValueLabel = computed(() => {
			const valueLabelTemp: any = valueLabel.value;
			let labels: any[] = [];
			if (Array.isArray(valueLabelTemp)) {
				valueLabelTemp.forEach((item: any) => {
					labels.push(item.label);
				});
			}
			else
				labels = valueLabelTemp;

			return labels;
		});

		const selectListener = computed(() => {
			return {
				mouseleave: (evt: any) => {
					if (evt.relatedTarget !== options.value) {
						targetSelectInput.value = false;
						targetSelect.value = false;
					}
				},
				mouseenter: () => {
					targetSelectInput.value = true;
				}
			};
		});

		const handleKeydown = function (evt: any) {
			const optionsTemp = options.value;
			for (let index = 0; index < 300; index++) {
				setTimeout(() => {
					setCords(optionsTemp, select.value);
				}, index);
			}
			if (evt.code === "ArrowDown") {
				evt.preventDefault();
				if (hoverOption.value < childOptions.value?.length - 1)
					hoverOption.value++;
				else
					hoverOption.value = 0;
			}
			else if (evt.code === "ArrowUp") {
				evt.preventDefault();
				if (hoverOption.value > 0)
					hoverOption.value--;
				else
					hoverOption.value = childOptions.value?.length - 1;
			}
			else if (evt.code === "Enter") {
				evt.preventDefault();
				if (hoverOption.value !== -1) {
					if (!childOptions.value?.[hoverOption.value].disabled) {
						clickOption(
							childOptions.value?.[hoverOption.value].value,
							childOptions.value?.[hoverOption.value].label
						);
						if (!props.multiple) {
							handleBlur();
							input.value?.blur();
						}
					}
				}
			}

			if (hoverOption.value !== -1) {
				(content.value as HTMLElement).scrollTop =
					childOptions.value?.[hoverOption.value].offsetTop - 66;
			}
		};

		const inputListener = computed(() => {
			return {
				keydown: handleKeydown,
				focus: (evt: Event) => {
					activeOptions.value = true;
					context.emit("focus", evt);
					if (props.filter)
						activeFilter.value = true;

					window.addEventListener("mousedown", handleWindowClick);
				},
				// blur: this.blur,
				input: (evt: any) => {
					textFilter.value = evt.target.value;
				}
			};
		});

		const chipsListener = computed(() => {
			return {
				keydown: handleKeydown,
				mouseover: event => {
					chipsHovered.value = true;
				},
				mouseout: event => {
					chipsHovered.value = false;
				},
				focus: (evt: Event) => {
					if (!targetClose.value) {
						activeOptions.value = true;
						context.emit("focus", evt);
					}
					if (props.filter && props.multiple)
						(chipsInput.value as HTMLElement).focus();

					window.addEventListener("mousedown", handleWindowClick);
				},
				blur: blur
			};
		});

		const chipsFilterListener = computed(() => {
			return {
				focus: (evt: Event) => {
					if (!targetClose.value) {
						activeOptions.value = true;
						context.emit("focus", evt);
					}
				},
				blur: blur,
				input: (evt: any) => {
					textFilter.value = evt.target.value;
				}
			};
		});

		const notData = computed(() => {
			let newChildOptions: any = [];

			childOptions.value.forEach((option: any): any => {
				if (!option.hiddenOption)
					newChildOptions.push(option);
			});

			newChildOptions = childOptions.value.filter(item => {
				if (item.optionGroup)
					return !item.hiddenOptionGroup;

				return true;
			});

			return newChildOptions.length === 0;
		});

		const iconClicked = function () {
			if (activeOptions.value)
				activeOptions.value = false;
			else
				input.value?.focus();
		};

		const beforeEnter = function (el: any) {
			el.style.height = 0;
		};

		const enter = function (el: any, done: any) {
			const h = el.scrollHeight;
			el.style.height = h - 1 + "px";
			done();
		};

		const leave = function (el: any, done: any) {
			el.style.minHeight = "0px";
			el.style.height = "0px";
		};

		const insertOptions = function () {
			const optionsTemp = options.value as HTMLElement;
			insertBody(optionsTemp, document.body);
			setCords(options.value, select.value);
		};

		watch(activeOptions, (val: boolean) => {
			nextTick(() => {
				if (val) insertOptions();
			});

			uids.value = [];
		});

		const handleResize = function () {
			const optionsTemp = options.value as HTMLElement;
			if (!optionsTemp)
				return;

			nextTick(() => {
				setCords(optionsTemp, select.value);
			});

			for (let index = 0; index < 300; index++) {
				setTimeout(() => {
					setCords(optionsTemp, select.value);
				}, index);
			}
		};

		const handleScroll = function () {
			const optionsTemp = options.value as HTMLElement;
			if (optionsTemp)
				setCords(optionsTemp, select.value);
		};

		onMounted(() => {
			// getValue();

			let children: any = instance?.slots.default?.();
			if (children?.length === 1 && typeof children[0].type === "symbol") children = children[0].children;
			const reduced = _.reduce(
				children,
				function (result: any[], item, index) {
					result.push({
						label: item.props?.label,
						value: item.props?.value
					});
					return result;
				},
				[]
			);

			// set the default value first. Since the list is not rendered to the dom, get it from the default slot.
			if (!Array.isArray(props.modelValue)) {
				const labelValue = _.find(reduced, { value: props.modelValue });
				if (labelValue) valueLabel.value = labelValue.label;
			}
			else {
				const newLabelValues: any[] = [];
				props.modelValue.forEach(value => {
					const labelValue = _.find(reduced, { value: value });
					if (labelValue) newLabelValues.push(labelValue);
				});

				valueLabel.value = newLabelValues;
			}

			window.addEventListener("resize", handleResize);
			window.addEventListener("scroll", handleScroll);
		});

		onBeforeUnmount(() => {
			if (activeOptions.value) {
				const optionsTemp = options.value as HTMLElement;
				removeBody(optionsTemp, document.body);
			}
		});

		return {
			chipsHovered,
			renderSelect,
			activeOptions,
			valueLabel,
			hoverOption,
			uids,
			childOptions,
			targetSelect,
			targetSelectInput,
			targetClose,
			activeFilter,
			textFilter,
			childVisibles,
			uid,
			getChips,
			selectListener,
			inputListener,
			getValueLabel,
			chipsListener,
			chipsFilterListener,
			notData,
			iconClicked,
			chips,
			chipsInput,
			input,
			options,
			select,
			content,
			beforeEnter,
			enter,
			leave,
			setHover,
			onClickOption,
			isValue
		};
	}
});
</script>

<style lang="scss" scoped>
@import "../../style/sass/_mixins";

@mixin state($color) {
	.tu-select__input {
		// border: 2px solid -getColor($color,.5)
		background: -getColor($color, 0.05);
		color: -getColor($color, 1);

		&:hover {
			// border: 2px solid -getColor($color, 0)
			color: -getColor("text", 1);
		}
	}

	.tu-select__chips {
		background: -getColor($color, 0.05);
		color: -getColor($color, 1);
		&:hover {
			&:after {
				opacity: 0;
			}
		}

		&:after {
			width: calc(100% - 4px);
			height: calc(100% - 4px);
			content: "";
			position: absolute;
			top: 0px;
			left: 0px;
			border: 2px solid -getColor($color, 0.5);
			border-radius: inherit;
			transition: all 0.25s ease;
		}
	}

	.tu-select__label {
		color: -getColor($color, 1);
	}

	.tu-select__icon {
		color: -getColor($color, 1);
		background: -getColor($color, 0.1);
		box-shadow: (-15px) 10px 10px -10px -getColor($color, 0.1);
	}

	.tu-icon-arrow {
		&:after {
			background: -getColor($color, 1);
		}

		&:before {
			background: -getColor($color, 1);
		}
	}
}

.tu-select-enter-active {
	transition: all 0.25s ease;
}

.tu-select-enter-from {
	opacity: 0;
	transform: translate(0, -10px);
	transition: all 0.25s ease;
	box-shadow: 0px 0px 0px 0px rgba(0, 0, 0, -var(shadow-opacity));

	&:after {
		opacity: 0 !important;
		box-shadow: 0px 0px 0px 0px rgba(0, 0, 0, -var(shadow-opacity));
	}
}

.tu-select-leave-active {
	transition: all 0.25s ease;
}

.tu-select-leave-to {
	opacity: 0;
	transform: translate(0, -10px);
	transition: all 0.25s ease;
	box-shadow: 0px 0px 0px 0px rgba(0, 0, 0, -var(shadow-opacity));

	&.top {
		transform: translate(0, 10px) !important;
	}

	&:after {
		opacity: 0 !important;
		box-shadow: 0px 0px 0px 0px rgba(0, 0, 0, -var(shadow-opacity));
	}
}

.tu-select-content {
	width: 100%;
	max-width: fit-content;
	display: flex;
	flex-direction: column;

	&.inline {
		display: inline-flex;
	}

	margin: 5px;
	padding: 5px;
	&.block {
		&.block {
			max-width: 100%;
		}
	}
}

.tu-select__chips {
	max-height: 50px;
	overflow: auto;
	&::-webkit-scrollbar {
		width: 5px;
		height: 5px;
		display: block;
		background: transparent;
	}
}

.tu-select {
	position: relative;
	display: flex;
	align-items: center;
	justify-content: center;
	min-height: 38px;
	width: 100%;

	&--state-success {
		@include state("success");
	}

	&--state-danger {
		@include state("danger");
	}

	&--state-warn {
		@include state("warn");
	}

	&--state-dark {
		@include state("dark");
	}

	&--state-primary {
		@include state("primary");
	}

	&.loading {
		pointer-events: none;

		*:not(.tu-select__loading):not(.tu-select__label) {
			opacity: 0.6;
		}

		.tu-select__label {
			pointer-events: none !important;
		}
	}

	&.top {
		&.activeOptions {
			.tu-select__input,
			.tu-select__chips {
				border-radius: 0px 0px 12px 12px !important;
				box-shadow: 0px -5px 25px -4px
					rgba(0, 0, 0, -var(shadow-opacity)) !important;
			}
		}
	}

	&--disabled {
		opacity: 0.6;
		pointer-events: none;

		label {
			pointer-events: none;
		}
	}

	.tu-icon-arrow {
		z-index: 600;
		position: absolute;
		right: 15px;
		margin-top: -2px;
		transition: all 0.25s ease;
		pointer-events: auto;
		cursor: pointer;
	}

	&.activeOptions {
		.tu-icon-arrow {
			transform: rotate(45deg);
			margin-top: (-2px) !important;
		}

		.tu-select__input {
			border-radius: 12px 12px 0px 0px;
			border: 1px solid -getColor("text", 0.09);
			background: -getColor("background");
			//box-shadow: 0px 5px 25px -4px rgba(0, 0, 0, -var(shadow-opacity));
			//transform: translate(0, -4px);
			transition: all 0.25s ease, height 0s;
			border: 2px solid transparent;
			color: -getColor("text", 1);
		}

		.tu-select__chips {
			border-radius: 12px 12px 0px 0px;
			background: -getColor("background");
			box-shadow: 0px 5px 25px -4px rgba(0, 0, 0, -var(shadow-opacity));
			transform: translate(0, -4px);
			transition: all 0.25s ease, height 0s;
			&:after {
				opacity: 0;
			}
		}

		.tu-select__label--placeholder {
			opacity: 1;
			visibility: visible;
			pointer-events: auto;
			transform: translate(-1%, -30px) !important;
			font-size: 0.8rem;
			margin-top: 0px !important;
		}

		.tu-select__label {
			margin-top: -4px;
		}
	}

	&__input {
		opacity: 1;
		background: transparent;
		padding: 7px 13px;
		border: 1px solid -getColor("text", 0.09);
		border-radius: 12px;
		cursor: pointer;
		transition: all 0.25s ease, height 0s;
		background: -getColor("gray-2");
		color: -getColor("text");
		min-height: 38px;
		padding-right: 30px;
		width: 100%;

		&.multiple {
			color: transparent;
			background: transparent;
			pointer-events: none;
		}

		&.simple {
			user-select: none;
		}

		&:focus {
			border-radius: 12px 12px 0px 0px;
			background: -getColor("background");
			//box-shadow: 0px 5px 25px -4px rgba(0, 0, 0, -var(shadow-opacity));
			//transform: translate(0, -4px);
			transition: all 0.25s ease;

			~ .tu-select__label--placeholder {
				opacity: 1;
				visibility: visible;
				pointer-events: auto;
				//transform: translate(-3%, -28px) !important;
				font-size: 0.8rem;
				margin-top: 0px !important;
			}
		}

		&:hover {
			background: -getColor("background");
			//box-shadow: 0px 5px 25px -4px rgba(0, 0, 0, -var(shadow-opacity));
			//transform: translate(0, -4px);

			~ .tu-select__label {
				margin-top: -4px;
			}

			~ .tu-icon-arrow {
				margin-top: -6px;
			}
		}
	}

	&__chips {
		width: 100%;
		height: auto;
		position: absolute;
		left: 0px;
		background: -getColor("gray-2");
		z-index: 300;
		border-radius: 12px;
		display: flex;
		border: 0px;
		display: flex;
		align-items: flex-start;
		justify-content: flex-start;
		flex-wrap: wrap;
		padding: 5px;
		min-height: 38px;
		transition: all 0.25s ease, height 0s;
		padding-right: 26px;

		&:focus {
			border-radius: 12px 12px 0px 0px;
			background: -getColor("background");
			//box-shadow: 0px 5px 25px -4px rgba(0, 0, 0, -var(shadow-opacity));
			//transform: translate(0, -4px);
			transition: all 0.25s ease;
		}

		&:hover {
			background: -getColor("background");
			//box-shadow: 0px 5px 25px -4px rgba(0, 0, 0, -var(shadow-opacity));
			//transform: translate(0, -8px);
			&.chips-hovered-multiple {
				transform: translate(0, -8px);
			}
			transition: all 0.25s ease;

			~ .tu-icon-arrow {
				margin-top: -6px;
				transition: all 0.25s ease;
			}
		}

		&__input {
			width: auto;
			flex: 1;
			max-width: 100%;
			position: relative;
			min-width: 0px;
			border: 0px solid transparent;
			background: transparent;
			margin: 2px 3px;
			min-width: 30px;

			&::placeholder {
				color: -getColor("text", 0.4);
			}
		}

		&__chip {
			flex: 0 1 auto;
			position: relative;
			background: -getColor("background");
			border-radius: 10px;
			display: flex;
			align-items: center;
			justify-content: center;
			padding: 0px 6px;
			margin: 2px 3px;
			padding-right: 10px;
			font-size: 0.84rem;
			border: 2px solid -getColor("gray-2");
			box-sizing: border-box;
			color: -getColor("text");

			&.isCollapse {
				padding-right: 6px !important;
			}

			&__close {
				position: absolute;
				top: -4px;
				right: -4px;
				width: 15px;
				height: 15px;
				background: -getColor("gray-4");
				display: flex;
				align-items: center;
				justify-content: center;
				border-radius: 50%;
				font-size: 0.75rem;
				cursor: pointer;
				transition: all 0.25s ease;

				&:hover {
					background: -getColor("danger");

					.tu-icon-close {
						--tu-color: var(--tu-background);

						&:after {
							width: 12px;
							transform: rotate(180deg);
						}

						&:before {
							width: 12px;
							transform: rotate(180deg);
						}
					}
				}

				.tu-icon-close {
					--tu-color: var(--tu-text);
					transform: scale(0.5);
				}
			}
		}
	}

	&__options {
		--tu-color: var(--tu-primary);
		position: absolute;
		z-index: 99999;
		background: -getColor("background");
		padding: 5px;
		border-radius: 0px 0px 12px 12px;
		overflow: hidden;
		box-shadow: 0px 10px 20px -5px rgba(0, 0, 0, -var(shadow-opacity));

		&.top {
			border-radius: 12px 12px 0px 0px;
			box-shadow: 0px -10px 20px -5px rgba(0, 0, 0, -var(shadow-opacity));

			&:after {
				top: auto;
				bottom: -10px;
			}
		}

		&:after {
			content: "";
			position: absolute;
			top: -10px;
			width: 80%;
			margin-left: 10%;
			left: 0px;
			height: 10px;
			background: -getColor("background");
			box-shadow: 0px 0px 20px 0px rgba(0, 0, 0, -var(shadow-opacity));
			z-index: 200;
			transition: all 0.25s ease 0.05s;
			opacity: 1;
		}

		&__content {
			max-height: 200px;
			overflow: auto;
			height: auto;
			z-index: 100;
			transform: scale(1);
			transition: all 0.25s ease;
			position: relative;
			scroll-behavior: smooth;

			&__not-data {
				font-size: 0.8rem;
				text-align: center;
				padding: 6px 10px;
			}

			&::-webkit-scrollbar {
				width: 5px;
				height: 5px;
				display: block;
				background: transparent;
			}

			&::-webkit-scrollbar-thumb {
				background: -getColor("gray-3");
				border-radius: 5px;
			}
		}
	}

	&__label {
		position: absolute;
		left: 14px;
		font-size: 0.8rem;
		transition: all 0.25s ease;
		cursor: text;
		user-select: none;
		pointer-events: none;
		width: 100%;
		display: flex;
		align-items: center;
		justify-content: flex-start;
		opacity: 0.4;
		z-index: 500;

		// top: 10px
		&--hidden {
			opacity: 0;
			visibility: hidden;

			&.tu-select__label--placeholder {
				opacity: 1;
				visibility: visible;
				pointer-events: auto;
				transform: translate(-3%, -28px) !important;
				font-size: 0.8rem;
				margin-top: 0px !important;
			}
		}

		&--label {
			opacity: 1;
			visibility: visible;
			pointer-events: auto;
			transform: translate(-3%, -28px) !important;
			font-size: 0.8rem;
			margin-top: 0px !important;
		}
	}

	&__loading {
		position: absolute;
		width: 22px;
		height: 22px;
		right: 7px;
		pointer-events: none;
		border-radius: 50%;
		box-sizing: border-box;
		background: inherit;
		cursor: default;
		z-index: 600;

		&:after {
			box-sizing: border-box;
			position: absolute;
			width: 100%;
			height: 100%;
			border: 2px solid -getColor("primary", 1);
			border-radius: inherit;
			border-top: 2px solid transparent;
			border-left: 2px solid transparent;
			border-right: 2px solid transparent;
			animation: rotateInputLoading 0.8s ease infinite;
			top: 0px;
			content: "";
		}

		&:before {
			box-sizing: border-box;
			top: 0px;
			position: absolute;
			width: 100%;
			height: 100%;
			border: 2px dashed -getColor("primary", 1);
			border-radius: inherit;
			border-top: 2px solid transparent;
			border-left: 2px solid transparent;
			border-right: 2px solid transparent;
			animation: rotateInputLoading 0.8s linear infinite;
			opacity: 0.2;
			content: "";
		}

		& ~ .tu-icon-arrow {
			opacity: 0 !important;
		}
	}

	&__message {
		font-size: 0.7rem;
		position: relative;
		padding: 0px 7px;
		transition: all 0.25s ease;
		overflow: hidden;
		height: unset !important;

		&--success {
			color: -getColor("success", 1);
		}

		&--danger {
			color: -getColor("danger", 1);
		}

		&--warn {
			color: -getColor("warn", 1);
		}

		&--dark {
			color: -getColor("dark", 1);
		}

		&--primary {
			color: -getColor("primary", 1);
		}
	}
}

.tu-darken {
	.tu-select__options {
		&.isColorDark {
			--tu-color: 0, 0, 0 !important;

			.tu-select__option {
				&:hover {
					background: -getColor("color", 0.2);
					color: -getColor("text");
				}
			}

			.activeOption {
				color: -getColor("text");
				background: -getColor("color", 0.6);
			}
		}
	}
}

.tu-select-content:not(.tu-select--dropdown)
	.tu-select
	.tu-select__placeholder {
	opacity: 1;
	margin-left: 10px;
}
</style>
