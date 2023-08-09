<template>
    <div class="ai_draw_ground">
      <main class=" main overflow-auto" id="ai_back">
            <div class="draw_bg">
              <iframe style="border-width: 0;height: 100%;position: absolute;top: 0;width: 100%" scrolling="no" src="https://qiniuchat.littlewheat.com/static/io-fx10.html"></iframe>

                <span class="bot_boxshadow"></span>
                <div class="container pt-44" style="position: relative;">
                    <h1 class="header_title text-center mb-5">
                      华丽的艺术作品
                    </h1>
                    <a-space class="justify-center w-full">
                        <a-select v-model="draw_cate" size="large" placeholder="请选择..." @change="cate_change">
                            <a-option value="all_draw" >全部</a-option>
                            <a-option :value="cate.other" v-for="(cate,c_index) in draw_cate_all" :key="c_index">{{ cate.title }}</a-option>
                        </a-select>
                        <a-input-search
                            class="search_input"
                            placeholder="搜索绘画关键词"
                            enter-button="搜索"
                            size="large"
                            :disabled="search_loading"
                            :loading="search_loading"
                            v-model="search_title"
                            @search="onSearch"
                            @keydown.enter.native="onSearch()"
                        />
                    </a-space>

                </div>

            </div>
            <div class="public_draw_list container">
                <div class="list_cate_all">
                    <div class="public_draw_cate_list">
                        <div class="pub_ul flex" ref="ulRef" :class="list_show?'drop_downof':''">
                            <div ref="liRefs" class="data_cate shadow-sm" @click="change_cate('all_draw')" :class="draw_cate=='all_draw'?'active':''">
                                <img src="@/assets/images/draw_all.png" data-style="">
                                <p>全部</p>
                            </div>
                            <div  v-for="(item,index) in draw_cate_all" ref="liRefs" class="data_cate shadow-sm" @click="change_cate(item.other)" :class="item.other==draw_cate?'active':''">
                                <img :src="item.icon" data-style="">
                                <p>{{ item.title }}</p>
                            </div>

                        </div>

                    </div>
                    <div class="data_cate_fall shadow-sm" v-if="showExpandButton" @click="expandList">
                        <icon-arrow-fall v-if="!list_show" />
                        <icon-arrow-rise  v-else />
                    </div>
                </div>
                <div class="draw_select flex justify-between">
                    <div class="left_select">
                        <a-space>
                            <h5 class="cursor-pointer mr-4" :class="select_d=='new'?'active':''" @click="select_change('new')"><icon-image />最新绘画</h5>
                            <h5 class="cursor-pointer" :class="select_d=='hot'?'active':''" @click="select_change('hot')"><icon-thunderbolt />热门绘画</h5>
                        </a-space>
                    </div>
                    <div class="right_select">
                        <a-dropdown trigger="hover" :popup-max-height=false @select="change_bili">
                            <div class="cursor-pointer">
                                {{ search_bili(bili_check) }} <icon-down />
                            </div>
                            <template #content>
                                <a-doption v-for="(b,b_index) in bili" :key="b_index" :value="b.value">{{b.name}}</a-doption>
                            </template>
                        </a-dropdown>
                    </div>
                </div>
                <client-only>
                <a-spin dot :loading="change_load">
                    <div class="draw_group pb-10" v-if="public_draw?.length > 0">

                            <Waterfall :list="public_draw" :loadProps="loadProps" :crossOrigin=false :width=315 :hasAroundGutter=false :breakpoints="breakpoints">
                                <template #item="{ item, url, index }">
                                    <div class="water_fall u-img relative item list-animation-bottomIn"
                                         :class="'delay-'+index"
                                         @click="draw_dialog_check(index)">
                                        <LazyImg :url="item.image" :loadProps="loadProps" />
                                        <div class="bottom_draw_info">
                                            <h5 class="text-white text-truncate mb-1">{{item.name}}</h5>
                                            <p class="text-white text-truncate mb-1 opacity-70">
                                                <a-avatar :size="24">
                                                    <img :src="item.get_userinfo.avatar" />
                                                </a-avatar>
                                                <span class="ml-1">{{ item.get_userinfo.name }}</span>

                                            </p>
                                            <p class="text-white text-truncate opacity-70">{{item.created_at}}</p>

                                        </div>
                                        <div class="round_bottom">
                                            <div class="num_info cursor-pointer mr-2" @click="send_drawlike(item.id,'star')">
                                                <icon-heart />
                                                <span class="ml-1">
                                            {{item.ai_drawlike_count}}
                                        </span>
                                            </div>
                                            <div class="num_info cursor-pointer" @click="send_drawlike(item.id,'like')">
                                                <icon-thumb-up />
                                                <span class="ml-1">
                                            {{item.like_count}}
                                        </span>
                                            </div>
                                        </div>
                                    </div>


                                </template>
                            </Waterfall>

                        <a-row :gutter="20" >
                            <div class="mt-10 flex justify-center w-full mb-4">
                                <a-button class="load_more" :loading="load_more_loading" @click="load_more()">
                                    <template #icon>
                                        <icon-refresh />
                                    </template>
                                    加载更多
                                </a-button>
                            </div>

                        </a-row>

                    </div>
                    <a-empty description="暂无数据哦~" v-else />
                </a-spin>
                </client-only>
                <a-back-top target-container="#ai_back">
                    <a-button shape="circle" class="bg-white">
                        <icon-arrow-rise />
                    </a-button>
                </a-back-top>
            </div>
        </main>
    </div>
    <a-modal :header=false :footer=false v-model:visible="draw_dialog" simple class="draws_dialog">
        <div class="flex draw_all_info h-100 overflow-hidden">
            <div class="left_draw_image text-center ">
                <a-image
                    :src="draw_info.image"
                />
                <div class="draw_dialog_left_point" @click="change_nextor_prev('prev')">
                    <div class="sc-cVtpRj dLvaiq" style="width: 28px; height: 28px; transform: rotate(180deg);">
                        <div class="baseIcon">
                            <svg width="28" height="28" viewBox="0 0 28 28" fill="none"
                                 xmlns="http://www.w3.org/2000/svg">
                                <path d="M9.02081 26.0416L21.0416 14.0208C16.3472 9.32639 13.7152 6.69442 9.02081 2"
                                      stroke="white" stroke-width="4"></path>
                            </svg>
                        </div>
                    </div>
                </div>
                <div class="draw_dialog_right_point" @click="change_nextor_prev('next')">
                    <div class="sc-cVtpRj dLvaiq" style="width: 28px; height: 28px;">
                        <div class="baseIcon">
                            <svg width="28" height="28" viewBox="0 0 28 28" fill="none"
                                 xmlns="http://www.w3.org/2000/svg">
                                <path d="M9.02081 26.0416L21.0416 14.0208C16.3472 9.32639 13.7152 6.69442 9.02081 2"
                                      stroke="white" stroke-width="4"></path>
                            </svg>
                        </div>
                    </div>
                </div>
            </div>
            <div class="right_draw_info p-4 flex-1">
                <h4>作品介绍</h4>
                <div class="flex right_all_info flex-column justify-between">
                    <div class="draw_info_item">
                        <div class="draw_dialog_card">
                            <div class="draw_dialog_card_title">
                                <h3>生成描述词</h3>
                                <div class="draw_dialog_card_icon"
                                     @click="code_copy()"
                                     style="width: 20px; height: 20px; position: relative; opacity: 0.5;">
                                    <icon-copy />
                                </div>
                            </div>
                            <p class="draw_dialog_card_des">
                                {{draw_info.title}}
                            </p>
                        </div>
                        <div class="draw_dialog_card">
                            <div class="draw_dialog_card_title">
                                <p>画面尺寸</p>
                                <p>
                                    {{draw_info.width}} x {{draw_info.height}}
                                </p>
                            </div>
                            <div class="draw_dialog_card_title">
                                <p>模型选择</p>
                                <p>
                                    {{draw_info.text_model}}
                                </p>
                            </div>
                            <div class="draw_dialog_card_title">
                                <p>图片后缀</p>
                                <p>
                                    {{draw_info.image_type}}
                                </p>
                            </div>
                            <div class="draw_dialog_card_title">
                                <p>日期</p>
                                <p>
                                    {{draw_info.created_at}}
                                </p>
                            </div>
                        </div>
                    </div>
                    <div class="author_info">
                        <div class="bottom_draw_info">
                            <h3 class="text-truncate mb-2">{{draw_info.name}}</h3>
                            <p class="text-truncate mb-2">
                                <a-avatar :size="30">
                                    <img :src="draw_info.get_userinfo?draw_info.get_userinfo.avatar:''" />
                                </a-avatar>
                                <NuxtLink :to="'/member/'+(draw_info.get_userinfo?draw_info.get_userinfo.id:0)" class="ml-1 text-gray-900">{{ draw_info.get_userinfo?draw_info.get_userinfo.name:'未命名' }}</NuxtLink>

                            </p>
                            <a-button-group class="w-full mb-2">
                                <a-button class="flex-1" type="primary" status="danger"  @click="onDownLoad(draw_info.image)">
                                    <icon-download />
                                    下载
                                </a-button>
                                <a-button class="flex-1" type="primary" status="warning" @click="send_drawlike(draw_info.id,'star')">
                                    <icon-star />
                                    收藏 {{draw_info.ai_drawlike_count}}

                                </a-button>
                                <a-button class="flex-1" type="primary" status="success"  @click="send_drawlike(draw_info.id,'like')">
                                    <icon-thumb-up />
                                    点赞 {{draw_info.like_count}}
                                </a-button>
                            </a-button-group>

                        </div>
                    </div>
                </div>

            </div>
        </div>
    </a-modal>

</template>
<script setup lang="ts">
import  {ref,onMounted} from "vue";
import {useCounter} from "~/store/counter";
import {
    IconArrowFall,
    IconArrowRise,
    IconRefresh,
    IconDown,
    IconThunderbolt,
    IconImage, IconThumbUp, IconHeart, IconCopy, IconDownload, IconStar,
} from "@arco-design/web-vue/es/icon";
import {Message} from "@arco-design/web-vue";
const counter = useCounter()
useHead({
    title: 'AI绘画广场' + counter.setting.title,
    meta: [
        {name: 'description', content: counter.setting.description},
        {name: 'keywords', content: counter.setting.keywords}
    ],

})
import { LazyImg, Waterfall } from 'vue-waterfall-plugin-next'
import 'vue-waterfall-plugin-next/dist/style.css'
const new_draw_bg = ref('')
const draw_cate = ref('all_draw')
const ulRef = ref(null);
const liRefs = ref([]);
const showExpandButton = ref(false);
const list_show = ref(false);

const public_draw = ref([])
const public_draw_count = ref(0)
const public_page = ref(1)
const public_page_size = ref(20)
const load_more_loading = ref(false)
const select_d = ref('new')
const draw_cate_all = ref([])
const search_title = ref('')
const columnCount = ref(4)
definePageMeta({
  middleware: ['mustlogin']
})
const breakpoints = {
    1200: { //当屏幕宽度小于等于1200
        rowPerView: 4,
    },
    800: { //当屏幕宽度小于等于800
        rowPerView: 3,
    },
    768: { //当屏幕宽度小于等于500
        rowPerView: 2,
    },
    480: { //当屏幕宽度小于等于480
        rowPerView: 1,
    },
}
import loading from '../assets/images/loading.gif'
import error from '../assets/images/loading.gif'

const loadProps= {
    loading: loading,
    error: error,
}
const send_drawlike = (id:number,type:any)=>{
    // 点赞或收藏
    draw_star({
        type:type,
        id:id
    }).then((res:any)=>{
        Message.success(res._rawValue.message)
    }).catch((err:any)=>{
        console.log(err)
    })
}
const draw_info = ref({})
const now_index = ref(0)
const draw_dialog = ref(false)
const draw_dialog_check = (item: any) => {
    now_index.value = item
    draw_info.value = public_draw.value[now_index.value]
    const img = new Image()
    img.src = draw_info.value.image
    img.onload = () => {
        draw_info.value.width = img.width
        draw_info.value.height = img.height
    }

    if (draw_info.value.title.indexOf('v 5') !== -1) {
        draw_info.value.text_model = 'Midjourney'
    } else if(draw_info.value.title.indexOf('niji') !== -1){
        draw_info.value.text_model = 'Niji'
    }else if(draw_info.value.type=='dall-e'){
        draw_info.value.text_model = 'Dall-E'
    }
    const suffix = draw_info.value.image.split('.').pop()
    draw_info.value.image_type = suffix
    draw_dialog.value = true
}
const change_nextor_prev = (type:any)=>{
    if (type=='prev'){
        if (now_index.value==0){
            now_index.value = public_draw.value.length-1
        }else{
            now_index.value = now_index.value-1
        }
    }else{
        if (now_index.value==public_draw.value.length-1){
            now_index.value = 0
        }else{
            now_index.value = now_index.value+1
        }
    }

    draw_info.value = public_draw.value[now_index.value]
    const img = new Image()
    img.src = draw_info.value.image
    img.onload = () => {
        draw_info.value.width = img.width
        draw_info.value.height = img.height
    }

    if (draw_info.value.title.indexOf('v 5') !== -1) {
        draw_info.value.text_model = 'Midjourney'
    } else if(draw_info.value.title.indexOf('niji') !== -1){
        draw_info.value.text_model = 'Niji'
    }else{
        draw_info.value.text_model = '未知'
    }
    const suffix = draw_info.value.image.split('.').pop()
    draw_info.value.image_type = suffix

    draw_dialog.value = true
}
const onResize = () => {
    const screenWidth = window.innerWidth

}
if (process.client) {
    onResize()
}
onMounted(() => {
    // window.addEventListener('resize', onResize)
})


const code_copy = () => {
    navigator.clipboard.writeText(draw_info.value.title).then(() => {
        Message.success('复制成功')
    }, () => {
        Message.error('复制失败')
    });
}
const get_draw_cate = () => {
    draw_cate_f().then((res: any) => {
        draw_cate_all.value = res._rawValue.data
    }).catch((err: any) => {
        console.log(err)
    })
}
get_draw_cate()

const bili_check = ref(1)
const bili = [
    {
        name: '全部',
        value: 1,
    },
    {
        name: '1:1',
        value: 2,
    },
    {
        name: '4:3',
        value: 3,
    },
    {
        name: '3:4',
        value: 4,
    },
    {
        name: '16:9',
        value: 5,
    },
    {
        name: '9:16',
        value: 6,
    },

]
const select_change = (type: string) => {
    select_d.value = type
    get_new_public()
}
const visible1 = ref(false)
const change_load = ref(false)
const get_new_public = () => {
    load_more_loading.value = true
    change_load.value = true
    get_public_draw({
        page: public_page.value,
        limit: public_page_size.value,
        type: select_d.value,
        search: search_title.value,
        cate_other: draw_cate.value,
        bili: bili_check.value,
    }).then((res: any) => {
        public_draw.value = res._rawValue.data
        public_draw_count.value = res._rawValue.count
        load_more_loading.value = false
        change_load.value = false

    }).catch((err: any) => {
        console.log(err)
        load_more_loading.value = false
        change_load.value = false
    })
}
get_new_public()
const onDownLoad = (imgsrc:any)=>{
    // 下载当前图片
    const image = new Image()
    image.src = imgsrc
    image.onload = function () {
        const a = document.createElement('a')
        a.href = imgsrc
        a.download = '图片'
        a.click()
    }
}
onMounted(async () => {
    await nextTick(); // 等待DOM渲染完成
    const ul = ulRef.value;
    const lis = liRefs.value;

    let totalWidth = 0;
    for (let i = 0; i < lis.length; i++) {
        totalWidth += lis[i].offsetWidth;
    }
    showExpandButton.value = totalWidth > ul.offsetWidth;
    new_draw_bg.value = counter.setting.draw_bg
});
const load_more = () => {
    public_page_size.value +=20
    get_new_public()
}
const expandList = () => {
    list_show.value = !list_show.value;
};
const search_loading = ref(false)
const onSearch = ()=>{
    if (search_title.value==''){
        Message.warning('请输入搜索内容')
        return false
    }
    search_loading.value = true
    get_new_public()
    search_loading.value = false
}

const change_cate = (other: any) => {
    draw_cate.value = other
    get_new_public()
}

const cate_change = (cate: any) => {
    draw_cate.value = cate
    get_new_public()
}

const search_bili = (bili_id: any) => {
    for (let i = 0; i < bili.length; i++) {
        if (bili[i].value == bili_check.value) {
            bili_check.value = bili[i].value
            return bili[i].name
        }
    }
    get_new_public()
}

const change_bili = (value: any) => {
    bili_check.value = value
    get_new_public()
}
</script>
<style>
.draw_group .lazy__img[lazy=loading] {
    padding: 5em 0;
    width: 100%;
}

.lazy__img[lazy=loaded] {
    width: 100%;
}

.lazy__img[lazy=error] {
    padding: 5em 0;
    width: 48px;
}
</style>
