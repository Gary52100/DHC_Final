<template>
  <el-form ref="form" label-width="0px">
    <el-form-item>
        <el-input type="textarea" id="editor"></el-input>
    </el-form-item>
    <el-form-item>
        <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
        <el-button @click="changeVisible()">返回</el-button>
    </el-form-item>
  </el-form>
</template>

<script>
import CKEDITOR from 'CKEDITOR'
export default {
  props: ['propsData'],
  data () {
    return {
      cData: []
    }
  },
  mounted () {
    CKEDITOR.replace('editor', {height: '300px', width: '100%', toolbar: 'full'})
    this.editor = CKEDITOR.instances.editor
  },
  watch: {
    propsData: function (newVal, oldVal) {
      this.cData = newVal
      setTimeout(() => {
        this.editor.setData(this.cData[2])
      }, 500)
    }
  },
  methods: {
    changeVisible () {
      this.editor.setData()
      this.$emit('changeVisible')
    },
    dataFormSubmit () {
      let content = this.cData[0] + this.editor.getData() + this.cData[1]
      this.$http({
        url: this.$http.adornUrl('/webpage/editContent'),
        method: 'post',
        data: this.$http.adornData({
          'id': this.cData[3],
          'content': content
        })
      }).then(({data}) => {
        if (data && data.code === 200) {
          this.$message({
            message: '操作成功',
            type: 'success',
            duration: 1500,
            onClose: () => {
              this.$emit('changeVisible')
              this.$emit('refreshDataList')
            }
          })
        }
      })
    }
  }
}
</script>
