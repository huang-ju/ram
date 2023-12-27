<template>
  <div id="app">
    <a-tabs default-active-key="1" @change="handleChangeTab">
      <a-tab-pane key="rams" tab="RAMS排行榜">
        <a-table
          :loading="isLoading"
          :columns="columns"
          :data-source="dataList"
          :pagination="pagination"
        >
        </a-table>
      </a-tab-pane>
      <a-tab-pane key="ram" tab="RAM排行榜" force-render>
        <a-table
          :loading="isLoading"
          :columns="ramColumns"
          :data-source="ramList"
          :pagination="false"
        >
        </a-table>
      </a-tab-pane>
    </a-tabs>
  </div>
</template>

<script>
import EosApi from "eosjs-api";
import axios from "axios";
const columns = [
  {
    title: "Index",
    key: "index",
    dataIndex: "index",
  },
  { title: "Account", dataIndex: "username", key: "username" },
  { title: "RAMS", dataIndex: "mintcount", key: "mintcount" },
];

const ramColumns = [
  {
    title: "Index",
    key: "index",
    dataIndex: "index",
  },
  { title: "Account", dataIndex: "username", key: "username" },
  { title: "RAM", dataIndex: "ram", key: "ram" },
  { title: "RAMS", dataIndex: "rams", key: "rams" },
];
export default {
  name: "App",
  components: {},
  data() {
    return {
      isLoading: false,
      columns: columns,
      dataList: [],
      pagination: {
        defaultPageSize: 100,
      },
      ramList: [],
      ramColumns: ramColumns,
    };
  },
  mounted() {
    this.getRamsList();
  },
  methods: {
    handleChangeTab(e) {
      if (e === "rams") {
        this.getRamsList();
      } else if (e === "ram") {
        this.getRam100List();
      }
      console.log("handleChangeTab", e);
    },
    getRamsList() {
      if (this.dataList.length != 0) {
        return;
      }
      this.isLoading = true;
      const eos = EosApi({
        httpEndpoint: "https://eos.greymass.com",
        verbose: false,
        fetchConfiguration: {},
      });
      eos
        .getTableRows({
          json: true,
          code: "rams.eos",
          scope: "rams.eos",
          table: "users",
          lower_bound: null,
          upper_bound: null,
          index_position: 1,
          key_type: "",
          limit: "10000",
          reverse: false,
          show_payer: true,
        })
        .then((res) => {
          this.dataList = res.rows.map((v) => {
            return {
              username: v.payer,
              mintcount: v.data.mintcount,
            };
          });
          this.dataList = this.dataList
            .sort((a, b) => b.mintcount - a.mintcount)
            .map((v, index) => {
              return {
                ...v,
                index: index + 1,
              };
            });
        })
        .catch((err) => {
          console.log("err", err);
        })
        .finally(() => {
          this.isLoading = false;
        });
    },
    getRam100List() {
      if (this.ramList.length != 0) {
        return;
      }
      this.isLoading = true;
      axios
        .get("https://api.light.xeos.me/api/topram/eos/100")
        .then((res) => {
          console.log("res", res);
          const { data = [] } = res || {};
          let ramsMap = {};
          this.dataList.forEach((v) => {
            ramsMap[v.username] = v.mintcount;
          });
          this.ramList = data.map((v, index) => {
            return {
              index: index + 1,
              username: v[0],
              ram: `${(v[1] / 1024 / 1024).toFixed(2)} MB`,
              rams: ramsMap[v[0]] || ''
            };
          });
        })
        .finally(() => {
          this.isLoading = false;
        });
    },
  },
};
</script>

<style>
#app {
  font-weight: bold;
}
</style>
