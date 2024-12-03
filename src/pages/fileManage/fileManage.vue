<template>
  <div class="mx-4 mt-2">
    <div class="d-flex justify-content-between">
      <el-space>
        <div style="margin-top: -5px;cursor: pointer" @click="handleBackParent">
          <SvgIcon icon="BackIcon" :color="'#66b1ff'" style="margin-right: 10px;" />
        </div>
        <SvgIcon icon="StorageIcon" :color="'#333333'" />
        <el-space v-for="(dirName, index) in dirNameList" :key="index">
          <el-icon>
            <ArrowRight />
          </el-icon>
          <span :class="nowDir === dirName ? 'fw-bold' : ''" style="cursor: pointer"
            :style="nowDir === dirName ? { color: '#66b1ff' } : ''" @click="handleBackPath(index)">
            {{ dirName }}
          </span>
        </el-space>
      </el-space>
      <el-space>
        <el-icon v-if="showSearchIcon" key="icon" style="cursor: pointer" :size="20" @click="showSearchIcon = false">
          <Search />
        </el-icon>
        <el-input v-else key="input" v-model="searchValue"
          placeholder="Search for files/folders in the current directory" size="small" style="min-width: 200px"
          clearable @change="searchFile" @input="searchFile" />
        <el-icon style="cursor: pointer" :size="20" class="mx-2" @click="getFileList('')">
          <Refresh />
        </el-icon>
        <el-popover placement="bottom" trigger="click" :popper-style="{ padding: 7 + 'px' }">
          <template #reference>
            <el-icon style="cursor: pointer" :size="20">
              <More />
            </el-icon>
          </template>
          <template #default>
            <el-space class="operationItemCss" @click="copyPath">
              <SvgIcon style="margin-left: 9px" icon="CopyIcon" />
              <span class="mx-2">Copy Path</span>
            </el-space>
            <el-space class="operationItemCss" @click="jumpPath">
              <SvgIcon style="margin-left: 9px" icon="JumpIcon" />
              <span class="mx-2">Jump to directory</span>
            </el-space>
            <el-space class="operationItemCss" @click="handleBackPath(0)">
              <SvgIcon style="margin-left: 11px;transform: rotate(180deg)"
                :style="{ width: 18 + 'px', height: 18 + 'px' }" icon="BackRootIcon" />
              <span style="margin-left: 11px">Return to the root directory</span>
            </el-space>
          </template>
        </el-popover>
      </el-space>
    </div>
    <el-space class="mt-4" :size="40">
      <el-space style="cursor: pointer" @click="createNewFolder">
        <SvgIcon icon="NewFolderIcon" />
        <span>Create a new folder</span>
      </el-space>
      <el-upload ref="upFileRef" style="margin-bottom: -5px" :auto-upload="false" :show-file-list="false"
        :on-change="handleUploadFile">
        <template #trigger>
          <el-space style="cursor: pointer;">
            <SvgIcon icon="UploadFileIcon" />
            <span>Upload files</span>
          </el-space>
        </template>
      </el-upload>
      <el-space
        :style="selectStatus === 3 || selectStatus === 2 || selectStatus === 1 ? { cursor: 'pointer' } : { color: '#a8a8a8', cursor: 'not-allowed' }"
        @click="topDownloadFile">
        <SvgIcon icon="DownloadIcon" />
        <span>download</span>
      </el-space>
      <el-space :style="selectStatus === 3 ? { cursor: 'pointer' } : { color: '#a8a8a8', cursor: 'not-allowed' }"
        @click="topRenameFile">
        <SvgIcon icon="RenameIcon" />
        <span>Rename</span>
      </el-space>
      <el-space
        :style="selectStatus === 3 || selectStatus === 2 || selectStatus === 1 ? { cursor: 'pointer' } : { color: '#a8a8a8', cursor: 'not-allowed' }"
        @click="topDeleteFile">
        <SvgIcon icon="DeleteIcon" />
        <span>delete</span>
      </el-space>
      <el-space :style="selectStatus === 3 ? { cursor: 'pointer' } : { color: '#a8a8a8', cursor: 'not-allowed' }"
        @click="topGetFileDetail">
        <SvgIcon icon="InfoIcon" :style="{ width: 18 + 'px', height: 18 + 'px' }" />
        <span>More Information</span>
      </el-space>
    </el-space>
    <div id="showFileCard" class="card mt-4" :class="{ 'drag-over': isDragging }" @dragover.prevent="handleDragOver"
      @dragleave="handleDragLeave" @drop.prevent="handleDrop">
      <div class="overlay" v-if="isDragging">
        Drag files here
      </div>
      <div class="card-header">
        <el-row :gutter="10">
          <el-col :span="12">
            <el-space>
              <el-icon v-if="selectStatus === 0" style="margin-right: 50px;margin-left: 10px;cursor: pointer" :size="20"
                @click="handleSelectAll">
                <CircleCheck />
              </el-icon>
              <el-icon v-if="selectStatus === 2 || selectStatus === 3"
                style="margin-right: 50px;margin-left: 10px;cursor: pointer" :size="20" color="#409EFF"
                @click="handleSelectAll">
                <RemoveFilled />
              </el-icon>
              <el-icon v-if="selectStatus === 1" style="margin-right: 50px;margin-left: 10px;cursor: pointer" :size="20"
                color="#409EFF" @click="handleSelectAll">
                <CircleCheckFilled />
              </el-icon>

              <span>name</span>
              <el-icon v-if="sortType === 'desc'" :size="18" style="cursor: pointer" @click="sortFileList('asc')">
                <Bottom />
              </el-icon>
              <el-icon v-else :size="18" style="cursor: pointer;margin-top: 3px" @click="sortFileList('desc')">
                <Top />
              </el-icon>
            </el-space>
          </el-col>
          <el-col :span="12">
            <span>size</span>
            <div style="margin-left: 100px; margin-top: -24px" class="d-flex justify-content-between">
              <span>Modify</span>
              <span>Create</span>
              <span style="margin-right: 15px">operate</span>
            </div>
          </el-col>
        </el-row>
      </div>
      <el-scrollbar :style="{ height: height - 250 + 'px' }">
        <div class="card-body">
          <div v-if="fileItemList.length > 0">

            <el-row v-for="(fileItem, index) in fileItemList" :key="index" class="fileItemCss"
              :style="fileItem.isSelect ? { backgroundColor: '#d1dbf5' } : ''" :gutter="10"
              style="margin-left: -16px; margin-right: -16px" @click="getFileList(fileItem.name)">
              <el-col :span="12">
                <el-space>
                  <div v-if="fileItem.isSelect" @click.stop="handelSelectItem(fileItem)"
                    style="margin-right: 14px;margin-left: 21px;margin-top: -3px">
                    <SvgIcon icon="RoundCheckIcon" color="#409EFF" />
                  </div>
                  <div v-else class="hidden-icon" @click.stop="handelSelectItem(fileItem)">
                    <SvgIcon icon="RoundIcon" />
                  </div>
                  <SvgIcon v-if="fileItem.type === ''" icon="FileTypeFolderIcon" style="margin-left: 8px" />
                  <SvgIcon v-else-if="imgType.includes(fileItem.type)" icon="FileTypeImgIcon"
                    style="margin-left: 8px" />
                  <SvgIcon v-else-if="videoType.includes(fileItem.type)" icon="FileTypeVideoIcon"
                    style="margin-left: 8px" />
                  <SvgIcon v-else-if="audioType.includes(fileItem.type)" icon="FileTypeAudioIcon"
                    style="margin-left: 8px" />
                  <SvgIcon v-else-if="textType.includes(fileItem.type)" icon="FileTypeTextIcon"
                    style="margin-left: 8px" />
                  <SvgIcon v-else icon="FileTypeNoneIcon" style="margin-left: 8px" />
                  <span class="fileNameCss">{{ fileItem.name }}</span>
                </el-space>
              </el-col>
              <el-col :span="12">
                <span v-if="fileItem.numType === 8">{{ fileItem.size }}</span>
                <span v-else>--</span>
                <div style="margin-left: 100px; margin-top: -24px" class="d-flex justify-content-between">
                  <span>{{ fileItem.createTime }}</span>
                  <span>{{ fileItem.modifyTime }}</span>
                  <el-popover placement="right" trigger="click" :popper-style="{ padding: 7 + 'px' }">
                    <template #reference>
                      <el-icon style="margin-right: 32px; margin-left: 9px;margin-top: 3px" :size="18" @click.stop>
                        <More />
                      </el-icon>
                    </template>
                    <template #default>
                      <el-space class="operationItemCss" @click="rightDownloadFile(fileItem.name, fileItem.numType)">
                        <SvgIcon style="margin-left: 9px" icon="DownloadIcon" />
                        <span class="mx-2">download</span>
                      </el-space>
                      <el-space class="operationItemCss" @click="renameFile(fileItem.name)">
                        <SvgIcon style="margin-left: 9px" icon="RenameIcon" />
                        <span class="mx-2">Rename</span>
                      </el-space>
                      <el-space class="operationItemCss" @click="deleteFileSingle(fileItem.name, fileItem.numType)">
                        <SvgIcon style="margin-left: 9px" icon="DeleteIcon" />
                        <span class="mx-2">delete</span>
                      </el-space>
                      <el-space class="operationItemCss" @click="getFileDetail(fileItem.name)">
                        <SvgIcon style="margin-left: 12px" icon="InfoIcon"
                          :style="{ width: 18 + 'px', height: 18 + 'px' }" />
                        <span style="margin-left: 9px">More Information</span>
                      </el-space>
                    </template>
                  </el-popover>
                </div>
              </el-col>
            </el-row>
          </div>
          <el-empty v-else description="No files yet" />
        </div>
      </el-scrollbar>
    </div>
  </div>
  <FileDetailDrawer ref="fileDetailDrawerRef" />
</template>
<script setup>
import {
  ArrowRight,
  Bottom,
  CircleCheck,
  CircleCheckFilled,
  More,
  Refresh,
  RemoveFilled,
  Search, Top,
} from "@element-plus/icons-vue";
import SvgIcon from "@/components/SvgIcon.vue";
import { getAdbInstance, executeCommand, formatSize } from "@/utils/adbManager.js";
import useWindowResize from "@/utils/useWindowResize.js";
import FileDetailDrawer from "@/pages/fileManage/fileDetailDrawer.vue";
import { Consumable } from "@yume-chan/stream-extra";

let syncObject
let imgType = ['jpg', 'jpeg', 'png', 'gif', 'bmp', 'webp', 'svg', 'ico', 'psd', 'raw', 'heif']
let videoType = ['mp4', 'avi', 'mov', 'wmv', 'flv', 'rmvb', 'mkv', 'webm', 'mpg', 'mpeg', '3gp']
let audioType = ['mp3', 'wav', 'wma', 'flac', 'aac', 'ape', 'ogg', 'm4a', 'amr', 'mid']
let textType = ['txt', 'md', 'log']

const { width, height } = useWindowResize()
const dirPathName = ref('/sdcard')
const fileItemList = ref([])
const fileDetailDrawerRef = ref(null)
const upFileRef = ref(null)
const nowFileInfo = ref({})
const sortType = ref('asc')
const showSearchIcon = ref(true)
const searchValue = ref('')
const isDragging = ref(false)
const copyFileList = ref([])

const nowDir = computed(() => {
  // 先判断路径是不是只有'/'，如果不是则用'/'切割路径字符串，获取最后一个元素
  if (dirPathName.value === '/') {
    return '根目录'
  } else {
    return dirPathName.value.split('/').filter(item => item !== '').pop()
  }
})
const selectStatus = computed(() => {
  if (fileItemList.value.length === 0) {
    return 0;
  }
  // 检查是否所有对象的 isSelect 值都为 true
  const allTrue = fileItemList.value.every(obj => obj.isSelect === true);

  // 检查是否至少有一个对象的 isSelect 值为 true
  const hasTrue = fileItemList.value.some(obj => obj.isSelect === true);

  // 检查是否只有一个对象的 isSelect 值为 true
  const onlyOneTrue = fileItemList.value.filter(obj => obj.isSelect === true).length === 1;

  if (allTrue) {
    return 1;
  } else if (onlyOneTrue) {
    return 3;
  } else if (hasTrue) {
    return 2;
  } else {
    return 0;
  }
})
const dirNameList = computed(() => {
  if (dirPathName.value === '/') {
    return ['Root Directory']
  } else {
    return ['Root Directory', ...dirPathName.value.split('/').filter(item => item !== '')]
  }
})
const getFileList = async (filePath) => {
  fileItemList.value = []
  let nowDirPath = dirPathName.value
  if (filePath) {
    nowDirPath = dirPathName.value + '/' + filePath
  }
  syncObject = await getAdbInstance().sync()
  console.log(filePath)
  // 在获取前先判断是否为文件夹
  const isDirectory = await syncObject.isDirectory(nowDirPath)
  if (isDirectory) {
    dirPathName.value = nowDirPath
  } else {
    console.log('Not a folder')
  }
  let num = 0
  for await (const entry of syncObject.opendir(dirPathName.value)) {
    if (entry.name === '.' || entry.name === '..') {
      continue
    }
    num++
    fileItemList.value.push({
      id: num,
      name: entry.name,
      size: formatSize(entry.size),
      createTime: timestampToTime(entry.ctime),
      modifyTime: timestampToTime(entry.mtime),
      isSelect: false,
      numType: entry.type,
      type: entry.type === 8 ? entry.name.split('.').pop() : '' // 8是文件，其他是文件夹
    })
  }
  await sortFileList('asc')
}
const searchFile = () => {
  if (searchValue.value === '') {
    fileItemList.value = copyFileList.value
  } else {
    fileItemList.value = copyFileList.value.filter(item => item.name.includes(searchValue.value))
  }
}
// 将类似这种1533657660n的bigint转换为xxxx-xx-xx xx:xx:xx格式
const timestampToTime = (bigIntValue) => {
  // 转换为毫秒级别的时间戳
  const timestamp = Number(bigIntValue) * 1000;
  // 创建一个Date对象
  const date = new Date(timestamp);
  // 获取年、月、日、小时、分钟、秒
  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, '0');
  const day = String(date.getDate()).padStart(2, '0');
  const hours = String(date.getHours()).padStart(2, '0');
  const minutes = String(date.getMinutes()).padStart(2, '0');
  const seconds = String(date.getSeconds()).padStart(2, '0');
  // 格式化为字符串
  return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;
}
// 根据点击的文件夹名字返回到指定路径
const handleBackPath = (index) => {
  console.log('Click Path')
  if (index === 0) {
    dirPathName.value = '/'
  } else {
    const dirNameList = dirPathName.value.split('/').filter(item => item !== '')
    dirPathName.value = '/' + dirNameList.slice(0, index).join('/')
  }
  getFileList('')
}
// 返回上一级目录
const handleBackParent = () => {
  console.log('Return to the previous directory')
  if (dirPathName.value === '/') {
    ElNotification({
      title: 'hint',
      message: 'Already the root directory',
      type: 'warning'
    })
    return
  }
  const dirNameList = dirPathName.value.split('/').filter(item => item !== '')
  dirPathName.value = '/' + dirNameList.slice(0, dirNameList.length - 1).join('/')
  getFileList('')
}
const handelSelectItem = (item) => {
  console.log('Select and click on the folder', selectStatus.value)
  fileItemList.value.forEach((fileItem) => {
    if (fileItem.id === item.id) {
      fileItem.isSelect = !fileItem.isSelect
    }
  })
}
const handleSelectAll = () => {
  console.log('Select All')
  if (selectStatus.value === 0) {
    fileItemList.value.forEach((fileItem) => {
      fileItem.isSelect = true
    })
  } else {
    fileItemList.value.forEach((fileItem) => {
      fileItem.isSelect = false
    })
  }
}
const topGetFileDetail = async () => {
  if (selectStatus.value === 3) {
    fileItemList.value.forEach((fileItem) => {
      if (fileItem.isSelect) {
        getFileDetail(fileItem.name)
      }
    })
  }
}
const getFileDetail = async (fileName) => {
  console.log('Get file details')
  // 判断dirPathName.value头部是否有多余的'/'，如果有多余的'/'，只保留一个，如果没有则不变
  dirPathName.value = dirPathName.value.replace(/^\/{2,}/, '/');
  let nowDirPath
  if (dirPathName.value === '/') {
    nowDirPath = dirPathName.value + fileName
  } else {
    nowDirPath = dirPathName.value + '/' + fileName
  }
  console.log(nowDirPath)
  const res = await executeCommand(`stat '${nowDirPath}'`)
  const accessRegex = /Access:\s*\(([^)]*)\)/;
  const gidRegex = /Gid:\s*\(\s*\d+\/([^)]*)\)/;

  const accessMatch = res.match(accessRegex);
  const gidMatch = res.match(gidRegex);

  const accessValue = accessMatch ? accessMatch[1] : null;
  const gidValue = gidMatch ? gidMatch[1] : null;

  const stat = await syncObject.stat(nowDirPath);
  nowFileInfo.value = {
    name: fileName,
    path: dirPathName.value,
    size: stat.size,
    createTime: timestampToTime(stat.ctime),
    modifyTime: timestampToTime(stat.mtime),
    accessTime: timestampToTime(stat.atime),
    mode: stat.mode,
    uid: stat.uid,
    gid: stat.gid + (gidValue ? `(${gidValue})` : ''),
    type: stat.type,
    permissions: accessValue.replace('/', ' ')
  }
  fileDetailDrawerRef.value.openDrawer(nowFileInfo.value)
}
const copyPath = () => {
  console.log('Copy Path')
  // 判断dirPathName.value头部是否有多余的'/'，如果有多余的'/'，只保留一个，如果没有则不变
  dirPathName.value = dirPathName.value.replace(/^\/{2,}/, '/');
  try {
    const input = document.createElement('input');
    input.style.position = 'fixed';
    input.style.opacity = '0';
    input.value = dirPathName.value;
    document.body.appendChild(input);
    input.select();
    navigator.clipboard.writeText(input.value).then(() => {
      ElNotification({
        title: 'hint',
        message: 'Copy Success',
        type: 'success'
      });
    }).catch(() => {
      ElNotification({
        title: 'error',
        message: 'Copy Failure',
        type: 'error'
      });
    }).finally(() => {
      document.body.removeChild(input);
    });
  } catch (error) {
    console.error('Error copying path:', error);
    ElNotification({
      title: 'error',
      message: 'Error copying path',
      type: 'error'
    });
  }
}
const jumpPath = () => {
  console.log('Jump to directory', dirPathName.value)
  // Check if there are extra '/' at the beginning of dirPathName.value, keep only one if there are, otherwise leave it unchanged
  dirPathName.value = dirPathName.value.replace(/^\/{2,}/, '/');
  // Display a prompt box and set the default value of the input box to the current path
  ElMessageBox.prompt('Please enter the directory path', 'Jump to directory', {
    inputValue: dirPathName.value,
    confirmButtonText: 'Confirm',
    cancelButtonText: 'Cancel',
    inputPattern: /.*/,
    inputErrorMessage: 'Please enter the directory path',
  }).then(({ value }) => {
    console.log(value)
    // After clicking the confirm button, assign the input box value to the current path
    dirPathName.value = value;
    // Retrieve the file list under the current path
    getFileList('');
  }).catch(() => {
    console.log('Cancel jumping to directory');
  });
}

const createNewFolder = () => {
  console.log('Create a new folder')
  // Check if there are extra '/' at the beginning of dirPathName.value, keep only one if there are, otherwise leave it unchanged
  dirPathName.value = dirPathName.value.replace(/^\/{2,}/, '/');
  ElMessageBox.prompt('Please enter the folder name', 'Create a new folder', {
    confirmButtonText: 'Confirm',
    cancelButtonText: 'Cancel',
    inputPattern: /.*/,
    inputErrorMessage: 'Please enter the folder name',
  }).then(({ value }) => {
    let creatPath = dirPathName.value === '/' ? '/' + value : dirPathName.value + '/' + value
    executeCommand('mkdir ' + creatPath).then((res) => {
      console.log('Create a new folder', res)
      if (res) {
        if (res.includes('File exists') || res.includes('Read-only')) {
          ElNotification({
            title: 'Creation Failed',
            message: res.split(':')[2],
            type: 'warning'
          });
          return
        }
      }
      ElNotification({
        title: 'Notice',
        message: 'Creation successful',
        type: 'success'
      });
      getFileList('')
    }).catch((err) => {
      console.log('Create a new folder', err)
      ElNotification({
        title: 'Error',
        message: 'Creation failed',
        type: 'error'
      });
    });
  });
}

const handleUploadFile = async (event, file, fileList, isDrag) => {
  try {
    // Create a FileReader object
    const reader = new FileReader();

    // Wrap file reading in a Promise for asynchronous processing
    const readFile = () => {
      return new Promise((resolve, reject) => {
        // Set the callback for successful file reading
        reader.onload = (event) => {
          // On success, return the file content via resolve
          resolve(event.target.result);
        };

        // Set the callback for file reading errors
        reader.onerror = (error) => {
          // On failure, return the error via reject
          reject(error);
        };

        // Start reading the file content
        reader.readAsArrayBuffer(isDrag ? file[0] : file[0].raw);
      });
    };

    // Read the file content (returns ArrayBuffer, which will be converted to Uint8Array for stream creation)
    const fileContent = await readFile();
    // Create a readable stream and pass the file content
    const fileStream = new ReadableStream({
      start(controller) {
        // At the start of the stream, enqueue the file content into the readable stream
        controller.enqueue(new Consumable(new Uint8Array(fileContent)));
        // Close the readable stream after adding all content
        controller.close();
      },
    });
    // Check if there are extra '/' at the beginning of dirPathName.value, keep only one if there are, otherwise leave it unchanged
    dirPathName.value = dirPathName.value.replace(/^\/{2,}/, '/');
    // Pass the readable stream to sync.write() to write to the target location
    await syncObject.write({
      filename: dirPathName.value === '/' ? '/' + file[0].name : dirPathName.value + '/' + file[0].name,
      file: fileStream,
    });
  } catch (error) {
    // Handle errors during file reading or writing
    console.error('File processing failed:', error);
  }
  upFileRef.value.clearFiles();
  await getFileList('');
}

const deleteFileSingle = async (fileName, type) => {
  // Check if there are extra '/' at the beginning of dirPathName.value, keep only one if there are, otherwise leave it unchanged
  dirPathName.value = dirPathName.value.replace(/^\/{2,}/, '/');
  console.log('Delete file', dirPathName.value === '/' ? '/' + fileName : dirPathName.value + '/' + fileName);
  // Display a confirmation dialog
  ElMessageBox.confirm('Do you want to delete the file/folder?', 'Delete File', {
    confirmButtonText: 'Confirm',
    cancelButtonText: 'Cancel',
    type: 'warning',
  }).then(async () => {
    if (type === 8) {
      await getAdbInstance().rm(dirPathName.value === '/' ? '/' + fileName : dirPathName.value + '/' + fileName);
    } else {
      await getAdbInstance().rm(dirPathName.value === '/' ? '/' + fileName : dirPathName.value + '/' + fileName, { recursive: true });
    }
    ElNotification({
      title: 'Notice',
      message: 'Deleted successfully',
      type: 'success'
    });
    await getFileList('');
  }).catch(() => {
    console.log('File deletion canceled');
  });
}

const topDeleteFile = async () => {
  ElMessageBox.confirm('Do you want to delete the selected files/folders?', 'Delete File', {
    confirmButtonText: 'Confirm',
    cancelButtonText: 'Cancel',
    type: 'warning',
  }).then(async () => {
    for (let i = 0; i < fileItemList.value.length; i++) {
      if (fileItemList.value[i].isSelect) {
        if (fileItemList.value[i].type === 8) {
          await getAdbInstance().rm(dirPathName.value === '/' ? '/' + fileItemList.value[i].name : dirPathName.value + '/' + fileItemList.value[i].name);
        } else {
          await getAdbInstance().rm(dirPathName.value === '/' ? '/' + fileItemList.value[i].name : dirPathName.value + '/' + fileItemList.value[i].name, { recursive: true });
        }
      }
    }
    ElNotification({
      title: 'Notice',
      message: 'Deleted successfully',
      type: 'success'
    });
    await getFileList('');
  }).catch(() => {
    console.log('File deletion canceled');
  });
}

const renameFile = async (fileName) => {
  ElMessageBox.prompt('Please enter the new file name', 'Rename', {
    confirmButtonText: 'Confirm',
    cancelButtonText: 'Cancel',
    inputPattern: /.*/,
    inputValue: fileName,
    inputErrorMessage: 'Please enter the file name',
  }).then(async ({ value }) => {
    if (value === '') {
      ElNotification({
        title: 'Notice',
        message: 'File name cannot be empty',
        type: 'warning'
      });
      return;
    }
    if (fileName === value) {
      ElNotification({
        title: 'Notice',
        message: 'File name is the same',
        type: 'warning'
      });
      return;
    }
    if (fileName.indexOf('/') !== -1) {
      ElNotification({
        title: 'Notice',
        message: 'File name cannot contain /',
        type: 'warning'
      });
      return;
    }
    // Check if there are extra '/' at the beginning of dirPathName.value, keep only one if there are, otherwise leave it unchanged
    dirPathName.value = dirPathName.value.replace(/^\/{2,}/, '/');
    let originalPath = dirPathName.value === '/' ? '/' + fileName : dirPathName.value + '/' + fileName;
    let renamePath = dirPathName.value === '/' ? '/' + value : dirPathName.value + '/' + value;
    console.log('Rename file', originalPath, renamePath);
    const res = await executeCommand(`mv ${originalPath} ${renamePath}`);
    if (res) {
      if (res.includes('File exists') || res.includes('Read-only')) {
        ElNotification({
          title: 'Rename Failed',
          message: res.split(':')[2],
          type: 'warning'
        });
        return;
      }
    }
    ElNotification({
      title: 'Notice',
      message: 'Renamed successfully',
      type: 'success'
    });
    await getFileList('');
  });
}

const topRenameFile = async () => {
  if (selectStatus.value === 3) {
    for (const item of fileItemList.value) {
      if (item.isSelect) {
        await renameFile(item.name);
      }
    }
  }
}

const downloadFile = async (fileName) => {
  // Check if there are extra '/' at the beginning of dirPathName.value, keep only one if there are, otherwise leave it unchanged
  dirPathName.value = dirPathName.value.replace(/^\/{2,}/, '/');
  let readPath = dirPathName.value === '/' ? '/' + fileName : dirPathName.value + '/' + fileName;
  console.log('Download file path', readPath);
  const connect = await syncObject.read(readPath);

  let chunks = [];
  const writableStream = new WritableStream({
    write(chunk) {
      chunks.push(chunk);
    },
  });
  await connect.pipeTo(writableStream);
  // Combine all Uint8Arrays into one
  const combinedUint8Array = new Uint8Array(chunks.reduce((acc, chunk) => acc + chunk.byteLength, 0));
  let offset = 0;
  for (const chunk of chunks) {
    combinedUint8Array.set(chunk, offset);
    offset += chunk.byteLength;
  }
  // Create Blob from Uint8Array
  const blob = new Blob([combinedUint8Array]);
  // Create download link
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.style.display = "none";
  a.href = url;
  a.download = fileName;
  document.body.appendChild(a);
  a.click();
  // Cleanup
  window.URL.revokeObjectURL(url);
  document.body.removeChild(a);
}

const rightDownloadFile = async (fileName, type) => {
  if (type === 8) {
    await downloadFile(fileName);
  } else {
    // Check if there are extra '/' at the beginning of dirPathName.value, keep only one if there are, otherwise leave it unchanged
    dirPathName.value = dirPathName.value.replace(/^\/{2,}/, '/');
    let filePath = dirPathName.value === '/' ? '/' + fileName : dirPathName.value + '/' + fileName;
    console.log('Download file path', filePath);
    await executeCommand(`tar -cvf ${filePath}.tar ${filePath}`);
    await downloadFile(fileName + '.tar');
    await executeCommand(`rm ${filePath}.tar`);
  }
}

const topDownloadFile = async () => {
  for (let i = 0; i < fileItemList.value.length; i++) {
    if (fileItemList.value[i].isSelect) {
      await rightDownloadFile(fileItemList.value[i].name, fileItemList.value[i].numType);
    }
  }
}

// Sorting
const sortFileList = async (type) => {
  if (type === 'asc') {
    fileItemList.value.sort((a, b) => {
      return a.name.localeCompare(b.name);
    });
    sortType.value = 'asc';
  } else {
    fileItemList.value.sort((a, b) => {
      return b.name.localeCompare(a.name);
    });
    sortType.value = 'desc';
  }
  copyFileList.value = JSON.parse(JSON.stringify(fileItemList.value));
};

const handleDrop = (e) => {
  isDragging.value = false;
  e.preventDefault();
  let files = e.dataTransfer.files;
  console.log(files);
  let fileList = [];
  handleUploadFile(e, files, fileList, true);
};

const handleDragOver = (e) => {
  e.preventDefault();
  console.log('Drag entered');
  isDragging.value = true;
  return false; // Prevent default behavior
};

const handleDragLeave = (el) => {
  console.log('Drag left');
  if (!document.getElementById('showFileCard').contains(el.relatedTarget)) {
    isDragging.value = false;
  }
};

onMounted(() => {
  getFileList('');
});

onUnmounted(() => {
  console.log('Dispose sync connection');
  syncObject?.dispose();
});

</script>

<style scoped>
.fileNameCss {
  cursor: pointer;
  transition: color 0.3s;

  &:hover {
    color: #409EFF;
  }
}

.fileItemCss {
  cursor: pointer;
  height: 50px;
  transition: background-color 0.3s;
  align-content: center;

  &:hover {
    background-color: #dedfe0;
  }
}

/* 新增规则：隐藏 SvgIcon 默认不可见 */
.hidden-icon {
  margin-right: 14px;
  margin-left: 21px;
  margin-top: -3px;
  opacity: 0;
  transition: opacity 0.3s;
  /* 添加过渡效果 */
}

/* 新增规则：在 fileItemCss 悬停时显示 .hidden-icon */
.fileItemCss:hover .hidden-icon {
  opacity: 1;
  /* 完全可见 */
}

.operationItemCss {
  cursor: pointer;
  height: 40px;
  width: 134px;
  border-radius: 6px;
  transition: background-color 0.3s;

  &:hover {
    background-color: #dedfe0;
  }
}

.drag-over {
  border: 2px dashed #409EFF;
}

.overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: #e2ebf4;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 18px;
  color: #333;
  z-index: 999;
  opacity: 0.6;
}
</style>