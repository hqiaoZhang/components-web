<template  >
  
    <div>
      <el-dialog title="发布" :visible.sync="$parent.publishPopVisible" width="50%">
        <div style="margin-bottom: 16px;"> 发布成功！当前图表的公开链接为：</div>
        <el-input v-model="publicUrl" readonly />
        <span slot="footer">
          <el-button type="primary" @click="handleCopyUrl">复制链接</el-button>
        </span>

        
      </el-dialog>
      <div class="edit_view"  tabindex="0"
        @keydown.space.prevent="handleSpaceDown"
        @keyup.space.prevent="handleSpaceUp"
        @click.self="handleActivated(-1)">
        <vue-draggable-resizable   :style="wrapStyle"
          :x="100"
          :y="50"
          :w="chartData.w"
          :h="chartData.h"
          class-name="screen_box"
          class-name-draggable="screen_box_draggable"
          :draggable="screenDraggable"
          :resizable="false">
          <div class="screen" :style="screenStyle" @click.self="handleActivated(-1)" ref="screen">
            <vue-drag-resize v-for="(item, index) in chartData.elements"
            :key="index"
            :isActive="item.active && !$parent.preview"
            :parentScaleX="scale"
            :parentScaleY="scale"
            :x="item.x"
            :y="item.y"
            :w="item.w"
            :h="item.h"
            :parentLimitation="true"
            :parentW="chartData.w"
            :parentH="chartData.h"
            :aspectRatio="false"
            :minw="20"
            :minh="20"
            :z="chartData.elements.length - index"
            :isDraggable="!$parent.preview"
            :isResizable="!$parent.preview"
            @activated="handleActivated(index)"
            @resizing="handleResize(item, arguments[0])"
            @dragging="handleDrag(item, arguments[0])"> 
              <div class="filler" v-if="item.data.type == 'chart'"
                :style="{width: '100%', height: '100%', backgroundColor: item.bgcolor}">
                <ve-map  v-if="item.data.settings.type=='map'"
                  :width="item.w + 'px'"
                  :height="item.h + 'px'"
                  :data="item.data.generated"
                  :settings="item.data.settings"
                  @ready-once="generateData(item)"> 
                </ve-map>
                <ve-liquidfill v-else-if="item.data.settings.type=='liquidfill'"
                  :width="item.w + 'px'"
                  :height="item.h + 'px'"
                  :data="item.data.generated"
                  @ready-once="generateData(item)"> 
                </ve-liquidfill>
                <ve-chart v-else
                :width="item.w + 'px'"
                :height="item.h + 'px'"
                :data="item.data.generated"
                :settings="item.data.settings"
                @ready-once="generateData(item)"> 
                </ve-chart>
              </div>

              <div class="filler" v-if="item.data.type == 'text'"
                :style="{width: '100%', height: '100%', backgroundColor: item.bgcolor}">
                <div class="textcontainer"  
                  :style="{fontFamily: item.data.datacon.fontFamily, 
                  fontWeight: item.data.datacon.bold ? 'bold' : 'normal', 
                  fontStyle: item.data.datacon.italic ? 'italic' : 'normal', 
                  color: item.data.datacon.color, 
                  fontSize: item.data.datacon.fontSize + 'px', 
                  textStroke: item.data.datacon.stroke ? item.data.datacon.strokeSize+'px '+item.data.datacon.strokeColor : '0', 
                  textShadow: item.data.datacon.shadow ? '5px 5px '+item.data.datacon.shadowBlur+'px '+item.data.datacon.shadowColor : 'none'}"
                  v-text="item.data.datacon.text"> 
                </div>
              </div>

              <div class="filler"  v-if="item.data.type == 'image'"
                :style="{width: '100%', height: '100%', backgroundColor: item.bgcolor}">
                <div class="imagecontainer" 
                  :style="{backgroundImage: `url(${item.data.datacon.img})`, 
                  backgroundSize: item.data.datacon.imgSize, 
                  backgroundRepeat: item.data.datacon.repeat,  
                  opacity: item.data.datacon.opacity}">
                  <div class="placeholder" v-show="!item.data.datacon.img"></div>
                </div>
              </div>

              <div class="filler" v-if="item.data.type == 'border'"
                :style="{width: '100%', height: '100%', backgroundColor: item.bgcolor}">
                <div class="bordercontainer" :class="'border' + item.data.datacon.borderId"
                :style="{opacity: item.data.datacon.opacity}"> 
                </div>
              </div> 
            </vue-drag-resize>
            <div class="mock" :class="{front: screenDraggable}"></div>
          </div>
        </vue-draggable-resizable>
      </div>
      <input id="input" style="opacity: 0; height: 1px; display: none" />        
    </div>
</template>

<script>
 import { messagePopup } from '@/utils/util' 
export default {
  props: ['scale'],
  data() {
    return {
      screenDraggable: false,
      timer: null
    };
  },
  computed: {
    publicUrl() {
      return `http://${window.location.host}${window.location.pathname}#/view/${this.$route.params.id}`;
    },
    chartData() {
      return this.$parent.chartData;
    },
    wrapStyle() {
      return {
        transform: `scale(${this.scale})`,
      };
    },
    screenStyle() {
      return {
        backgroundColor: this.chartData.bgcolor,
        backgroundImage: `url(${this.chartData.bgimage})`,
        backgroundSize: this.chartData.bgimagesize,
      };
    },
  },
  methods: {
    handleCopyUrl() {
       var input = document.getElementById("input");
      input.value = this.publicUrl; // 修改文本框的内容
      input.select(); // 选中文本
      document.execCommand("copy"); // 执行浏览器复制命令 
       messagePopup('复制成功', 'success')
       clearTimeout(this.timer)
      this.timer = setTimeout(() => {
          this.$parent.publishPopVisible = false
      }, 1000)
    },
    handleSpaceDown() {
      this.screenDraggable = true;
    },
    handleSpaceUp() {
      this.screenDraggable = false;
    },
    handleActivated(index) {
      this.$parent.setActiveComponentByIndex(index);
    },
    handleResize(widget, arg) { 
      const item = widget;
      item.x = arg.left;
      item.y = arg.top;
      item.w = arg.width;
      item.h = arg.height;
    },
    handleDrag(widget, arg) {
      const item = widget;
      item.x = arg.left;
      item.y = arg.top;
    },
    generateData(item) {
      this.$parent.generateData(item);
    }
  },
};
</script>

<style lang="scss" scoped>
.edit_view {
  position: relative;
  width: 100%;
  height: 100%;
  box-sizing: border-box;
  overflow: visible;
  outline: 0;
}

.screen_box {
  // width: 1220px;
  // height: 400px;
  top: 16px;
  left: 16px;
  position: relative;
  background: #ffffff;
  transform-origin: 0 0;
  box-shadow: 0px 4px 8px rgba(203, 219, 234, 0.3);
  transition: transform 0.5s ease;

  &.screen_box_draggable {
    cursor: grab;
  }
}

.screen {
  position: relative;
  width: 100%;
  height: 100%;
  .vdr {
    border: 0;
    &.active  {
      cursor: move;
    }
  }
  .filler {
    .textcontainer {
      word-wrap: break-word;
    }
    .imagecontainer {
      width: 100%;
      height: 100%;
      .placeholder {
        width: 100%;
        height: 100%;
        background: rgba(0, 0, 0, 0.3);
      }
    }
    .bordercontainer {
      width: 100%;
      height: 100%;
      box-sizing: border-box;
      &.border1 {
        border: 50px solid transparent;
        // border-image: url('@/assets/img/borders/1.png') 50;
      }
      &.border2 {
        border: 50px solid transparent;
        // border-image: url('@/assets/img/borders/2.png') 50;
      }
      &.border3 {
        border: 50px solid transparent;
        // border-image: url('@/assets/img/borders/3.png') 50;
      }
    }
  }
}

.mock {
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  bottom: 0;
  background: rgba(255, 255, 255, 0.1);
  z-index: -1;

  &.front {
    z-index: 999;
  }
}
</style>
