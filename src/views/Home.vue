<template>
  <div class="home">
    <el-form :model="dynamicValidateForm" ref="dynamicValidateForm" class="demo-dynamic">
      <el-form-item
        prop="title"
        label="标题"
        :rules="[
          { required: true, message: '请输入标题', trigger: 'blur' }
        ]"
      >
        <el-input v-model="dynamicValidateForm.title"></el-input>
      </el-form-item>
      <el-divider><i class="el-icon-present"></i></el-divider>
      <el-form-item
        prop="brief"
        label="概要"
        :rules="[
          { required: true, message: '请输入概要', trigger: 'blur' }
        ]"
      >
        <el-input type="textarea" v-model="dynamicValidateForm.brief"></el-input>
      </el-form-item>

      <div v-for="(content, index) in dynamicValidateForm.contents" :key="content.key">
        <el-divider>
          <i class="el-icon-document padding-right-25"></i>
          <i class="el-icon-picture-outline-round padding-right-25"></i>
          <i class="el-icon-video-camera-solid"></i>
        </el-divider>
        <el-form-item
          :label="'内容卡片' + (index+1)"
          :prop="'contents.' + index + '.contentBrief'"
          :rules="{
            required: true, message: '内容描述不能为空', trigger: 'blur'
          }"
        >
          <el-input v-model="content.contentBrief"></el-input>
        </el-form-item>
        <el-form-item class="button-group float-left content-type"
          :key="content.key"
          :prop="'contents.' + index + '.type'"
          :rules="{
            required: true, message: '请选择内容类型', trigger: 'change'
          }"
        >
          <el-select v-model="dynamicValidateForm.contents[index].type" placeholder="请选择类型">
            <el-option label="仅文字" value="text"></el-option>
            <el-option label="图片" value="image"></el-option>
            <el-option label="视频" value="video"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item class="button-group float-left upload-area">
          <el-upload
            class="upload-demo"
            action="http://localhost:9001/article/uploadFiles"
            :on-preview="handlePreview"
            :on-success="(response, file, fileList)=>{return handleUploadSuccess(response, file, fileList, index)}"
            :on-error="handleUploadError"
            :on-remove="(file, fileList)=>{return handleRemove(file, fileList, index)}"
            :file-list="content.fileList"
            accept="image/*, video/*"
            list-type="picture">
            <el-button size="small" type="primary" class="button-group">点击上传</el-button>
            <div slot="tip" class="el-upload__tip button-group">只能上传jpg/png文件，且不超过3mb</div>
          </el-upload>
        </el-form-item>
        <el-form-item class="button-group float-right">
          <el-button @click.prevent="removeContent(content)">删除卡片</el-button>
        </el-form-item>
        <div class="clear"></div>
      </div>
      <el-form-item>
        <el-divider></el-divider>
        <el-button @click="addContent">新增内容卡片</el-button>
      </el-form-item>
      <el-divider><i class="el-icon-chat-line-round"></i></el-divider>
      <p>
        是否展示用户评价
      </p>
      <el-switch
        v-model="dynamicValidateForm.displayComment"
        active-text="是"
        inactive-text="否">
      </el-switch>
      <el-form-item
        prop="comment"
        label="用户评价内容"
        :rules="[
          { required: dynamicValidateForm.displayComment, message: '请输入用户评价内容', trigger: 'blur' }
        ]"
        v-show="dynamicValidateForm.displayComment"
      >
        <el-input type="textarea" v-model="dynamicValidateForm.comment"></el-input>
      </el-form-item>
      <el-divider><i class="el-icon-mic"></i></el-divider>
      <p>
        是否展示评分与反馈
      </p>
      <el-switch
        v-model="dynamicValidateForm.displayFeedback"
        active-text="是"
        inactive-text="否">
      </el-switch>
      <el-divider></el-divider>
      <el-form-item class="bottom-button">
        <el-button type="primary" @click="submitForm('dynamicValidateForm')">提交</el-button>
        <el-button @click="resetForm('dynamicValidateForm')">重置</el-button>
      </el-form-item>
    </el-form>
    <el-dialog
      title="预览"
      :visible.sync="dialogVisible">
      <el-image
        :src="previewImgSrc"
        fit="none"
        v-show="previewImgVisible"></el-image>
      <video
        :src="previewVideoSrc"
        controls="controls"
        preload="auto"
        v-show="previewVideoVisible"></video>
    </el-dialog>
  </div>
</template>

<script>
// @ is an alias to /src
// import HelloWorld from '@/components/HelloWorld.vue'

export default {
  name: 'Home',
  components: {
    // HelloWorld
  },
  data () {
    return {
      dynamicValidateForm: {
        contents: [{
          type: '',
          contentBrief: '',
          fileList: []
        }],
        title: '',
        brief: '',
        displayComment: false,
        comment: '',
        displayFeedback: true
      },
      dialogVisible: false,
      previewImgSrc: '',
      previewVideoSrc: '',
      previewImgVisible: true
    }
  },
  computed: {
    previewVideoVisible: function () {
      return !this.previewImgVisible
    }
  },
  methods: {
    submitForm (dynamicValidateForm) {
      this.$refs[dynamicValidateForm].validate((valid) => {
        if (valid) {
          this.$http({
            method: 'post',
            url: 'http://localhost:9001/article/addArticle',
            headers: {
              'Content-Type': 'application/json'
            },
            data: this.dynamicValidateForm
          }).then(function (response) {
            if (response.data.status === 200) {
              this.$notify({
                title: '提交成功',
                message: '',
                type: 'success'
              })
            } else {
              this.$notify({
                title: '提交失败',
                message: '请联系管理员查看',
                type: 'warning'
              })
            }
          }.bind(this)).catch(function (error) {
            console.log(error)
          })
        } else {
          this.$notify({
            title: '表单校验失败',
            message: '请检查表单必填项是否填写完整',
            type: 'error'
          })
          return false
        }
      })
    },
    resetForm (dynamicValidateForm) {
      this.$refs[dynamicValidateForm].resetFields()
    },
    removeContent (item) {
      var index = this.dynamicValidateForm.contents.indexOf(item)
      if (index !== -1) {
        this.dynamicValidateForm.contents.splice(index, 1)
      }
    },
    addContent () {
      this.dynamicValidateForm.contents.push({
        type: '',
        contentBrief: '',
        fileList: []
        // key: Date.now()
      })
    },
    handleRemove (file, fileList, index) {
      var fileListContents = this.dynamicValidateForm.contents[index].fileList
      fileListContents.forEach((item, index) => {
        if (file.uid === item.uid) {
          fileListContents.splice(index, 1)
        }
      })
      this.$notify({
        title: '移除成功',
        message: file.name + '已从文件列表移除成功',
        type: 'success'
      })
    },
    handlePreview (file) {
      this.dialogVisible = true
      if (file.type.substring(0, 5) === 'image') {
        this.previewImgVisible = true
        this.previewImgSrc = file.url
      } else if (file.type.substring(0, 5) === 'video') {
        this.previewImgVisible = false
        this.previewVideoSrc = file.url
      }
    },
    handleUploadSuccess (response, file, fileList, index) {
      if (response.status === 200) {
        this.dynamicValidateForm.contents[index].fileList.push({ name: response.data.fileName, url: response.data.filePath, type: response.data.fileType })
        this.$notify({
          title: '上传成功',
          message: file.name + '已上传成功,大小为' + file.size / 1000 + 'kb',
          type: 'success'
        })
      }
    },
    handleUploadError (err, file, fileList) {
      this.$notify.error({
        title: '上传失败',
        message: '原因：' + err
      })
      console.log(err, file, fileList)
    }
  }
}
</script>
<style scoped>
.padding-right-25 {
  padding-right: 25px;
}
.bottom-button {
  margin-top: 30px;
}
.home {
  width: 50%;
  margin: auto;
}
.button-group {
  display: inline-block;
  margin-left: 10px;
  margin-right: 10px;
}
.float-left {
  float: left;
}
.float-right {
  float: right;
}
.clear {
  clear:both;
}
.content-type {
  width: 100px;
}
.upload-area {
  max-width: 62%;
}
</style>
