<template>
    <view :class="theme_view">
        <view class="padding-main">
            <view v-if="detail != null">
                <form @submit="formSubmit" class="form-container">
                    <view v-for="(item, index) in detail" :key="index" class="form-gorup oh">
						<block v-if="plugins_name == 'wallet'">
							<view class="oh">
								<view class="item-base fl margin-top-sm">
									<view>充值单号：{{item.recharge_no}}</view>
									<view>充值金额：{{item.money}}</view>
								</view>
							</view>
						</block>
						<block v-else>
							<view class="oh">
								<navigator hover-class="none">
									<image class="goods-image fl radius br margin-right-lg" :src="item.items[0].images" mode="aspectFill"></image>
								</navigator>
								<view class="item-base fl margin-top-sm">
									<view>订单号：{{item.order_no}}</view>
									<view>订单价：{{item.total_price}}</view>
								</view>
							</view>
						</block>
						<view class="margin-top-main br-t">
							<view class="form-container-upload oh">
																  
								<view v-if="(form_images_list[index] || null) != null && form_images_list[index].length > 0" class="form-upload-data fl margin-top-main">
									<view v-for="(fx, ix) in form_images_list[index]" :key="ix" class="item fl">
										<!--<text class="delete-icon" @tap="upload_delete_event" :data-index="index" :data-ix="ix">x</text>-->
										<image :src="fx" @tap="upload_show_event" :data-index="index" :data-ix="ix" mode="aspectFill" class="padding-xs dis-block"></image>
									</view>
											
								</view>
								<view v-else class="form-upload-data fl margin-top-main">
									<image class="upload-icon" :src="common_static_url+'upload-icon.png'" mode="aspectFill" @tap="file_upload_event" :data-index="index"></image>
								</view>
							</view>
						</view>
                    </view>
					<block v-if="is_all_up > 0">
						<view class="form-gorup anonymous">
							<text class="cr-grey margin-top-lg">凭证一旦提交，不支持修改，请谨慎操作，以免影响付款审核！（如有多个订单一起付款，请每个订单均上传一次凭证）</text>
						</view>

						<view class="form-gorup form-gorup-submit">
							<button form-type="submit" class="bg-main br-main cr-white round text-size" type="default" hover-class="none" :disabled="form_button_disabled">提交</button>
						</view>
					</block>
                </form>
            </view>
            <view v-else>
                <!-- 提示信息 -->
                <component-no-data :propStatus="data_list_loding_status" :propMsg="data_list_loding_msg"></component-no-data>
            </view>
        </view>
    </view>
</template>
<script>
    const app = getApp();
    import componentNoData from "../../components/no-data/no-data";
    import componentBottomLine from "../../components/bottom-line/bottom-line";

    var common_static_url = app.globalData.get_static_url('common');
    export default {
        data() {
            return {
				theme_view: app.globalData.get_theme_value_view(),
                common_static_url: common_static_url,
                data_list_loding_status: 1,
                data_list_loding_msg: '',
                params: null,
                detail: null,
                editor_path_type: '',
                form_orderid_list: [],
                form_images_list: [],
                form_button_disabled: false,
				is_all_up:1,
				plugins_name:'',
				backurl:'',
            };
        },
        components: {
            componentNoData,
            componentBottomLine
        },
        props: {},

        onLoad(params) {
            //params['goods_id']=9;
            this.setData({
                params: params
            });
			this.init();
        },
        
        onShow() {
        },

        // 下拉刷新
        onPullDownRefresh() {
            this.init();
        },

        methods: {
            // 初始化
			init() {
			    var self = this;
			    uni.showLoading({
			        title: "加载中..."
			    });
			    this.setData({
			        data_list_loding_status: 1
			    });
			    uni.request({
			        url: app.globalData.get_request_url("index", "index", "bankext"),
			        method: "POST",
			        data: {
			            ids: this.params.ids,
						business_type: this.params.business_type || ''
			        },
			        dataType: "json",
			        success: res => {console.log(res);
			            uni.hideLoading();
			            uni.stopPullDownRefresh();
			            if (res.data.code == 0) {
			                var data = res.data.data;
							// 得到订单集合
							var temp_data_list = [];
							var temp_img_list = [];
							var img_t = [];
							var temp_data = res.data.data.data;
							for (var i in temp_data) {
							    temp_data_list.push(temp_data[i].id);
								
								img_t = [];
								if(temp_data[i].pingzheng_img != ''){
									img_t.push(temp_data[i].pingzheng_img);
								}
								temp_img_list.push(img_t);
							}
			                self.setData({
			                    editor_path_type: data.editor_path_type || '',
			                    detail: data.data,
								form_orderid_list:temp_data_list,
								form_images_list:temp_img_list,
			                    data_list_loding_status: 3,
			                    data_list_loding_msg: '',
								is_all_up:data.is_all_up,
								plugins_name: data.plugins_name,
								backurl: data.backurl
			                });
			            } else {
			                self.setData({
			                    data_list_loding_status: 2,
			                    data_list_loding_msg: res.data.msg
			                });
			                if (app.globalData.is_login_check(res.data, self, 'init')) {
			                    app.globalData.showToast(res.data.msg);
			                }
			            }
			        },
			        fail: () => {
			            uni.hideLoading();
			            uni.stopPullDownRefresh();
			            self.setData({
			                data_list_loding_status: 2,
			                data_list_loding_msg: '服务器请求出错'
			            });
			            app.globalData.showToast("服务器请求出错");
			        }
			    });
			},

            // 上传图片预览
            upload_show_event(e) {
                var index = e.currentTarget.dataset.index;
                var ix = e.currentTarget.dataset.ix;
                uni.previewImage({
                    current: this.form_images_list[index][ix],
                    urls: this.form_images_list[index]
                });
            },
            
            // 图片删除
            upload_delete_event(e) {
                var index = e.currentTarget.dataset.index;
                var ix = e.currentTarget.dataset.ix;
                var self = this;
                uni.showModal({
                    title: '温馨提示',
                    content: '删除后不可恢复、继续吗？',
                    success(res) {
                        if (res.confirm) {
                            var list = self.form_images_list;
                            list[index].splice(ix, 1);
                            self.setData({
                                form_images_list: list
                            });
                        }
                    }
                });
            },
            
            // 文件上传
            file_upload_event(e) {
                // 数据初始化
                var index = e.currentTarget.dataset.index;
                var temp_list = this.form_images_list;
                var length = this.detail.length;
                for (var i = 0; i < length; i++) {
                    if (temp_list[i] == undefined) {
                        temp_list[i] = [];
                    }
                }
                this.setData({
                    form_images_list: temp_list
                });

                // 处理上传文件
                var self = this;
                uni.chooseImage({
                    count: 1,
                    success(res) {
                        var success = 0;
                        var fail = 0;
                        var length = res.tempFilePaths.length;
                        var count = 0;
                        self.upload_one_by_one(index, res.tempFilePaths, success, fail, count, length);
                    }
                });
            },
            
            // 采用递归的方式上传多张
            upload_one_by_one(index, img_paths, success, fail, count, length) {
                var self = this;
                if ((self.form_images_list[index] || null) == null || self.form_images_list[index].length < 1) {
                    uni.uploadFile({
                        url: app.globalData.get_request_url("index", "ueditor"),
                        filePath: img_paths[count],
                        name: 'upfile',
                        formData: {
                            action: 'uploadimage',
                            path_type: self.editor_path_type
                        },
                        success: function(res) {
                            success++;
                            if (res.statusCode == 200) {
                                var data = typeof res.data == 'object' ? res.data : JSON.parse(res.data);
                                if (data.code == 0 && (data.data.url || null) != null) {
                                    var list = self.form_images_list;
                                    if ((list[index] || null) == null) {
                                        list[index] = [];
                                    }
                                    list[index].push(data.data.url);
                                    self.setData({
                                        form_images_list: list
                                    });
                                } else {
                                    app.globalData.showToast(data.msg);
                                }
                            }
                        },
                        fail: function(e) {
                            fail++;
                        },
                        complete: function(e) {
                            count++;
            
                            // 下一张
                            if (count >= length) {
                                // 上传完毕，作一下提示
                                //app.showToast('上传成功' + success +'张', 'success');
                            } else {
                                // 递归调用，上传下一张
                                self.upload_one_by_one(index, img_paths, success, fail, count, length);
                            }
                        }
                    });
                }
            },
            
            // 表单
            formSubmit(e) {
                // 商品数量
                var length = this.detail.length;
                
                // 图片校验
                if (this.form_images_list.length > 0) {
                    for (var i in this.form_images_list) {
                        if (this.form_images_list[i].length > 1) {
                            app.globalData.showToast('每个订单凭证图片不能超过1张');
                            return false;
                        }
						if (this.form_images_list[i].length == 0) {
						    app.globalData.showToast('请上传凭证图片');
						    return false;
						}
                    }
                }
                
                // 表单数据
                var form_data = e.detail.value;
                form_data['order_id'] = JSON.stringify(this.form_orderid_list);
                form_data['images'] = this.form_images_list.length > 0 ? JSON.stringify(this.form_images_list) : '';
                //console.log(form_data);return;
				
				form_data['business_type'] = this.params.business_type || '';
				
                // 提交表单
                var self = this;
                uni.showLoading({
                    title: "处理中..."
                });
                self.setData({
                    form_button_disabled: true
                });
                uni.request({
                    url: app.globalData.get_request_url("somesave", "index", "bankext"),
                    method: "POST",
                    data: form_data,
                    dataType: "json",
                    success: res => {
                        uni.hideLoading();
                        if (res.data.code == 0) {
                            app.globalData.showToast(res.data.msg, "success");
							var value = self.backurl;
							if (app.globalData.is_tabbar_pages(value)) {
								var temp = value.split('?');
								if(temp.length > 1 && (temp[1] || null) != null)
								{
									value = temp[0];
								}
								setTimeout(function() {
									// 回到订单首页
									uni.switchTab({
									    url: value
									});
								}, 2000);
							} else {
								setTimeout(function() {
									// 回到订单首页
									uni.navigateTo({
									    url: value
									});
								}, 2000);
							}
                        } else {
                            self.setData({
                                form_button_disabled: false
                            });
                            app.globalData.showToast(res.data.msg);
                        }
                    },
                    fail: () => {
                        uni.hideLoading();
                        self.setData({
                            form_button_disabled: false
                        });
                        app.globalData.showToast("服务器请求出错");
                    }
                });
            }
        }
    };
</script>
<style>
    @import './bankext.css';
</style>