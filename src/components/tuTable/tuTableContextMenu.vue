<template>
	<tu-popper arrow>
		<tu-icon v-bind="customIcon"></tu-icon>
		<template #content>
			<tu-popup-menu>
				<tu-popup-item v-for="(item, index) of modelValue" :key="index"
					@click="onOptionClicked(item.onClicked)" :onClickClose="item.closeOnClick" :divider="item.divider"
				>
					<tu-popper v-if="item.hasSubMenu" arrow placement='right'>
						{{ item.caption }}
						<template #content>
							<tu-popup-menu>
								<tu-popup-item v-for="(itemSub, indexSub) of item.subMenu" :key="indexSub" @click="onOptionClicked(itemSub.onClicked)"
								:onClickClose="item.onClicked ?? item.closeOnClick" :divider="item.divider">
									<tu-icon v-if="itemSub.icon">{{ itemSub.icon }}</tu-icon>
									{{ itemSub.caption }}
								</tu-popup-item>
							</tu-popup-menu>
						</template>
					</tu-popper>
					<span v-else>
						{{item.caption}}
					</span>
				</tu-popup-item>
			</tu-popup-menu>
		</template>
	</tu-popper>
</template>

<script lang="ts">
import { defineComponent, inject, PropType } from "vue";
import tuComponent from "../tuComponent";
import { TuTableContextMenuEntry, TuTableStore } from "./tuTableStore";

export default defineComponent({
	extends: tuComponent,
	name: "TuTableContextMenu",
	props: {
		customIcon: {
			type: Object,
			default: () => {
				return {
					icon: "more_horiz"
				};
			}
		},
		rowData: {
			type: Object
		},
		rowIndex: {
			type: Number,
			default: 0
		},
		modelValue: {
			type: Object as PropType<Array<TuTableContextMenuEntry>>,
			default: () => {}
		}
	},
	setup (props, context) {
		const tableInstance = inject<TuTableStore>("tableInstance");
		const onOptionClicked = function (callback: Function) {
			if (callback)
				callback(props.rowData);
		};
		return {
			onOptionClicked
		};
	}
});
</script>

<style lang="scss">
</style>
