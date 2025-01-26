<script lang="tsx" setup>
import type { UploadFileInfo } from 'naive-ui';

import { h, ref } from 'vue';

import { NButton, NCalendar, NCard, NTag, NTooltip, NUpload } from 'naive-ui';

// 修改文件列表类型
interface FileInfo extends UploadFileInfo {
  uploadTime?: number;
}

// 修改文件列表的类型
const fileList = ref<FileInfo[]>([]);

// 修改日历事件数据的类型
const calendarEvents = ref<Record<string, FileInfo[]>>({});

// 修改处理文件上传函数
const handleUpload = (options: {
  file: UploadFileInfo;
  fileList: UploadFileInfo[];
}) => {
  const fileInfo: FileInfo = {
    ...options.file,
    uploadTime: Date.now(),
  };
  fileList.value.push(fileInfo);
  return false; // 阻止默认上传行为
};

// 处理拖拽开始
const handleDragStart = (e: DragEvent, file: FileInfo) => {
  e.dataTransfer!.setData('fileId', file.id!);
};

// 处理拖拽放置
const handleDrop = (e: DragEvent, date: string) => {
  e.preventDefault();
  const fileId = e.dataTransfer!.getData('fileId');
  const file = fileList.value.find((f) => f.id === fileId);

  if (file) {
    // 创建新对象而不是修改原对象
    calendarEvents.value = {
      ...calendarEvents.value,
      [date]: [...(calendarEvents.value[date] || []), file],
    };
  }
};

// 允许放置
const handleDragOver = (e: DragEvent) => {
  e.preventDefault();
};

// 添加日期格式化辅助函数
const formatDate = (date: Date) => {
  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, '0');
  const day = String(date.getDate()).padStart(2, '0');
  return `${year}-${month}-${day}`;
};

// 格式化文件大小
const formatFileSize = (size: number) => {
  if (size < 1024) return `${size} B`;
  if (size < 1024 * 1024) return `${(size / 1024).toFixed(2)} KB`;
  if (size < 1024 * 1024 * 1024)
    return `${(size / (1024 * 1024)).toFixed(2)} MB`;
  return `${(size / (1024 * 1024 * 1024)).toFixed(2)} GB`;
};

// 格式化日期时间
const formatDateTime = (date: Date) => {
  return date.toLocaleString('zh-CN', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit',
  });
};

// 修改渲染文件信息表格函数
const renderFileInfo = (file: FileInfo) => {
  return h(
    'div',
    {
      class: 'min-w-[200px] text-sm',
    },
    [
      h('div', { class: 'mb-2 font-medium' }, '文件信息'),
      h('table', { class: 'w-full' }, [
        h('tbody', [
          h('tr', [
            h('td', { class: 'py-1 pr-4 text-gray-500' }, '名称：'),
            h('td', file.name),
          ]),
          h('tr', [
            h('td', { class: 'py-1 pr-4 text-gray-500' }, '大小：'),
            h('td', formatFileSize(file.file?.size || 0)),
          ]),
          h('tr', [
            h('td', { class: 'py-1 pr-4 text-gray-500' }, '上传时间：'),
            h('td', formatDateTime(new Date(file.uploadTime || Date.now()))),
          ]),
          h('tr', [
            h('td', { class: 'py-1 pr-4 text-gray-500' }, '类型：'),
            h('td', file.file?.type || '未知'),
          ]),
        ]),
      ]),
    ],
  );
};

// 修改渲染函数
const renderDate = (props: { date: number; month: number; year: number }) => {
  const date = new Date(props.year, props.month, props.date);
  const dateStr = formatDate(date);
  const events = calendarEvents.value[dateStr] || [];

  return h(
    'div',
    {
      class:
        'hover:border-primary relative h-[105px] border border-dashed border-transparent p-3 transition-colors',
      onDragenter: (e: DragEvent) => e.preventDefault(),
      onDragover: (e: DragEvent) => handleDragOver(e),
      onDrop: (e: DragEvent) => handleDrop(e, dateStr),
    },
    [
      h(
        'div',
        { class: 'h-full overflow-y-auto pr-1' },
        events.map((file) =>
          h(
            NTooltip,
            {
              trigger: 'hover',
              placement: 'right',
            },
            {
              trigger: () =>
                h(
                  'div',
                  {
                    class: 'mb-1',
                  },
                  h(
                    NTag,
                    {
                      class: 'block w-full',
                      key: file.id,
                      size: 'small',
                    },
                    { default: () => file.name },
                  ),
                ),
              default: () => renderFileInfo(file),
            },
          ),
        ),
      ),
    ],
  );
};
</script>

<template>
  <div class="flex h-full gap-4 p-4">
    <!-- 左侧文件列表 -->
    <NCard
      class="w-[300px] overflow-y-auto"
      title="文件列表"
      content-class="h-full overflow-y-scroll"
    >
      <template #header-extra>
        <NUpload
          v-if="fileList.length > 0"
          :default-file-list="fileList"
          :on-before-upload="handleUpload"
          :show-file-list="false"
          multiple
          directory-dnd
        >
          <NButton size="small" type="primary"> 上传文件 </NButton>
        </NUpload>
      </template>

      <NUpload
        v-if="fileList.length === 0"
        :default-file-list="fileList"
        :on-before-upload="handleUpload"
        class="w-full"
        multiple
        directory-dnd
      >
        <div
          class="flex cursor-pointer items-center justify-center rounded border-2 border-dashed border-gray-300 p-4"
        >
          点击或拖拽文件上传
        </div>
      </NUpload>

      <div v-else class="mt-4 max-h-[calc(100vh-240px)] overflow-y-scroll">
        <div
          v-for="file in fileList"
          :key="file.id"
          class="mb-2 cursor-move rounded border p-2"
          draggable="true"
          @dragstart="(e) => handleDragStart(e, file)"
        >
          {{ file.name }}
        </div>
      </div>
    </NCard>

    <!-- 右侧日历 -->
    <NCard class="flex-1">
      <NCalendar v-slot="props">
        <component :is="renderDate(props)" />
      </NCalendar>
    </NCard>
  </div>
</template>

<style scoped>
.n-calendar {
  /* height: calc(100% - 200px); */
  height: 100%;
}

:deep(.n-upload-trigger) {
  width: 100%;
}

:deep(.n-calendar-date__date) {
  font-size: 12px !important;
}

:deep(.n-card__content) {
  overflow: hidden;
}

:deep(.n-tag) {
  display: flex;
  align-items: center;
  max-width: 100%;
}

:deep(.n-tag__content) {
  display: block;
  width: 100%;
  overflow: hidden;
  line-height: 1.2;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* 自定义滚动条样式 */
:deep(*::-webkit-scrollbar) {
  width: 4px;
}

:deep(*::-webkit-scrollbar-track) {
  background: transparent;
}

:deep(*::-webkit-scrollbar-thumb) {
  background-color: rgb(0 0 0 / 20%);
  border-radius: 2px;
}

:deep(*::-webkit-scrollbar-thumb:hover) {
  background-color: rgb(0 0 0 / 30%);
}

:deep(.n-tooltip) {
  max-width: none !important;
}
</style>
