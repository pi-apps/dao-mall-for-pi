<template>
    <view>
    </view>
</template>
<script>
    const app = getApp();
    export default {
        data() {
            return {
                params: null,
				isdone:false,
            };
        },
        components: {},
        props: {},

        onLoad(params) {
            /*this.setData({
                params: params
            });*/
			this.params = params;
        },
        
        onShow() {
			this.init();
        },

        // 下拉刷新
        onPullDownRefresh() {
            this.init();
        },

        methods: {
            // 初始化
			init() {
				var self = this;
			    let time = setInterval(() => {
			    	let theme = self.params || null;
			    	if (theme != null) {
						clearInterval(time);
						setTimeout(function() {
						    uni.navigateBack();
						}, 500);
			    		app.globalData.text_copy_event(theme.no);
			    	}
			    }, 50);
			}
        }
    };
</script>
<style>
</style>