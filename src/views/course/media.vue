<template>
  <div>
    <div style="float: left; padding: 20px">
      <el-button type="primary" icon="el-icon-edit">新增图文</el-button>
    </div>
    <div style="float: right; margin-right: 50px; margin-top: 20px">
      <el-dropdown>
        <i class="el-dropdown-link">
          商品状态
          <i class="el-icon-arrow-down el-icon--right"></i>
        </i>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item>已上架</el-dropdown-item>
          <el-dropdown-item>已下架</el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
      <el-input
        v-model="input"
        style="width: 200px; margin-right: 5px"
        placeholder="标题"
      ></el-input>
      <el-button type="primary" icon="el-icon-search">搜索</el-button>
    </div>
    <div style="margin: 20px">
      <el-table :data="tableData" style="width: 100%;height：100px" border>
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
        <el-table-column prop="address" label="操作" width="260">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" @click="handleUpdate(scope.row)"
              >编辑</el-button
            >
            <el-button size="mini" @click="putAway(scope.$index, scope.row)"
              >上架</el-button
            >
            <el-button
              size="mini"
              type="danger"
              @click="handleDelete(scope.$index, scope.row)"
              >删除</el-button
            >
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
        <el-form-item label="封面" prop="title">
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
      </el-form>
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
      currentPage1: 5,
      tableData: null,
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
      dialogStatus: "",
      textMap: {
        update: "修改",
        create: "Create",
      },
      rules: {
        type: [
          { required: true, message: "type is required", trigger: "change" },
        ],
        timestamp: [
          {
            type: "date",
            required: true,
            message: "timestamp is required",
            trigger: "change",
          },
        ],
        title: [
          { required: true, message: "title is required", trigger: "blur" },
        ],
      },
      input: "",
      dialogImageUrl: "",
      dialogVisible: false,
      disabled: false,
      content: '',
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
    handleUpdate(row) {
      this.temp = Object.assign({}, row); // copy obj
      this.temp.timestamp = new Date(this.temp.timestamp);
      this.dialogStatus = "update";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
    },
    handleDelete(index, row) {
      console.log(index, row);
    },
    putAway(index, row) {
      console.log(index, row);
    },
    handleRemove(file) {
      console.log(file);
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
