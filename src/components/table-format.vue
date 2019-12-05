<template>
  <div>
    <div v-if="config.opRowItem.length > 0" style="padding:5px 0px;display:inline-block">
      <el-button-group v-for="i in config.opRowItem.length" :key="i - 1" style="margin-right:15px;">
        <el-button v-for="button in config.opRowItem[i - 1]" :key="button" :disabled="!runtime.availableButtons[opSet[button].affectOn]" :type="opSet[button].type" size="mini" :icon="opSet[button].icon" @click.stop="operate(button)">{{ opSet[button].text }}</el-button>
      </el-button-group>
    </div>
    <div v-if="runtime.enableSearch" style="padding:5px 0px;display:inline-block">
      <el-tooltip :disabled="runtime.searchTextValidated" :value="!runtime.searchTextValidated" :manual="true" class="item" effect="dark" content="请不要输入特殊符号" placement="bottom">
        <el-input placeholder="搜索" v-model="config.search.text" clearable @change="search()" class="input-with-select" style="width:400px" size="mini">
          <el-select multiple collapse-tags v-model="config.search.colList" @change="filterData()" slot="prepend" placeholder="请选择搜索列" style="width:160px">
            <el-option v-for="col in runtime.searchColSelectionList" :key="col.text" :label="col.label" :value="col.value"></el-option>
          </el-select>
          <el-button slot="append" icon="el-icon-search"></el-button>
        </el-input>
      </el-tooltip>
    </div>
    <table cellspacing="0" cellpadding="0" class="el-table el-table--fit el-table--striped el-table--border el-table--enable-row-hover el-table--enable-row-transition">
      <thead>
        <tr>
          <th v-if="config.showSelectCol" :style="'width:' + runtime.colWidth.selectCol + '%;'"></th>
          <th v-if="config.showNumber" :style="'width:' + runtime.colWidth.num + '%;'">序号</th>
          <th v-for="col in runtime.tableItemList" :key="col" :style="'width:' + runtime.colWidth[col] + '%;'">
            <div class="thdiv">
              <el-date-picker v-if="runtime.editRow == -1 && (colSet[col].type == 'datetime' || colSet[col].type == 'date' || colSet[col].type == 'time') && colSet[col].filterable" :value-format="colSet[col].format" size="mini" v-model="config.filter[col]" :type="colSet[col].type + 'range'" :placeholder="colSet[col].label" :start-placeholder="colSet[col].label" @change="filterData()">
              </el-date-picker>
              <el-select v-else-if="runtime.editRow == -1 && colSet[col].type == 'color' && colSet[col].filterable" v-model="config.filter[col]" multiple clearable collapse-tags :placeholder="colSet[col].label" :value-key="col" @change="filterData()">
                <el-option value="191,63,63"   label="偏红色"><span style="color:#F00;">●</span> 偏红色</el-option>
                <el-option value="63,191,63"   label="偏绿色"><span style="color:#0F0;">●</span> 偏绿色</el-option>
                <el-option value="63,63,191"   label="偏蓝色"><span style="color:#00F;">●</span> 偏蓝色</el-option>
                <el-option value="191,191,63"  label="偏黄色"><span style="color:#FF0;">●</span> 偏黄色</el-option>
                <el-option value="191,63,191"  label="偏洋红"><span style="color:#F0F;">●</span> 偏洋红</el-option>
                <el-option value="63,191,191"  label="偏青色"><span style="color:#0FF;">●</span> 偏青色</el-option>
                <el-option value="63,63,63"    label="偏黑色"><span style="color:#000;">●</span> 偏黑色</el-option>
                <el-option value="191,191,191" label="偏白色"><span style="color:#FFF;">●</span> 偏白色</el-option>
                <el-option value="127,127,127" label="偏灰色"><span style="color:#777;">●</span> 偏灰色</el-option>
              </el-select>
              <el-select v-else-if="runtime.editRow == -1 && colSet[col].filterable" v-model="config.filter[col]" multiple collapse-tags :placeholder="colSet[col].label" :value-key="col" @change="filterData()">
                <el-option v-for="item in colSet[col].item" :key="item.value" :label="item.text" :value="item.value"></el-option>
              </el-select>
              <span v-else class="tftip">{{ colSet[col].label }}</span>
              <span v-if="runtime.editRow == -1 && colSet[col].sortable" class="caret-wrapper">
                <i class="sort-caret ascending"  :style="'border-bottom-color:' + (config.sort[0] == col && config.sort[1] == 'ASC'  ? '#409EFF' : '#C0C4CC')" @click="sort(col, 'ASC')" ></i>
                <i class="sort-caret descending" :style="'border-top-color:'    + (config.sort[0] == col && config.sort[1] == 'DESC' ? '#409EFF' : '#C0C4CC')" @click="sort(col, 'DESC')"></i>
              </span>
            </div>
          </th>
          <th v-if="config.opColItem.length > 0" :style="'width:' + runtime.colWidth.op + '%;'">快捷操作</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="row in dataIndexListCurrentPage.length" @click.stop="selectRow(row - 1)" :key="row - 1" :class="runtime.selectedRowList.indexOf(row - 1) == -1 ? ('row' + (row % 2)) : 'selected'">
          <td v-if="config.showSelectCol"></td>
          <td v-if="config.showNumber">{{ (config.page - 1) * config.pageSize + row }}</td>
          <td v-for="col in runtime.tableItemList" :key="col">
            <div class="cell" :style="'text-align:' + colSet[col].align">
              <div v-if="runtime.editRow != row - 1 || !colSet[col].inlineEditable" :title="runtime.editRow == row - 1 && colSet[col].editable ? '请在对话框模式下编辑该项' : colSet[col].label" v-html="config.search.text == '' || !colSet[col].searchable || !runtime.searchTextValidated ? dataListRendered[dataListOriginal[dataIndexListCurrentPage[row - 1]]._renderIndex][col] : highlight(dataListRendered[dataListOriginal[dataIndexListCurrentPage[row - 1]]._renderIndex][col], config.search.text, col)"></div>
              <component v-else :is="col + '-component'" :value="dataListOriginal[dataIndexListCurrentPage[row - 1]][col]" :dataIndex="dataIndexListCurrentPage[row - 1]" :col="colSet[col]" @edited="editedResponse" :title="colSet[col].label"></component>
            </div>
          </td>
          <td v-if="config.opColItem.length > 0 && runtime.editRow != row - 1">
            <el-button-group>
              <el-button v-for="button in config.opColItem" :disabled="runtime.editRow != -1" @click.stop="operate(button, row - 1)" :key="button" :type="opSet[button].type" size="mini" :icon="opSet[button].icon">{{ opSet[button].text }}</el-button>
            </el-button-group>
          </td>
          <td v-if="config.opColItem.length > 0 && runtime.editRow == row - 1">
            <el-button-group>
              <el-button v-for="button in config.opColEditItem" :key="button" :type="opSet[button].type" @click.stop="operate(button, row - 1)" size="mini" :icon="opSet[button].icon">{{ opSet[button].text }}</el-button>
            </el-button-group>
          </td>
        </tr>
        <tr v-if="dataIndexListCurrentPage.length == 0"><td :colspan="runtime.colCount">{{ runtime.ajaxing ? config.waitingTip : config.noFoundTip }}</td></tr>
      </tbody>
      <tfoot v-if="config.pagination != 'none'">
        <tr>
          <td :colspan="runtime.colCount" :style="'text-align:' + config.pagination">
            <el-pagination :disabled="runtime.editRow != -1" @size-change="selectPageSize" @current-change="selectPage" :current-page="config.page" :page-sizes="config.pageSizeSelection" :page-size="config.pageSize" layout="total, sizes, prev, pager, next, jumper" :total="runtime.itemCount">
            </el-pagination>
          </td>
        </tr>
      </tfoot>
    </table>
    <el-dialog title="编辑数据" :before-close="dialogCancel" v-if="runtime.dialogEditing != -1" :modal-append-to-body="false" :visible.sync="runtime.dialogEditing != -1" width="70%" top="30px" :close-on-click-modal="false" :close-on-press-escape="false">
      <div v-for="col in colSet" :key="col.index" class="dcell">
        <span v-if="!col.hideInEditDialog">{{ col.label }}：</span>
        <div v-if="!col.hideInEditDialog">
          <component :is="col.index + '-component'" :value="runtime.dataEditing[col.index]" :col="col" @edited="editedResponse"></component>
          <p style="color:#F00;" v-show="col.validateMessage.length > 0" v-html="col.validateMessage"></p>
        </div>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogSave" :disabled="!runtime.validatedPass">保 存</el-button>
        <el-button type="danger" @click="dialogCancel">关 闭</el-button>
      </span>
    </el-dialog>
    <div class="ajaxMask" v-show="runtime.ajaxing">
    </div>
  </div>
</template>

<script>
import Vue from 'vue'
import axios from 'axios'
import VueUeditorWrap from 'vue-ueditor-wrap'
Vue.component('vue-ueditor-wrap', VueUeditorWrap)
import { Button, Select, ButtonGroup, Option, Input, Autocomplete, Switch, Dialog, Upload, Pagination, Tooltip, Message, Popover, MessageBox } from 'element-ui'
Vue.component(Button.name, Button)
Vue.use(Tooltip)
Vue.use(Autocomplete)
Vue.use(Dialog)
Vue.use(Upload)
Vue.use(Switch)
Vue.use(Select)
Vue.use(ButtonGroup)
Vue.use(Option)
Vue.use(Input)
Vue.use(Pagination)
Vue.use(Popover)
Vue.prototype.$msgbox = MessageBox
Vue.prototype.$alert = MessageBox.alert
Vue.prototype.$confirm = MessageBox.confirm
Vue.prototype.$prompt = MessageBox.prompt
Vue.prototype.$message = Message
import 'element-ui/lib/theme-chalk/index.css'
export default {
  name: 'tf',
  props: {
    set: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      dataListOriginal        : [],
      dataListRendered        : [],
      dataIndexListFiltered   : [],
      dataIndexListCurrentPage: [],
      debug: true,
      colSet: {},
      colDefault: {
        type              : 'text',   //can be one of ['text', 'number', 'discrete', 'datetime', 'color', 'select', 'file', 'ueditor', 'tinymce', 'multiSelect', 'rate', 'switch', 'tag']
                                      //if a column is html, it won't be shown in the table but can only be edited in dialog
        index             : null,     //the name of this column
        editable          : true,     //can this column be edited?
        item              : [],       //filter for discrete data or limit the range for consecutive data
                                      //this is required if you want to filter on number
                                      //for example, if you define item like this: [60, 70, 80, 90]
                                      //the filter ranges will be automaticlly calculated into: (min, 60), [60, 70), [70, 80), [80, 90), [90, max),
        filterItemAutoAdd : true,
        itemMethod        : 'get',
        align             : 'center', //text-align of css
        width             : 70,       //flex width
        //is_Array          : false,    //弃用
        editTip           : '',     //only shown in dialog edit model
        validate          : [],
        validateMessage   : '',
        hideInTable       : false,    //
        hideInEditDialog  : false,
        staticItemLength  : 0
      },
      dataTypeSet: {
        number:       {filterable: false,sortable: true, searchable: false,inlineEditable: true, default: 0, parse: function(value){return parseFloat(value)},
                        editComponent: '<el-input type="number" v-model="new_value" size="mini" :disabled="!col.editable"></el-input>',
                        _readOnly: {searchable: true}},
        discrete:     {filterable: true, sortable: true, searchable: false,inlineEditable: true, default: 1, parse: function(value){return parseInt(value)},
                        editComponent: '<el-input type="number" v-model="new_value" size="mini" :disabled="!col.editable"></el-input>',
                        _readOnly: {searchable: true}},
        text:         {filterable: false,sortable: true, searchable: true, inlineEditable: true, default: '',
                        editComponent: `<el-input v-if="!col.filterable" type="text" v-model="new_value" size="mini" :disabled="!col.editable"></el-input>
                                        <el-autocomplete v-else class="inline-input" v-model="new_value" :fetch-suggestions="querySearch" size="mini" :disabled="!col.editable"></el-autocomplete>`,
                        methods: {
                          querySearch (queryString, cb) {
                            var item = this.col.item
                            var results = queryString ? item.filter(this.createFilter(queryString)) : item
                            cb(results)
                          },
                          createFilter(queryString) {
                            return (item) => {
                              return (item.value.toLowerCase().indexOf(queryString.toLowerCase()) !== -1);
                            };
                          },
                        },
                        _readOnly: {}},
        select:       {filterable: true, sortable: false,searchable: false,inlineEditable: true, default: '',
                        editComponent: `<el-select v-model="new_value" :allow-create="col.filterItemAutoAdd" filterable :default-first-option="col.filterItemAutoAdd" size="mini" :disabled="!col.editable">
                                          <el-option v-for="item in col.item" :key="item.value" :label="item.label" :value="item.value"></el-option>
                                        </el-select>`,
                        _readOnly: {}},
        multiSelect:  {filterable: true, sortable: false,searchable: false,inlineEditable: true, default: [],
                        editComponent: `<el-select v-model="new_value" multiple :allow-create="col.filterItemAutoAdd" filterable :default-first-option="col.filterItemAutoAdd" size="mini" :disabled="!col.editable">
                                          <el-option v-for="item in col.item" :key="item.value" :label="item.label" :value="item.value"></el-option>
                                        </el-select>`,
                        _readOnly: {sortable: true}},
        file:         {filterable: false,sortable: false,searchable: false,inlineEditable: false,default: [], overWrite: false, allowDownload: true, accept: 'image/png, image/jpeg', maxSize: 1000,
                        editComponent: `<el-upload ref="upload" :http-request="tfUpload" :on-success="success" :before-upload="beforeUpload" :multiple="col.maxFile > 1" :action="col.uploadUrl" :accept="col.accept" style="width:340px">
                                          <div v-for="file in me_fileList.length" class="tf_img_list">
                                            <img class="el-upload-list__item-thumbnail" :src="fileicon(me_fileList[file - 1])" style="width:100%;height:100%" />
                                            <div class="tf_img_mask">
                                              <i class="el-icon-delete" slot="reference" style="text-align:right;padding-right:5%" title="删除文件" @click.stop="remove(file - 1)"></i>
                                              <i class="el-icon-download" style="text-align:left;padding-left:5%" title="下载文件" @click.stop="download(file - 1)"></i>
                                              <span @click.stop="">{{ filename(me_fileList[file - 1]) }}</span>
                                            </div>
                                          </div>
                                          <div class="tf_img_uploadicon">
                                            <i class="el-icon-plus" style="margin-top:62px;font-size:24px;color:#909296;"></i>
                                          </div>
                                          <div slot="tip" class="el-upload__tip">{{ col.editTip }}</div>
                                        </el-upload>`,
                        methods: {
                          filename (file) {
                            return file === undefined ? '' : file.replace(/[\s\S]*\//, '')
                          },
                          fileicon (file) {
                            if(file !== undefined){
                              var ext = file.replace(/[\s\S]*\./, '')
                              if(ext == 'jpg' || ext == 'png' || ext == 'gif'){
                                return file
                              }
                              var knowns = ['doc', 'docx', 'rar', 'zip', 'xls', 'xlsx', 'ppt', 'pptx', 'psd', 'txt', 'mp3', 'mov', 'pdf', 'ini', 'wmv', 'wma']
                              if(knowns.indexOf(ext) != -1){
                                return '/' + ext + '.png'
                              }
                            }
                            return '/unknown.png'
                          },
                          beforeUpload (file) {
                            //console.log(file)
                            if(this.col.maxFile == 1 && this.col.overWrite){
                              this.me_fileList.length = 0
                            }
                            else if(this.col.maxFile < this.fileCount + 1){
                              Message.error({message: '超出允许的文件数量，请先删除部分已上传文件后重试。允许的文件数量：' + this.col.maxFile})
                              return false
                            }
                            const typeValid = this.col.accept.indexOf(file.type) !== -1
                            const sizeValid = file.size / 1024 < this.col.maxSize
                            if (!typeValid) {
                              Message.error({message: '上传文件格式非法！允许的文件格式：' + this.col.accept})
                            }
                            else if (!sizeValid) {
                              Message.error({message: '上传文件大小不能超过 ' + this.col.maxSize + 'KB!'})
                            }
                            this.fileCount += (typeValid && sizeValid ? 1 : 0)
                            return typeValid && sizeValid
                          },
                          tfUpload (operate) {
                            //console.log(operate)
                            if(this.beforeUpload(operate.file))
                            {
                              let formdata = new FormData()
                              formdata.append('file', operate.file)
                              let that = this
                              Message.warning('上传中，请稍候')
                              axios({
                                method: 'POST',
                                url: this.col.uploadUrl,
                                headers: {'Content-Type': 'multipart/form-data'},
                                data: formdata
                              })
                              .then(function(response){
                                //console.log(response.data)
                                if(response.data.result == 'success'){
                                  that.success(response.data, that.me_fileList.concat(response.data.address))
                                }
                                else{
                                  that.error(response.data, operate.file, that.me_fileList)
                                }
                              })
                            }
                          },
                          success (res, file) {
                            Message.success({message: '上传成功！记得保存数据。'})
                            this.refreshFileList(file)
                            this.new_value = this.col.maxFile == 1 && Object.prototype.toString.call(this.value) !== '[object Array]' ? this.me_fileList[0] : this.me_fileList
                          },
                          error (/*err, file, fileList*/) {
                            Message.error({message: '上传失败，请重试'})
                          },
                          download (file) {
                            var a = document.createElement('a')
                            a.href = this.me_fileList[file]
                            a.download = this.filename(this.me_fileList[file])
                            a.style.display = 'none';
                            a.target = '_blank'
                            document.body.appendChild(a);
                            a.click()
                            document.body.removeChild(a);
                          },
                          remove (file) {
                            var that = this
                            this.$confirm('此操作将从服务器删除该文件, 是否继续?', '提示', {
                              confirmButtonText: '确定',
                              cancelButtonText: '取消',
                              type: 'warning'
                            }).then(() => {
                              //let formdata = new FormData()
                              axios({
                                method: 'POST',
                                url: this.col.deleteUrl,
                                data: {operate: 'delete', file: that.me_fileList[file]}
                              })
                              .then(function (response) {
                                //console.log(response)
                                if(response.data.result == 'success'){
                                  that.me_fileList.splice(file, 1)
                                  that.fileCount--
                                  that.new_value = that.col.maxFile == 1 && Object.prototype.toString.call(that.value) !== '[object Array]' ? '' : that.me_fileList
                                  that.$message({
                                    type: 'success',
                                    message: '删除成功!'
                                  });
                                }
                                else{
                                  this.$alert(response.data)
                                  console.error(response.data)
                                }
                              })
                            }).catch(() => {
                              that.$message({
                                type: 'info',
                                message: '已取消删除'
                              });          
                            });
                          },
                          refreshFileList (fileList) {
                            var temp = fileList || this.value
                            if(Object.prototype.toString.call(temp) !== '[object Array]' && temp.length > 0){
                              this.me_fileList = [temp]
                            }
                            else{
                              for(let i = 0;i < temp.length;i++){
                                this.me_fileList.push(temp[i])
                              }
                            }
                            this.fileCount = this.me_fileList.length
                          }
                        },
                        mounted () {
                          this.refreshFileList()
                        },
                        _readOnly: {filterable: true,sortable: true,inlineEditable: true}},
        color:        {filterable: true, sortable: false,searchable: false,inlineEditable: true, default: '#000000',
                        editComponent: `<el-color-picker v-model="new_value" size="mini" style="margin:0px aut" :disabled="!col.editable"></el-color-picker><span :style="'vertical-align: super;color: ' + new_value">{{ new_value }}</span>`, 
                        _readOnly: {sortable: true,searchable: true,}},
        html:         {filterable: false,sortable: false,searchable: true, inlineEditable: false,default: '',
                        ueditorConfig: {
                          autoHeightEnabled: false,
                          initialFrameHeight: 360,
                          initialFrameWidth: '100%',
                          serverUrl: 'notset',
                          enableAutoSave: false
                        },
                        editComponent: `<vue-ueditor-wrap v-model="new_value" :config="ueditorConfig" style="margin-top:9px;"></vue-ueditor-wrap>`, 
                        methods: {
                        },
                        mounted () {
                          this.col.ueditorConfig.serverUrl = this.col.serverUrl || this.col.ueditorConfig.serverUrl
                        },
                        _readOnly: {filterable: true,sortable: true,inlineEditable: true}
                      },
        tinymce:      {filterable: false,sortable: false,searchable: true, inlineEditable: false,default: '',
                        tinymceConfig: {
                          autoHeightEnabled: false,
                          initialFrameHeight: 240,
                          initialFrameWidth: '100%',
                          serverUrl: 'notset',
                          enableAutoSave: false
                        },
                        editComponent: `<tinymce-editor :api-key="col.apikey" v-model="new_value" :init="tinymceConfig"></tinymce-editor>`, 
                        methods: {
                        },
                        mounted () {
                          //this.col.ueConfig.serverUrl = this.col.serverUrl || this.col.ueConfig.serverUrl
                        },
                        _readOnly: {filterable: true, sortable: true, inlineEditable: true}
                      },
        date:         {filterable: true, sortable: true, searchable: false,inlineEditable: true, default: '',
                        editComponent: `<el-date-picker v-model="new_value" type="date" :format="col.format" size="mini" :disabled="!col.editable"></el-date-picker>`, 
                        _readOnly: {searchable: true}},
        time:         {filterable: true, sortable: true, searchable: false,inlineEditable: true, default: '',
                        editComponent: `<el-date-picker v-model="new_value" type="time" :format="col.format" size="mini" :disabled="!col.editable"></el-date-picker>`, 
                        _readOnly: {searchable: true}},
        datetime:     {filterable: true, sortable: true, searchable: false,inlineEditable: true, default: '',
                        editComponent: `<el-date-picker v-model="new_value" type="datetime" :format="col.format" size="mini" :disabled="!col.editable"></el-date-picker>`, 
                        _readOnly: {searchable: true}},
        tag:          {filterable: true, sortable: false,searchable: true, inlineEditable: true, default: [],
                        editComponent: `<el-tag :key="tag" v-for="tag in new_value" closable :disable-transitions="false" @close="handleClose(tag)" :disabled="!col.editable">{{ tag }}</el-tag>
                                        <el-input class="input-new-tag" v-if="inputVisible" v-model="inputValue" ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm" @blur="handleInputConfirm" :disabled="!col.editable">
                                        <el-button v-else class="button-new-tag" size="small" @click="showInput" :disabled="!col.editable">+ New Tag</el-button>`, 
                        _readOnly: {sortable: true}},
        rate:         {filterable: true, sortable: true, searchable: false,inlineEditable: true, default: 0,
                        editComponent: `<el-rate v-model="new_value" :disabled="!col.editable"></el-rate>`, 
                        _readOnly: {searchable: true}},
        switch:       {filterable: true, sortable: false,searchable: false,inlineEditable: true, default: false,
                        editComponent: `<el-switch v-model="new_value" active-color="#13ce66" inactive-color="#CCCCCC" :disabled="!col.editable"></el-switch>`, 
                        _readOnly: {sortable: true, searchable: true}}
      },
      opSet: {
        add       : {affectOn: 'any',   type: 'primary', text:'添加', icon:'el-icon-plus',
                      operate () {
                        this.runtime.editedWithoutSave = false//editingCol这个变量用来记录是否有数据被修改，如果是添加新数据默认是有数据被修改
                        this.runtime.editType = 'add'
                        this.runtime.dataEditing = {}
                        for(let index in this.colSet){
                          this.runtime.dataEditing[index] = this.colSet[index].default
                          this.colSet[index].validateMessage = ''
                        }
                        this.runtime.dialogEditing = this.dataListOriginal.length
                        this.runtime.renderIndex = this.dataListOriginal.length
                        this.runtime.validatedPass = true
                      },
                    },
        edit      : {affectOn: 'single',type: 'success', text:'编辑', icon:'el-icon-edit-outline',
                      operate (row) {
                        this.runtime.editedWithoutSave = false
                        this.runtime.editType = 'edit'
                        this.runtime.dialogEditing = row[0]
                        this.runtime.dataEditing = this.objCopy(this.dataListOriginal[this.dataIndexListCurrentPage[this.runtime.dialogEditing]])
                        this.runtime.renderIndex = this.runtime.dataEditing._renderIndex
                        delete this.runtime.dataEditing._readOnly
                        delete this.runtime.dataEditing._filtered
                        delete this.runtime.dataEditing._renderIndex
                        this.runtime.validatedPass = true
                        for(let index in this.colSet){
                          this.colSet[index].validateMessage = ''
                        }
                      }},
        copy      : {affectOn: 'multi', type: 'warning', text:'复制', icon:'el-icon-document-copy'},
        delete    : {affectOn: 'multi', type: 'danger',  text:'删除', icon:'el-icon-delete', confirm: '是否确认删除数据，该操作无法撤销。'},
        save      : {affectOn: 'edit',  type: 'primary', text:'保存', icon:'el-icon-success', operate: function(){}},
        cancel    : {affectOn: 'edit',  type: 'warning', text:'取消', icon:'el-icon-close',
                      operate () {
                        this.runtime.editRow = -1
                        this.runtime.selectedRowList.length = 0
                      }
                    },
        refresh   : {affectOn: 'any',   type: 'success', text:'刷新', icon:'el-icon-refresh',
                      operate () {
                        this.fetch()
                      }
                    },
        import    : {affectOn: 'any',   type: 'primary', text:'导入', icon:'el-icon-upload'},
        export    : {affectOn: 'any',   type: 'primary', text:'导出', icon:'el-icon-download'},
        selectAll : {affectOn: 'any',   type: 'primary', text:'全选', icon:'el-icon-circle-check',
                      operate () {
                        var length = this.runtime.selectedRowList.length
                        this.runtime.selectedRowList.length = 0
                        if(length != this.dataIndexListCurrentPage.length){
                          for(let i = 0;i < this.dataIndexListCurrentPage.length;i++){
                            this.runtime.selectedRowList.push(i)
                          }
                        }
                        this.runtime.availableButtons = {
                          none: this.runtime.selectedRowList.length == 0,
                          single: this.runtime.selectedRowList.length == 1,
                          multi: this.runtime.selectedRowList.length > 0,
                          any: true,
                          edit: false
                        }
                      }
                    },
        clearFilter:{affectOn: 'any',   type: 'primary', text:'清空筛选条件', icon:'el-icon-refresh',
          operate () {
            this.config.filter = {}
            this.config._filter = {}
            this.config.search = {colList: [], text: ''},
            this.config.sort.length = 0
            this.filterData(false)
            this.runtime.selectedRowList.length = 0
          }
        }
      },
      opDefault: {
        type          : 'primary',
        text          : '操作',
        icon          : 'el-icon-s-operation',
        affectOn      : 'multi',  //can be 'any', 'none', 'single', 'multi', 'edit'
        confirm       : false,    //in case of wrong doing, but false by default for a conciser UI
        url           : null,
        method        : 'POST',
        operate (row, op) {
          var ids = []
          for(let i = 0;i < row.length;i++){
            ids.push(this.dataListOriginal[this.dataIndexListCurrentPage[row[i]]].id)
          }
          var that = this
          this.runtime.ajaxing = true
          var data = {
            operate: op,
            ids
          }
          data = this.objCopy(this.config.extendData[this.opSet[op].method], data)
          delete data['_readOnly']
          axios({
            method: that.opSet[op].method,
            url:  that.opSet[op].url || that.config.editUrl,
            data: JSON.stringify(data)
          })
          .then(function(response){
            //console.log(response)
            that.runtime.ajaxing = false
            that.runtime.dialogEditing = -1
            that.runtime.messageDialog.show = false
            if(typeof(response.data) != 'object'){
              throw {message: "操作失败，从服务器获取的数据格式错误，请返回JSON格式的数据，详情请见控制台。", text: response.data}
            }
            var data = Object.prototype.toString.call(response.data) === '[object Array]' ? response.data : [response.data]
            for(let i = 0;i < data.length;i++){
              if(data[i].hasOwnProperty('_deleted') && data[i]._deleted){//删除数据
                if(that.runtime.dmap.hasOwnProperty('_' + data[i].id)){
                  let index = that.runtime.dmap['_' + data[i].id]
                  that.dataListOriginal[index]._deleted = true
                  let fi = that.dataIndexListFiltered.indexOf(index)//filtered data index
                  that.dataIndexListFiltered.splice(fi, 1)
                  let ci = that.dataIndexListCurrentPage.indexOf(index)//current index
                  if(ci !== -1){
                    that.dataIndexListCurrentPage.splice(ci, 1)
                    let rowi = that.runtime.selectedRowList.indexOf(ci)//row selected index
                    if(rowi !== -1){
                      that.runtime.selectedRowList.splice(rowi, 1)
                      for(let j = 0;j < that.runtime.selectedRowList.length;j++){
                        if(that.runtime.selectedRowList[j] > rowi){
                          that.runtime.selectedRowList[j]--
                        }
                      }
                    }
                  }
                }
              }
              else if(that.runtime.dmap.hasOwnProperty('_' + data[i].id)){//更新数据
                that.updateData(data[i], that.runtime.dmap['_' + data[i].id])}
              else{//新增数据
                that.appendData(data[i], false)
              }
            }
            if(that.dataIndexListCurrentPage.length == 0){
              that.fetch()
            }
            Message.success({message: '操作成功'})
          })
          .catch(function (error) {
            that.$alert(error.message, 'error')
            console.log(error)
            that.runtime.ajaxing = false
          })
        }
      },
      config: {
        dataUrl             : null,             //
        dataMethod          : 'GET',            //fetch type
        pickSource          : 'local',          //'local' or 'remote', which can effect filter, sort and search
        waitingTip          : '加载中，请稍候...',
        noFoundTip          : '没有任何数据',
        page                : 1,                //
        pageSize            : 12,               //operator can change this pageSize at frontend selector
        pageSizeSelection   : [6, 12, 24],      //
        maxPage             : 100,              //
        pagination          : 'center',         //set this to null if do not need pagination, or this is the text-align attribute
        editUrl             : null,             //
        editMethod          : 'POST',           //It is difficult to image what would happen if someone set this to 'GET'
        singleSelect        : false,
        showNumber          : true,             //
        numWidth            : 10,
        //enableEditInline    : false,            //下一个版本再启用吧
        //inlineEditTimeout   : 5000,             //auto active inline edit mode in ms, false means never automatically active
        //timeout             : 1000,             //AJAX timeout in ms
        opRowItem           : [['add', 'edit', 'delete', 'copy'], ['refresh', 'selectAll'], ['clearFilter']],
        opColItem           : ['edit', 'delete', 'copy'],
        opWidth             : 10,
        //opColEditItem       : ['save', 'cancel'],
        quiteEditOnSave     : false,            //if true, the editing dialog will be automatically closed, inline model will be quit after successfully saving
        extendData          : {'POST':{}, 'GET':{}},  //this will be attached to POST or GET data in each AJAX
        searchTextRegexp    : /^[a-zA-Z0-9 \u4e00-\u9fa5]*$/,
        sort                : [],               //[0]: index to be ordered by, [1]: 'ASC' or 'DESC'
        filter              : {},               //format: {class:1,[[null, 60],[70,80]]}, 
        _filter             : {},               //special filter such as number and color
        search              : {colList: [], text: ''},
        //validateMessage     : {}
      },
      runtime: {
        enableSearch     : false,
        totalWidth       : 0,
        selectedRowList  : [],
        colWidth         : {},
        itemCount        : 0,        //
        colCount         : 0,        //
        editedWithoutSave: false,    //true means it's now editing, and some value has been changed without saved
        editRow          : -1,
        dmap             : {},       //data primary key to table row number map
        fdmap            : {},       //this can be used to locate which row is to be modified after AJAX from server
        renderIndex      : null,
        explorer         : 'unknown',
        dialogEditRCMD   : false,
        availableButtons : {any: true, none: true, single: false, multi: false},
        searchColSelectionList : [],
        dialogEditing    : -1,
        messageDialog    : {
          show           : false,
          title          : '',
          content        : ''
        },
        dialogTitle      : {
          message : '提示',
          error   : '错误',
          warning : '警告',
          success : '成功'
        },
        dataEditing      : {},
        confirmFunction () {
          console.log('null')
        },
        //validateMessage  : {answer:''},
        unvalidated      : {},
        validatedPass    : true,
        rowList          : [],
        dialogItemList   : [],
        tableItemList    : [],
        ajaxing          : false, //mouse pointer
        colListForSearch : [],    //record columns that can be searched, this will be useful if anyone search some text without any columns selected
        searchTextValidated: true,
        editType         : 'edit',//edit or add
      },
    }
  },
  methods:{
    init () {
      var agent = navigator.userAgent.toLowerCase()
      if(agent.indexOf('edge') > 0)           {this.runtime.explorer = 'edge'}
      else if(agent.indexOf('chrome') > 0)    {this.runtime.explorer = 'chrome'}
      else if(agent.indexOf('firefox') > 0)   {this.runtime.explorer = 'firefox'}
      else if(agent.indexOf('trident') > 0)   {this.runtime.explorer = 'ie'}
      for(let op in this.set.operate){
        this.opSet[op] = this.objCopy(this.opSet[op], this.set.operate[op])
      }
      for(let op in this.opSet){
        this.opSet[op] = this.objCopy(this.opDefault, this.opSet[op])
        this['operate-' + op] = this.opSet[op].operate
      }
      this.config = this.objCopy(this.config, this.set)
      //col init:
      this.runtime.colCount = 0
      this.runtime.totalWidth = 0
      if(this.config.showNumber){
          this.runtime.colWidth.num = this.config.numWidth
          this.runtime.totalWidth += this.config.numWidth
          this.runtime.colCount++
      }
      if(this.config.opColItem.length > 0){
          this.runtime.colWidth.op = this.config.opWidth
          this.runtime.totalWidth += this.config.opWidth
          this.runtime.colCount++
      }
      if(this.config.showSelectCol){
          this.runtime.colWidth.selectCol = this.config.selectColWidth
          this.runtime.totalWidth += this.config.selectColWidth
          this.runtime.colCount++
      }
      for(let i in this.config.columnList){
        var index = this.config.columnList[i].index
        var type = this.config.columnList[i].type || 'text'
        this.colSet[index] = this.objCopy(this.colDefault, this.dataTypeSet[type], this.config.columnList[i])
        this.colSet[index].searchable = this.colSet[index].searchable && !this.colSet[index].hideInTable
        this.colSet[index].filterable = this.colSet[index].filterable && !this.colSet[index].hideInTable
        this.colSet[index].sortable = this.colSet[index].sortable && !this.colSet[index].hideInTable
        this.colSet[index].inlineEditable = !this.colSet[index].editable ? false : this.colSet[index].inlineEditable
        this.colSet[index].filterItemAutoAdd = this.colSet[index].filterItemAutoAdd && this.colSet[index].filterable
        this.colSet[index].editable = this.colSet[index].editable && !this.colSet[index].hideInEditDialog
        
        //蛋疼
        this.colSet[index].filterable = this.colSet[index].filterable && !(this.colSet[index].type == 'file' || this.colSet[index].type == 'html')
        this.colSet[index].searchable = this.colSet[index].searchable && !(this.colSet[index].type == 'rate' || this.colSet[index].type == 'switch' || this.colSet[index].type == 'date' || this.colSet[index].type == 'time' || this.colSet[index].type == 'datetime' || this.colSet[index].type == 'discrete' || this.colSet[index].type == 'number')
        this.colSet[index].sortable = this.colSet[index].sortable && !(this.colSet[index].type == 'file' || this.colSet[index].type == 'html' || this.colSet[index].type == 'tag' || this.colSet[index].type == 'switch')

        this.runtime.dialogEditRCMD = this.colSet[index].inlineEditable ? true : this.runtime.dialogEditRCMD
        if(this.colSet[index].searchable){
          this.runtime.searchColSelectionList.push({value: index, label: this.config.columnList[i].label})
          this.runtime.enableSearch = true
          this.runtime.colListForSearch.push(index)
        }
        this.runtime.totalWidth += this.colSet[index].hideInTable ? 0 : this.colSet[index].width
        if(!this.config.columnList[i].hideInTable){
          this.runtime.colCount++
        }
        if(this.colSet[index].filterable){//处理item：
          if(typeof(this.colSet[index].item) == 'string'){
            var url = this.colSet[index].item
            this.colSet[index].item.length = 0
            var that = this
            var tempIndex = index
            axios
              .get(url)
              .then(function(response){
                that.colSet[tempIndex].item = response.data
                that.filterArrayToItem(tempIndex, that.colSet[tempIndex].type == 'number')
              })
              .catch(function(error){
                this.$alert(error)
                console.log(error)
              })
          }
          else{
            this.filterArrayToItem(index, this.colSet[index].type == 'number')
          }
        }
        if(!this.colSet[index].hideInEditDialog){
          this.runtime.dialogItemList.push(index)
          //处理component
          var template = this.colSet[index].editComponent
          var methods = this.colSet[index].methods
          var mounted = this.colSet[index].mounted
          Vue.component(index + '-component', {
            props: ['value', 'col'],
            data () {
              return {
                new_value: this.value,
                me_fileList: [],
                fileCount: 0,
                ueditorConfig: this.col.type == 'html' ? this.col.ueditorConfig : null,
                tinymceConfig: this.col.type == 'tinymce' ? this.col.tinymceConfig : null
              }
            },
            watch: {
              value (newValue) {
                this.new_value = newValue
              },
              new_value (newValue) {
                this.$emit('edited', newValue, this.col.index)
              }
            },
            mounted,
            template: `<div class="component">` + template + `</div>`,
            methods,
            _readOnly () {
              return null
            }
          })
        }
        if(!this.colSet[index].hideInTable){
          this.runtime.tableItemList.push(index)
        }
      }
      
      for(let i in this.runtime.tableItemList){
        let colIndex = this.runtime.tableItemList[i]
        this.runtime.colWidth[colIndex] = this.colSet[colIndex].width * 100 / this.runtime.totalWidth
      }
      if(this.config.showNumber){
        this.runtime.colWidth.num *= (100 / this.runtime.totalWidth)
      }
      if(this.config.opColItem.length > 0){
        this.runtime.colWidth.op *= (100 / this.runtime.totalWidth)
        this.runtime.colCount++
      }
      if(this.config.showSelectCol){
        this.runtime.colWidth.selectCol *= (100 / this.runtime.totalWidth)
      }
    },
    operate (op, row) {
      var that = this
      row = row == undefined ? this.runtime.selectedRowList : [row]
      if(this.opSet[op].confirm !== false){
        this.$confirm(this.opSet[op].confirm, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          that['operate-' + op](row, op)
        }).catch(() => {})
      }
      else{
        this['operate-' + op](row, op)
      }
    },
    selectRow (row) {
      if(this.runtime.editRow == -1){
        if(this.config.singleSelect){
          //console.log(row)
          if(this.runtime.selectedRowList.length === 1 && this.runtime.selectedRowList[0] === row){
            this.runtime.selectedRowList.length = 0
            this.runtime.selectedRowList = []//why??
          }
          else{
            this.runtime.selectedRowList = [row]
          }
        }
        else{
          var index = this.runtime.selectedRowList.indexOf(row)
          if(index == -1){
            this.runtime.selectedRowList.push(row)
          }
          else{
            this.runtime.selectedRowList.splice(index, 1)
          }
          this.runtime.availableButtons = {
            none: this.runtime.selectedRowList.length == 0,
            single: this.runtime.selectedRowList.length == 1,
            multi: this.runtime.selectedRowList.length > 0,
            any: true,
            edit: false
          }
        }
      }
    },
    dialogSave () {
      if(this.runtime.editType == 'edit' && !this.runtime.editedWithoutSave){//未做任何更改
        if(this.config.quiteEditOnSave){
          this.runtime.dialogEditing = -1
        }
        return
      }
      for(let index in this.colSet){//前端验证数据
        if(this.colSet[index].editable){
          this.colSet[index].validateMessage = this.validate(index, this.runtime.dataEditing[index])
          this.runtime.validatedPass = this.runtime.validatedPass && this.colSet[index].validateMessage.length == 0
          this.runtime.unvalidated[index] = this.colSet[index].validateMessage.length == 0
        }
      }
      if(!this.runtime.validatedPass){//未通过验证
        return
      }
      var that = this
      this.runtime.ajaxing = true
      var data = {
        operate: this.runtime.editType,
        data: JSON.stringify(this.runtime.dataEditing)
      }
      data = this.objCopy(this.config.extendData[this.config.editMethod], data)
      delete data['_readOnly']
      //console.log(JSON.stringify(data))
      axios({
        method: this.config.editMethod,
        url: this.config.editUrl,
        data
      })
      .then(function(response){
        that.runtime.ajaxing = false
        //console.log(response.data)
        if(typeof(response.data) != 'object'){
          throw {message: "操作失败，从服务器获取的数据格式错误，请返回JSON格式的数据，详情请见控制台。", text: response.data}
        }
        that.runtime.editedWithoutSave = false
        if(that.runtime.editType == 'add'){
          that.appendData(response.data, false)
        }
        else{
          that.updateData(response.data, that.runtime.dmap['_' + response.data.id])
        }
        if(that.config.quiteEditOnSave){
          that.runtime.dialogEditing = -1
        }
        Message.success({message: '保存成功'})
      })
      .catch(function (error) {
        that.$alert(error.message)
        //that.dialog('error', error.message)
        console.log(error.text)
        that.runtime.ajaxing = false
      })
    },
    dialogCancel () {
      var that = this
      if(this.runtime.editedWithoutSave){
        this.$confirm('您有未保存的编辑项目，是否确认放弃修改？', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          that.runtime.dialogEditing = -1
          that.runtime.messageDialog.show = false
        }).catch(() => {})
      }
      else{
        this.runtime.dialogEditing = -1
      }
    },
    dialog(type, content, callback){
      var title = this.runtime.dialogTitle[type]
      this.runtime.messageDialog = {
        type,
        content,
        title,
        show: true
      }
      this.runtime.confirmFunction = callback
    },
    selectPageSize (val) {
      if(this.runtime.editRow == -1){
        this.config.pageSize = val
        this.config.page = 1
        if(this.config.pickSource == 'remote'){
          this.fetch()
        }
        else{
          this.dataIndexListCurrentPage = this.dataIndexListFiltered.slice((this.config.page - 1) * this.config.pageSize, this.config.page * this.config.pageSize)
        }
      }
    },
    selectPage (val) {
      if(this.runtime.editRow == -1){
        this.config.page = val
        if(this.config.pickSource == 'remote'){
          if(this.runtime.itemCount <= val * this.config.pageSize){
            this.runtime.itemCount += this.config.pageSize
          }
          this.fetch()
        }
        else{
          this.dataIndexListCurrentPage = this.dataIndexListFiltered.slice((this.config.page - 1) * this.config.pageSize, this.config.page * this.config.pageSize)
        }
      }
    },
    highlight (html, keyword, index) {
      if(keyword === undefined || keyword == '' || typeof(html) != 'string' || (0 < this.config.search.colList.length && this.config.search.colList.indexOf(index) == -1)){
        return html
      }
      var text = html//.replace(/<[^>]*>/g, '')
      var arr = keyword.split(' ')
      for(let k = 0;k < arr.length;k++){
        if(arr[k].length > 0){
          var i = text.indexOf(arr[k])
          var newText = ''
          while(i !== -1){
            newText += (text.substr(0, i) + '<span style="color:red;font-weight:bold">' + text.substr(i, arr[k].length) + '</span>')
            text = text.substr(i + arr[k].length)
            i = text.indexOf(arr[k])
          }
          text = newText + text
        }
      }
      return text
    },
    sort (index, order) {
      this.config.sort = [index, order]
      //this.config.sort[index] = order
      if(this.config.pickSource == 'local'){
        this.dataListOriginal = this.dataListOriginal.sort(function(obj1, obj2){
          let ret = 0
          if(typeof(obj1[index]) === 'string'){
            ret = obj1[index] < obj2[index] ? -1 : (obj1[index] === obj2[index] ? 0 : 1)
          }
          else{
            ret = obj1[index] - obj2[index]
          }
          return order === 'ASC' ? ret : ret * -1
        })
        for(let i = 0;i < this.dataListOriginal.length;i++){
          this.runtime.dmap['_' + this.dataListOriginal[i].id] = i
        }
      }
      this.filterData(true)
    },
    editedResponse (newValue, index) {
      this.runtime.editedWithoutSave = true
      var value = this.colSet[index].hasOwnProperty('parse') ? this.colSet[index].parse(newValue) : newValue
      this.runtime.dataEditing[index] = value
      this.colSet[index].validateMessage = this.validate(index, value)
      this.runtime.unvalidated[index] = this.colSet[index].validateMessage.length == 0
      this.runtime.validatedPass = !this.runtime.unvalidated[index] ? false : this.validatePass()
    },
    validatePass () {
      for(let index in this.runtime.unvalidated){
        if(!this.runtime.unvalidated[index]){
          return false
        }
      }
      return true
    },
    filterArrayToItem (index, number) {//this function is called only in initCol function, dealing with static items not those automatically added ones
      if(typeof(this.colSet[index].item[0]) !== 'object'){
        if(number){
          var new_item = []
          this.colSet[index].item.sort(function(a, b){return a - b})
          new_item.push({text: '< ' + this.colSet[index].item[0], value: '-Infinity,' + this.colSet[index].item[0]})
          for(let i = 0;i < this.colSet[index].item.length - 1;i++){
            new_item.push({text: '[' + this.colSet[index].item[i] + ', ' + this.colSet[index].item[i + 1] + ')', value: this.colSet[index].item[i] + ',' + this.colSet[index].item[i + 1]})
          }
          new_item.push({text: '>= ' + this.colSet[index].item[this.colSet[index].item.length], value: this.colSet[index].item[this.colSet[index].item.length] + 'Infinity'})
          this.colSet[index].item = new_item
        }
        else{
          var temp = this.colSet[index].item.slice(0)
          this.colSet[index].staticItemLength = temp.length
          this.colSet[index].item.length = 0
          for(let i = 0;i < temp.length;i++){
            this.colSet[index].item.push({text:temp[i], value:temp[i]})
          }
        }
      }
    },
    render (data, i) {
      var ret = {}
      for(let index in this.colSet){
        var originVal = data[index] || this.colSet[index].default
        ret[index] = this.colSet[index].hasOwnProperty('render') ? this.colSet[index].render(originVal, data) : originVal
        if(this.colSet[index].filterItemAutoAdd){
          let add = true
          for(let j = 0;j < this.colSet[index].staticItemLength;j++){
            if(this.colSet[index].item[j].value == this.dataListOriginal[i][index]){
              add = false
              break
            }
          }
          if(add){
            this.addFilterItem(index, ret[index], this.dataListOriginal[i][index])
          }
        }
        if(!this.dataListOriginal[i].hasOwnProperty(index)){
          this.dataListOriginal[i][index] = ret[index]
        }
      }
      ret._dataIndex = i
      return ret
    },
    urlSearchParams (obj) {
      var ret = new URLSearchParams()
      for(let index in obj){
        ret.set(index, obj[index])
      }
      return ret
    },
    fetch () {
      this.runtime.ajaxing = true
      var that = this
      var temp = this.objCopy(this.config.filter)
      delete temp._readOnly
      for(let index in this.config._filter){
        delete temp[index]
      }
      delete temp['_readOnly']
      var tempSearch = {
        colList: that.config.search["colList"].length == 0 ? that.runtime.colListForSearch : that.config.search["colList"],
        text: that.config.search["text"]
      }
      var data = {
        page: that.config.page,
        pageSize: that.config.pageSize,
        filter: temp,
        _filter: that.config._filter,
        search: tempSearch,
        sort: that.config.sort
      }
      data = this.objCopy(this.config.extendData[this.config.dataMethod], data)
      delete data['_readOnly']
      //data = this.urlSearchParams(data)
      axios({
          method: that.config.dataMethod,
          url:  that.config.dataUrl,
          //headers:{'content-type': 'application/x-www-form-urlencoded'},
          data
        })
        .then(function(response){
          //console.log(response.data)
          that.runtime.ajaxing = false
          if(typeof(response.data) != 'object'){
            throw {message: "操作失败，从服务器获取的数据格式错误，请返回JSON格式的数据，详情请见控制台。", text: response.data}
          }
          that.dataListRendered.length = 0
          that.dataIndexListFiltered.length = 0
          that.dataListOriginal.length = 0
          that.dataIndexListCurrentPage = 0
          for(let i = 0;i < response.data.length;i++){
            that.appendData(response.data[i], true)
          }
          if(that.config.pickSource == 'local'){
            that.filterData(false)
            that.dataIndexListCurrentPage = that.dataIndexListFiltered.slice((that.config.page - 1) * that.config.pageSize, that.config.page * that.config.pageSize)
            that.runtime.itemCount = that.dataIndexListFiltered.length
          }
          else{
            that.dataIndexListCurrentPage = that.dataIndexListFiltered.slice(0)//直接赋值是浅拷贝
            that.runtime.itemCount = that.config.maxPage * that.config.pageSize
          }
          that.runtime.waitingFilterSingle = true
          that.runtime.ajaxing = false
          that.runtime.selectedRowList.length = 0
        })
        .catch(function (error) {
          that.$alert(error.message)
          console.log(error)
          that.runtime.ajaxing = false
        })
    },
    appendData (data, FromFetch) {//If from fetch, data are not filtered until all data ready and not append to table
      let index = this.dataListOriginal.length
      data._deleted = false
      data._renderIndex = index
      this.dataListOriginal.push(data)
      this.runtime.dmap['_' + data.id] = index
      this.dataListRendered.push(this.render(data, index))
      if(!FromFetch){
        this.filterSingleData(index)
        this.dataIndexListCurrentPage.unshift(index)
        for(let i = 0;i < this.runtime.selectedRowList.length;i++){
          this.runtime.selectedRowList[i]++
        }
      }
      else if(this.config.pickSource == 'remote'){
        this.dataIndexListFiltered.push(index)
      }
    },
    updateData (data, index) {
      this.dataListOriginal[index] = this.objCopy(this.dataListOriginal[index], data)
      delete this.dataListOriginal[index]._readOnly
      this.dataListRendered[this.dataListOriginal[index]._renderIndex] = this.render(this.dataListOriginal[index], index)
      this.filterSingleData(index)
    },
    theadFilter (value, row, column) {
      if(this.runtime.waitingFilterSingle)
      {
        this.runtime.waitingFilterSingle = false
        this.config.filter[column.property] = column.filteredValue
        this.filterData()
      }
      return true
    },
    search () {
      this.runtime.searchTextValidated = this.config.search.text === '' || this.config.searchTextRegexp.test(this.config.search.text)
      if(this.runtime.searchTextValidated && this.config.search.text.length > 0){
        this.filterData(false)
      }
    },
    filterData (filtered) {
      filtered = filtered || false
      this.config.page = 1
      for(let index in this.colSet){
        if(this.colSet[index].filterable && this.config.filter.hasOwnProperty(index)){
          if(this.colSet[index].type == 'number'){
            this.config._filter[index] = []
            for(let i = 0;i < this.config.filter[index].length;i++){
              var min = parseInt(this.config.filter[index][i].split(',')[0])
              min = isNaN(min) ? null : min
              var max = parseInt(this.config.filter[index][i].split(',')[1])
              max = isNaN(max) ? null : max
              this.config._filter[index].push([min, max])
            }
          }
          else if(this.colSet[index].type == 'color'){
            this.config._filter[index] = []
            for(let i = 0;i < this.config.filter[index].length;i++){
              var coord = this.config.filter[index][i].split(',')
              this.config._filter[index].push({dataListRendered: parseInt(coord[0]), G: parseInt(coord[1]), B: parseInt(coord[2])})
            }
          }
        }
      }
      if(this.config.pickSource == 'remote'){
        this.fetch()
      }
      else{
        this.dataIndexListFiltered.length = 0
        for(let i = 0;i < this.dataListOriginal.length;i++){
          if(!filtered && !this.dataListOriginal[i]._deleted){
            this.filterSingleData(i)
          }
          if(this.dataListOriginal[i]._filtered && !this.dataListOriginal[i]._deleted){
            this.dataIndexListFiltered.push(i)
          }
        }
        this.runtime.itemCount = this.dataIndexListFiltered.length
        this.dataIndexListCurrentPage = this.dataIndexListFiltered.slice(0, this.config.pageSize)
      }
    },
    filterSingleData (i) {
      if(this.config.pickSource == 'local'){
        this.dataListOriginal[i]._filtered = true
        for(let index in this.colSet){
          if(!this.filterMatch(this.colSet[index].type, this.dataListOriginal[i][index], (this.colSet[index].type == 'number' || this.colSet[index].type == 'color') ? this.config._filter[index] : this.config.filter[index])){
            this.dataListOriginal[i]._filtered = false
            break
          }
        }
        if(this.dataListOriginal[i]._filtered && this.config.search.text != ''){
          if(0 < this.config.search.colList.length){
            for(var j = 0;j < this.config.search.colList.length;j++){
              if(this.searchMatch(this.config.search.colList[j], this.dataListOriginal[i][this.config.search.colList[j]], this.config.search.text)){
                break
              }
            }
            this.dataListOriginal[i]._filtered = j < this.config.search.colList.length
          }
          else{
            for(var k = 0;k < this.runtime.searchColSelectionList.length;k++){
              if(this.searchMatch(this.runtime.searchColSelectionList[k].value, this.dataListOriginal[i][this.runtime.searchColSelectionList[k].value], this.config.search.text)){
                break
              }
            }
            this.dataListOriginal[i]._filtered = k < this.runtime.searchColSelectionList.length
          }
        }
      }
    },
    filterMatch (type, data, filter) {
      if(filter === undefined || filter === 'all' || filter.length === 0){
        return true
      }
      if(Object.prototype.toString.call(data) == '[object Array]'){
        var ret = false
        for(let i = 0;i < data.length;i++){
          if(this.filterMatch(type, data[i], filter)){
            ret = true
            break
          }
        }
        return ret
      }
      if(type == 'datetime' || type == 'time'  || type == 'date'){
        return data <= filter[1] && filter[0] <= data
      }
      if(type == 'number'){
        for(let i = 0;i < filter.length;i++){
          if(filter[i][0] <= data && data < filter[i][1]){
            return true
          }
        }
      }
      else if(type == 'color'){
        for(let i = 0;i < filter.length;i++){
          var dataListRendered = parseInt(data.substr(1,2), 16)
          var G = parseInt(data.substr(3,2), 16)
          var B = parseInt(data.substr(5,2), 16)
          var dist = Math.max(Math.abs(dataListRendered - filter[i].dataListRendered), Math.abs(G - filter[i].G), Math.abs(B - filter[i].B))
          if(dist < 64){
            return true
          }
        }
      }
      else{
        for(let i = 0;i < filter.length;i++){
          if(filter[i] === data){
            return true
          }
        }
      }
      return false
    },
    searchMatch (index, data, text) {
      if(Object.prototype.toString.call(data) == '[object Array]'){
        for(let i = 0;i < data.length;i++){
          if(data[i].indexOf(text) !== -1){
            return true
          }
        }
        return false
      }
      return data.indexOf(text) !== -1
    },
    addFilterItem (index, text, value) {
      if(!this.colSet[index].filterable || this.colSet[index].type == 'datetime' || this.colSet[index].type == 'color' || this.colSet[index].type == 'number'){
        return false
      }
      text = text.toString().replace(/<[^>]*>/g, '')
      var low = this.colSet[index].staticItemLength, high = this.colSet[index].item.length
      while(low < high){
        var middle = (low + high) >> 1
        text < this.colSet[index].item[middle].text ? high = middle : low = middle + 1
      }
      low--
      if(low == -1){
        this.colSet[index].item.splice(0, 0, {text, value})
      }
      else if(this.colSet[index].item[low].text == text && this.colSet[index].item[low].value != value){
        console.warn('Duplicate key inserted for text of "' + text + '" at column "' + index + '"', this.colSet[index].item[low].value, value)
        return false
      }
      else if(this.colSet[index].item[low].text != text){
        this.colSet[index].item.splice(low + 1, 0, {text, value})
      }
      return text
    },
    objCopy (...objectList) {
      //var ret = {_readOnly: {}}
      var ret = {}
      for(let i = 0;i < objectList.length;i++){
        for(let key in objectList[i]){
          //if(!ret._readOnly[key] || !ret.hasOwnProperty(key)){
          //if(!ret.hasOwnProperty(key)){
            if(Object.prototype.toString.call(objectList[i][key]) == '[object Array]'){
              ret[key] = objectList[i][key].slice(0)
            }
            else if(Object.prototype.toString.call(objectList[i][key]) == '[object RegExp]')
            {
              ret[key] = objectList[i][key]
            }
            else if(typeof(objectList[i][key]) === 'object'){
              ret[key] = this.objCopy(objectList[i][key])
            }
            else{
              ret[key] = objectList[i][key]
            }
          //}
        }
      }
      return ret
    },
    validate (index, value) {
      var ret = true
      var reg = ''
      for(let i = 0;i < this.colSet[index].validate.length;i++){
        var temp = this.colSet[index].validate[i].split(':')
        var type = temp[0]
        var params = temp.length > 1 ? temp[1].split(',') : []
        switch(type){
          case 'require':
            if(value === '' || value === null || value === undefined){
              ret = '{label}为必填项'
            }
            break
          case 'length':
            if(params.length == 1){
              if(value.length !== params[0]){
                ret = '{label}长度必须为' + params[0] + '位'
              }
            }
            else{
              if(value.length < params[0]){
                ret = '{label}长度不得低于' + params[0] + '位'
              }
              else if(value.length > params[1]){
                ret = '{label}长度不得超过' + params[1] + '位'
              }
            }
            break
          case 'minLength':
            if(value.length < params[0]){
              ret = '{label}长度不得低于' + params[0] + '位'
            }
            break
          case 'maxLength':
            if(value.length > params[0]){
              ret = '{label}长度不得超过' + params[0] + '位'
            }
            break
          case 'regexp':
            reg = new RegExp(params[0])
            if(!reg.test(value)){
              ret = '{label}正则表达式不匹配'
            }
            break
          case 'email':
            reg = /^([A-Za-z0-9_\-\.])+\@([A-Za-z0-9_\-\.])+\.([A-Za-z]{2,4})$/
            if(!reg.test(value)){
              ret = '必须输入有效的email地址'
            }
            break
          case 'integer':
            reg = /^-?\d+$/
            if(!reg.test(value)){
              ret = '必须输入一个整数'
            }
            break
          case 'number':
            reg = /^-?\d*\.?\d+$/
            if(!reg.test(value)){
              ret = '必须输入一个数字'
            }
            break
          case 'min':
            reg = /^-?\d*\.?\d+$/
            if(!reg.test(value)){
              ret = '必须输入一个数字'
            }
            else if(parseFloat(value) < params[0]){
              ret = '不得输入小于' + params[0] + '的数字'
            }
            break
          case 'max':
            reg = /^-?\d*\.?\d+$/
            if(!reg.test(value)){
              ret = '必须输入一个数字'
            }
            else if(parseFloat(value) > params[0]){
              ret = '不得输入大于' + params[0] + '的数字'
            }
            break
          case 'between':
            reg = /^-?\d*\.?\d+$/
            if(!reg.test(value)){
              ret = '必须输入一个数字'
            }
            else if(parseFloat(value) < params[0] || parseFloat(value) > params[1]){
              ret = '必须输入' + params[0] + '和' + params[1] + '之间的数字'
            }
            break
          default:
            if(this.set.customValidate.hasOwnProperty(type)){
              ret = this.set.customValidate[type](value, params)
            }
            else{
              ret = '验证方法 ' + type + ' 不存在，请检查配置'
            }
        }
        if(ret !== true){
          break
        }
      }
      return ret === true ? '' : ret.replace('{label}', this.colSet[index].label)
    }
  },
  mounted () {
    console.clear()
    this.init()
    this.fetch()
  }
}
</script>
<style scoped>
.el-table >>> td{padding:1px !important;}
.el-table >>> td div.cell div.component{display:block;}
.el-table >>> td input.el-input__inner{padding-right:0px}
.el-table >>> th{padding:0px !important;background-color:#FAFAFA;overflow:hidden}
.el-table >>> th div.thdiv{padding:0px;height:40px;line-height:40px;width:100%;}
.el-table >>> th div.thdiv span.tftip{padding-left:.7em;}
.el-table >>> th span.el-input__suffix{line-height:40px;}
.el-table >>> th i{cursor: pointer;}
.el-table >>> tr.selected{background:#d9e7f2 !important}
.el-table >>> tr.row1{background:#FFF}
.el-table >>> tr.row0{background:#FAFAFA}
.el-table >>> tr:hover{background-color:#edf4fa}
.el-table >>> img{display:block}
.caret-wrapper{position:absolute !important;right:0px;top:4px;}
.cell{width:100%;}
.el-table >>> th div.el-select{left:0px;position:absolute;padding:0px 8px 0px 0px;height:40px;}
.el-table >>> th div.el-input input{border:0px;padding:0px;height:40px;background:none;display:block;font-weight:bold}
.el-table >>> th div.el-input input::-webkit-input-placeholder{color:#909399;}
.el-table >>> th div.el-input input::-moz-input-placeholder{color:#909399;}
.el-table >>> th div.el-input input::-ms-input-placeholder{color:#909399;}
.el-select__caret{height:40px;}
.el-dialog__wrapper >>> .el-dialog__body{padding:10px 20px !important;}
.dcell{padding:0px;}
.dcell span{display:inline-block;width:12%;text-align:right;padding:9px 20px 0px 0px;vertical-align:top}
.dcell >>> input{max-width:304px;width:304px;margin:5px 0px;}
.dcell div{display:inline-block;vertical-align:top;max-width:90%;}
.dcell >>> button{margin:5px}
.dcell >>> .el-upload__tip{margin-top:0px !important}
.dcell >>> .el-color-picker{margin:5px 0px !important}
.dcell >>> input[type=number]{padding-right:0px;}
.component >>> .el-date-editor.el-input{width:300px !important;}
div >>> .el-upload-list--text{display:none;}
div >>> .tf_img_mask{z-index:9;background:#000;width:148px;height:148px;position:absolute;top:0px;left:0px;opacity:0;transition: all 0.5s;color: #FFF;font-size: 24px;line-height: 148px;}
div >>> .tf_img_mask:hover{opacity:0.7}
div >>> .tf_img_mask i{height:80%;width:45%;float:left;line-height:148px;display:block}
div >>> .tf_img_mask span{height:20%;width:100%;float:left;z-index:10;display:block;line-height:30px;font-size:14px;}
div >>> div.tf_img_uploadicon{border: 1px dotted #c0ccda;border-radius: 6px;box-sizing: border-box;width: 148px;height: 148px;margin-top:8px;display: inline-block;overflow:hidden}
div >>> div.tf_img_list{border: 1px solid #c0ccda;border-radius: 6px;box-sizing: border-box;width: 148px;height: 148px;margin: 8px 4px 0px 0;display: inline-block;overflow:hidden;position:relative}
div >>> .v-modal{z-index:2000 !important}
.component >>> .el-switch{margin-top:9px}
p{margin:0px}
div.ajaxMask{position:fixed;top:0px;left:0px;z-index:2000;width:100%;height:100%;opacity:0;cursor:wait}
</style>