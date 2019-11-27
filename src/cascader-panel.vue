<template>
  <span>
    <ul v-if="hasData" class="enhance-mul-cascader-menu">
      <CascaderItem
        v-for="(item, idx) of data"
        :key="idx"
        :item="item"
        :multiple="multiple"
        :disabled="disabled"
        :only-leaf="onlyLeaf"
        :prop-alias="propAlias"
        :allow-select-by-parent-node="allowSelectByParentNode"
        @click.native.stop="handleClickItem(item, $event)"
        @mouseenter.native.stop="handleHoverItem(item)"
      />
    </ul>
    <CascaderPanel
      v-if="hasSubList"
      :data="subList"
      :trigger="trigger"
      :disabled="disabled"
      :multiple="multiple"
      :only-leaf="onlyLeaf"
      :prop-alias="propAlias"
      :allow-select-by-parent-node="allowSelectByParentNode"
    />
  </span>
</template>

<script>
import CascaderItem from './cascader-item.vue'

export default {
  name: 'CascaderPanel',
  components: { CascaderItem },
  inject: ['rootProp'],
  props: {
    data: {
      type: Array,
      default: () => []
    },
    disabled: {
      type: Boolean,
      default: false
    },
    trigger: {
      type: String,
      default: 'hover'
    },
    multiple: {
      type: Boolean,
      default: true
    },
    onlyLeaf: {
      type: Boolean,
      default: true
    },
    allowSelectByParentNode: {
      type: Boolean,
      default: false
    },
    propAlias: {
      type: Object,
      default() {
        return {
          label: 'label',
          value: 'value',
          disabled: 'disabled',
          children: 'children'
        }
      }
    }
  },
  data() {
    return {
      subList: []
    }
  },
  computed: {
    hasData() {
      return !!(Array.isArray(this.data) && this.data.length)
    },
    hasSubList() {
      return !!(Array.isArray(this.subList) && this.subList.length)
    },
    childrenProp() {
      return this.propAlias.children
    },
    disabledProp() {
      return this.propAlias.disabled
    }
  },
  watch: {
    data() {
      this.subList = []
    }
  },
  methods: {
    handleClickItem(item) {
      if (this.trigger === 'click') {
        this.triggerItem(item)
      }
    },
    handleHoverItem(item) {
      if (this.trigger === 'hover') {
        this.triggerItem(item)
      }
    },
    isValidChildren(item) {
      return (
        Array.isArray(item[this.childrenProp]) && item[this.childrenProp].length
      )
    },
    triggerItem(item) {
      if (item[this.disabledProp] || this.disabled) return

      if (this.isValidChildren(item)) {
        this.subList = item[this.childrenProp]
      }
    }
  }
}
</script>

<style lang="less" scoped>
.enhance-mul-cascader-menu {
  display: inline-block;
  min-width: 97px;
  height: 180px;
  margin: 0;
  padding: 5px;
  vertical-align: top;
  list-style: none;
  border-right: 1px solid #e8eaec;
  overflow: auto;
  .enhance-mul-cascader-item {
    cursor: pointer;
  }
}
</style>