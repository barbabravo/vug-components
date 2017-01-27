<template>
  <div class="well" v-show="uploading_file">
    <div>
      <labe>正在上传:   </label>
      <span v-text="uploading_file.name"></span>
      <a class="ml-short" @click="cancelUploadFile(uploading_file)" role="button">删除</a>
    </div>
    <div class="mt-low progress">
      <div class="progress-bar progress-bar-success progress-bar-striped active" role="progressbar" :style="{width:uploading_file.percent+'%'}">{{uploading_file.percent}}%</div>
    </div>
  </div>
  <div id="attachments" class="hidden"></div>
</template>

<style lang="less">
.well{
  width:100%;
  max-width:100%;
}
</style>

<script>
 export default {
   data:function(){
     return {
       _upload:null,
       uploading_file:null
     }
   },
   ready:function(){
     if(!this._upload){
       this.initUploadComponent();
     }
   },
   props:['relate_id', 'upload_config'],
   methods:{
     cancelUploadFile: function(file){
       if(this._upload){
         this._upload.uploader.stop();
         this._upload.uploader.removeFile(file.id);
       }
       this.uploading_file = null;
     },
     initUploadComponent: function(){
       var self = this;
       requirejs(['web_be_upload_standard'], function(QuiUpload){
         var opt = {
           uptoken_url: self.upload_config,
           browse_button: self.relate_id,
           container: 'attachments',
           multi_selection:true,
           save_key:false,
           unique_names:false,
           init: {
             'BeforeUpload': function(up, file) {
               self.uploading_file = file;
               self.$dispatch('before-upload', up, file);
             },
             'UploadProgress': function(up, file) {
               self.uploading_file = file;
               self.$dispatch('upload-progress', up, file);
             },
             'UploadComplete': function() {
               self.$dispatch('upload-complete');
             },
             'FileUploaded': function(up, file, info) {
               self.$dispatch('file-uploaded', up, file, info);
               self.uploading_file = null;
             },
             'Error': function(up, err, errTip) {
               self.$dispatch('error', err, errTip);
             }
           }
         };
         self._upload = new QuiUpload(opt);
       });
     }
   }
 }
</script>
