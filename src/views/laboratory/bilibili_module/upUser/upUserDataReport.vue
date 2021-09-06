<template>
  <div class="app-container">
    <conditional-filter
      :listQuery.sync="listQuery"
      :defaultListQuery="defaultListQuery"
      :addButton="false"
      :batchDelete="false"
      :list="list"
      :columns="columns"
      :multipleSelection="multipleSelection"
      @getList="getList"
      @handleBatchDelete="handleBatchDelete"
      excelTitle="Up主数据报表"
    >
      <template slot="extraForm">
        <el-form-item label="UP主ID搜索：">
          <el-select
            style="width:300px"
            v-model="listQuery.mid"
            filterable
            remote
            reserve-keyword
            placeholder="请填写搜索UP主的ID"
            :remote-method="remoteMethod"
            :loading="loading"
            clearable
          >
            <el-option v-for="item in options" :key="item.mid" :label="item.name" :value="item.mid"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="时间范围：">
          <el-date-picker
            v-model="listQuery.date"
            :picker-options="pickerOptions"
            type="daterange"
            range-separator="至"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
            :clearable="true"
            :editable="false"
            value-format="yyyy-MM-dd HH:mm:ss"
            :default-time="['00:00:00', '23:59:59']"
          />
        </el-form-item>
        <el-form-item label="趋势预览：">
          <el-select
            style="width:200px"
            v-model="listQuery.is_trend"
            filterable
            placeholder="是否开启趋势预览"
            clearable
          >
            <el-option label="开启" :value="1"></el-option>
            <el-option label="关闭" :value="0"></el-option>
          </el-select>
        </el-form-item>
      </template>
    </conditional-filter>
    <div class="table-container">
      <el-table ref="upUserDataReport" :data="list" style="width: 100%;" size="mini" border>
        <el-table-column label="time" sortable width="150" prop="time" v-if="columns[0].visible"></el-table-column>
        <el-table-column label="关注数" sortable prop="following" v-if="columns[1].visible">
          <template slot-scope="scope">
            {{scope.row.following}}
            <span v-if="listQuery.is_trend == 1">
              （
              <svg-icon icon-class="upward_trend" v-if="scope.row.following_trend > 0"></svg-icon>
              <svg-icon icon-class="down_trend" v-if="scope.row.following_trend < 0"></svg-icon>
              {{scope.row.following_trend}}
              ）
            </span>
          </template>
        </el-table-column>
        <el-table-column label="粉丝数" sortable prop="follower" v-if="columns[2].visible">
          <template slot-scope="scope">
            {{scope.row.follower}}
            <span v-if="listQuery.is_trend == 1">
              （
              <svg-icon icon-class="upward_trend" v-if="scope.row.follower_trend > 0"></svg-icon>
              <svg-icon icon-class="down_trend" v-if="scope.row.follower_trend < 0"></svg-icon>
              {{scope.row.follower_trend}}
              ）
            </span>
          </template>
        </el-table-column>
        <el-table-column label="视频播放数" sortable prop="video_play" v-if="columns[3].visible">
          <template slot-scope="scope">
            {{scope.row.video_play}}
            <span v-if="listQuery.is_trend == 1">
              （
              <svg-icon icon-class="upward_trend" v-if="scope.row.video_play_trend > 0"></svg-icon>
              <svg-icon icon-class="down_trend" v-if="scope.row.video_play_trend < 0"></svg-icon>
              {{scope.row.video_play_trend}}
              ）
            </span>
          </template>
        </el-table-column>
        <el-table-column label="阅读数" sortable prop="readling" v-if="columns[4].visible">
          <template slot-scope="scope">
            {{scope.row.readling}}
            <span v-if="listQuery.is_trend == 1">
              （
              <svg-icon icon-class="upward_trend" v-if="scope.row.readling_trend > 0"></svg-icon>
              <svg-icon icon-class="down_trend" v-if="scope.row.readling_trend < 0"></svg-icon>
              {{scope.row.readling_trend}}
              ）
            </span>
          </template>
        </el-table-column>
        <el-table-column label="获赞数" sortable prop="likes" v-if="columns[5].visible">
          <template slot-scope="scope">
            {{scope.row.likes}}
            <span v-if="listQuery.is_trend == 1">
              （
              <svg-icon icon-class="upward_trend" v-if="scope.row.likes_trend > 0"></svg-icon>
              <svg-icon icon-class="down_trend" v-if="scope.row.likes_trend < 0"></svg-icon>
              {{scope.row.likes_trend}}
              ）
            </span>
          </template>
        </el-table-column>
        <el-table-column label="月充电数" sortable prop="recharge_month" v-if="columns[6].visible">
          <template slot-scope="scope">
            {{scope.row.recharge_month}}
            <span v-if="listQuery.is_trend == 1">
              （
              <svg-icon icon-class="upward_trend" v-if="scope.row.recharge_month_trend > 0"></svg-icon>
              <svg-icon icon-class="down_trend" v-if="scope.row.recharge_month_trend < 0"></svg-icon>
              {{scope.row.recharge_month_trend}}
              ）
            </span>
          </template>
        </el-table-column>
        <el-table-column label="总充电数" sortable prop="recharge_total" v-if="columns[7].visible">
          <template slot-scope="scope">
            {{scope.row.recharge_total}}
            <span v-if="listQuery.is_trend == 1">
              （
              <svg-icon icon-class="upward_trend" v-if="scope.row.recharge_total_trend > 0"></svg-icon>
              <svg-icon icon-class="down_trend" v-if="scope.row.recharge_total_trend < 0"></svg-icon>
              {{scope.row.recharge_total_trend}}
              ）
            </span>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <div class="pagination-container">
      <Pagination
        v-show="total>0"
        :total="total"
        :page.sync="listQuery.cur_page"
        :limit.sync="listQuery.page_size"
        @pagination="getList"
      ></Pagination>
    </div>
  </div>
</template>
<script>
import {
  upUserDataReport,
  upUserSearch,
} from '@/api/laboratory/bilibili_module/upUser'
import { getDefaultDate } from '@/utils/date'
const defaultListQuery = {
  cur_page: 1,
  page_size: 50,
  mid: null,
  name: null,
  date: null,
  is_trend: null,
}
export default {
  components: {},
  data() {
    return {
      listQuery: Object.assign({}, defaultListQuery),
      defaultListQuery: Object.assign({}, defaultListQuery),
      list: [],
      total: 0,
      options: [],
      multipleSelection: [],
      timedStatusOptions: [],
      columns: [
        { key: 0, field: 'time', label: `时间`, visible: true },
        { key: 1, field: 'following', label: `关注数`, visible: true },
        { key: 2, field: 'follower', label: `粉丝数`, visible: true },
        { key: 3, field: 'video_play', label: `视频播放数`, visible: true },
        { key: 4, field: 'readling', label: `阅读数`, visible: true },
        { key: 5, field: 'likes', label: `获赞数`, visible: true },
        { key: 6, field: 'recharge_total', label: `月充电数`, visible: true },
        { key: 7, field: 'recharge_month', label: `总充电数`, visible: true },
      ],
    }
  },
  created() {
    upUserSearch().then((response) => {
      this.options = response.data.list
    })

    this.listQuery.date = getDefaultDate(1)

    const mid = this.$route.query && this.$route.query.mid
    const name = this.$route.query && this.$route.query.name
    this.listQuery.mid = mid
    this.listQuery.name = name

    this.getList()
  },
  methods: {
    getList() {
      upUserDataReport(this.listQuery).then((response) => {
        this.list = response.data.list
        this.total = response.data.total
      })
    },
    remoteMethod(query) {
      if (query !== '') {
        this.loading = true
        setTimeout(() => {
          this.loading = false
          upUserSearch().then((response) => {
            this.options = response.data.list.filter((item) => {
              return item.mid.toLowerCase().indexOf(query.toLowerCase()) > -1
            })
          })
        }, 200)
      } else {
        this.options = []
      }
    },
  },
}
</script>
<style scoped>
.input-width {
  width: 203px;
}
</style>
