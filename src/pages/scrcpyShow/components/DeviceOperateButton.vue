<template>
  <el-button-group class="pressKey">
    <el-tooltip effect="light" content="Menu" placement="bottom">
      <el-button style="width: 25%" type="info" @mousedown="(e) => pressKey(e, 'ContextMenu')" @mouseup="(e) => pressKey(e, 'ContextMenu')">
        <el-icon :size="13" style="vertical-align: middle">
          <Menu />
        </el-icon>
      </el-button>
    </el-tooltip>
    <el-tooltip effect="light" content="Task" placement="bottom">
      <el-button style="width: 25%" type="info" @mousedown="(e) => pressKey(e, 'AndroidAppSwitch')" @mouseup="(e) => pressKey(e, 'AndroidAppSwitch')">
        <el-icon :size="13" style="vertical-align: middle">
          <CopyDocument />
        </el-icon>
      </el-button>
    </el-tooltip>
    <el-tooltip effect="light" content="Home" placement="bottom">
      <el-button style="width: 25%" type="info" @mousedown="(e) => pressKey(e, 'AndroidHome')" @mouseup="(e) => pressKey(e, 'AndroidHome')">
        <el-icon :size="13" style="vertical-align: middle">
          <House />
        </el-icon>
      </el-button>
    </el-tooltip>
    <el-tooltip effect="light" content="Return" placement="bottom">
      <el-button style="width: 25%" type="info" @mousedown="(e) => pressKey(e, 'AndroidBack')" @mouseup="(e) => pressKey(e, 'AndroidBack')">
        <el-icon :size="13" style="vertical-align: middle">
          <Back />
        </el-icon>
      </el-button>
    </el-tooltip>
  </el-button-group>
</template>

<script setup>
import {Back, CopyDocument, House, Menu} from "@element-plus/icons-vue";

defineProps({
  pressKey: Function,
})

</script>

<style scoped lang="scss">
.pressKey {
  padding: 5px;
  width: 100%;
  margin-top: 1px;
  margin-left: 2px;
}
</style>
