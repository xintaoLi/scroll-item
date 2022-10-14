<template>
  <div ref="refScrollWrapper"
    :style="localWrapperStyle">
    <div ref="refScrollInner"
      :style="localInnerStyle">
      <span v-for="item in localList"
        :key="item.uuid"
        :style="localChildStyle">
        <slot name="child"
          :item="item" />
      </span>
    </div>
    <div :style="imgContainerStyle">
      <slot name="img"></slot>
    </div>
    <div :style="localPreStyle"
      @click="handlePreBtnClick">
      <slot name="pre">
        {{ "<" }}
      </slot>
    </div>
    <div :style="localNextStyle"
      @click="handleNextBtnClick">
      <slot name="next">
        {{ '>' }}
      </slot>
    </div>
  </div>
</template>
<script>
export default {
  props: {
    list: {
      type: Array,
      default: () => ([])
    },

    wrapperStyle: {
      type: Object,
      default: () => ({})
    },

    innerStyle: {
      type: Object,
      default: () => ({})
    },

    preStyle: {
      type: Object,
      default: () => ({})
    },

    nextStyle: {
      type: Object,
      default: () => ({})
    },

    childStyle: {
      type: Object,
      default: () => ({})
    },

    /**
     * Pre & Next disabled style
     */
    disabledStyle: {
      type: Object,
      default: () => ({})
    },

    /**
     * 行高
     */
    lineHeight: {
      type: Number,
      default: 48
    },

    /**
     * 上一页、下一页按钮宽度
     */
    preNextBtnWidth: {
      type: Number,
      default: 24
    },

    /**
     * 动画执行时间
     */
    duration: {
      type: Number,
      default: 160
    },

    // 点击上一步或者下一步，每次点击移动的步长
    // 如果是number类型，则会按照对应的px值进行位移
    // 可以设置为 childNode,则会按照子项长度进行位移
    stepSize: {
      type: [Number, String],
      default: 50
    },

    /**
     * 如果stepSize设置childNode
     * 需要设置childSelector，用来作为childNodes选择器
     */
    childSelector: {
      type: String,
      default: 'child'
    },

    /**
     * 是否需要图片占位区域
     * 可以设置为bool，true 开启，false 关闭
     * 可以设置为对象, width & height 值为number
     * 默认配置：{ width: 24 }
     */
    imgContainer: {
      type: [Boolean, Object],
      default: true
    }
  },
  data() {
    return {
      offsetX: 0, // x轴偏移量
      scrollXWidth: 0, // x轴可滚动宽度
      stepSizeValue: 0, // 每次点击可偏移量
      isRunning: false, // 标识是否正在执行动画
      runningCount: 0, // 动画执行次数
      totalCount: 10 // 动画总计可执行次数，可以通过调整此数值和props.duration调整动画
    };
  },
  computed: {
    baseStyle() {
      return {
        position: 'absolute',
        top: 0,
        bottom: 0,
        width: `${this.preNextBtnWidth}px`,
        background: '#fff',
        display: 'flex',
        alignItems: 'center',
      }
    },
    localList() {
      return this.list.map(item => ({ ...item, uuid: this.generateId() }));
    },
    /**
     * 向前位移是否可用
     */
    isPreBtnEnable() {
      return this.scrollXWidth > 0 && this.offsetX > 0;
    },

    /**
     * 向后位移是否可用
     */
    isNextBtnEnable() {
      return this.scrollXWidth > 0 && this.offsetX < this.scrollXWidth;
    },

    localDisabledStyle() {
      return {
        ...this.disabledStyle
      };
    },

    resolvedImgContainer() {
      if (typeof this.imgContainer === 'boolean') {
        if (this.imgContainer) {
          return { 
            width: 24, 
          };
        }

        return {
          width: 0,
          height: 0
        }
      }

      return this.imgContainer;
    },

    imgContainerStyle() {
      const { width, height } = this.resolvedImgContainer;
       return {
        ...this.baseStyle,
        right: `${this.preNextBtnWidth}px`,
        width: `${width}px`,
        height: `${height || this.lineHeight}px`,
       }
    },

    localWrapperStyle() {
      const paddingRight = `${this.resolvedImgContainer.width + this.preNextBtnWidth}px`;
      return {
        position: 'relative',
        overflow: 'hidden',
        width: '100%',
        boxSizing: 'border-box',
        padding: `0 ${this.preNextBtnWidth}px`,
        height: `${this.lineHeight}px`,
        lineHeight: `${this.lineHeight}px`,
        paddingRight,
        ...this.wrapperStyle
      };
    },

    localInnerStyle() {
      return {
        ...this.baseStyle,
        height: `${this.lineHeight}px`,
        transform: `translate(${-this.offsetX}px, 0)`,
        left: `${this.preNextBtnWidth}px`,
        width: 'auto',
        whiteSpace: 'nowrap',
        ...this.innerStyle
      };
    },

    localPreStyle() {
      return {
        ...this.baseStyle,
        left: 0,
        justifyContent: 'center',
        ...this.preStyle,
        ...(this.isPreBtnEnable ? {} : this.localDisabledStyle),
        cursor: this.isPreBtnEnable ? 'pointer' : 'not-allowed',
      };
    },

    localNextStyle() {
      return {
        ...this.baseStyle,
        right: 0,
        justifyContent: 'center',
        ...this.nextStyle,
        ...(this.isNextBtnEnable ? {} : this.localDisabledStyle),
        cursor: this.isNextBtnEnable ? 'pointer' : 'not-allowed',
      };
    },

    localChildStyle() {
      return {
        display: 'inline-flex',
        flex: 1,
        ...this.childStyle
      };
    },

    stepDuration() {
      return this.duration / this.totalCount;
    },
  },
  watch: {
    list: {
      handler() {
        this.$nextTick(() => {
          this.computeStyle();
        });
      },
      deep: true,
      immediate: true
    }
  },
  methods: {
    /**
     * 执行动画
     * fn: 动画回调函数
     * 实现transform 动画
     */
    runAnimation(fn) {
      this.runningCount ++;
      this.isRunning = true;
      fn();
      if (this.runningCount < this.totalCount) {
        setTimeout(()=> {
          this.runAnimation(fn);
        }, this.stepDuration);
      } else {
        this.runningCount = 0;
        this.isRunning = false;
      }
    },
    generateId(prefix = '', length = 6) {
      let d = new Date().getTime();
      const uuid = new Array(length).fill('x').join('').replace(/[xy]/g, c => {
        const r = (d + Math.random() * 16) % 16 | 0;
        d = Math.floor(d / 16);
        return (c === 'x' ? r : (r & 0x7) | 0x8).toString(16);
      });
      return `${prefix}${uuid}`;
    },
    handlePreBtnClick() {
      if (this.isRunning) {
        return;
      }
      if(this.isPreBtnEnable) {
        // 抛出当前实例，方便外层组件获取相关参数进行样式调整
        this.$emit('pre-click', { scope: this });
        this.runAnimation(() => {
          this.offsetX -= this.stepSizeValue / this.totalCount;
          if (this.offsetX < 0) {
            this.offsetX = 0;
          }
        });
      }
    },
    handleNextBtnClick() {
      if (this.isRunning) {
        return;
      }
      if (this.isNextBtnEnable) {
        // 抛出当前实例，方便外层组件获取相关参数进行样式调整
        this.$emit('next-click', { scope: this });
        this.runAnimation(() => {
          this.offsetX += this.stepSizeValue / this.totalCount;
          if (this.offsetX > this.scrollXWidth) {
            this.offsetX = this.scrollXWidth;
          }
        });
      }
    },

    /**
     * 计算相关样式
     * 如果外层改变list样式计算不对，可在外层调用此方法
     */
    computeStyle() {
      const wrapper = this.$refs.refScrollWrapper;
      const inner = this.$refs.refScrollInner;

      const wrapperStyle = window.getComputedStyle(wrapper);
      const paddingWidth = ['padding-left', 'padding-right'].reduce((width, prop) => {
        return width + Number(wrapperStyle.getPropertyValue(prop).replace('px', ''));
      }, 0);

      const wrapperWidth = wrapper.clientWidth - paddingWidth;
      const innerWidth = inner.clientWidth;

      this.scrollXWidth = innerWidth > wrapperWidth ? (innerWidth - wrapperWidth) : 0;

      if (typeof this.stepSize === 'number') {
        this.stepSizeValue = this.stepSize;
      }

      if (this.stepSize === 'childNode') {
        let childLength = inner.childNodes.length || 1;
        if (this.itemSelector) {
          const childNodes = inner.querySelector(this.childSelector);
          childLength = childNodes ? childNodes.length : 1;
        }
        this.stepSizeValue = this.scrollXWidth / childLength;
      }
    }
  }
};
</script>