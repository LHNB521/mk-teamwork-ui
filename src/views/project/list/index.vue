<template>
  <div class="project-list-index">
    <wrapper-content :showHeader="false">
      <div class="action">
        <a-button type="primary" icon="plus" @click="doAction(null, 'new')">
          创建新项目
        </a-button>
      </div>
      <!-- <template>
        <a-tree
          class="draggable-tree"
          :default-expanded-keys="expandedKeys"
          draggable
          :tree-data="gData"
          @dragenter="onDragEnter"
          @drop="onDrop"
        />
      </template> -->

      <template>
        <!-- :data-source="dataSource" -->
        <a-table
          :data-source="dataSource"
          :columns="columns"
          childrenColumnName="children"
          :loading="loading"
          rowKey="code"
        >
          <!-- <p slot="expandedRowRender" slot-scope="record" style="margin: 0">
            {{ record.children }}
          </p> -->
          <template slot="name" slot-scope="text, record">
            <router-link :to="'/project/space/task/' + record.code">
              {{ record.name }}
            </router-link>
            <!--            <a-tag color="green" class="m-l" v-show="!item.privated">公开</a-tag>-->
            <a-tag color="green" class="m-l" v-show="record.privated == 0">
              公开
            </a-tag>
          </template>

          <template slot="schedule" slot-scope="text, record">
            <div class="info-item schedule">
              <a-progress :strokeWidth="5" :percent="record.schedule" />
            </div>
          </template>

          <template slot="action" slot-scope="text, record, index">
            <span slot="actions" @click="inviteProjectMember(record)">
              <a-tooltip title="添加成员">
                <a-icon type="user-add" />
              </a-tooltip>
            </span>
            <a-divider type="vertical" />
            <a-icon
              slot="actions"
              type="delete"
              @click="doAction(record, 'del', index)"
            />
            <a-divider type="vertical" />
            <span slot="actions" @click="doAction(record, 'edit', index)">
              <a-tooltip title="项目设置">
                <a-icon type="setting" />
              </a-tooltip>
            </span>
            <a-divider type="vertical" />
            <span slot="actions">
              <a-tooltip
                :title="record.collected ? '取消收藏' : '加入收藏'"
                @click="doAction(record, 'collect', index)"
              >
                <a-icon type="star" v-show="!record.collected" />
                <a-icon
                  type="star"
                  theme="filled"
                  style="color: #ffaf38"
                  v-show="record.collected"
                />
              </a-tooltip>
            </span>
          </template>
        </a-table>
      </template>

      <!--      <template>-->
      <!--        <a-table-->

      <!--            :columns="columns"-->
      <!--            :loading="loading"-->
      <!--            :dataSource="dataSource"-->
      <!--            bordered-->
      <!--        >-->

      <!--        </a-table>-->
      <!--      </template>-->
      <!--            <a-list-->
      <!--                    class="project-list"-->
      <!--                    :loading="loading"-->
      <!--                    itemLayout="horizontal"-->
      <!--                    :dataSource="dataSource"-->
      <!--            >-->
      <!--                <div v-if="showLoadingMore" slot="loadMore"-->
      <!--                     :style="{ textAlign: 'center', marginTop: '12px', height: '32px', lineHeight: '32px' }">-->
      <!--                    <a-spin v-if="loadingMore"/>-->
      <!--                    <a-button v-else @click="onLoadMore">查看更多项目</a-button>-->
      <!--                </div>-->
      <!--                <a-list-item slot="renderItem" slot-scope="item,index">-->
      <!--                    <span slot="actions" @click="inviteProjectMember(item)">-->
      <!--                         <a-tooltip title="添加成员">-->
      <!--                             <a-icon type="user-add"/>-->
      <!--                         </a-tooltip>-->
      <!--                    </span>-->
      <!--                    <span slot="actions" @click="doAction(item,'del',index)">-->
      <!--                         <a-tooltip title="移至回收站">-->
      <!--                              <a-icon type="delete"/>-->
      <!--                         </a-tooltip>-->
      <!--                    </span>-->
      <!--                    <span slot="actions" @click="doAction(item,'edit',index)">-->
      <!--                         <a-tooltip title="项目设置">-->
      <!--                              <a-icon type="setting"/>-->
      <!--                         </a-tooltip>-->
      <!--                    </span>-->
      <!--                    <span slot="actions">-->
      <!--                         <a-tooltip :title="item.collected ? '取消收藏' : '加入收藏'" @click="doAction(item,'collect',index)">-->
      <!--                             <a-icon type="star" v-show="!item.collected"/>-->
      <!--                             <a-icon type="star" theme="filled" style="color: #ffaf38;" v-show="item.collected"/>-->
      <!--                         </a-tooltip>-->
      <!--                    </span>-->
      <!--                    <a-list-item-meta-->
      <!--                            :description="item.description"-->
      <!--                    >-->
      <!--                        <div slot="title">-->
      <!--                            <router-link :to="'/project/space/task/' + item.code">{{item.name}}</router-link>-->
      <!--                            <a-tag color="green" class="m-l" v-show="!item.privated">公开</a-tag>-->
      <!--                        </div>-->
      <!--                        <a-avatar slot="avatar" icon="user" :src="item.cover"/>-->
      <!--                    </a-list-item-meta>-->
      <!--                    <div class="other-info muted">-->
      <!--                        <div class="info-item">-->
      <!--                            <span>创建日期</span>-->
      <!--                            <span>{{moment(item.create_time).format('YYYY-MM-DD')}}</span>-->
      <!--                        </div>-->
      <!--                        <div class="info-item">-->
      <!--                            <span>创建人</span>-->
      <!--                            <span>{{item.owner_name}}</span>-->
      <!--                        </div>-->
      <!--                        <div class="info-item schedule">-->
      <!--                            <span>进度</span>-->
      <!--                            <a-progress :strokeWidth="5" :percent="item.schedule"/>-->
      <!--                        </div>-->
      <!--                    </div>-->
      <!--                </a-list-item>-->
      <!--            </a-list>-->
    </wrapper-content>
    <a-modal
      destroyOnClose
      :width="360"
      v-model="actionInfo.modalStatus"
      :title="actionInfo.modalTitle"
      :bodyStyle="{ paddingBottom: '1px' }"
      :footer="null"
    >
      <a-form @submit.prevent="handleSubmit" :form="form">
        <a-form-item>
          <a-input
            placeholder="项目名称（必填）"
            v-decorator="[
              'name',
              { rules: [{ required: true, message: '请输入项目名称' }] },
            ]"
          />
        </a-form-item>
        <a-form-item>
          <a-select placeholder="项目模板" v-decorator="['templateCode']">
            <a-select-option :key="0">空白项目</a-select-option>
            <a-select-option
              :key="template.code"
              v-for="template in templateList"
              >{{ template.name }}</a-select-option
            >
          </a-select>
        </a-form-item>

        <a-form-item>
          <a-select placeholder="等级" v-decorator="['code']">
            <a-select-option :key="p1.code" v-for="p1 in dataSource">{{
              p1.name
            }}</a-select-option>
          </a-select>
        </a-form-item>

        <a-form-item>
          <a-textarea
            placeholder="项目简介"
            :rows="2"
            v-decorator="['description']"
          />
        </a-form-item>
        <a-form-item>
          <div class="action-btn">
            <a-button
              type="primary"
              htmlType="submit"
              block
              size="large"
              :loading="actionInfo.confirmLoading"
              class="middle-btn"
              >完成并创建</a-button
            >
          </div>
        </a-form-item>
      </a-form>
    </a-modal>
    <a-modal
      destroyOnClose
      class="project-config-modal"
      :width="800"
      v-model="projectModal.modalStatus"
      :title="projectModal.modalTitle"
      :footer="null"
    >
      <project-config
        :code="currentProjectCode"
        @update="updateProject"
      ></project-config>
    </a-modal>
    <invite-project-member
      v-model="showInviteMember"
      :project-code="currentProjectCode"
      v-if="showInviteMember"
    ></invite-project-member>
  </div>
</template>

<script>
import inviteProjectMember from "../../../components/project/inviteProjectMember";
import projectConfig from "@/components/project/projectConfig";
import { list, doData, recycle } from "@/api/project";
import { checkResponse } from "@/assets/js/utils";
import pagination from "@/mixins/pagination";
import moment from "moment";
import { collect } from "../../../api/projectCollect";
import { list as projectTemplates } from "../../../api/projectTemplate";
const columns = [
  {
    key: "name",
    title: "项目名称",
    dataIndex: "name",
    width: "15%",
    scopedSlots: { customRender: "name" },
  },
  {
    key: "create_time",
    title: "创建日期",
    dataIndex: "create_time",
    width: "20%",
    // scopedSlots: {
    //   customRender: "create_time",
    // },
  },
  {
    key: "schedule",
    title: "项目进度",
    dataIndex: "schedule",
    width: "20%",
    scopedSlots: {
      customRender: "schedule",
    },
  },
  {
    key: "action",
    title: "操作",
    dataIndex: "action",
    scopedSlots: {
      customRender: "action",
    },
  },
];

export default {
  components: {
    inviteProjectMember,
    projectConfig,
  },
  mixins: [pagination],
  data() {
    return {
      columns,
      dataSource: [],
      loading: true,
      showLoadingMore: false,
      loadingMore: false,
      showInviteMember: false,
      currentProject: {},
      currentProjectCode: 0,
      currentProjectIndex: 0,
      newData: {
        code: "",
      },
      form: this.$form.createForm(this),
      searchForm: {},
      actionInfo: {
        modalStatus: false,
        confirmLoading: false,
        modalTitle: "编辑项目",
      },
      /*项目设置*/
      projectModal: {
        modalStatus: false,
        modalTitle: "项目设置",
      },
      templateList: [],
    };
  },
  watch: {
    $route: function () {
      this.init();
    },
  },
  created() {
    this.init();
    this.projectTemplates();
  },
  methods: {
    moment,
    init(reset = true) {
      let app = this;
      if (reset) {
        this.dataSource = [];
        this.pagination.page = 1;
        this.pagination.pageSize = 1000;
        this.showLoadingMore = false;
      }
      this.requestData.type = this.$route.params.type;
      app.loading = true;
      list(app.requestData).then((res) => {
        app.dataSource = app.dataSource.concat(res.data.list);
        app.pagination.total = res.data.total;
        app.showLoadingMore = app.pagination.total > app.dataSource.length;
        app.loading = false;
        app.loadingMore = false;
      });
    },
    projectTemplates() {
      projectTemplates({ pageSize: 100, viewType: -1 }).then((res) => {
        this.templateList = res.data.list;
      });
    },
    onLoadMore() {
      this.loadingMore = true;
      this.pagination.page++;
      this.init(false);
    },
    inviteProjectMember(item) {
      this.currentProject = item;
      this.currentProjectCode = item.code;
      this.showInviteMember = true;
    },
    doAction(record, action, index) {
      this.currentProject = record;
      this.currentProjectIndex = index;
      let app = this;
      app.newData = { id: 0 };
      if (action == "new") {
        setTimeout(function () {
          app.form && app.form.resetFields();
        }, 0);
        app.actionInfo.modalStatus = true;
        app.actionInfo.modalTitle = "创建项目";
      } else if (action == "edit") {
        app.currentProjectCode = record.code;
        app.projectModal.modalStatus = true;
      } else if (action == "del") {
        this.$confirm({
          title: "确定放入回收站？",
          content: `一旦将项目「${this.currentProject.name}」放入回收站，所有与项目有关的信息将会被放入回收站`,
          okText: "放入回收站",
          okType: "danger",
          cancelText: "再想想",
          onOk() {
            recycle(record.code).then(() => {
              app.dataSource.splice(index, 1);
              // app.init();
            });
            return Promise.resolve();
          },
        });
      } else if (action == "collect") {
        const type = record.collected ? "cancel" : "collect";
        collect(record.code, type).then(() => {
          app.$set(app.dataSource[index], "collected", !record.collected);
          if (this.requestData.type == "collect") {
            app.dataSource.splice(index, 1);
          }
        });
      }
    },
    updateProject(data) {
      this.dataSource[this.currentProjectIndex] = JSON.parse(
        JSON.stringify(data)
      );
    },
    handleSubmit() {
      let app = this;
      app.form.validateFields((err) => {
        if (!err) {
          app.handleOk();
        }
      });
    },
    handleOk() {
      let app = this;
      app.actionInfo.confirmLoading = true;
      let obj = app.form.getFieldsValue();
      if (app.newData.code) {
        //编辑
        obj.projectCode = app.newData.code;
      } else {
        //新增
        Object.assign(obj, app.newData);
        //设置项目封面20200202
        var j, len;
        for (j = 0, len = this.templateList.length; j < len; j++) {
          if (obj && obj.templateCode == this.templateList[j].code) {
            obj.cover = this.templateList[j].cover;
            break;
          }
        }
      }

      doData(obj).then((res) => {
        app.actionInfo.confirmLoading = false;
        if (!checkResponse(res)) {
          return;
        }
        app.form.resetFields();
        app.newData = { id: 0 };
        app.actionInfo.modalStatus = false;
        app.init();
      });
    },
    handleSearchSubmit() {
      let app = this;
      app.searchForm.validateFields((err) => {
        if (!err) {
          app.search();
        }
      });
    },
    search() {
      let obj = this.searchForm.getFieldsValue();
      Object.assign(this.requestData, obj);
      this.init();
    },
  },
};
</script>
<style lang="less">
@import "~ant-design-vue/lib/style/themes/default";

.project-list-index {
  .project-list {
    .ant-list-item-meta-avatar {
      .ant-avatar {
        width: 50px;
        height: 50px;
        border-radius: 3px;
      }
    }

    .ant-list-item-content {
      .other-info {
        display: flex;

        .info-item {
          display: flex;
          flex-direction: column;
          padding-left: 48px;
        }

        .schedule {
          width: 250px;
        }
      }
    }

    .ant-list-item-action {
      .anticon:hover {
        svg {
          color: @primary-color;
        }
      }
    }
  }
}
</style>
