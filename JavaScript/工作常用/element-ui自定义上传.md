```vue
<template>
<el-upload
           ref="upload"
           action="#"
           :auto-upload="false"
           :on-remove="handleRemove"
           :file-list="materials_list"
           :http-request="editCourse"
           :on-change="fileChange"
           :before-upload="beforeUpload"
           >
  <v-btn slot="trigger" small color="blue-grey" class="white--text">
    选取文件
    <v-icon small right dark>mdi-file-document-multiple</v-icon>
  </v-btn>
  </el-upload>
</template>
<script>
	
</script>
```

