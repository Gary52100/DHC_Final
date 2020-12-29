<template>
  <el-dialog :title="'上传页面'" :close-on-click-modal="false" :visible.sync="visible">
    <el-form :model="dataForm" ref="dataForm" :rules="dataRule" label-width="100px">
      <el-form-item label="页面名称" prop="pageName">
        <el-input v-model="dataForm.pageName" placeholder="页面名称"></el-input>
      </el-form-item>
      <el-form-item label="上传文件" prop="file" ref="uploadElement">
        <el-checkbox-group v-show="false" v-model="dataForm.file"></el-checkbox-group>
        <el-upload class="upload-demo" ref="upload" :on-change="fileChange" drag action="" :http-request="httpRequest" :limit="1" accept="text/html" :auto-upload="false">
          <i class="el-icon-upload"></i>
          <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
          <div slot="tip" style="color: red" class="el-upload__tip">只能上传html文件，且文件个数不超过1个</div>
        </el-upload>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button type="success" @click="dataFormSubmit()">确定</el-button>
      <el-button @click="closeUpload()">返回</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import axios from 'axios'
  import Vue from 'vue'
  export default {
    data () {
      var validateFile = (rule, value, callback) => {
        if (this.dataForm.fileLength === 0) {
          callback(new Error('上传文件不能为空'))
        } else {
          callback()
        }
      }
      return {
        visible: false,
        dataForm: {
          pageName: '',
          file: '',
          fileLength: 0
        },
        dataRule: {
          pageName: [
            { required: true, message: '页面名称不能为空', trigger: 'blur' }
          ],
          file: [
            { validator: validateFile, trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init () {
        this.visible = true
        this.dataForm.fileLength = 0
      },
      fileChange (file, fileList) {
        this.dataForm.fileLength = fileList.length
        if (fileList.length > 0) {
          this.$refs['uploadElement'].clearValidate()
        }
      },
      closeUpload () {
        this.$refs.upload.clearFiles()
        this.dataForm.pageName = ''
        this.visible = false
      },
      httpRequest (param) {
        let fileObj = param.file
        let fd = new FormData()
        let createUser = Vue.cookie.get('createUser')
        fd.append('file', fileObj)
        fd.append('webPageName', this.dataForm.pageName)
        fd.append('createUser', createUser)
        let url = this.$http.adornUrl('/webpage/upload')
        let config = {
          headers: {
            'Content-Type': 'multipart/form-data'
          },
          xhrFields: {
            withCredentials: true
          },
          crossDomain: true
        }
        axios.post(url, fd, config).then(res => {
          if (res.data.code === 200) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.$refs['uploadElement'].clearValidate()
                this.closeUpload()
                this.$emit('refreshDataList')
              }
            })
          } else {
            this.$refs['uploadElement'].clearValidate()
            this.$message.error(res.data.msg)
          }
        })
      },
      dataFormSubmit () {
        let _this = this
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            _this.$refs.upload.submit()
          }
        })
        _this.$refs.upload.submit()
      }
    }
  }
</script>