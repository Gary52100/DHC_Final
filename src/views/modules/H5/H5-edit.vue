<template>
  <el-dialog :title="'编辑页面'" :visible.sync="visible" :close-on-click-modal="false" width="70%">
    <CKEditorPage :propsData="propsData" @changeVisible="changeVisible"></CKEditorPage>
  </el-dialog>
</template>

<script>
  import CKEditorPage from './CKeditorPage'
  export default {
    data () {
      return {
        visible: false,
        propsData: []
      }
    },
    components: {
      CKEditorPage
    },
    methods: {
      changeVisible () {
        this.visible = !this.visible
      },
      init (id) {
        this.visible = true
        this.$http({
          url: this.$http.adornUrl('/webpage/getDetail'),
          method: 'get',
          params: this.$http.adornParams({
            'id': id
          })
        }).then(({data}) => {
          if (data && data.code === 200) {
            let content = data.content
            // <body class='name'>
            if (content !== '') {
              this.preData = content.split('<body>')[0] + '<body>'
              this.sufData = '</body>' + content.split('</body>')[1]
              this.editorData = content.split('<body>')[1].split('</body>')[0]
              this.id = id
              this.propsData = [this.preData, this.sufData, this.editorData, this.id]
            }
          } else {
            this.propsData = []
          }
        }).catch((err) => {
          console.log(err)
        })
      }
    }
  }
</script>
