<template>
  <div id="app">
    <tf :set="set"></tf>
  </div>
</template>

<script>
import tf from './components/table-format.vue'

export default {
  name: 'app',
  components: {
    tf
  },
  data () {
    return {
      set:{
        showNumber: false,
        showOperateCol: true,
        opColItem: ['edit', 'delete'],
        opWidth: 15,
        dataUrl: 'http://localhost/basic/web/data.json',
        dataMethod: 'GET',
        editUrl: 'http://localhost/basic/web/?r=api/save',
        editMethod: 'POST',
        quiteEditOnSave: true,
        deleteMethod: 'POST',
        pickSource: 'local',
        pagination: 'right',
        columnList: [
          {
            label: 'ID',
            index: 'id',
            type: 'number',
            filterable: false,
            editable: false,
            width: 6
          },{
            label: '类型',
            index: 'type',
            type: 'select',
            filterable: true,
            width: 12,
            item: ['单项选择题','多项选择题','不定项选择题','填空题','判断题','简答题'],
            itemAutoAdd: false,
            validate: ['require']
          },{
            label: '题目',
            index: 'title',
            type: 'text',
            filterable: false,
            editable: true,
            searchable: true,
            sortable: false,
            width: 20,
            align: 'left',
            validate: ['require']
          },{
            label: '配图',
            index: 'image',
            type: 'file',
            maxFile: 1,
            overWrite: true,
            hideInTable: true,
            uploadUrl: 'http://localhost/basic/web/?r=ajaxfile/upload',
            deleteUrl: 'http://localhost/basic/web/?r=ajaxfile/delete'
          },{
            label: '选项',
            index: 'selectionList',
            type: 'text',
            editable: false,
            hideInEditDialog: true,
            searchable: true,
            filterable: false,
            sortable: false,
            width: 27,
            align: 'left',
            render: function(value, data){
              if(data.type.indexOf('选择题') == -1){
                return '<span title="该项仅对选择题有效">-</span>'
              }
              var selectionList = []
              for(var i = 0;i < 8;i++){
                if(data['selection' + i] && data['selection' + i] != ''){
                  selectionList.push(String.fromCharCode(65 + i) + '.' + data['selection' + i])
                }
                else{
                  break
                }
              }
              return selectionList.join('<br />')
            },
            tip: '非选择题请无视该项'
          },{
            label: '选项A',
            index: 'selection0',
            type: 'text',
            filterable: false,
            sortable: false,
            searchable: false,
            hideInTable: true,
            tip: '最多支持8个选项'
          },{
            label: '选项B',
            index: 'selection1',
            type: 'text',
            filterable: false,
            sortable: false,
            searchable: false,
            hideInTable: true
          },{
            label: '选项C',
            index: 'selection2',
            type: 'text',
            filterable: false,
            sortable: false,
            searchable: false,
            hideInTable: true
          },{
            label: '选项D',
            index: 'selection3',
            type: 'text',
            filterable: false,
            sortable: false,
            searchable: false,
            hideInTable: true
          },{
            label: '选项E',
            index: 'selection4',
            type: 'text',
            filterable: false,
            sortable: false,
            searchable: false,
            hideInTable: true
          },{
            label: '选项F',
            index: 'selection5',
            type: 'text',
            filterable: false,
            sortable: false,
            searchable: false,
            hideInTable: true
          },{
            label: '选项G',
            index: 'selection6',
            type: 'text',
            filterable: false,
            sortable: false,
            searchable: false,
            hideInTable: true
          },{
            label: '选项H',
            index: 'selection7',
            type: 'text',
            filterable: false,
            sortable: false,
            searchable: false,
            hideInTable: true
          },{
            label: '答案',
            index: 'answer',
            type: 'text',
            filterable: false,
            sortable: false,
            searchable: false,
            tip: '填空题的多个答案请用竖线|分割',
            width: 8,
            validate: ['require']
          },{
            label: '严格答案顺序',
            index: 'answerOrder',
            type: 'switch',
            filterable: false,
            sortable: false,
            searchable: false,
            tip: '若开启，则考生必须按顺序写出答案才算得分，仅对填空题有效',
            width: 10,
            default: true,
            render: function(value, data){
              return data.type == '填空题' ? (value ? '是' : '否') : '<span title="该项仅对填空题有效">-</span>'
            }
          },{
            label: '正确率（%）',
            index: 'ratio',
            type: 'number',
            filterable: true,
            sortable: true,
            searchable: false,
            editable: false,
            default: 0,
            width: 15,
            item: [60, 70, 80, 90],
            hideInEditDialog: true
          },{
            label: '解析',
            index: 'analysis',
            type: 'html',
            filterable: false,
            sortable: false,
            searchable: false,
            editable: true,
            default: '',
            hideInTable: true,
            ueditorConfig: {
              serverUrl: 'http://localhost/basic/web/?r=ajaxfile/upload',
              initialFrameHeight: 360,
            }
          }
        ],
        extendData: {
          GET   :{},
          POST  :{
            _csrf  : 'csrf',
            table  : 'question',
            tableOp: 'questionManage'
          }
        },
        operate: {
          delete: {
            url: 'http://localhost/basic/web/?r=api/delete'
          },
          copy: {
            url: 'http://localhost/basic/web/?r=api/copy'
          }
        },
        opRowItem: [['add', 'edit', 'delete', 'copy'], ['refresh', 'selectAll'], ['clearFilter']]
      }
    }
  }
}
</script>
