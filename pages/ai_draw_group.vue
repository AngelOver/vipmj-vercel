<template>
    <div class="ai_draw_ground">

        <main class=" main overflow-auto">
            <div class="draw_bg">
              <iframe style="border-width: 0;height: 100%;position: absolute;top: 0;width: 100%" scrolling="no" src="https://qiniuchat.littlewheat.com/static/io-fx10.html"></iframe>
                <span class="bot_boxshadow"></span>
                <div class="container pt-44">
                    <h1 class="header_title text-center mb-5">
                      华丽的艺术作品
                    </h1>

                    <a-space class="justify-center w-full" style="position: relative;">
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
                <div class="draw_group pb-10" v-if="public_draw?.length > 0">
                    <a-row :gutter="20" >
                        <waterFull
                            :images="public_draw"
                            :columnCount="columnCount"
                            :columnGap="10"
                            width="100%"
                            mode="JS"
                            backgroundColor="#F2F4F8"
                            @get_new_public="get_new_public"
                        />
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
            </div>
        </main>
    </div>
</template>
<script setup lang="ts">
import  {ref,onMounted} from "vue";
import {useCounter} from "~/store/counter";
import {
    IconArrowFall,
    IconArrowRise,
    IconRefresh,
    IconDown,
    IconHighlight,
    IconThunderbolt,
    IconImage
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
//definePageMeta({
//middleware: ['mustlogin']
//})
const onResize = () => {
    const screenWidth = window.innerWidth
    if (screenWidth < 768) {
        columnCount.value = 2
    }else {
        columnCount.value = 4
    }
}
if (process.client) {
    onResize()
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
const get_new_public = () => {
    load_more_loading.value = true
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
    }).catch((err: any) => {
        console.log(err)
        load_more_loading.value = false
    })
}
get_new_public()
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
