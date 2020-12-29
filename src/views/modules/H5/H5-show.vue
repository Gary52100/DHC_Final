<template>
  <el-dialog :title="'浏览页面'" :close-on-click-modal="false" :visible.sync="visible">
    <div v-html="htmlValue"></div>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">返回</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        loading: false,
        htmlValue: ''
      }
    },
    methods: {
      init (id) {
        let _this = this
        _this.visible = true
        _this.$http({
          url: _this.$http.adornUrl('/webpage/getDetail'),
          method: 'get',
          params: _this.$http.adornParams({
            'id': id
          })
        }).then(({data}) => {
          if (data && data.code === 200) {
            _this.htmlValue = data.content
          } else {
            _this.htmlValue = ''
          }
        })
      }
    }
  }
</script>