<template>
  <div id="configInfo">
    <el-dialog
      title="系统信息"
      width="=80%"
      top="2rem"
      :close-on-click-modal="false"
      :visible.sync="showDialog"
      :destroy-on-close="true"
      @close="close()"
    >
      <div id="shared" style="margin-top: 1rem;margin-right: 100px;">
        <el-descriptions title="国标服务信息" v-if="configInfoData.sip" :span="2">
          <el-descriptions-item label="编号" >{{configInfoData.sip.id}}</el-descriptions-item>
          <el-descriptions-item label="域">{{configInfoData.sip.domain}}</el-descriptions-item>
          <el-descriptions-item label="IP">{{configInfoData.sip.showIp}}</el-descriptions-item>
          <el-descriptions-item label="端口">{{configInfoData.sip.port}}</el-descriptions-item>
          <el-descriptions-item label="密码">
            <el-tag size="small">{{configInfoData.sip.password}}</el-tag>
          </el-descriptions-item>
        </el-descriptions>
        <el-descriptions title="技术支持">
          <el-descriptions-item label="微信">lingtianxinxi</el-descriptions-item>
          <el-descriptions-item label="电话">0731-85155911</el-descriptions-item>
          <el-descriptions-item label="QRH"><el-link href="https://mp.weixin.qq.com/s?__biz=MzkwNzIxMTExNQ==&mid=2247484236&idx=1&sn=b632b9439c14817b89424b9390a2371a&chksm=c0ddfb48f7aa725e96da281a818d615c821f3c92c3678c4000b7179e93d80529f9abd38960bd#rd" target="_blank">快速参考手册</el-link></el-descriptions-item>
        </el-descriptions>
        © 2024 长沙零天信息科技有限公司
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "configInfo",
  props: {},
  computed: {},
  created() {},
  data() {
    return {
      showDialog: false,
      configInfoData: {
        sip:{},

      }
    };
  },
  methods: {
    openDialog: function (data) {
      console.log(data)
      this.showDialog = true;
      this.configInfoData = data;
    },
    close: function () {
      this.showDialog = false;
    },
  },
};
</script>
