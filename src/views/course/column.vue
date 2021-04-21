<template>
  <div>
    <div style="float:left;padding:20px">
      <el-button type="primary" icon="el-icon-edit">新增专栏</el-button>
    </div>
    <div style="float:right;margin-right:50px;margin-top:20px">
      <el-dropdown>
        <i class="el-dropdown-link" >
          商品状态
          <i class="el-icon-arrow-down el-icon--right"></i>
        </i>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item>黄金糕</el-dropdown-item>
          <el-dropdown-item>狮子头</el-dropdown-item>
          <el-dropdown-item>螺蛳粉</el-dropdown-item>
          <el-dropdown-item disabled>双皮奶</el-dropdown-item>
          <el-dropdown-item divided>蚵仔煎</el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
      <el-input
        v-model="input"
        style="width: 200px;margin-right:5px"
        placeholder="标题"
      ></el-input>
      <el-button type="primary" icon="el-icon-search">搜索</el-button>
    </div>
    <div style="margin:20px;">
       <el-table :data="tableData" style="width: 100%;height：100px" border>
      <el-table-column prop="date" label="ID" width="110" sortable>
        <template slot-scope="scope">{{ scope.row.id }}</template>
      </el-table-column>
      <el-table-column prop="name" label="专栏内容" >
        <template slot-scope="scope">
          <img
            :src="scope.row.cover"
            alt=""
            style="float: left; width: 100px; height: 80px"
          />
          <div style="float: left;margin-left: 50px； ">
            <p>{{ scope.row.title }}</p>
            <p style="color: red">￥{{ scope.row.price }}</p>
          </div>
        </template>
      </el-table-column>
        <el-table-column prop="address" label="更新状态" width="110">
        <template slot-scope="scope" >                                    {{ scope.row.isend|upIsend }}</template>
      </el-table-column>
      <el-table-column prop="address" label="订阅量" width="110">
        <template slot-scope="scope">{{ scope.row.sub_count }}</template>
      </el-table-column>
      <el-table-column prop="date" label="状态" width="100">
        <template slot-scope="scope">
          <el-button  size="mini" :type="scope.row.status|setStatusColor" plain disabled> {{ scope.row.status|setStatus }}</el-button>
         </template>
      </el-table-column>
      <el-table-column prop="name" label="创建时间" width="260">
        <template slot-scope="scope">{{ scope.row.created_time }}</template>
      </el-table-column>
      <el-table-column prop="address" label="操作" width="300">
        <template slot-scope="scope">
         <el-button
          size="mini"
           type="warning"
          @click="catalogue(scope.$index, scope.row)">目录</el-button>
        <el-button
          size="mini"
           type="primary"
          @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
        <el-button
          size="mini"
          @click="putAway(scope.$index, scope.row)">上架</el-button>  
        <el-button
          size="mini"
          type="danger"
          @click="handleDelete(scope.$index, scope.row)">删除</el-button>
      </template>
      </el-table-column>
    </el-table>
    </div>
    <div class="block" style="margin-left:50px">
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage1"
      :page-sizes="[20,40,60,80,100]"
      :page-size="20"
      layout="total, sizes, prev, pager, next, jumper"
      :total="100">
    </el-pagination>
  </div>
  </div>
</template>

<script>
import { fetchList,fetchDetail,fetchDetailCourse,createColumn,updateColumn,deleteColumn } from "@/api/column";
export default {
  data() {
    return {
      tableData: null,
      
    };
  },
  created() {
    this.fetchData();
  },
  methods: {
    async fetchData() {
      const { data } = await fetchList();
      this.tableData = data.items;
    },
    catalogue(index,row){
        console.log(index, row);
    },
     handleEdit(index, row) {
        console.log(index, row);
      },
      handleDelete(index, row) {
        console.log(index, row);
      },
      putAway(index,row){
        console.log(index, row);
      },

      handleSizeChange(val) {
        console.log(`每页 ${val} 条`);
      },
      handleCurrentChange(val) {
        console.log(`当前页: ${val}`);
      }
  },
  filters:{
    setStatus(value){
      if(value===0){
        return "已下架";
        
      }else if(value===1){
        return "已上架";
      }
    },
    setStatusColor(value){
      if(value===0){
        return "danger";

      }else if(value===1){
        return "success";
      }
    },
    upIsend(value){
      if(value===0){
        return "连载中"
      }else if(value===1){
        return "已完结"
      }
    },
    upIsendColor(value){
      if(value===0){
        return "red"
      }else if(value){
        return "black"
      }
    },

  }
};
</script>
<style scoped>
</style>

