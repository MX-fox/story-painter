<template>
  <div v-show="isShow">
    <n-tooltip class="box-item" placement="top-start">
      <template #trigger>
        <n-checkbox label="对单次发言，从次行进行空格缩进" style="margin-bottom: 0.5rem;"
                     v-model:checked="store.bbsUseSpaceWithMultiLine" />
      </template>
      注意: 预览中看起来错位的原因是扣去了论坛代码的宽度。但实际能否对其仍取决于论坛使用的字体
    </n-tooltip>

    <n-tooltip class="box-item" placement="top-start">
      <template #trigger>
        <n-checkbox label="使用颜色名，而非16进制颜色码" style="margin-bottom: 0.5rem;"
                    v-model:checked="store.bbsUseColorName" />
      </template>
      注意: 这种模式下会随机分配部分颜色，因为无法完成与hex色彩的对应
    </n-tooltip>
  </div>

  <div class="preview" ref="preview" id="preview" v-show="isShow">
    <div style="position: absolute; right: 2rem; direction: rtl;">
      <n-button secondary type="primary" @click="copied" id="btnCopyPreviewBBS" style="z-index: 100">一键复制</n-button>
      <div class="mt-0.5 text-xs">注意: 长文本速度较慢</div>
    </div>

    <div v-if="previewItems.length === 0">
      <div>染色失败，内容为空或无法识别此格式。</div>
      <div>已知支持的格式有: 海豹Log(json)、赵/Dice!原始文件、塔原始文件</div>
      <div>请先清空编辑框，再行复制</div>
    </div>

    <VirtualList class="list-dynamic scroll-touch scroller" :data-key="'index'" :data-sources="previewItems"
      :data-component="Item" :estimate-size="20" :item-class="''" />
  </div>
</template>

<script setup lang="ts">
import dayjs from 'dayjs';
import ClipboardJS from 'clipboard';
import { h, nextTick, onMounted, render, watch } from 'vue';
import { useStore } from '~/store';
import { LogItem, packNameId } from '~/logManager/types';
// @ts-ignore
import VirtualList from 'vue3-virtual-scroll-list';
import Item from './preview-bbs-item.vue'
import { useMessage } from 'naive-ui';

const props = defineProps<{
  isShow: boolean,
  previewItems: LogItem[],
}>();

const store = useStore();
const message = useMessage();

const copied = () => {
  message.success('进行了复制！')
}

onMounted(() => {
})

let clip: ClipboardJS;

watch(() => props.isShow, (val: any) => {
  if (val) {
    store.exportOptions.imageHide = true

    nextTick(() => {
      if (clip) return;
      clip = new ClipboardJS('#btnCopyPreviewBBS', {
        text: () => {
          // 这个实现很好，很简单，可惜太慢
          // 先用着吧
          const el = document.createElement('span');
          const items = [];
          for (let i of props.previewItems) {
            const html = h(Item, { source: i });
            render(html, el);
            items.push(el.textContent);
          }
          return items.join('\n');
        }
      })
    });
  }
})

</script>
