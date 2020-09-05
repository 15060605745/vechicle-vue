<!--  -->
<template>
  <div>
    <el-button type="danger" @click="batchDel" round>批量删除</el-button>
    <el-tree
      :data="dataTree"
      :props="defaultProps"
      :expand-on-click-node="false"
      node-key="catId"
      :default-expanded-keys="expandKey"
      show-checkbox
      ref="tree"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="data.catLevel < 3"
            type="text"
            size="mini"
            @click="() => append(data)"
          >append</el-button>
          <el-button
            v-if="data.catLevel == 3"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >delete</el-button>
          <el-button type="text" size="mini" @click="() => edit(node, data)">edit</el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog title="分类信息" :visible.sync="dialogFormVisible" :close-on-click-modal="false">
      <el-form :model="categoryForm">
        <el-form-item label="分类名称" >
          <el-input v-model="categoryForm.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位" >
          <el-input v-model="categoryForm.productUnit" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="排    序" >
          <el-input v-model="categoryForm.sort" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitCategory">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},
  data() {
    return {
      dialogFormVisible: false,
      dataTree: [],
      expandKey: [],
      categoryForm: {
        name: "",
        productUnit: null,
        sort: 0,
        showStatus: 1,
        catLevel: "",
        parentCid: 1,
        icon: null,
        productCount: 0,
      },
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {
    // 获取数据列表
    getCategoryTree() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        this.dataTree = data.page;
      });
    },

    //新增节点
    append(data) {
      this.categoryForm.catLevel = data.catLevel + 1;
      this.categoryForm.parentCid = data.catId;
      this.categoryForm.name = "";
      this.categoryForm.productUnit = null;
      this.categoryForm.catId = "";
      this.dialogFormVisible = true;
    },

    //删除节点
    remove(node, data) {
      const ids = [data.catId];
      this.$confirm(
        "此操作将永久删除节点：" + data.name + ", 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      )
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          })
            .then(({ data }) => {
              if (data && data.code === 0) {
                this.$message({
                  type: "success",
                  message: "删除成功!",
                });
                //刷新树
                this.getCategoryTree();
                //展开节点
                this.expandKey = [node.parent.data.catId];
              } else {
                this.$message.error(data.msg);
              }
            })
            .catch(() => {
              this.$message.error("程序异常！请联系管理人员");
            });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },

    //编辑节点
    edit(node, data) {
      this.$http({
        url: this.$http.adornUrl("/product/category/info/" + data.catId),
        method: "get",
      })
        .then(({ data }) => {
          if (data && data.code === 0) {
            this.categoryForm = { ...data.category };
          } else {
            this.$message.error(data.msg);
          }
        })
        .catch(() => {
          this.$message.error("程序异常！请联系管理人员");
        });
      this.dialogFormVisible = true;
    },

    //提交表单
    submitCategory() {
      let url = "/product/category/save";
      if (
        this.categoryForm.catId != null &&
        this.categoryForm.catId != "" &&
        this.categoryForm.catId != undefined
      ) {
        url = "/product/category/update";
      }
      this.$http({
        url: this.$http.adornUrl(url),
        method: "post",
        data: this.$http.adornData(this.categoryForm, false),
      })
        .then(({ data }) => {
          if (data && data.code === 0) {
            this.$message({
              type: "success",
              message: "节点添加成功!",
            });
            //刷新树
            this.getCategoryTree();
            //展开节点
            this.expandKey = [this.categoryForm.parentCid];
          } else {
            this.$message.error(data.msg);
          }
        })
        .catch(() => {
          this.$message.error("程序异常！请联系管理人员");
        });
      this.dialogFormVisible = false;
    },

    //批量删除
    batchDel() {
      let nodes = this.$refs.tree.getCheckedNodes();
      let names = [];
      let ids = [];

      for (var i = 0; i < nodes.length; i++) {
        ids.push(nodes[i].catId);
        names.push(nodes[i].name);
      }

      this.$confirm(`批量删除【${names}】节点, 是否继续?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          })
            .then(({ data }) => {
              if (data && data.code === 0) {
                this.$message({
                  type: "success",
                  message: "批量删除成功!",
                });
                //刷新树
                this.getCategoryTree();
              } else {
                this.$message.error(data.msg);
              }
            })
            .catch(() => {
              this.$message.error("程序异常！请联系管理人员");
            });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
  },
  //生命周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getCategoryTree();
  },
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {},
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {}, //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style scoped>
</style>