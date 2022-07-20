<template>
    <div class="recent-activity-main-container">
        <div
            v-infinite-scroll="loadMore"
            infinite-scroll-disabled="loading"
            infinite-scroll-distance="10"
        >
            <mt-loadmore
                ref="loadmore"
                :top-method="pullDownRefresh"
                :topPullText="topPullText"
                :topDropText="topDropText"
                :topLoadingText="topLoadingText"
            >
                <div
                    v-for="(item, index) in regionList"
                    :key="index"
                    @click="toDetailPage(item)"
                >
                    <div class="recent-activity-view-container">
                        <div style="flex: 1">
                            <div class="recent-activity-title-container">
                                <div
                                    style="flex: 1; font: bold 16px PingFangSC"
                                >
                                    {{ item.activity_name }}
                                </div>
                                <a
                                    @click.stop="toDetail(item)"
                                    style="
                                        font: bold 16px PingFangSC;
                                        margin-right: 0;
                                    "
                                    :style="getJoinStyle(item.residents_id)"
                                >
                                    {{
                                        item.residents_id
                                            | joinState(
                                                residentInfo.residentsId
                                            )
                                    }}
                                </a>
                            </div>
                            <div class="recent-activity-info-container">
                                <div class="info-common-type">
                                    <img
                                        class="img"
                                        src="../img/i-time.png"
                                        alt=""
                                    />
                                    <span class="span">{{
                                        item.hold_time | timeFormate
                                    }}</span>
                                </div>
                                <div class="info-common-type">
                                    <img
                                        class="img"
                                        src="../img/i-position.png"
                                        alt=""
                                    />
                                    <span class="span">{{ item.address }}</span>
                                </div>
                                <div class="info-common-type">
                                    <img
                                        class="img"
                                        src="../img/i-username.png"
                                        alt=""
                                    />
                                    <span class="span">{{
                                        item.participateNum
                                    }}</span>
                                </div>
                            </div>
                        </div>
                    </div>

                    <div class="divider"></div>
                </div>
            </mt-loadmore>
        </div>

        <div v-if="(!regionList || regionList.length <= 0) && !isLoading">
            <img
                style="width: 100%; margin-top: 28px"
                src="../img/no_result.png"
            />
            <div style="text-align: center; font-weight: 600">
                没有找到相关结果
            </div>
        </div>
    </div>
</template>
<style scoped>
.recent-activity-main-container {
    background-color: #f5f5f5;
    display: flex;
    flex-direction: column;
}

.recent-activity-view-container {
    background-color: #ffffff;
    display: flex;
    padding: 12px 16px;
}

.recent-activity-title-container {
    font-weight: 600;
    display: flex;
    padding: 0;
    line-height: 24px;
}

.recent-activity-info-container {
    padding: 0;
}

.info-common-type {
    line-height: 20px;
    font: 14px PingFangSC;
    padding: 4px 0 0;
}
/* 
    .info-common-type::after{
        position:relative;
        content:'';
        width:0;
        height:100%;
        vertical-align:middle;
    } */
.info-common-type .span {
    vertical-align: middle;
}

.info-common-type .img {
    vertical-align: middle;
}

.divider {
    height: 1px;
    width: 100%;
}
</style>
<script>
import { bindNavbar } from 'egova/js-interface';
import { initBaseGData } from '../../hwPointManage/api/index';
import { Search, InfiniteScroll, Loadmore } from 'mint-ui';
import Vue from 'vue';
import api from '../api/index';
import pointApi from '../../hwPointManage/api/index';
import { parseDate, formatDateByDate } from 'egova/date-util';
import { Indicator } from 'egova-ui';
import { getRequestParam, resetObject } from 'egova/util';
import { MessageBox as iMessageBox } from 'egova-ui';
import { Toast } from 'mint-ui';
import ActionSheet from 'egova/components/ActionSheet.vue';
import { gData } from 'egova/data';
import { getServerUrl } from 'egova/http';
import TreeActionSheet from '../../hwPointManage/view/TreeActionSheet';

import pubApi, {
    thirdLogin,
    getBaseConfig,
} from '../../lib/egova/api/hwCommon';
Vue.use(InfiniteScroll);
Vue.component(Loadmore.name, Loadmore);

function defaultData() {
    return {
        isLoading: false,
        wxGarbageDetailPath: '/pages/main/homeGarbage', // 跳转详情的地址，smt传过来
        selectRegionText: gData.regionName,
        regionId: null,
        regionType: null,
        originalRegionType: null, // 原始的regionType
        basicUnitFlag: null,
        topPullText: '加载更多',
        topDropText: '加载更多',
        topLoadingText: '加载更多',
        hasMore: true,
        regionList: [],
        currentPage: 1,
        numPerPage: 15,
        canLoading: false,
        isRefresh: false,
        selectPointName: null,
        unitTypeId: null,
        unitList: [],
        regionFlag: 0,
        smtUserId: '',
        residentInfo: {},
    };
}
export default {
    name: 'regionList',
    components: {},
    data() {
        return defaultData();
    },
    computed: {},
    created() {},
    mounted() {},
    filters: {
        joinState: function (value, residentsId) {
            if (value) {
                value = value + '';
                console.log('value : ' + value);
                let ids = value.split(',');
                const content = ids.find((id) => id == residentsId);
                if (content) {
                    return '已报名';
                }
            }
            return '我要报名';
        },
        timeFormate: function (value) {
            return formatDateByDate(value, 'yyyy-MM-dd');
        },
    },
    methods: {
        getJoinStyle(value) {
            // item.residents_id && item.residents_id == residentInfo.residentsId ? 'color: #7e8c8d;':'color: #0091fa;'
            if (value) {
                value = value + '';
                console.log(
                    'value : ' +
                        JSON.stringify(value) +
                        ',,' +
                        this.residentInfo.residentsId
                );
                let ids = value.split(',');
                const content = ids.find(
                    (id) => id == this.residentInfo.residentsId
                );
                console.log('content' + content);
                if (content) {
                    return 'color: #7e8c8d;';
                }
            }
            return 'color: #0091fa;';
        },
        toDetailPage(item) {
            console.log(JSON.stringify(item));
            this.$router.push({
                name: 'DetailPage',
                params: {
                    id: item.activity_id,
                },
            });
        },
        getResidents() {
            pubApi.getHWResidentBySmtUserId(this.smtUserId).then((data) => {
                console.log('查询到居民信息 -- ' + JSON.stringify(data));
                if (data && data.success) {
                    this.residentInfo = data.data.result || {};
                    this.getRegionList();
                } else {
                    console.log(JSON.stringify(data));
                    if (data.code == 501) {
                        // todo 新增居民信息
                        pubApi.getSmtUserInfo(this.smtUserId).then((data) => {
                            // 获取到市民通user信息，再构建一个环卫居民，然后新增居民
                            console.log(
                                'getSmtUserInfo -- ' + JSON.stringify(data)
                            );
                            if (data && data.success) {
                                // let realName = data.data.userInfo.realName || "";
                                let userName =
                                    data.data.userInfo.userName || '';
                                let phone = data.data.userInfo.phone || '';
                                let resident = {
                                    residentsId: null,
                                    residentsName: userName,
                                    phoneNumber: phone,
                                    smtUserId: this.smtUserId,
                                };
                                pubApi
                                    .saveResidentInfo(resident)
                                    .then((data) => {
                                        if (data && data.success) {
                                            this.residentInfo =
                                                data.data.result;
                                            this.getRegionList();
                                        }
                                    });
                            } else {
                                Toast('获取市民通用户数据出错！');
                            }
                        });
                    } else {
                        Toast(data.message);
                    }
                }
            });
        },
        init() {
            //根据humanID获取token
            console.log('gData.token ?? ' + gData.token);
            if (!gData.token) {
                thirdLogin()
                    .then((data) => {
                        gData.token = data;
                        this.getResidents();
                    })
                    .catch((err) => {
                        Toast('获取token失败：' + JSON.stringify(err));
                        console.log('获取token失败：' + JSON.stringify(err));
                    });
            } else {
                pubApi.getHWResidentBySmtUserId(this.smtUserId).then((data) => {
                    console.log('查询到居民信息 -- ' + JSON.stringify(data));
                    if (data && data.success) {
                        this.residentInfo = data.data.result || {};
                        this.getRegionList();
                    } else {
                        console.log(JSON.stringify(data));
                        if (data.code == 501) {
                            // todo 新增居民信息
                            pubApi
                                .getSmtUserInfo(this.smtUserId)
                                .then((data) => {
                                    // 获取到市民通user信息，再构建一个环卫居民，然后新增居民
                                    console.log(
                                        'getSmtUserInfo -- ' +
                                            JSON.stringify(data)
                                    );
                                    if (data && data.success) {
                                        // let realName = data.data.userInfo.realName || "";
                                        let userName =
                                            data.data.userInfo.userName || '';
                                        let phone =
                                            data.data.userInfo.phone || '';
                                        let resident = {
                                            residentsId: null,
                                            residentsName: userName,
                                            phoneNumber: phone,
                                            smtUserId: this.smtUserId,
                                        };
                                        pubApi
                                            .saveResidentInfo(resident)
                                            .then((data) => {
                                                if (data && data.success) {
                                                    this.residentInfo =
                                                        data.data.result;
                                                    this.getRegionList();
                                                }
                                            });
                                    } else {
                                        Toast('获取市民通用户数据出错！');
                                    }
                                });
                        } else {
                            Toast(data.message);
                        }
                    }
                });
            }
        },
        onselectUnit(selectedList) {
            let selectUnit = selectedList.pop();
            this.regionId = selectUnit.value;
            this.selectRegionText = selectUnit.text;
            if (
                selectUnit.attributes &&
                selectUnit.attributes.data &&
                selectUnit.attributes.data.regionType
            ) {
                this.regionType = selectUnit.attributes.data.regionType;
            } else {
                this.regionType = this.originalRegionType;
            }
            if (this.regionType < 4) {
                this.regionType = parseInt(parseInt(this.regionType) + 1);
                this.basicUnitFlag = 0;
            } else {
                this.basicUnitFlag = 1;
            }
            this.refresh();
        },
        getRegionList() {
            let that = this;
            Indicator.open();
            api.getActivityList(
                this.residentInfo.residentsId,
                this.currentPage,
                this.numPerPage
            ).then(
                (data) => {
                    Indicator.close();
                    that.isLoading = false;
                    if (data && data.success) {
                        let dataList =
                            (data.data && data.data.resultList) || [];
                        let pageInfo =
                            (data.data && data.data.pageInfo) || null;
                        if (that.isRefresh) {
                            that.regionList = dataList;
                            that.isRefresh = false;
                        } else {
                            let list = that.regionList;
                            list = list.concat(dataList);
                            that.regionList = list;
                        }
                        that.currentPage = that.currentPage + 1;
                        if (pageInfo) {
                            if (
                                that.regionList.length >= pageInfo.totalRecord
                            ) {
                                that.loading = false;
                                that.canLoading = false;
                            } else {
                                that.loading = true;
                                that.canLoading = true;
                            }
                        }
                    }
                },
                (error) => {
                    // Indicator.close();
                    that.isLoading = false;
                }
            );
        },
        pullDownRefresh() {
            this.refresh();
            this.$refs.loadmore.onTopLoaded();
        },
        loadMore() {
            if (this.canLoading) {
                var that = this;
                that
                    .getRegionList
                    // this.regionId,
                    // this.regionType,
                    // this.basicUnitFlag,
                    // this.selectPointName,
                    // this.unitTypeId,
                    // that.currentPage,
                    // that.numPerPage
                    ();
            }
        },
        refresh() {
            var that = this;
            that.currentPage = 1;
            that.isRefresh = true;
            that.getRegionList(
                this.regionId,
                this.regionType,
                this.basicUnitFlag,
                this.selectPointName,
                this.unitTypeId,
                that.currentPage,
                that.numPerPage
            );
        },
        // 跳转详情页
        toDetail(item) {
            if (item.residents_id) {
                let value = item.residents_id;
                if (value) {
                    value = value + '';
                    let ids = value.split(',');
                    const content = ids.find(
                        (id) => id == this.residentInfo.residentsId
                    );
                    console.log('content' + content);
                    if (content) {
                        Toast('已成功报名');
                        return;
                    }
                }
            }
            iMessageBox.confirm({
                vm: this,
                title: '报名确认',
                message:
                    "<div style='font: 14px PingFangSC;line-height: 20px'>" +
                    "<span style='margin-right: 8px'>活动时间</span><span style='color: #000000;'>" +
                    formatDateByDate(item.hold_time, 'yyyy-MM-dd') +
                    '</span>' +
                    '<br>' +
                    "<span style='margin-right: 8px'>活动地点</span><span style='color: #000000;'>" +
                    item.address +
                    '</span>' +
                    '</div>',
                confirmText: '确认',
                cancelText: '取消',
                onConfirm: () => {
                    //   api.releaseAlarm(item.alarmId).then(data=>{
                    //      if(data && data.success){
                    //          this.refresh();
                    //      }
                    //   })
                    this.joinActivity(item);
                },
                onCancel: () => {},
                closeOnModal: false,
            });
        },
        joinActivity(activity) {
            api.residentsparticipate(
                this.residentInfo.residentsId,
                activity.activity_id
            ).then((data) => {
                console.log('报名结束 ： ' + JSON.stringify(data));
                if (data && data.success) {
                    this.refresh();
                } else {
                    Toast('活动报名失败：' + data.message);
                }
            });
        },
    },
    beforeRouteEnter(to, from, next) {
        next((vm) => {
            if (!vm.regionList || vm.regionList.length < 1) {
                getBaseConfig(vm);
                vm.smtUserId = gData.smtUserCode;
                vm.init();
            }
            bindNavbar({
                title: '近期活动',
            });
        });
    },
};
</script>
