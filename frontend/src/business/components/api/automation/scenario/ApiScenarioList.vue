<template>
  <el-card class="table-card" v-loading="result.loading">
    <template v-slot:header>
      <ms-table-header :condition.sync="condition" @search="search" :title="$t('commons.test')"
                       :show-create="false"/>
    </template>

    <el-table ref="scenarioTable" border :data="tableData" class="adjust-table" @select-all="select" @select="select">
      <el-table-column type="selection"/>
      <el-table-column width="40" :resizable="false" align="center">
        <template v-slot:default="{row}">
          <show-more-btn :is-show="isSelect(row)" :buttons="buttons" :size="selection.length"/>
        </template>
      </el-table-column>
      <el-table-column prop="name" :label="$t('api_test.automation.scenario_name')" width="200"
                       show-overflow-tooltip/>
      <el-table-column prop="level" :label="$t('api_test.automation.case_level')" width="100"
                       show-overflow-tooltip/>
      <el-table-column prop="tagName" :label="$t('api_test.automation.tag')" width="200" show-overflow-tooltip/>
      <el-table-column prop="userId" :label="$t('api_test.automation.creator')" width="150" show-overflow-tooltip/>
      <el-table-column prop="updateTime" :label="$t('api_test.automation.update_time')" width="160">
        <template v-slot:default="scope">
          <span>{{ scope.row.updateTime | timestampFormatDate }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="stepTotal" :label="$t('api_test.automation.step')" width="100" show-overflow-tooltip/>
      <el-table-column prop="status" :label="$t('api_test.automation.last_result')" width="100">
        <template v-slot:default="{row}">
          <el-link type="success" v-if="row.status === 'Success'">{{ $t('api_test.automation.success') }}</el-link>
          <el-link type="danger" v-if="row.status === 'Fail'">{{ $t('api_test.automation.fail') }}</el-link>
          <el-link type="info" v-if="row.status === 'Saved'">{{ $t('api_test.automation.saved') }}</el-link>
          <el-link type="warning" v-if="row.status === 'Trash'">{{ $t('api_test.automation.trash') }}</el-link>
        </template>
      </el-table-column>
      <el-table-column prop="passingRate" :label="$t('api_test.automation.passing_rate')" width="100"
                       show-overflow-tooltip/>
      <el-table-column :label="$t('commons.operating')">
        <template v-slot:default="{row}">
          <el-button type="text" @click="edit(row)">{{ $t('api_test.automation.edit') }}</el-button>
          <el-button type="text" @click="execute(row)">{{ $t('api_test.automation.execute') }}</el-button>
          <el-button type="text" @click="copy(row)">{{ $t('api_test.automation.copy') }}</el-button>
          <el-button type="text" @click="remove(row)">{{ $t('api_test.automation.remove') }}</el-button>
        </template>
      </el-table-column>
    </el-table>
    <ms-table-pagination :change="search" :current-page.sync="currentPage" :page-size.sync="pageSize"
                         :total="total"/>
  </el-card>
</template>

<script>
import MsTableHeader from "@/business/components/common/components/MsTableHeader";
import MsTablePagination from "@/business/components/common/pagination/TablePagination";
import ShowMoreBtn from "@/business/components/track/case/components/ShowMoreBtn";

export default {
  name: "MsApiScenarioList",
  components: {ShowMoreBtn, MsTablePagination, MsTableHeader},
  props: {
    currentProject: Object,
    currentModule: Object,
  },
  data() {
    return {
      result: {},
      condition: {},
      selectAll: false,
      selection: [],
      tableData: [],
      currentPage: 1,
      pageSize: 10,
      total: 0,
      buttons: [
        {
          name: this.$t('api_test.automation.batch_add_plan'), handleClick: this.handleBatchAddCase
        }, {
          name: this.$t('api_test.automation.batch_execute'), handleClick: this.handleBatchExecute
        }
      ],
    }
  },
  watch: {
    currentProject() {
      this.search();
    },
    currentModule() {
      this.search();
    },
  },
  methods: {
    search() {
      this.condition.filters = ["Saved", "Success", "Fail"];
      if (this.currentModule != null) {
        if (this.currentModule.id === "root") {
          this.condition.moduleIds = [];
        } else if (this.currentModule.id === "gc") {
          this.condition.moduleIds = [];
          this.condition.filters = ["Trash"];
        } else {
          this.condition.moduleIds = this.currentModule.ids;
        }
      }
      if (this.currentProject != null) {
        this.condition.projectId = this.currentProject.id;
      }

      let url = "/api/automation/list/" + this.currentPage + "/" + this.pageSize;
      this.result = this.$post(url, this.condition, response => {
        let data = response.data;
        this.total = data.itemCount;
        this.tableData = data.listObject;
      });
    },
    handleCommand(cmd) {
      let table = this.$refs.scenarioTable;
      switch (cmd) {
        case "table":
          this.selectAll = false;
          table.toggleAllSelection();
          break;
        case "all":
          this.selectAll = true;
          break
      }
    },
    handleBatchAddCase() {

    },
    handleBatchExecute() {

    },
    selectAllChange() {
      this.handleCommand("table");
    },
    select(selection) {
      this.selection = selection.map(s => s.id);
      console.log(this.selection)
    },
    isSelect(row) {
      console.log(this.selection.includes(row.id))
      return this.selection.includes(row.id)
    },
    edit(row) {
      this.$emit('edit', row);
    },
    execute(row) {

    },
    copy(row) {

    },
    remove(row) {
      if (this.currentModule !== undefined && this.currentModule.id === "gc") {
        this.$get('/api/automation/delete/' + row.id, () => {
          this.$success(this.$t('commons.delete_success'));
          this.search();
        });
        return;
      }
      this.$alert(this.$t('api_test.definition.request.delete_confirm') + ' ' + row.name + " ？", '', {
        confirmButtonText: this.$t('commons.confirm'),
        callback: (action) => {
          if (action === 'confirm') {
            let ids = [row.id];
            this.$post('/api/automation/removeToGc/', ids, () => {
              this.$success(this.$t('commons.delete_success'));
              this.search();
            });
          }
        }
      });
    },
  }
}
</script>

<style scoped>

</style>
