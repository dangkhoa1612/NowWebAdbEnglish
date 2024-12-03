<template>
  <el-drawer v-model="drawerDisplay">
    <template #header>
      <el-space>
        <el-icon :size="20">
          <Warning/>
        </el-icon>
        <span>More Information</span>
      </el-space>
    </template>
    <el-text size="large">
      File name
      <el-icon style="cursor: pointer;margin-left: 7px">
        <CopyDocument/>
      </el-icon>
    </el-text>
    <div class="my-3 fw-bold">
      {{ nowFileInfo.name }}
    </div>
    <el-text size="large">
      Location
      <el-icon style="cursor: pointer;margin-left: 7px">
        <CopyDocument/>
      </el-icon>
    </el-text>
    <div class="my-3 fw-bold">
      {{ nowFileInfo.path }}
    </div>
    <el-text size="large">
      Type
    </el-text>
    <div class="my-3 fw-bold">
      <span v-if="nowFileInfo.type === 4">Folders</span>
      <span v-else-if="nowFileInfo.type === 8">document</span>
      <span v-else-if="nowFileInfo.type === 10">Soft Link</span>
      <span v-else>unknown</span>
    </div>
    <div v-if="nowFileInfo.type === 8">
      <el-text size="large">
        size
      </el-text>
      <div class="my-3 fw-bold">
        {{ formatSize(nowFileInfo.size) }}
      </div>
    </div>
    <el-text size="large">
      Create
    </el-text>
    <div class="my-3 fw-bold">
      {{ nowFileInfo.createTime }}
    </div>
    <el-text size="large">
      Modify
    </el-text>
    <div class="my-3 fw-bold">
      {{ nowFileInfo.modifyTime }}
    </div>
    <el-text size="large">
      Last visit time
    </el-text>
    <div class="my-3 fw-bold">
      {{ nowFileInfo.accessTime }}
    </div>
    <el-text size="large">
      Owner
    </el-text>
    <div class="my-3 fw-bold">
      {{ nowFileInfo.uid }}
    </div>
    <el-text size="large">
      User Groups
    </el-text>
    <div class="my-3 fw-bold">
      {{ nowFileInfo.gid }}
    </div>
    <el-text size="large">
      Permissions
    </el-text>
    <div class="my-3 fw-bold">
      {{ nowFileInfo.permissions }}
    </div>
  </el-drawer>
</template>
<script setup>
import {CopyDocument, Warning} from "@element-plus/icons-vue";
import {formatSize} from "@/utils/adbManager.js";

const drawerDisplay = ref(false);
const nowFileInfo = ref({
  name: '',
  path: '',
  type: 0,
  size: 0,
  createTime: '',
  modifyTime: '',
  accessTime: '',
  uid: '',
  gid: '',
  permissions: ''
});
defineExpose({
  openDrawer(fileInfo) {
    drawerDisplay.value = true;
    nowFileInfo.value = fileInfo;
  },
})
</script>
<style scoped>
</style>