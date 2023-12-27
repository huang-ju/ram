<template>
  <div id="app">
    <a-table
      :loading="isLoading"
      :columns="columns"
      :data-source="dataList"
      :pagination="pagination"
    >
    </a-table>
  </div>
</template>

<script>
import EosApi from "eosjs-api";

const columns = [
  {
    title: "序号",
    key: "index",
    dataIndex: "index",
  },
  { title: "账号", dataIndex: "username", key: "username" },
  { title: "RAMS数量", dataIndex: "mintcount", key: "mintcount" },
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
    };
  },
  mounted() {
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
  methods: {},
};
</script>

<style>
#app {
  font-weight: bold;
}
</style>
