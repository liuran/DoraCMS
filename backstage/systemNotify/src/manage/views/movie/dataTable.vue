<!--
  -  Copyright © 2019  Liuran All Rights Reserved.
  -->

<template>
  <div>
    <el-table
      align="center"
      v-loading="loading"
      ref="multipleTable"
      :data="dataList"
      tooltip-effect="dark"
      style="width: 100%"
      @selection-change="handleMovieSelectionChange"
    >
      <el-table-column type="selection" width="55"></el-table-column>
      <el-table-column prop="isTop" :label="$t('contents.rec')" width="55" show-overflow-tooltip>
        <template slot-scope="scope">
          <i
            @click="topMovie(scope.$index, dataList)"
            :class="scope.row.isTop === 1 ? 'fa fa-star' : 'fa fa-star-o'"
            :style="scope.row.isTop === 1 ? yellow : gray"
          ></i>
        </template>
      </el-table-column>
      <el-table-column
        prop="roofPlacement"
        :label="$t('contents.roofPlacement')"
        width="55"
        show-overflow-tooltip
      >
        <template slot-scope="scope">
          <i
            v-if="scope.row.isTop === 1"
            @click="roofMovie(scope.$index, dataList)"
            :class="scope.row.roofPlacement=='1' ? 'fa fa-thumbs-up ' : 'fa fa-thumbs-o-up'"
            :style="scope.row.roofPlacement=='1' ? green : gray"
          ></i>
        </template>
      </el-table-column>
      <el-table-column prop="title" :label="$t('contents.title')" width="200" show-overflow-tooltip>
        <template slot-scope="scope">
          <div v-if="scope.row.state">
            <a :href="'/detaiss/'+scope.row._id+'.html'" target="_blank">{{scope.row.title}}</a>
          </div>
          <div v-else>{{scope.row.title}}</div>
        </template>
      </el-table-column>
      <el-table-column prop="date" :label="$t('contents.date')" width="180">
        <template slot-scope="scope">{{scope.row.updateDate}}</template>
      </el-table-column>
      <el-table-column prop="type" :label="$t('contents.movieType')" width="180">
        <template slot-scope="scope">
          <span v-if="scope.row.type == '1'">普通</span>
        
        </template>
      </el-table-column>
      <el-table-column
        prop="categories"
        :label="$t('contents.movieCategories')"
        show-overflow-tooltip
        width="120"
      >
        <template slot-scope="scope">
          <span>{{(scope.row.categories&&scope.row.categories[0])?scope.row.categories[0].name:''}}</span>
        </template>
      </el-table-column>

      <el-table-column prop="tags" :label="$t('contents.movieTags')" show-overflow-tooltip>
        <template slot-scope="scope">
          <span v-for="tag in scope.row.tags" :key="tag._id">{{tag.name+','}}</span>
        </template>
      </el-table-column>

      <el-table-column prop="clickNum" :label="$t('contents.clickNum')" show-overflow-tooltip></el-table-column>
      <el-table-column prop="commentNum" :label="$t('contents.commentNum')" show-overflow-tooltip></el-table-column>
      <el-table-column prop="state" :label="$t('contents.enable')" show-overflow-tooltip>
        <template slot-scope="scope">
          <i
            :class="scope.row.state=='2' ? 'fa fa-check-circle' : 'fa fa-minus-circle'"
            :style="scope.row.state=='2' ? green : red"
          ></i>
        </template>
      </el-table-column>
      <el-table-column prop="author.name" :label="$t('contents.author')" show-overflow-tooltip>
        <template
          slot-scope="scope"
        >{{scope.row.uAuthor?scope.row.uAuthor.userName:(scope.row.author?scope.row.author.userName:'')}}</template>
      </el-table-column>
      <el-table-column :label="$t('main.dataTableOptions')" width="150" fixed="right">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="primary"
            plain
            round
            @click="editMovieInfo(scope.$index, dataList)"
          >
            <i class="fa fa-edit"></i>
          </el-button>
          <el-button
            size="mini"
            type="danger"
            plain
            round
            icon="el-icon-delete"
            @click="deleteMovie(scope.$index, dataList)"
          ></el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>
<style lang="scss">
.fa-star,
.fa-thumbs-up {
  cursor: pointer;
}

.fa-star-o,
.fa-thumbs-o-up {
  cursor: pointer;
}
</style>
<script>
import services from "../../store/services.js";
import _ from "lodash";
export default {
  props: {
    dataList: Array,
    pageInfo: Object
  },
  data() {
    return {
      loading: false,
      multipleSelection: [],
      yellow: {
        color: "#F7BA2A"
      },
      gray: {
        color: "#CCC"
      },
      green: { color: "#13CE66" },
      red: { color: "#FF4949" }
    };
  },

  methods: {
    handleMovieSelectionChange(val) {
      if (val && val.length > 0) {
        let ids = val.map((item, index) => {
          return item._id;
        });
        this.multipleSelection = ids;
        this.$emit("changeMovieSelectList", ids);
      }
    },
    editMovieInfo(index, rows) {
      let rowData = rows[index];
      rowData = JSON.parse(JSON.stringify(rowData));
      let categoryIdArr = [],
        tagsArr = [];
      !_.isEmpty(rowData.categories) &&
        rowData.categories.map((item, index) => {
          item && categoryIdArr.push(item._id);
        });
      !_.isEmpty(rowData.tags) &&
        rowData.tags.map((item, index) => {
          item && tagsArr.push(item._id);
        });
      !_.isEmpty(rowData.keywords) &&
        (rowData.keywords = rowData.keywords.join());
      rowData.categories = categoryIdArr;
      rowData.tags = tagsArr;
      this.$store.dispatch("showMovieForm", {
        edit: true,
        formData: rowData
      });
      this.$router.push("/editMovie/" + rowData._id);
    },
    topMovie(index, rows) {
      let movieData = rows[index];
      // movieData.isTop = movieData.isTop == 1 ? 0 : 1;
      let targetParams = {
        _id: movieData._id,
        isTop: movieData.isTop == 1 ? 0 : 1
      };
      services.updateMovieToTop(targetParams).then(result => {
        if (result.data.status === 200) {
          this.$store.dispatch("getMovieList", this.pageInfo);
        } else {
          this.$message.error(result.data.message);
        }
      });
    },
    roofMovie(index, rows) {
      let movieData = rows[index];
      // 推荐的才允许置顶
      if (movieData.isTop != 1) {
        return false;
      }
      let targetParams = {
        _id: movieData._id,
        roofPlacement: movieData.roofPlacement == "1" ? "0" : "1"
      };
      services.roofMovie(targetParams).then(result => {
        if (result.data.status === 200) {
          this.$store.dispatch("getMovieList", this.pageInfo);
        } else {
          this.$message.error(result.data.message);
        }
      });
    },
    deleteMovie(index, rows) {
      this.$confirm(
        this.$t("main.del_notice"),
        this.$t("main.scr_modal_title"),
        {
          confirmButtonText: this.$t("main.confirmBtnText"),
          cancelButtonText: this.$t("main.cancelBtnText"),
          type: "warning"
        }
      )
        .then(() => {
          return services.deleteMovie({
            ids: rows[index]._id
          });
        })
        .then(result => {
          if (result.data.status === 200) {
            Object.assign(this.pageInfo);
            this.$store.dispatch("getMovieList", this.pageInfo);
            this.$message({
              message: this.$t("main.scr_modal_del_succes_info"),
              type: "success"
            });
          } else {
            this.$message.error(result.data.message);
          }
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: this.$t("main.scr_modal_del_error_info")
          });
        });
    }
  },
  computed: {}
};
</script>
