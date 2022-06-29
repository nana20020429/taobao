<template>
  <!-- 评价详情页 -->
  <div id="commDetail">
    
  </div>
</template>

<script>
export default {
  name: "commDetail",
  data() {
    return {
      commid: "",
    };
  },
  created() {
    this.getCommid();
  },
  methods: {
    //获取路由传来的评论id
    getCommid() {
      this.commid = this.$route.params.commid;
    },
  },
};
</script>

<style>
</style>