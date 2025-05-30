<template>
  <div class="batch-container">
    <h2>批量巡检</h2>

    <el-form label-position="top" class="form-area">
      <el-form-item label="选择设备信息 Excel 文件路径">
        <el-input v-model="deviceFilePath" placeholder="点击右边按钮选择文件" readonly>
          <template #append>
            <el-button @click="selectDeviceFile">选择</el-button>
          </template>
        </el-input>
      </el-form-item>

      <el-form-item label="选择巡检命令文件夹路径">
        <el-input v-model="commandFolderPath" placeholder="点击右边按钮选择文件夹" readonly>
          <template #append>
            <el-button @click="selectCommandFolder">选择</el-button>
          </template>
        </el-input>
      </el-form-item>

      <el-form-item label="选择导出目录">
        <el-input v-model="outputFolderPath" placeholder="点击右边按钮选择文件夹" readonly>
          <template #append>
            <el-button @click="selectOutputFolder">选择</el-button>
          </template>
        </el-input>
      </el-form-item>

      <el-form-item>
        <el-button type="primary" @click="startInspection" :loading="isLoading">开始巡检</el-button>
      </el-form-item>
    </el-form>

    <div class="status-area" v-if="devices.length">
      <h3>巡检状态</h3>
      <el-table :data="devices" style="width: 100%">
        <el-table-column prop="ip" label="IP 地址" width="180" />
        <el-table-column prop="brand" label="品牌" width="180" />
        <el-table-column prop="status" label="巡检状态" />
      </el-table>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// 模拟路径和状态数据
const deviceFilePath = ref('')
const commandFolderPath = ref('')
const outputFolderPath = ref('')
const isLoading = ref(false)

// 模拟设备状态
const devices = ref([])

// 以下函数将通过 Electron 调用后端或系统文件选择器
const selectDeviceFile = async () => {
  const result = await window.electronAPI.selectFile() // 后端用 dialog 实现
  if (result) deviceFilePath.value = result
}

const selectCommandFolder = async () => {
  const result = await window.electronAPI.selectFolder()
  if (result) commandFolderPath.value = result
}

const selectOutputFolder = async () => {
  const result = await window.electronAPI.selectFolder()
  if (result) outputFolderPath.value = result
}

const startInspection = async () => {
  if (!deviceFilePath.value || !commandFolderPath.value || !outputFolderPath.value) {
    ElMessage.warning('请先选择所有路径！')
    return
  }

  isLoading.value = true
  try {
    // 请求后端读取设备信息并初始化状态
    const response = await fetch('http://127.0.0.1:5000/start_batch_inspection', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        device_file: deviceFilePath.value,
        command_folder: commandFolderPath.value,
        output_folder: outputFolderPath.value
      })
    })
    const data = await response.json()
    devices.value = data.devices // 后端返回设备数组，每个设备包含 ip、brand、status
  } catch (err) {
    ElMessage.error('巡检启动失败')
    console.error(err)
  } finally {
    isLoading.value = false
  }
}
</script>

<style scoped>
.batch-container {
  padding: 20px 40px;
}

.form-area {
  max-width: 600px;
}

.status-area {
  margin-top: 40px;
}
</style>
