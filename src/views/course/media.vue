<template>
  <div>
    <div style="float: left; padding: 20px">
      <el-button type="primary" icon="el-icon-edit" @click="handleCreate"
        >新增图文</el-button
      >
    </div>
    <div style="float: right; margin-right: 50px; margin-top: 20px">
      <el-select v-model="value" placeholder="请选择">
        <el-option
          v-for="item in options"
          :key="item.value"
          :label="item.label"
          :value="item.value"
        >
        </el-option>
      </el-select>
      <el-input
        v-model="input"
        style="width: 200px; margin-right: 5px"
        placeholder="标题"
      ></el-input>
      <el-button type="primary" icon="el-icon-search" @click="handleFilter"
        >搜索</el-button
      >
    </div>
    <div style="margin: 20px">
      <el-table
        :data="tableData"
        style="width: 100%;height：100px"
        border
        @sort-change="sortChange"
      >
        <el-table-column prop="date" label="ID" width="110" sortable>
          <template slot-scope="scope">{{ scope.row.id }}</template>
        </el-table-column>
        <el-table-column prop="name" label="图文内容">
          <template slot-scope="scope">
            <img
              :src="scope.row.cover"
              alt=""
              style="float: left; width: 100px; height: 80px"
            />
            <div style="float: left; margin-left: 50px；">
              <p>{{ scope.row.title }}</p>
              <p style="color: red">￥{{ scope.row.price }}</p>
            </div>
          </template>
        </el-table-column>
        <el-table-column prop="address" label="订阅量" width="110">
          <template slot-scope="scope">{{ scope.row.sub_count }}</template>
        </el-table-column>
        <el-table-column prop="date" label="状态" width="100">
          <template slot-scope="scope">
            <el-button
              size="mini"
              :type="scope.row.status | setStatusColor"
              plain
              disabled
            >
              {{ scope.row.status | setStatus }}</el-button
            >
          </template>
        </el-table-column>
        <el-table-column prop="name" label="创建时间" width="260">
          <template slot-scope="scope">{{ scope.row.created_time }}</template>
        </el-table-column>
        <el-table-column
          label="操作"
          align="center"
          width="230"
          class-name="small-padding fixed-width"
        >
          <template slot-scope="{ row, $index }">
            <el-button type="primary" size="mini" @click="handleUpdate(row)">
              编辑
            </el-button>
            <el-button
              v-if="row.status == 0"
              size="mini"
              type="success"
              @click="handleModifyStatus(row, 1)"
            >
              上架
            </el-button>
            <el-button
              v-if="row.status == 1"
              size="mini"
              @click="handleModifyStatus(row, 0)"
            >
              下架
            </el-button>
            <el-popconfirm
              title="是否要删除该记录？"
              @onConfirm="handleDelete(row, $index)"
              style="margin-left: 10px"
            >
              <el-button
                v-if="row.status != 'deleted'"
                size="mini"
                type="danger"
                slot="reference"
                >删除</el-button
              >
            </el-popconfirm>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <el-dialog
      fullscreen
      :title="textMap[dialogStatus]"
      :visible.sync="dialogFormVisible"
    >
      <el-form
        ref="dataForm"
        :rules="rules"
        :model="temp"
        label-position="left"
        label-width="70px"
        style="width: 400px; margin-left: 50px"
      >
        <el-form-item label="标题" prop="title">
          <el-input v-model="temp.title" />
        </el-form-item>
        <el-form-item label="封面">
          <el-upload action="#" list-type="picture-card" :auto-upload="false">
            <i slot="default" class="el-icon-plus"></i>
            <div slot="file" slot-scope="{ file }">
              <img
                class="el-upload-list__item-thumbnail"
                :src="file.url"
                alt=""
              />
              <span class="el-upload-list__item-actions">
                <span
                  class="el-upload-list__item-preview"
                  @click="handlePictureCardPreview(file)"
                >
                  <i class="el-icon-zoom-in"></i>
                </span>
                <span
                  v-if="!disabled"
                  class="el-upload-list__item-delete"
                  @click="handleDownload(file)"
                >
                  <i class="el-icon-download"></i>
                </span>
                <span
                  v-if="!disabled"
                  class="el-upload-list__item-delete"
                  @click="handleRemove(file)"
                >
                  <i class="el-icon-delete"></i>
                </span>
              </span>
            </div>
          </el-upload>
        </el-form-item>
        <el-form-item label="试看内容" prop="title">
          <tinymce v-model="content" :width="600" />
        </el-form-item>
        <el-form-item label="课程价格">
          <el-input-number
            v-model="num"
            :min="0"
            :max="100"
            label="描述文字"
          ></el-input-number>
        </el-form-item>
        <el-form-item label="状态">
          <el-radio v-model="radio" label="1">下架</el-radio>
          <el-radio v-model="radio" label="2">上架</el-radio>
        </el-form-item>
        <el-form-item label="更新状态">
          <el-radio v-model="niradio" label="1">连载中</el-radio>
          <el-radio v-model="niradio" label="2">已完结</el-radio>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false"> 取消 </el-button>
        <el-button
          type="primary"
          @click="dialogStatus === 'create' ? createData() : updateData()"
        >
          提交
        </el-button>
      </div>
    </el-dialog>
    <el-dialog :visible.sync="dialogVisible">
      <el-table
        :data="pvData"
        border
        fit
        highlight-current-row
        style="width: 100%"
      >
        <el-table-column prop="key" label="Channel" />
        <el-table-column prop="pv" label="Pv" />
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogVisible = false"
          >Confirm</el-button
        >
      </span>
    </el-dialog>

    <div class="block" style="margin-left: 50px">
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage1"
        :page-sizes="[20, 40, 60, 80, 100]"
        :page-size="20"
        layout="total, sizes, prev, pager, next, jumper"
        :total="100"
      >
      </el-pagination>
    </div>
  </div>
</template>

<script>
import Tinymce from "@/components/Tinymce";
import { fetchList, createMedia, updateMedia, deleteMedia } from "@/api/media";
export default {
  components: { Tinymce },
  data() {
    return {
      options: [
        {
          value: "选项1",
          label: "已下架",
        },
        {
          value: "选项2",
          label: "已上架",
        },
      ],
      value: "",
      radio: "1",
      niradio: "2",
      num: 0,
      currentPage1: 5,
      tableData: null,
      tableDataLoading: true,
      tableDataQuery: {
        page: 1,
        limit: 20,
        status: undefined,
        title: undefined,
        sort: "+id",
      },
      sortOptions: [
        {
          label: "ID Ascending",
          key: "+id",
        },
        {
          label: "ID Descending",
          key: "-id",
        },
      ],
      temp: {
        id: undefined,
        importance: 1,
        remark: "",
        timestamp: new Date(),
        title: "",
        type: "",
        status: "published",
      },
      dialogFormVisible: false,
      pvData: [],
      dialogStatus: "",
      textMap: {
        update: "编辑",
        create: "新增",
      },
      rules: {
        title: [
          {
            required: true,
            message: "标题不能为空",
            trigger: "blur",
          },
        ],
        try: [
          {
            required: true,
            message: "试看内容不能为空",
            trigger: "blur",
          },
        ],
        content: [
          {
            required: true,
            message: "课程内容不能为空",
            trigger: "blur",
          },
        ],
        dialogImageUrl: "",
        dialogVisible: false,
      },
      timestamp: [
        {
          type: "date",
          required: true,
          message: "timestamp is required",
          trigger: "change",
        },
      ],
      input: "",

      disabled: false,
      content: "",
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
    handleFilter() {
      this.tableData();
    },
    handleModifyStatus(row, status) {
      this.$message({
        message: "操作Success",
        type: "success",
      });
      row.status = status;
    },
    sortChange(data) {
      const { prop, order } = data;
      if (prop === "id") {
        this.sortByID(order);
      }
    },
    sortByID(order) {
      if (order === "ascending") {
        this.listQuery.sort = "+id";
      } else {
        this.listQuery.sort = "-id";
      }
      this.handleFilter();
    },

    resetTemp() {
      this.temp = {
        id: undefined,
        title: "",
        status: 1,
        price: 0,
        try: "",
        content: "",
        cover: "",
      };
    },
    handleCreate() {
      this.resetTemp();
      this.dialogStatus = "create";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
    },
    createData() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          this.temp.id = parseInt(Math.random() * 100) + 1024; // mock a id
          this.temp.author = "vue-element-admin";
          createMedia(this.temp).then(() => {
            this.tableData.unshift(this.temp);
            this.dialogFormVisible = false;
            this.$notify({
              title: "Success",
              message: "Created Successfully",
              type: "success",
              duration: 2000,
            });
          });
        }
      });
    },
    handleUpdate(row) {
      this.temp = Object.assign({}, row); // copy obj
      this.temp.timestamp = new Date(this.temp.timestamp);
      this.dialogStatus = "update";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
    },
    updateData() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          const tempData = Object.assign({}, this.temp);
          tempData.timestamp = +new Date(tempData.timestamp); // change Thu Nov 30 2017 16:41:05 GMT+0800 (CST) to 1512031311464
          updateMedia(tempData).then(() => {
            const index = this.tableData.findIndex(
              (v) => v.id === this.temp.id
            );
            this.tableData.splice(index, 1, this.temp);
            this.dialogFormVisible = false;
            this.$notify({
              title: "Success",
              message: "Update Successfully",
              type: "success",
              duration: 2000,
            });
          });
        }
      });
    },
    handleDelete(row, index) {
      deleteMedia(row).then((response) => {
        this.$notify({
          title: "提示",
          message: "删除成功",
          type: "success",
          duration: 2000,
        });
        //console.log(this.tableData)
        this.tableData.splice(index, 1);
      });
    },
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    },
    handleDownload(file) {
      console.log(file);
    },
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`);
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
    },
  },
  filters: {
    setStatus(value) {
      if (value === 0) {
        return "已下架";
      } else if (value === 1) {
        return "已上架";
      }
    },
    setStatusColor(value) {
      if (value === 0) {
        return "danger";
      } else if (value === 1) {
        return "success";
      }
    },
  },
};
</script>
<style scoped>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
.editor-content {
  margin-top: 20px;
}
</style>
