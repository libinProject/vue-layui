# campaigntools-project

> 营销活动

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev
```

## 项目代码规范及使用方法

### 一、基于layui 封装的form 表单的组件(默认不支持v-model)，我们重写表单组件，支持v-model 数据绑定

>#### 1.  select 组件 封装  pg-select
>
>```vue
>1.1 引用 组件 文件
>import pgSelect from '../../components/layui/pgSelect.vue'
>
>1.2 挂载 组件
>components:{
>        pgSelect
>     }
>     
>1.3 调用 方式
><pg-select v-model="templateType" :options="templateTypeOptions"></pg-select>
>
>templateType : 绑定值
>templateTypeOptions ： 绑定的options
>```
>
>
>
>#### 2. 日期组件
>
>```html
>laydate v-model 支持不是很好
>1.1 引入 input
><input type="text" class="layui-input" name="rangeTime" id="rangeTime" v-model="rangeTime" placeholder="开始时间~结束时间">
>
>1.2 初始化 日期控件
>initDate(){
>      var self=this;
>      self.laydate.render({
>        elem:'#rangeTime',
>        type:'datetime',
>        range:'~',
>        done(value){
>          self.rangeTime=value;
>        }
>      })
>    }
>
>1.3 为了让日期控件支持 v-model 我们在初始化的时候 done 的回掉函数中 手动赋值
>
>1.4 在 initComponent() 中 调用 initDate 即可
>```
>
>
>
>#### 3. 开关组件 pg-switch
>
>```
>1.1 引用 组件 文件
>import pgSwitch from '../../components/layui/pgSwitch.vue'
>
>1.2 挂载 组件
>components:{
>        pgSwitch
>     }
>     
>1.3 调用 方式
><pg-switch v-model="sexValue" :text="男|女"></pg-switch>
>
>sexValue 绑定值
>text  男|女  开关选项  默认 sexValue==true 是第一个值
>```
>
>
>
>#### 4. 基本 复选框组件 pg-checkbox-primary
>
>```
>1.1 引用 组件 文件
>import pgCheckboxPrimary from '../../components/layui/pgCheckBoxPrimary.vue'
>
>1.2 挂载 组件
>components:{
>        pgCheckboxPrimary
>     }
>     
>1.3 调用 方式
><pg-checkbox-primary v-model="upOrigin" text="起"></pg-checkbox-primary>
>```
>
>



### 二、 Layui 组件 属于按需加载，使用方法如下(以Table 为例)

>#### 1. 页面引入组件方式 （mounted引用一次即可，不需要在方法里面多次引用） 
>
>>```go
>>说明：
>>1. 为什么 在 mounted 调用
>>因为 table 渲染数据的时候，需要 去找 dom 元素 因此，需要在页面加载完成的时候去调用render 方法 
>>2. 为什么 initTableDate 调用 需要放到 initComponent 里面
>>因为 layui.use(['form','table'] 是异步方法，首次调用的时候，如果不等table 组件加载完成，会导致 找不到table 组件 页面加载完成后，以后的查询事件 便可以 直接调用initTableDate()了 ，因此，此时，不需要再table 组件已经加载进来了
>>3. self.table.render 里面的 url 的解释
>>这边是接口的请求地址 ，由于prod 和 dev 的环境不一样，所以，我们定义了一个js常量，使用 export 暴露到外部使用
>>```
>>
>>
>>
>>##### 1.1  data 中 定义全局组件变量
>>
>>table:null
>>
>>##### 1.2  引入 所需要的layui 组件 封装成方法
>>
>>```javascript
>>initComponent(){
>>  var self=this;
>>  layui.use(['form','table'],function(){
>>    self.form = layui.form;
>>    self.table = layui.table;
>>
>>    self.initTableDate(); // 初始化 table 数据的公共方法
>>  })
>>}
>>```
>>
>>##### 1.3  初始化 table 数据的方法 table.render里面的参数 具体参考 layui 官网
>>
>>```javascript
>>initTableDate() {
>>  var self = this;
>>  var parms = {
>>  };
>>  if(self.keywords!==''){
>>    parms["title"]=self.keywords
>>  }
>>  self.table.render({
>>    id: "id",
>>    elem: "#tableContent",
>>    url: constUrl.messageListUrl,
>>    method: "post",
>>    cellMinWidth: 80,
>>    where: parms,
>>    cols: [
>>      [
>>        { field: "id", title: "编号", width: 80, unresize: "true" },
>>        { field: "title", title: "模板标题", minWidth: 80, unresize: "true" },
>>        { field: "content", title: "模板内容", unresize: "true" },
>>        { field: "platform", title: "类型", minWidth: 80, unresize: "true" },
>>        { field: "createPerson", title: "创建人", minWidth: 80, unresize: "true" },
>>        { field: "createTime", title: "创建时间", minWidth: 80, unresize: "true" },
>>        { field: "updatePerson", title: "更新人", minWidth: 80, unresize: "true" },
>>        { field: "updateTime", title: "更新时间", minWidth: 80, unresize: "true" },
>>        { align: "center", width: 120, toolbar: "#barToolOptions", title: "操作",unresize: "true" }
>>      ]
>>    ],
>>    request: {
>>      pageName: "pageNum", //页码的参数名称，默认：page
>>      limitName: "pageSize" //每页数据量的参数名，默认：limit
>>    },
>>    parseData: function(res){ //res 即为原始返回的数据
>>      console.log(res)
>>      return {
>>        "code": res.isSuccess?0:1, //解析接口状态
>>        "msg": res.message, //解析提示文本
>>        "count": res.data.total, //解析数据长度
>>        "data": res.data.list //解析数据列表
>>      };
>>    },
>>    loading: true,
>>    page: true
>>  });
>>}
>>```
>>
>>##### 1.4 在mounted 生命周期调用
>>
>>```
>>self.initComponent();
>>```
>>
>>###### 1.5 注意：我们修改和删除的时候，不应该刷新整个table 这样会使页面重置为1，正确的方法应该是 直接reload 当前页 ，源代码reload 的时候，如果会使table 页面结构变了，我们修改了table.js 目前 reload 的功能，就只是刷新当前页
>>
>>```javascript
>>self.table.reload('id');
>>```
>>
>>###### 1.7 vue layui tmpl 的一个小坑：tmpl 的{{}} 分隔符 与Vue 的插值表达式 冲突，所以，带参数的模板，不可以写在页面里面，否则编译就会出错，我们解决方法：
>>
>>>1.我们写一个模板的方法 将 值通过参数 传过去
>>>
>>>```javascript
>>>switchBtnTmpl(d){
>>>      var self=this;
>>>      console.log(d.status)
>>>      if(d.status){
>>>        return '<input type="checkbox" name="status" value="true" lay-skin="switch" lay-text="上架|下架" lay-filter="statusTpl" checked>';
>>>      }else{
>>>        return '<input type="checkbox" name="status" value="false" lay-skin="switch" lay-text="上架|下架" lay-filter="statusTpl">';
>>>      }
>>>    }
>>>```
>>>
>>>2.调用
>>>
>>>>```javascript
>>>>table cols 
>>>>templet:(d)=>{return self.switchBtnTmpl(d)}
>>>>```
>>>
>>>3.form.render 这个需要在table.render 的done 完成 否则 无效
>>>
>>>>```javascript
>>>>table.render({
>>>>    done(res, curr, count){
>>>>          self.form.render();
>>>>          self.form.on('switch(statusTpl)',function(obj){
>>>>            self.testHandler(obj)
>>>>          })
>>>>        }
>>>>})
>>>>```



### 三、提交表单校验(提交规范)

>#### 1. 所有校验，我们手动书写，校验的过程，单独写到一个方法里面去，方便 add  /  edit 的时候公用
>
>```javascript
>verityForm(){
>      var self=this;
>      if(self.templateType==''){
>        layer.msg('请选择模板类型', {icon: 5});
>        return 1;
>      }
>      if(self.templateTitle==''){
>        layer.msg('请输入模板名称', {icon: 5});
>        self.$refs.templateTitle.focus();
>        return 1;
>      }
>      if(self.templateContent==''){
>        layer.msg('请输入模板内容', {icon: 5});
>        self.$refs.templateContent.focus();
>        return 1;
>      }
>    }
>```
>
>#### 2. 所有表单按钮事件 例如 创建/修改 ajax 请求的时候增加 通用弹窗  可以防止重复点击
>
>>##### 2.1 loading 引用
>>
>>```javascript
>>import loadingUtil from '../../utility/loading.js'
>>```
>>
>>##### 2.2 loading 显示
>>
>>```javascript
>>loadingUtil.showOwnFullLoading();
>>```
>>
>>##### 2.3 loading 隐藏
>>
>>```javascript
>>loadingUtil.hideOwnFullLoading();
>>```
>
>



### 四、消息提示框

>##### 1. 校验 消息提示框 （弹窗自动消失）
>
>###### 1.1 错误提示
>
>````javascript
>layer.msg('请输入姓名',{icon: 5})
>````
>
>###### 1.2 成功 提示
>
>```javascript
>layer.msg('消息模板创建成功', {icon: 6});
>```
>
>##### 1.3 删除/消息发送 等需要【确认】框
>
>```javascript
>layer.msg('确定发送吗？',{
>    time:0,
>    btn:['确定','取消'],
>    yes(index){               //  yes 里面写确定的回调函数
>      self.sendMsgHandler(index);
>    }
>  })
>```
>
>



### 五、form 表单重置方法

>```
>说明：
>	因为 重构后的表单组件 支持数据双向绑定，所以，重置的时候，去掉绑定值即可
>特殊
>	有几个表单在（弹窗中）通过修改值重置不了，如下方法
>```
>
>##### 1.1 select 在弹窗中 的重置方法（重置方法写在子组件中，需要通过父组件调用子组件的重置方法）
>
>```vue
>self.$refs.templateType.resetHandler(value); 
>// value 表示 重置为某个值 ，如果是清空，value=''
>```
>
>



### 六、Icon 使用

>>说明：
>>
>>	标签，使用 iconfont 和 layui 自带的，以下列几个常用的【操作按钮】
>>
>>	使用方法：a标签里面嵌套i 标签 《a》<i></i>《/a》，
>>
>>			   a标签默认样式需要增加： layui-btn layui-btn-xs
>>		
>>			   i 标签 默认样式增加： layui-icon
>
>##### 1. 修改
>
>>a  标签 增加 事件过滤器：lay-event="edit"
>>
>>i 标签 增加 class : icon-font-edit
>>
>>例如：
>>
>>```html
>><a class="layui-btn layui-btn-xs" lay-event="edit"><i class="layui-icon icon-font-edit"></i></a>
>>```
>
>##### 2. 测试发送
>
>>```html
>><a class="layui-btn icon-font-send-btn layui-btn-xs" lay-event="send"><i class="layui-icon icon-font-send"></i></a>
>>```
>
>##### 3. 删除
>
>>```html
>><a class="layui-btn layui-btn-danger layui-btn-xs" lay-event="del"><i class="layui-icon icon-font-del"></i></a>
>>```
>
>##### 4. 查看详情
>
>>```html
>><a class="layui-btn icon-font-view-btn layui-btn-xs" lay-event="view"><i class="layui-icon icon-font-view"></i></a>
>>```
>
>##### 5.
>
>>```
>>
>>```
>>
>>
>
>##### 6.



### 七、命名规范

>```
>说明：
>	禁止使用 demo,a,b,c 这种没有语义的方式命名变量和方法
>	css 样式命名 也不可以
>```
>
>##### 1. 点击类事件方法命名 
>
>```
>以 Handler 结尾 前面是语义，比如 保存消息 saveMsgHandler()
>```
>
>##### 2. 数组
>
>```
>以 Arra 结尾  比如  rewardTypeArra
>```
>
>##### 3. 校验表单方法命名
>
>```
>verityForm
>```
>
>##### 4. 创建按钮方法命名
>
>```
>add+功能+Handler   比如 addMsgHandler
>```
>
>
>
>