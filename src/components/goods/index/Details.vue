<template>
    <div class="main">
        <a-spin tip="正在加载中..." v-if="dataLoading" class="loading"></a-spin>
        <div v-else>
            <a-tabs v-model="tabsKey" @change="callback">
                <a-tab-pane key="tabs1" tab="基础信息"></a-tab-pane>
                <a-tab-pane key="tabs2" tab="商品详情"></a-tab-pane>
            </a-tabs>
        
            <a-form :form="form" :label-col="labelCol" :wrapper-col="wrapperCol" class="margin-tb-lg formCol">
                <div v-show="tabsKey == 'tabs1'">
                    <a-form-item label="商品分类">
                        <a-cascader :options="classifyList" v-decorator="['category_id',{ initialValue: customForm.category_id}]" class="extent" paramsName="category_id" disabled/>
                    </a-form-item>
                    <a-form-item label="商品类型">
                        <a-select v-decorator="['kind_id',{ initialValue: customForm.kind_id}]" paramsName="kind_id" class="extent" disabled >
                            <a-select-option v-for="(item,index) in kindList" :key="index" :value="item.kind_id" >
                                {{ item.name }}
                            </a-select-option>
                        </a-select>
                    </a-form-item>
                    
                    <a-form-item label="商品名称">
                        <a-input v-decorator="['name',{ initialValue: customForm.name }]" paramsName="name" class="extent" disabled />
                    </a-form-item>
                    <a-form-item label="商品单位">
                        <a-input v-decorator="['unit',{ initialValue: customForm.unit }]" paramsName="unit" class="extent" disabled/>
                    </a-form-item>
                    <a-form-item label="商品简介">
                        <textarea-type :ruleForm="textarea" paramsName="short_description" ></textarea-type>
                    </a-form-item>
                    <a-form-item label="商品轮播图">
                        <div class="carousel">
                            <div v-for="(item,index) in carousel" :key="index">
                                <div class="img-album cursor radius-xs" >
                                    <img :src="customForm.url+ '?name=' + item">
                                    <span v-if="index == 0" class="first">宝贝主图</span>
                                </div>
                            </div>
                        </div>
                    </a-form-item>
    
                    <a-form-item label="商品规格" required>
        
                        <div class="norms">
                            <!-- 规格列表 -->
                            <div v-show="Object.keys(normsData).length > 0" class="normsMeter">
                                <div class="meterList" v-for="(item,index) in normsData" :key="index">
                                    <div class="meterName">{{ index }}：</div>
                                    <div class="normsText">
                                        <a-checkbox v-for="(value,index1) in item" :key="index1" :checked="value.check" @change="checkChange(index,index1)">{{ value.title }}</a-checkbox>
                                    </div>
                                </div>
                            </div>
                            <table>
                                <thead>
                                <tr>
                                    <th v-for="(item,index) in columns" :key="index" >
                                        <span class="text-red" v-show="item.need">*</span>{{ item.title }}
                                    </th>
                                </tr>
                                </thead>
                
                                <tbody>
                                <tr v-for="(item,index) in manyNormsList" :key="index">
                                    <td v-for="(value,index1) in columns" :key="index1" class="text-cut">
                                             <span v-if="value.label == 'image'" @click="handleChangeFocus(index)"><!-- 多规格-图片 -->
                                                <div class="upload-table">
                                                    <img v-if="item.image" :src="customForm.url + '?name=' + item.image" />
                                                    <div v-else>
                                                        <a-icon type="camera" />
                                                    </div>
                                                </div>
                                            </span>
                                        <a-input v-model="item.price" v-else-if="value.label == 'price'" /><!-- 售价 -->
                                        <a-input v-model="item.cost_price" v-else-if="value.label == 'cost_price'" /><!-- 成本价 -->
                                        <a-input v-model="item.old_price" v-else-if="value.label == 'old_price'" /><!-- 原价 -->
                                        <a-input v-model="item.stock" v-else-if="value.label == 'stock'" /><!-- 库存 -->
                                        <a-input v-model="item.sku" v-else-if="value.label == 'sku'" /><!-- 编号 -->
                                        <a-input v-model="item.weight" v-else-if="value.label == 'weight'" /><!-- 重量 -->
                                        <a-input v-model="item.volume" v-else-if="value.label == 'volume'" /><!-- 体积 -->
                                        <span v-else-if="value.label == 'action'">
                                                <span @click="deleteManyNorms(index)" class="text-blue cursor"> 删除 </span>
                                            </span><!-- 操作 -->
                                        <span v-else>
                                                {{ item[value.label] }}
                                            </span>
                                    </td>
                                </tr>
                                </tbody>
                            </table>
                        </div>
                    </a-form-item>
                    
                    
                    <a-form-item label="商品状态" >
                        <a-radio-group v-decorator="['status',{ initialValue: customForm.status }]" disabled>
                            <a-radio :value="1">上架</a-radio>
                            <a-radio :value="2">下架</a-radio>
                        </a-radio-group>
                    </a-form-item>
                    <a-form-item label="商品标签" >
                        <a-select mode="multiple" v-decorator="['tags',{ initialValue: customForm.tags }]" paramsName="tags" disabled>
                            <a-select-option v-for="(item,index) in labelList" :key="index" :value="item.value">
                                {{ item.label }}
                            </a-select-option>
                        </a-select>
                        
                    </a-form-item>
                </div>
                <div v-show="tabsKey == 'tabs2'">
                    <a-row :gutter="24">
                        <a-col :span="12" v-for="(item,key,index) in paramterData" :key="index">
                            <a-form-item :label="key" > <!-- 文本框 -->
                                <div v-html="item" class="radius-xs text-indent" style="background-color: rgba(200,200,200,0.15)"></div>
                            </a-form-item>
                        </a-col>
                        
                    </a-row>
    
                    <a-form-item label="手机端描述" class="phoneBewrite" :label-col="{ span: 22 }" :wrapper-col="{ span: 22 }" >
                        <div class="bewrite">
                            <div class="phones radius-xs">
                                <div class="text">
                                    <div v-for="(item,index) in phoneList" :key="index">
                                        <div v-if="item.type == 1" class="cursor radius-xs">
                                            <div v-if="item.status == 2" v-html="item.text" class="htmlText" :class="{ textBlack: item.focus }"></div>
                                        </div>
                                        <div v-else class="cursor radius-xs text-center">
                                            <img :src="customForm.url+ '?name=' + item.text"  />
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </a-form-item>
                </div>
                
                <a-form-item :wrapper-col="{ span: 16, offset: 6 }" class="subButton" >
                    <a-button type="primary" v-if="tabsKey == 'tabs1'" @click="nextTab()" class="margin-right-sm">下一步</a-button>
                    <a-button type="primary" @click="cancelTab()">取消</a-button>
                </a-form-item>
            </a-form>
        </div>
    </div>
</template>

<script>
    import { ProductListAPI,ProductCategoryListAPI,ProductKindListAPI,ProductTagListAPI,GetImageApi } from '../../../config/api'
    
    import {Warning} from "../../../common/mixin/Warning";  //公共类-警告
    import { SpecsParam } from "../../../common/mixin/EditGoods";  //公共类-商品详情
    import TextareaType from '../../common/Textarea'      //文本框

    import { mapState } from 'vuex'
    export default {
        mixins: [Warning,SpecsParam],
        name: "Edit",
        data() {
            return {
                dataLoading: true,      //loading加载状态
                tabsKey: 'tabs1',           //tabs页切换
                form: this.$form.createForm(this, { name: 'editGoods' }),
                labelCol: { span: 6 },
                wrapperCol: { span: 16 },
                /* 数据 */
                customForm:{
                    url: GetImageApi,     //图片路径
                    category_id: [],  //商品分类
                    kind_id: '',   //商品类型
                    name: '',  //商品名称
                    unit: '',  //商品单位
                    spec_type: '1',    //商品规格
                    status: 2,   //商品状态
                },
                /* 商品简介--文本框-数据 */
                textarea:{
                    text: '',
                    limitInput: 0,  //输入框限制
                    total: 100,  //文本输入最大长度
                    disabled: true,  //(true不可编辑，false:可以编辑)
                    isEmpty: false,   //是否为空（true是，false不是）
                },
    
                kindList:[],   //商品类型列表
                labelList:[],   //商品标签列表
                carousel:[],    /* 轮播图片 */
                phoneList:[],  //手机端描述列表
            }
        },
        created () {
            /* 面包屑 */
            this.$store.state.breadList = [
                {value: "商品管理", url: ""},
                {value: "商品列表", url: "/goods/index"},
                {value: "商品详情", url: ""},
            ];
            this.$store.state.breadValue = '商品详情';
        },
        computed: {
            ...mapState({
                classifyList: 'classifyList',  //分类列表
            })
        },
        mounted(){
            this.asyncFunc();   //初始化
        },
        components:{
            TextareaType
        },
        methods: {
            /* tabs切换页 */
            callback(key){
                this.tabsKey = key;
            },
            /* 下一个 */
            nextTab(){
                this.tabsKey = 'tabs2';
            },
            /* 取消 */
            cancelTab(){
                this.$router.push("/goods/index");
            },
           
            /* 初始化 */
            async asyncFunc(){
                this.dataLoading = true;
                await this.getClassify();  //商品分类列表
                await this.getKind();       //商品类型列表
                await this.getLabel();      //商品标签列表
                await this.loadData();  //商品详情
                this.dataLoading = false;
            },
            /* 商品类型--列表 */
            getKind(){
                return new Promise((resolve, reject) => {
                    let params = {
                        page_size: 999,
                        page: 1,
                    };
                    this.$http.get(ProductKindListAPI, {params}).then(resp => {
                        this.dataLoading = false;
                        if (resp.data.code == 1) {
                            let list = resp.data.data;
                            for (let item of list.kinds){
                                item.kind_id = item.kind_id.toString();
                            }
                            this.kindList = list.kinds;
                            resolve(true);
                        }
                    });
                });
            },
            /* 商品分类--列表 */
            getClassify(){
                return new Promise((resolve, reject) => {
                    if (this.classifyList.length > 0) {
                        resolve(true);
                        return false;
                    }
                    let params = {
                        page_size: 999,
                        page: 1,
                    };
                    this.$http.get(ProductCategoryListAPI, {params}).then(resp => {
                        this.dataLoading = false;
                        if (resp.data.code == 1) {
                           
                            let list = resp.data.data;
                            for (let item of list.categories) {
                                item.value = item.category_id;
                                item.label = item.name;
                            }
                            let array = this.authority.recursion(list.categories);
                            for (let item of array) {
                                if (item.type == 1) {
                                    this.classifyList.push(item);
                                }
                            }
                            resolve(true);
                        }
                    });
                });
            },
            /* 商品标签--列表 */
            getLabel(){
                return new Promise((resolve, reject) => {
                    let params = {
                        page_size: 999,
                        page: 1,
                    };
                    this.$http.get(ProductTagListAPI, {params}).then(resp => {
                        this.dataLoading = false;
                        if (resp.data.code == 1) {
                            this.labelList = [];
                            let list = resp.data.data;
                            for (let item of list.tags) {
                                let str = {};
                                str.value = item.tag_id;
                                str.label = item.name;
                                this.labelList.push(str);
                            }
                            resolve(true);
                        }
                    });
                });
            },
            /* 商品详情--数据 */
            loadData(){
                let params = {
                    page_size: 1,
                    page: 1,
                    product_id: this.$route.params.id,     //商品ID
                };
                this.$http.get(ProductListAPI, {params}).then(resp => {
                    if (resp.data.code == 1) {
                        let list = resp.data.data;
                        if(!list.products){
                            return false;
                        }
                        this.normsDataList = [];
                        let product = list.products[0];
                        this.customForm.category_id =  product.category_path.map(Number);  //商品分类
                
                        let kind_id = '';
                        for (let item of this.kindList){
                            if(product.kind_id == item.kind_id){
                                kind_id = product.kind_id.toString();
                            }
                        }
                        this.customForm.kind_id = kind_id;   //商品类型
                        if(kind_id){
                            this.setParams(this.customForm.kind_id);
                        } else {
                            this.form.validateFields(['kind_id',this.rules.kind_id],(errors, values)=>{});
                        }
                        this.customForm.name = product.name;  //商品名称
                        this.customForm.unit = product.unit;  //商品单位
                        this.customForm.status = product.status;  //商品状态
                
                        /* 商品简介 */
                        this.textarea.text = product.short_description;   //商品简介
                        this.textarea.limitInput = product.short_description.length;  //商品简介长度
                
                        /* 商品标签 */
                        this.customForm.tags = product.tags?product.tags:[];
                
                        /* 商品轮播 */
                        this.carousel = product.images?product.images:[];
                
                        /* 商品规格 */
                        this.customForm.spec_type = product.spec_type.toString();    //商品规格
                
                        let spec = product.spec?product.spec:[];
                        let specFocus = [];
                        for (let item of spec) {
                            specFocus = specFocus.concat(item.spec_value_id);
                        }
                        specFocus = Array.from(new Set(specFocus));
                
                        /* 计算商品规格选中 */
                        for (let i in this.normsData){
                            for (let t in this.normsData[i]){
                                if(specFocus.includes(this.normsData[i][t].id)){
                                    this.checkChange(i,t);
                                }
                            }
                        }
                
                        // console.log("多规格-商家列表：",JSON.stringify(this.normsData),JSON.stringify(this.normsDataList),JSON.stringify(this.manyNormsList),JSON.stringify(spec));
                        /* 计算商品规格列表--数据 */
                        let many = [];
                        console.log(spec[0].spec_value_id,spec[0].spec_value_id[0]);
                        if(spec.length == 1 && spec[0].spec_value_id[0] == 0){
                            many.push(spec[0]);
                        } else {
                            for (let value of spec) {
                                for (let item of this.manyNormsList) {
                                    let sen = value.spec_value_id.reduce((num, value) => {
                                        if (item.spec_value_id.includes(value)) {
                                            num++;
                                        }
                                        return num;
                                    }, 0);
                                    if (item.spec_value_id.length == sen) {
                                        let str = Object.assign(item, value);
                                        many.push(str);
                                    }
                                }
                            }
                        }
                        this.manyNormsList = many;
                
                        /* 商品参数 */
                        let paramTion = JSON.parse(product.param_description);
                        this.paramterData = {};
                        for (let value of paramTion){
                            let children = [];
                            for (let values in value.children){
                                children.push(value.children[values].content);
                            }
                            this.paramterData[value.name] = children.join('&nbsp;&nbsp;&nbsp;&nbsp;');
                        }
                        
                        
                
                        /* 手机端描述 */
                        this.phoneList = [];
                        for (let item of JSON.parse(product.description)){
                            let str = {
                                type: item.type, //类型(1文字,2图片)
                                status: 2,
                                focus: false,
                                text: item.text,
                            };
                            this.phoneList.push(str)
                        }
                    }
                })
            },
        }
    }
</script>

<style scoped>
    /* 手机端描述--start */
    .bewrite{
        width: 100%;
        height: 686px;
        position: relative;
        background-color: #656565;
    }
    .bewrite .phones{
        margin: auto;
        width: 375px;
        height: 667px;
        position: relative;
        top: 8px;
        background-color: #ffffff;
    }
    /** 文本 */
    .bewrite .phones .text{
        padding: 2px 12px 12px;
        width: 100%;
        height: 654px;
        line-height: 24px;
        overflow: auto;
    }
    .bewrite .phones .text .cursor{
        margin-top: 10px;
        position: relative;
    }
    .bewrite .phones .text .cursor .htmlText{
        padding: 4px 6px;
    }
    .bewrite .phones .text .cursor img{
        max-width: 100%;
    }
    /* 手机端描述--end */
    
    /* 规格--start */
    .norms{
        width: 100%;
        min-height: 100px;
        overflow-x: auto;
    }
    .norms .normsMeter{
        margin: 10px 0 20px;
        padding: 10px 0;
        width: 100%;
        background-color: #acc1e00d;
    }
    .normsMeter .meterList{
        width: 100%;
        display: grid;
        grid-template-columns: 90px auto;
        column-gap: 20px;
    }
    .meterList .meterName{
        text-align: right;
    }
    .meterList .normsText{
        width: 100%;
        display: grid;
        grid-template-columns: repeat(auto-fill,minmax(70px,118px));
        grid-template-rows: auto;
        column-gap: 10px;
    }
    .meterList .normsText label{
        white-space: nowrap;
        word-break: keep-all;
        text-overflow: ellipsis;
        overflow: hidden;
    }
    .norms table{
        min-width: calc(100% - 4px);
    }
    .norms>table thead{
        background-color: rgba(193, 193, 193, 0.24);
    }
    .norms>table td,.norms>table th{
        padding: 14px 10px 6px;
        line-height: 24px;
        border: 1px solid #dddddd;
    }
    /* 规格--end */
    
    /* 轮播图--start */
    .carousel{
        width: 100%;
        display: grid;
        grid-template-columns: repeat(5,100px);
        gap: 8px;
    }
    .carousel .img-album{
        padding: 2px;
        position: relative;
        width: 100px;
        height: 100px;
        border-radius: 4px;
        border: 1px dashed #d6c5c5;
    }
    .carousel .img-album img{
        width: 100%;
        height: 100%;
        border-radius: 4px;
    }
    .carousel .img-album .first{
        position: absolute;
        top: 0;
        left: 0;
        z-index: 2;
        width: 100%;
        height: 22px;
        line-height: 22px;
        text-align: center;
        display: inline-block;
        color: #f9f9f9;
        font-size: 12px;
        background-color: #5656569e;
    }
    .carousel .img-album .first:before{
        content: '*';
        margin-right: 4px;
        color: red;
    }
    .carousel .img-album .img-action{
        width: 100%;
        height: 24px;
        position: absolute;
        bottom: 0;
        left: 0;
        z-index: -1;
        display: grid;
        grid-template-columns: repeat(3,1fr);
        column-gap: 27px;
        color: #f9f9f9;
        line-height: 24px;
        background-color: #5656569e;
    }
    .carousel .img-album:hover .img-action{
        z-index: 1;
    }
    /* 轮播图--end */
    
    /* 提交按钮 */
    .subButton{
        margin-top: 50px;
    }
</style>
