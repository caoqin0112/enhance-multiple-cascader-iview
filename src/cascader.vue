<template>
  <div
    v-if="!disabled"
    v-click-outside="() => switchDropdown(false)"
    class="enhance-mul-cascader"
  >
    <input type="hidden" :name="name" :value="publicValue" />
    <CascaderHead
      :filterable="filterable"
      :visible="visible"
      :multiple="multiple"
      :values="currentValue"
      :clearable="clearable"
      :disabled="disabled"
      :placeholder="placeholder"
      :max-tag-count="maxTagCount"
      :max-tag-placeholder="maxTagPlaceholder"
      :prop-alias="localPropAlias"
      :head-style="headStyle"
      :allow-delete-by-close-icon="allowDeleteByCloseIcon"
      @click.native.stop="() => switchDropdown(true)"
      @on-query-change="handleQuery"
      @on-input-focus="isFocused = true"
      @on-input-blur="isFocused = false"
      @on-show-dropdown="switchDropdown"
      @on-clear="clearSingleSelect"
    />
    <transition name="drop">
      <Dropdown v-show="visible" ref="drop" :transfer="transfer">
        <div
          :class="[
            'enhance-mul-cascader-dropdown',
            { 'no-data-wrapper': noData },
          ]"
        >
          <CascaderPanel
            v-if="currentData.length > 0"
            ref="panel"
            :data="currentData"
            :values="currentValue"
            :disabled="disabled"
            :trigger="trigger"
            :multiple="multiple"
            :only-leaf="onlyLeaf"
            :prop-alias="localPropAlias"
            :allow-select-by-parent-node="allowSelectByParentNode"
            :unique-field-in-leaf="uniqueFieldInLeaf"
          ></CascaderPanel>
          <div v-else class="no-data">{{ notFoundText }}</div>
        </div>
      </Dropdown>
    </transition>
  </div>
  <span v-else class="enhance-mul-cascader">{{ displayValue }}</span>
</template>

<script>
import { Dropdown } from 'iview'
import { directive as clickOutside } from 'v-click-outside-x'

import CascaderHead from './cascader-head.vue'
import CascaderPanel from './cascader-panel.vue'

const isObject = (obj) =>
  Object.prototype.toString.call(obj).slice(8, -1) === 'Object'
const defaultPropAlias = {
  label: 'label',
  value: 'value',
  disabled: 'disabled',
  children: 'children',
}

export default {
  name: 'EnhanceCascader',
  cnName: '增强的级联选择组件',
  components: {
    Dropdown,
    CascaderHead,
    CascaderPanel,
  },
  directives: { clickOutside },
  provide() {
    return {
      rootProp: this.rootProp,
    }
  },
  props: {
    disabled: {
      type: Boolean,
      default: false,
      __description: '是否被禁用',
    },
    value: {
      type: Array,
      default() {
        return []
      },
      __description: '双向绑定使用的值',
    },
    names: {
      type: Array,
      default() {
        return []
      },
      __description:
        '该属性用来校验传入的value的标签是否与data中的数据保持一致，如果不一致，则清空组件的选中内容。',
    },
    data: {
      type: Array,
      default() {
        return [
          // {
          //   label: '',
          //   value: '-1',
          //   children: [],
          // },
        ]
      },
      __description: '待使用的级联数据',
    },
    notFoundText: {
      type: String,
      default: '没有数据可用',
      __description: '当没有找到数据时，显示的文本内容',
    },
    propAlias: {
      type: Object,
      __description:
        '传递下来的data数据的属性的别名。有可能从服务端获取到的data树的属性名并不能符合当前使用中的属性名，所以做一次映射操作，以避免对服务端数据的处理，方便操作。',
    },
    onlyLeaf: {
      type: Boolean,
      default: true,
      __description: '仅仅只能选择叶子节点',
    },
    uniqueFieldInLeaf: {
      type: String,
      default: '',
      __description: '叶子节点独有的字段，用来标识是否为叶子节点',
    },
    allowSelectByParentNode: {
      type: Boolean,
      default: false,
      __description: '允许通过父节点来选择相应的子节点',
    },
    maxTagCount: {
      type: Number,
      default: -1,
      validator(value) {
        return Number.isInteger(value) && value >= -1
      },
      __description: '当选中多个项目时，是否对显示在输入框中的条目精简显示',
    },
    maxTagPlaceholder: {
      type: Function,
      default: (v) => `+ ${v}...`,
      __description: '当maxTagCount值设置后，选择超过指定数目的项后，级联器的输入值会精简显示为该方法返回的内容'
    },
    filterable: {
      type: Boolean,
      default: false,
      __description: '是否通过输入过滤级联数据',
    },
    clearable: {
      type: Boolean,
      default: false,
      __description: '是否可以通过清空输入框的方式删除全部选中的条目',
    },
    allowDeleteByCloseIcon: {
      type: Boolean,
      default: true,
      __description: '允许通过Tag的关闭按钮来删除相应选中的条目',
    },
    labelInValue: {
      type: Boolean,
      default: false,
      __description:
        '触发on-change事件时，返回的参数是否要被包装为传入的对象形式',
    },
    multiple: {
      type: Boolean,
      default: true,
      __description: '默认为多选级联',
    },
    transfer: {
      type: Boolean,
      default() {
        return !this.$IVIEW || this.$IVIEW.transfer === ''
          ? false
          : this.$IVIEW.transfer
      },
      __description: '下拉组件是否挂载到body元素上',
    },
    placeholder: {
      type: String,
      default: '请选择',
      __description: '未操作时的占位符',
    },
    name: {
      type: String,
      default: 'cascader',
      __description: '该值用在级联器的输入框中的name属性上'
    },
    headStyle: {
      type: Object,
      default: () => ({}),
      __description:
        '控制级联框的行内样式，多用来控制级联框宽度，默认宽度为240px',
    },
  },
  data() {
    return {
      currentValue: [],
      currentData: [],
      visible: false,
      isFocused: false,
    }
  },
  computed: {
    localPropAlias() {
      return isObject(this.propAlias)
        ? Object.assign({}, defaultPropAlias, this.propAlias)
        : defaultPropAlias
    },
    labelProp() {
      return this.localPropAlias.label
    },
    valueProp() {
      return this.localPropAlias.value
    },
    disabledProp() {
      return this.localPropAlias.disabled
    },
    childrenProp() {
      return this.localPropAlias.children
    },
    rootProp() {
      return {
        setCurrentValue: this.setCurrentValue,
      }
    },
    noData() {
      return this.data.length === 0
    },
    trigger() {
      if (this.multiple) {
        return 'hover'
      } else {
        return 'click'
      }
    },
    // 展平后的data数组
    flatData() {
      return this.flattenData()
    },
    publicValue() {
      return this.currentValue.map((item) => item[this.labelProp])
    },
    displayValue() {
      const isLabelInValue = this.value.every((item) => isObject(item))

      const labelList = isLabelInValue
        ? this.value.map((item) => item[this.labelProp])
        : this.publicValue

      return labelList.join('，')
    },
    isAlignedNameInValue() {
      return !!this.names.length && this.value.length === this.names.length
    },
    flattenItemList() {
      const list = []

      this.value.forEach((val, idx) => {
        const item = this.flatData.find((item) => item[this.valueProp] === val)

        if (
          item &&
          (!this.isAlignedNameInValue ||
            this.names[idx] === item[this.labelProp])
        ) {
          list.push(item)
        }
      })

      return list
    },
  },
  watch: {
    flattenItemList(newList) {
      if (this.flatData.length) {
        this.currentValue = newList
      }
    },
    data: {
      immediate: true,
      handler(newVal) {
        this.currentData = newVal
      },
    },
    currentValue(newItemList) {
      const newValueList = newItemList.map((item) => item[this.valueProp])

      // 如果是相同的值，则不触发事件
      if (this.value.join('') === newValueList.join('')) return

      // 对value属性做双向绑定处理
      this.$emit('update:value', newValueList)
      this.$emit('on-change', this.labelInValue ? newItemList : newValueList)
    },
  },
  methods: {
    handleQuery(query) {
      if (query.length) {
        const queriedData = this.queryData(query)

        this.currentData = queriedData
      } else {
        this.currentData = this.data
      }
    },
    switchDropdown(status) {
      this.visible = status
    },
    clearSingleSelect() {
      // 清理掉选择的内容
      this.currentValue = []
    },
    // list: 待处理的数组
    // action: true:新增；false：删除
    setCurrentValue(list, action) {
      list.forEach((item) => {
        const idx = this.currentValue.findIndex(
          (ite) => ite[this.valueProp] === item[this.valueProp]
        )
        if (action && idx === -1) {
          this.currentValue.push(item)
        } else if (!action && idx > -1) {
          this.currentValue.splice(idx, 1)
        } else {
          throw new Error('错误的参数')
        }
      })
    },
    queryData(query, list) {
      const result = []
      const data = Array.isArray(list) ? list : this.data
      data.forEach((item) => {
        if (item[this.labelProp].includes(query)) {
          result.push(item)
        } else if (
          Array.isArray(item[this.childrenProp]) &&
          item[this.childrenProp].length
        ) {
          const subResult = this.queryData(query, item[this.childrenProp])
          result.push(...subResult)
        } else {
          // 无效的项，不做任何操作
        }
      })
    },
    flattenData(list) {
      const data = Array.isArray(list) ? list : this.data
      const result = []

      data.forEach((item) => {
        result.push(item)

        if (
          Array.isArray(item[this.childrenProp]) &&
          item[this.childrenProp].length
        ) {
          const subResult = this.flattenData(item[this.childrenProp])

          result.push(...subResult)
        }
      })

      return result
    },
  },
}
</script>

<style lang="less" scoped>
.enhance-mul-cascader {
  display: inline-block;
  width: max-content;
  min-width: 240px;
  position: relative;
  .ivu-dropdown {
    display: block;
    width: max-content;
    position: absolute;
    z-index: 999;
    width: 100%;
  }
  .enhance-mul-cascader-dropdown {
    min-width: 100%;
    border: 1px solid #dcdee2;
    border-radius: 4px;
    border-right: 0 none;
    background-color: #fff;
    width: max-content;
    &.no-data-wrapper {
      border-right: 1px solid #dcdee2;
    }
  }
  .no-data {
    text-align: center;
    height: 50px;
    line-height: 50px;
    color: #333;
    // background: #f8f8f8;
  }
}
.drop-enter-active {
  // max-height: 180px;
  // transition: max-height 0.15s ease-in;
  animation: drop-down-enter 0.26s ease-in;
}
.drop-leave-active {
  // max-height: 0;
  // transition: max-height 0.15s ease-out;
  // overflow: hidden;
  animation: drop-down-leave 0.26s ease-out;
}
@keyframes drop-down-enter {
  from {
    transform-origin: top;
    // transform: translateY(-100%);
    transform: scaleY(0);
    opacity: 0;
  }
  to {
    transform-origin: top;
    // transform: translateY(0);
    transform: scaleY(1);
    opacity: 1;
  }
}
@keyframes drop-down-leave {
  from {
    transform-origin: top;
    // transform: translateY(0);
    transform: scaleY(1);
    opacity: 1;
  }
  to {
    transform-origin: top;
    // transform: translateY(-100%);
    transform: scaleY(0);
    opacity: 0;
  }
}
</style>
