<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="个人编号" prop="id">
        <el-input
          v-model="queryParams.id"
          placeholder="请输入个人编号"
          clearable
          size="small"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="订单号" prop="outTradeNo">
        <el-input
          v-model="queryParams.outTradeNo"
          placeholder="请输入订单号"
          clearable
          size="small"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="订单生成时间" prop="timeStart">
        <el-date-picker clearable size="small"
          v-model="queryParams.timeStart"
          type="date"
          value-format="yyyy-MM-dd"
          placeholder="选择订单生成时间">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="订单状态" prop="status">
        <el-select v-model="queryParams.status" placeholder="请选择订单状态" clearable size="small">
          <el-option
            v-for="dict in statusOptions"
            :key="dict.dictValue"
            :label="dict.dictLabel"
            :value="dict.dictValue"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="交易类型" prop="tradeType">
        <el-select v-model="queryParams.tradeType" placeholder="请选择交易类型" clearable size="small">
          <el-option
            v-for="dict in tradeTypeOptions"
            :key="dict.dictValue"
            :label="dict.dictLabel"
            :value="dict.dictValue"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="平台订单号" prop="transactionId">
        <el-input
          v-model="queryParams.transactionId"
          placeholder="请输入平台订单号"
          clearable
          size="small"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="el-icon-download"
          size="mini"
		  :loading="exportLoading"
          @click="handleExport"
          v-hasPermi="['system:record:export']"
        >导出</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="recordList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="主键id" align="center" prop="id" />
      <el-table-column label="交易订单号" align="center" prop="outTradeNo" />
      <el-table-column label="ip" align="center" prop="machineIp" />
      <el-table-column label="id" align="center" prop="machineId" />
      <el-table-column label="账户id" align="center" prop="userId" />
      <el-table-column label="授权码" align="center" prop="authCode" />
      <el-table-column label="qronline —— 二维码支付 qrofflne——离线二维码支付 cardonline——联机卡支付 face—人脸支付 cardoffline——脱机卡支付 minicard-小程序充值一卡通账户" align="center" prop="mediaType" />
      <el-table-column label="商品主题" align="center" prop="body" />
      <el-table-column label="订单生成时间" align="center" prop="timeStart" width="180">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.timeStart, '{y}-{m}-{d}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="订单超时时间" align="center" prop="timeExpire" width="180">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.timeExpire, '{y}-{m}-{d}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作员-现金充值时必须有" align="center" prop="opUserId" />
      <el-table-column label="订单状态" align="center" prop="status" :formatter="statusFormat" />
      <el-table-column label="支付总金额" align="center" prop="totalFee" />
      <el-table-column label="修改账户操作类型" align="center" prop="accOperTypeId" />
      <el-table-column label="交易渠道编码" align="center" prop="channelCode" />
      <el-table-column label="交易渠道名称" align="center" prop="channelName" />
      <el-table-column label="状态" align="center" prop="accStatus" />
      <el-table-column label="交易时间" align="center" prop="tradeTime" width="180">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.tradeTime, '{y}-{m}-{d}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="状态修改次数" align="center" prop="updateStatusCount" />
      <el-table-column label="最后状态修改时间" align="center" prop="lastStatusTime" width="180">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.lastStatusTime, '{y}-{m}-{d}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="受理商户appid" align="center" prop="appId" />
      <el-table-column label="用户在受理商户 appid 下的唯一标识" align="center" prop="openid" />
      <el-table-column label="用户在子商户appid下的唯一标识" align="center" prop="subOpenid" />
      <el-table-column label="交易类型" align="center" prop="tradeType" :formatter="tradeTypeFormat" />
      <el-table-column label="平台订单号" align="center" prop="transactionId" />
      <el-table-column label="第三方订单号" align="center" prop="outTransactionId" />
      <el-table-column label="支付宝账号" align="center" prop="buyerLogonId" />
      <el-table-column label="实际付款金额" align="center" prop="buyerPayAmount" />
      <el-table-column label="支付宝用户id" align="center" prop="buyerUserId" />
      <el-table-column label="银行类型" align="center" prop="bankType" />
      <el-table-column label="银行订单号" align="center" prop="bankOrderNo" />
      <el-table-column label="支付完成时间" align="center" prop="timeEnd" width="180">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.timeEnd, '{y}-{m}-{d}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="渠道支付错误码" align="center" prop="errCode" />
      <el-table-column label="渠道支付错误描述" align="center" prop="errMsg" />
      <el-table-column label="扩展参数1" align="center" prop="paramOne" />
      <el-table-column label="扩展参数2" align="center" prop="paramTwo" />
      <el-table-column label="创建人" align="center" prop="createBy" />
      <el-table-column label="更新人" align="center" prop="updateBy" />
      <el-table-column label="创建时间" align="center" prop="createTime" width="180">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.createTime, '{y}-{m}-{d}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="更新时间" align="center" prop="updateTime" width="180">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.updateTime, '{y}-{m}-{d}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="备注" align="center" prop="remark" />
      <el-table-column label=" 删除" align="center" prop="delFlag" />
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            icon="el-icon-edit"
            @click="handleUpdate(scope.row)"
            v-hasPermi="['system:record:edit']"
          >修改</el-button>
          <el-button
            size="mini"
            type="text"
            icon="el-icon-delete"
            @click="handleDelete(scope.row)"
            v-hasPermi="['system:record:remove']"
          >删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total>0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="getList"
    />

  </div>
</template>

<script>
import { listRecord, getRecord, delRecord, addRecord, updateRecord, exportRecord } from "@/api/system/record";

export default {
  name: "Record",
  data() {
    return {
      // 遮罩层
      loading: true,
      // 导出遮罩层
      exportLoading: false,
      // 选中数组
      ids: [],
      // 非单个禁用
      single: true,
      // 非多个禁用
      multiple: true,
      // 显示搜索条件
      showSearch: true,
      // 总条数
      total: 0,
      // 消费交易原始记录表格数据
      recordList: [],
      // 弹出层标题
      title: "",
      // 是否显示弹出层
      open: false,
      // 订单状态：0待支付 1支付成功 2支付失败，3已退款，4 支付中 5 已撤销 6加密金额错误字典
      statusOptions: [],
      // 返回的交易类型：pay.weixin.micropay——微信被扫支付pay.alipay.micropay——支付宝被扫支付pay.unionpay.micropay——银联被扫支付字典
      tradeTypeOptions: [],
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        id: null,
        outTradeNo: null,
        timeStart: null,
        status: null,
        tradeType: null,
        transactionId: null,
      },
      // 表单参数
      form: {},
      // 表单校验
      rules: {
      }
    };
  },
  created() {
    this.getList();
    this.getDicts("sys_oper_type").then(response => {
      this.statusOptions = response.data;
    });
    this.getDicts("sys_notice_type").then(response => {
      this.tradeTypeOptions = response.data;
    });
  },
  methods: {
    /** 查询消费交易原始记录列表 */
    getList() {
      this.loading = true;
      listRecord(this.queryParams).then(response => {
        this.recordList = response.rows;
        this.total = response.total;
        this.loading = false;
      });
    },
    // 订单状态：0待支付 1支付成功 2支付失败，3已退款，4 支付中 5 已撤销 6加密金额错误字典翻译
    statusFormat(row, column) {
      return this.selectDictLabel(this.statusOptions, row.status);
    },
    // 返回的交易类型：pay.weixin.micropay——微信被扫支付pay.alipay.micropay——支付宝被扫支付pay.unionpay.micropay——银联被扫支付字典翻译
    tradeTypeFormat(row, column) {
      return this.selectDictLabel(this.tradeTypeOptions, row.tradeType);
    },
    // 取消按钮
    cancel() {
      this.open = false;
      this.reset();
    },
    // 表单重置
    reset() {
      this.form = {
        id: null,
        outTradeNo: null,
        machineIp: null,
        machineId: null,
        userId: null,
        authCode: null,
        mediaType: null,
        body: null,
        timeStart: null,
        timeExpire: null,
        opUserId: null,
        status: null,
        totalFee: null,
        accOperTypeId: null,
        channelCode: null,
        channelName: null,
        accStatus: 0,
        tradeTime: null,
        updateStatusCount: null,
        lastStatusTime: null,
        appId: null,
        openid: null,
        subOpenid: null,
        tradeType: null,
        transactionId: null,
        outTransactionId: null,
        buyerLogonId: null,
        buyerPayAmount: null,
        buyerUserId: null,
        bankType: null,
        bankOrderNo: null,
        timeEnd: null,
        errCode: null,
        errMsg: null,
        paramOne: null,
        paramTwo: null,
        createBy: null,
        updateBy: null,
        createTime: null,
        updateTime: null,
        remark: null,
        delFlag: null,
        requestData: null,
        responseData: null
      };
      this.resetForm("form");
    },
    /** 搜索按钮操作 */
    handleQuery() {
      this.queryParams.pageNum = 1;
      this.getList();
    },
    /** 重置按钮操作 */
    resetQuery() {
      this.resetForm("queryForm");
      this.handleQuery();
    },

    /** 导出按钮操作 */
    handleExport() {
      const queryParams = this.queryParams;
      this.$confirm('是否确认导出所有消费交易原始记录数据项?', "警告", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }).then(() => {
          this.exportLoading = true;
          return exportRecord(queryParams);
        }).then(response => {
          this.download(response.msg);
          this.exportLoading = false;
        }).catch(() => {});
    }
  }
};
</script>
