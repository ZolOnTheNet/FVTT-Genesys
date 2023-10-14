<script lang="ts" setup>
import VehicleDataModel from '@/actor/data/VehicleDataModel';
import { inject, ref } from 'vue';
import { ActorSheetContext, RootContext } from '@/vue/SheetContext';
import GenesysActor from '@/actor/GenesysActor';

import ContextMenu from '@/vue/components/ContextMenu.vue';
import MenuItem from '@/vue/components/MenuItem.vue';

const props = withDefaults(
	defineProps<{
		id: string;
		dragging?: boolean;
		mini?: boolean;
	}>(),
	{
		dragging: false,
		mini: false,
	},
);

const emit = defineEmits<{
	(e: 'dragstart', event: DragEvent): void;
	(e: 'dragend', event: DragEvent): void;
	(e: 'removeMember'): void;
}>();

const context = inject<ActorSheetContext<VehicleDataModel>>(RootContext)!;

const isBeingDragged = ref(false);

const editLabel = game.i18n.localize('Genesys.Labels.Edit');
const deleteLabel = game.i18n.localize('Genesys.Labels.Delete');

const targetActor = game.actors.get(props.id) as GenesysActor;
const career = targetActor.type === 'character' ? targetActor.items.find((item) => item.type === 'career')?.name : undefined;

function dragStart(event: DragEvent) {
	isBeingDragged.value = true;

	event.dataTransfer?.setData('text/plain', JSON.stringify({ id: props.id }));

	emit('dragstart', event);
}

function dragEnd(event: DragEvent) {
	isBeingDragged.value = false;

	emit('dragend', event);
}

async function openActorSheet() {
	await targetActor.sheet?.render(true);
}
</script>

<template>
	<div
		v-if="mini"
		:class="{
			'actor-tile-mini': true,
			'drag-dim': props.dragging && !isBeingDragged,
			'drag-source': isBeingDragged,
		}"
		@dragstart="dragStart"
		@dragend="dragEnd"
	>
		<img :src="targetActor.img" :alt="targetActor.name" draggable="false" />

		<div class="details">
			<span class="name">
				<a @click="openActorSheet">{{ targetActor.name }}</a>
			</span>

			<div v-if="career">Career: {{ career }}</div>
		</div>

		<ContextMenu class="actions" orientation="left" use-primary-click :disable-menu="!context.data.editable">
			<template v-slot:menu-items>
				<MenuItem @click="openActorSheet">
					<template v-slot:icon><i class="fas fa-edit"></i></template>
					{{ editLabel }}
				</MenuItem>

				<MenuItem @click="emit('removeMember')">
					<template v-slot:icon><i class="fas fa-trash"></i></template>
					{{ deleteLabel }}
				</MenuItem>
			</template>

			<a><i class="fas fa-ellipsis-vertical"></i></a>
		</ContextMenu>
	</div>

	<div
		v-else
		:class="{
			'actor-tile-normal': true,
			'drag-dim': props.dragging && !isBeingDragged,
			'drag-source': isBeingDragged,
		}"
		:data-tooltip="targetActor.name"
		@dragstart="dragStart"
		@dragend="dragEnd"
	>
		<ContextMenu class="actions" position="above" :disable-menu="!context.data.editable">
			<template v-slot:menu-items>
				<MenuItem @click="openActorSheet">
					<template v-slot:icon><i class="fas fa-edit"></i></template>
					Edit
				</MenuItem>

				<MenuItem @click="emit('removeMember')">
					<template v-slot:icon><i class="fas fa-trash"></i></template>
					Delete
				</MenuItem>
			</template>

			<img :src="targetActor.img" :alt="targetActor.name" draggable="false" />
		</ContextMenu>
	</div>
</template>

<style lang="scss" scoped>
@use '@/scss/vars/colors.scss';

.actor-tile-mini {
	display: grid;
	grid-template-columns: [icon] 2rem [name] auto [spacer] 1fr [actions] auto;
	grid-template-rows: auto auto;
	align-items: center;
	column-gap: 0.5em;

	background: transparentize(colors.$light-blue, 0.85);
	border: 1px solid transparentize(colors.$light-blue, 0.7);
	border-radius: 0.25em;
	padding: 2px;

	margin-bottom: 0.25em;

	&:hover {
		background: transparentize(colors.$light-blue, 0.75);
		border: 1px solid transparentize(colors.$light-blue, 0.5);
	}

	img {
		grid-column: icon / span 1;
		border-radius: 0.25em;
	}

	& > .name {
		grid-column: name / span 1;
	}

	.name {
		font-family: 'Roboto Serif', serif;
		font-size: 1.15em;
		display: flex;
		align-items: center;
		gap: 0.25em;
	}

	.details {
		display: grid;
		grid-template-rows: auto auto;
		grid-column: name / span 2;
		font-family: 'Roboto Serif', serif;

		& > div {
			grid-row: 2 / span 1;
			display: flex;
			flex-wrap: wrap;
			font-size: 0.9em;
			color: colors.$dark-blue;
			gap: 0.25em;

			&:not([data-item-type='container']) {
				margin-left: 0.5em;
			}

			& > div {
				&:after {
					display: inline;
					content: ';';
				}

				&:last-child:after {
					display: none;
				}
			}
		}
	}

	.actions {
		grid-column: actions / span 1;
		width: 1.25em;
		text-align: center;
	}
}

.actor-tile-normal {
	&:hover {
		background: transparentize(colors.$light-blue, 0.75);
		border: 2px solid transparentize(colors.$light-blue, 0.5);
		border-radius: 1em;

		img {
			padding: 2px;
			border: none;
		}
	}

	img {
		display: block;
		background: transparentize(colors.$gold, 0.5);
		border: 1px solid colors.$gold;
		border-radius: 1em;
	}
}

.actor-tile-normal,
.actor-tile-mini {
	user-select: none;
	transition: all 0.25s ease-in-out;

	&.drag-source {
		background: transparentize(colors.$light-blue, 0.6) !important;
		border: 1px solid transparentize(colors.$light-blue, 0.4) !important;
	}

	&.drag-dim {
		position: relative;

		background: transparentize(#333, 0.85);
		border: 1px solid transparentize(#333, 0.7);

		opacity: 65%;
	}
}
</style>