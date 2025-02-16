<template>
  <div class="relative flex w-full flex-1 flex-col overflow-x-auto">
    <div
      class="flex w-max min-w-full flex-col overflow-y-hidden"
      :class="$attrs.class"
    >
      <slot v-bind="{ showGroupedRows, selectable }">
        <ListHeader />
        <template v-if="props.rows.length">
          <ListGroups v-if="showGroupedRows" />
          <ListRows v-else />
        </template>
        <ListEmptyState v-else />
        <ListSelectBanner v-if="selectable" />
      </slot>
    </div>
  </div>
</template>
<script setup>
import ListEmptyState from './ListEmptyState.vue'
import ListHeader from './ListHeader.vue'
import ListRows from './ListRows.vue'
import ListGroups from './ListGroups.vue'
import ListSelectBanner from './ListSelectBanner.vue'
import { reactive, computed, provide, watch } from 'vue'

defineOptions({
  inheritAttrs: false,
})

const props = defineProps({
  columns: {
    type: Array,
    default: [],
  },
  rows: {
    type: Array,
    default: [],
  },
  rowKey: {
    type: String,
    required: true,
  },
  options: {
    type: Object,
    default: () => ({
      getRowRoute: null,
      onRowClick: null,
      showTooltip: true,
      selectable: true,
      resizeColumn: false,
      rowHeight: 40,
      emptyState: {
        title: 'No Data',
        description: 'No data available',
      },
    }),
  },
})

let selections = reactive(new Set())

const emit = defineEmits(['update:selections'])

watch(selections, (value) => {
  emit('update:selections', value)
})

let _options = computed(() => {
  function defaultTrue(value) {
    return value === undefined ? true : value
  }

  function defaultFalse(value) {
    return value === undefined ? false : value
  }

  return {
    getRowRoute: props.options.getRowRoute || null,
    onRowClick: props.options.onRowClick || null,
    showTooltip: defaultTrue(props.options.showTooltip),
    selectable: defaultTrue(props.options.selectable),
    resizeColumn: defaultFalse(props.options.resizeColumn),
    rowHeight: props.options.rowHeight || 40,
    emptyState: props.options.emptyState,
  }
})

const allRowsSelected = computed(() => {
  if (!props.rows.length) return false
  return selections.size === props.rows.length
})

const selectable = computed(() => {
  return _options.value.selectable
})

let showGroupedRows = computed(() => {
  return props.rows.every((row) => row.group)
})

function toggleRow(row) {
  if (!selections.delete(row)) {
    selections.add(row)
  }
}

function toggleAllRows(select) {
  if (!select || allRowsSelected.value) {
    selections.clear()
    return
  }
  props.rows.forEach((row) => selections.add(row[props.rowKey]))
}

provide(
  'list',
  computed(() => ({
    rowKey: props.rowKey,
    rows: props.rows,
    columns: props.columns,
    options: _options.value,
    selections: selections,
    allRowsSelected: allRowsSelected.value,
    toggleRow,
    toggleAllRows,
  }))
)
</script>
