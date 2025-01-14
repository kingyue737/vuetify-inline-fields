<template>
	<div
		ref="inlineFieldsContainer"
		:class="inlineFieldsContainerClass"
		:style="inlineFieldsContainerStyle"
	>
		<div
			v-if="(!showField && !settings.fieldOnly) || settings.cardField"
			:class="displayContainerClass"
		>
			<div :class="displayInputControlClasses">
				<DisplayedValue
					v-bind="bindingDisplay"
					@toggleField="toggleField"
				>

					<!-- Pass on all scoped slots -->
					<template
						v-for="(_, slot) in slots"
						#[slot]="scope"
					>
						<slot
							:name="slot"
							v-bind="{ ...scope }"
						/>
					</template>
				</DisplayedValue>
			</div>
		</div>

		<div
			v-if="showField || settings.fieldOnly || settings.cardField"
			class="d-flex align-center py-2"
			:class="fieldContainerClass"
		>
			<Teleport
				:disabled="!settings.cardField"
				:to="cardFieldRef"
			>
				<slot
					name="default"
					v-bind="slotBindings"
				/>

				<SaveFieldButtons
					v-model="modelValue"
					:cancel-button-color="settings.cancelButtonColor"
					:cancel-button-size="settings.cancelButtonSize"
					:cancel-button-title="settings.cancelButtonTitle"
					:cancel-button-variant="settings.cancelButtonVariant"
					:cancel-icon="settings.cancelIcon"
					:cancel-icon-color="settings.cancelIconColor"
					:error="error"
					:field-only="settings.fieldOnly"
					:hide-save-icon="settings.hideSaveIcon"
					:loading="loadingProp"
					:loading-icon="settings.loadingIcon"
					:loading-icon-color="settings.loadingIconColor"
					:required="settings.required"
					:save-button-color="settings.saveButtonColor"
					:save-button-size="settings.saveButtonSize"
					:save-button-title="settings.saveButtonTitle"
					:save-button-variant="settings.saveButtonVariant"
					:save-icon="settings.saveIcon"
					:save-icon-color="settings.saveIconColor"
					@close="closeField"
					@save="saveValue"
				/>
			</Teleport>
		</div>

		<!-- Card Field-->
		<div
			v-if="settings.cardField"
			:class="cardContainerClass"
			:style="cardContainerStyle"
		>
			<v-card v-bind="bindingCard">
				<v-card-text>
					<div ref="cardFieldRef"></div>
				</v-card-text>
			</v-card>
		</div>
	</div>
</template>

<script setup lang="ts">
import {
	CloseSiblingsBus,
	FieldValue,
	SharedProps,
	TimeOpened,
	VInlineTextFieldProps,
} from '@/types';
import { IconOptions } from 'vuetify';
import {
	defaultCardProps,
	textFieldProps,
} from './utils/props';
import { DisplayedValue, SaveFieldButtons } from './components/index';
import {
	useCheckForErrors,
	useToggleField,
	useTruncateText,
} from './composables/methods';
import {
	useCardContainerClass,
	useDisplayContainerClass,
	useDisplayInputControlClasses,
	useFieldContainerClass,
	useInlineFieldsContainerClass,
} from './composables/classes';
import {
	useCardContainerStyle,
	useInlineFieldsContainerStyle,
} from './composables/styles';
import inlineEmits from './utils/emits';


const modelValue = defineModel<FieldValue>();

const attrs = useAttrs();
const slots = useSlots();
const emit = defineEmits([...inlineEmits]);

const iconOptions = inject<IconOptions>(Symbol.for('vuetify:icons'));

const props = withDefaults(defineProps<VInlineTextFieldProps>(), { ...textFieldProps });
let settings = reactive({ ...attrs, ...props });
const loadingProp = computed(() => props.loading);
const disabledProp = computed(() => props.disabled);

const empty = ref<boolean>(false);
const error = ref<boolean>(false);
const showField = ref<boolean>(false);
const timeOpened = ref<TimeOpened>(null);
let originalValue = modelValue.value;


// ------------------------------------------------ Loading //
watch(() => loadingProp.value, (newVal, oldVal) => {
	if (!newVal && oldVal && showField.value) {
		toggleField();
	}
});


// ------------------------------------------------ The displayed value //
const displayValue = computed(() => {
	if (modelValue.value) {
		empty.value = false;

		if (settings.truncateLength) {
			return useTruncateText({
				length: settings.truncateLength,
				suffix: settings.truncateSuffix,
				text: modelValue.value as string,
			});
		}

		return modelValue.value;
	}

	empty.value = true;
	return settings.emptyText;
});


// ------------------------------------------------ Binding Events & Props //
const slotBindings = computed(() => ({
	...settings,
	...{
		loading: loadingProp.value,
		modelValue: modelValue.value,
		originalValue: originalValue,
	}
}));

const bindingDisplay = computed(() => {
	return {
		color: settings.color,
		displayAppendIcon: props.displayAppendIcon,
		displayAppendIconColor: props.displayAppendIconColor,
		displayAppendIconSize: props.displayAppendIconSize,
		displayAppendInnerIcon: props.displayAppendInnerIcon,
		displayAppendInnerIconColor: props.displayAppendInnerIconColor,
		displayAppendInnerIconSize: props.displayAppendInnerIconSize,
		displayPrependIcon: props.displayPrependIcon,
		displayPrependIconColor: props.displayPrependIconColor,
		displayPrependIconSize: props.displayPrependIconSize,
		displayPrependInnerIcon: props.displayPrependInnerIcon,
		displayPrependInnerIconColor: props.displayPrependInnerIconColor,
		displayPrependInnerIconSize: props.displayPrependInnerIconSize,
		displayValue: displayValue.value,
		empty: empty.value,
		error: error.value,
		field: 'v-text-field',
		underlineColor: settings.underlineColor,
		underlineStyle: settings.underlineStyle,
		underlineWidth: settings.underlineWidth,
		underlined: settings.underlined,
		valueColor: settings.valueColor,
	};
});

const bindingCard = computed(() => ({
	...defaultCardProps,
	...props.cardProps,
}) as SharedProps['cardProps']);


// ------------------------------------------------ Class & Styles //
const inlineFieldsContainerClass = computed(() => useInlineFieldsContainerClass({
	density: settings.density,
	disabled: disabledProp.value,
	field: 'v-text-field',
	iconSet: iconOptions?.defaultSet,
	loading: loadingProp.value,
	loadingWait: settings.loadingWait,
	tableField: settings.tableField,
	variant: settings.variant,
}));

const displayContainerClass = computed(() => useDisplayContainerClass({
	density: settings.density,
	field: 'v-text-field',
}));

const displayInputControlClasses = useDisplayInputControlClasses({
	density: settings.density,
	variant: settings.variant,
});

const fieldContainerClass = computed(() => useFieldContainerClass({
	active: showField.value,
	name: 'text-field',
}));

const cardContainerClass = computed(() => useCardContainerClass({
	name: 'custom-field',
	showField: showField.value,
}));

const inlineFieldsContainerStyle = computed(() => useInlineFieldsContainerStyle());
const cardContainerStyle = computed(() => fieldCoordinates.value);


// ------------------------------------------------ Key event to cancel/close field //
function closeField() {
	error.value = false;
	modelValue.value = originalValue;
	toggleField();
}


// ----------------------------------------------- Card Field//
const fieldCoordinates = ref<CSSProperties>();
const inlineFieldsContainer = ref<HTMLElement | null>(null);
const cardFieldRef = ref<HTMLElement | string | null>('body');


// ------------------------------------------------ Toggle the field //
function toggleField() {
	if (disabledProp.value || (settings.loadingWait && loadingProp.value)) {
		return;
	}

	fieldCoordinates.value = useCardContainerStyle({
		cardMinWidth: settings.cardProps?.minWidth,
		cardOffsetX: settings.cardOffsetX,
		cardOffsetY: settings.cardOffsetY,
		cardWidth: settings.cardProps?.width,
		field: inlineFieldsContainer.value,
	});

	const response = useToggleField({
		attrs,
		closeSiblings: settings.closeSiblings,
		fieldOnly: settings.fieldOnly,
		props,
		showField,
		timeOpened: timeOpened.value,
	});

	settings = { ...settings, ...response.settings };
	showField.value = response.showField;
	timeOpened.value = response.timeOpened;

	if (closeSiblingsBus !== null && settings.closeSiblings && showField.value && !settings.fieldOnly) {
		closeSiblingsBus.emit(response.timeOpened);
	}
}


// ------------------------------------------------ Check for errors //
const internalErrors = ref();

watch(() => showField.value, () => {
	if (showField.value) {
		checkInternalErrors();
	}
});

watch(() => modelValue.value, () => {
	if (showField.value) {
		checkInternalErrors();
	}
});

function checkInternalErrors() {
	const response = useCheckForErrors({
		required: settings.required,
		rules: settings.rules,
		value: modelValue,
	});

	error.value = response.errors;

	internalErrors.value = response.results;
	return response.results;
}


// ------------------------------------------------ Save the value / Emit update //
function saveValue() {
	if (error.value) {
		error.value = true;
		return;
	}

	originalValue = modelValue.value;
	emit('update', modelValue.value);

	if (!settings.loadingWait) {
		toggleField();
	}
}


// ------------------------------------------------ Close siblings bus event //
// eslint-disable-next-line @typescript-eslint/no-explicit-any
let closeSiblingsBus: unknown | any;
let unsubscribeBus: () => void;

if (settings.closeSiblings) {
	import('@vueuse/core').then(({ useEventBus }) => {
		closeSiblingsBus = useEventBus(CloseSiblingsBus);
		unsubscribeBus = closeSiblingsBus.on(closeSiblingsListener);
	});
}

function closeSiblingsListener(identifier: TimeOpened) {
	emit('update:closeSiblingFields', timeOpened);

	if (showField.value && timeOpened.value !== identifier) {
		closeField();
	}
}

onUnmounted(() => {
	if (typeof unsubscribeBus !== 'undefined') {
		closeSiblingsBus.off(closeSiblingsListener);
	}
});
</script>

<style lang="scss" scoped>
</style>
