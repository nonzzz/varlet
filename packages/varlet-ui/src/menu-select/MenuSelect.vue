<template>
  <var-menu
    ref="menu"
    tabindex="-1"
    :class="n()"
    :disabled="disabled"
    :trigger="trigger"
    :reference="reference"
    :placement="placement"
    :strategy="strategy"
    :offset-x="offsetX"
    :offset-y="offsetY"
    :teleport="teleport"
    :same-width="sameWidth"
    :elevation="elevation"
    :default-style="false"
    :popover-class="popoverClass"
    :close-on-click-reference="closeOnClickReference"
    :close-on-key-escape="false"
    v-model:show="show"
    @open="onOpen"
    @opened="onOpened"
    @close="onClose"
    @closed="onClosed"
    @click-outside="onClickOutside"
  >
    <slot />

    <template #menu>
      <div
        ref="menuOptionsRef"
        :class="classes(n('menu'), formatElevation(elevation, 3), [scrollable, n('--scrollable')])"
      >
        <template v-if="options.length">
          <var-menu-option
            v-for="option in options"
            :key="option[valueKey]"
            :label="option[labelKey]"
            :value="option[valueKey]"
            :option="option"
            :ripple="option.ripple"
            :disabled="option.disabled"
          />
        </template>
        <slot name="options" />
      </div>
    </template>
  </var-menu>
</template>

<script lang="ts">
import VarMenu from '../menu'
import VarMenuOption from '../menu-option'
import { defineComponent, computed, ref } from 'vue'
import { props } from './props'
import { createNamespace, formatElevation } from '../utils/components'
import { useMenuOptions, type MenuSelectProvider } from './provide'
import { useSelectController } from '../select/useSelectController'
import { type MenuOptionProvider } from '../menu-option/provide'
import { call, preventDefault } from '@varlet/shared'
import { useEventListener, useVModel } from '@varlet/use'
import { focusChildElementByKey } from '../utils/elements'

const { name, n, classes } = createNamespace('menu-select')

export default defineComponent({
  name,
  components: { VarMenu, VarMenuOption },
  props,
  setup(props) {
    const menu = ref<null | typeof VarMenu>(null)
    const menuOptionsRef = ref<null | HTMLElement>(null)
    const show = useVModel(props, 'show')
    const { menuOptions, length, bindMenuOptions } = useMenuOptions()
    const { computeLabel, getSelectedValue } = useSelectController({
      modelValue: () => props.modelValue,
      multiple: () => props.multiple,
      optionProviders: () => menuOptions,
      optionProvidersLength: () => length.value,
    })

    const menuSelectProvider: MenuSelectProvider = {
      size: computed(() => props.size),
      multiple: computed(() => props.multiple),
      computeLabel,
      onSelect,
    }

    bindMenuOptions(menuSelectProvider)

    useEventListener(() => window, 'keydown', handleKeydown)

    function onSelect(option: MenuOptionProvider) {
      const { multiple, closeOnSelect } = props

      call(props['onUpdate:modelValue'], getSelectedValue(option))

      if (!multiple && closeOnSelect) {
        menu.value!.$el.focus()
        close()
      }
    }

    function handleKeydown(event: KeyboardEvent) {
      if (props.disabled || !show.value) {
        return
      }

      const { key } = event

      if (['Escape', 'ArrowDown', 'ArrowUp'].includes(key)) {
        preventDefault(event)
      }

      if (key === 'Escape') {
        menu.value!.$el.focus()
        close()
        return
      }

      if (key === 'ArrowDown' || key === 'ArrowUp') {
        focusChildElementByKey(menu.value!.$el, menuOptionsRef.value!, key)
      }
    }

    // expose
    function open() {
      menu.value?.open()
    }

    // expose
    function close() {
      menu.value?.close()
    }

    // expose
    function resize() {
      menu.value?.resize()
    }

    return {
      show,
      menu,
      menuOptionsRef,
      n,
      classes,
      formatElevation,
      open,
      close,
      resize,
    }
  },
})
</script>

<style lang="less">
@import '../styles/elevation';
@import '../styles/common';
@import '../menu/menu';
@import '../menu-option/menuOption';
@import './menuSelect';
</style>
