<template>
  <div>
    <div>
      <div style="margin-left: 1rem">
        <n-h4>查询： {{ query_text_han }}，此为第 {{ curr_page }} 页，共 {{ poem_list_length }}条结果</n-h4>
      </div>

      <div>
        <n-grid
          :cols="n_col"
          :collapsed="gridCollapsed"
          :collapsed-rows="gridCollapsedRows"
          :x-gap="30"
          :y-gap="20"
          v-if="loaded"
        >
          <n-gi v-for="item in poem_list" :key="item">
            <n-card
              title=""
              @click="
                () => {
                  this.rt_push(item.to_poem_link)
                }
              "
            >
              <n-h3>{{ item.poem_title.length > 15 ? item.poem_title.substr(0, 15) + "..." : item.poem_title }}</n-h3>
              <n-h4>{{ item.a_name }}</n-h4>
              <n-p>{{
                item.p_paragraph.length > 10
                  ? item.p_paragraph.substr(0, 10).replace("b", "").replace("r", "").replace("<", "").replace(">", "") +
                    "..."
                  : item.p_paragraph
              }}</n-p>
            </n-card>
          </n-gi>
        </n-grid>
      </div>
      <n-divider />
      <div>
        <n-h4 style="text-align: center"
          >查询： {{ query_text_han }}，此为第 {{ curr_page }} 页，共 {{ poem_list_length }}条结果</n-h4
        >
        <div style="width: 200px; margin: 0 auto">
          <n-button circle style="margin-left: 2rem" :disabled="prev_disabled" @click="prev_page">
            <template #icon>
              <n-icon><FrontIcon /></n-icon>
            </template>
          </n-button>
          <n-button circle style="margin-left: 2rem" :disabled="next_disabled" @click="next_page">
            <template #icon>
              <n-icon><BackIcon /></n-icon>
            </template>
          </n-button>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import { defineComponent } from "vue"
import { NButton, NGrid, NGi, NCard, NH3, NP, NH4, NDivider, NIcon } from "naive-ui"
import axios from "axios"
import { ArrowBackCircleOutline as FrontIcon, ArrowForwardCircleOutline as BackIcon } from "@vicons/ionicons5"
import router from "../../router/index.ts"
export default defineComponent({
  components: {
    NButton,
    NGrid,
    NGi,
    NCard,
    NH3,
    NP,
    NH4,
    NDivider,
    NIcon,
    FrontIcon,
    BackIcon
  },
  methods: {
    rt_push: function (addr) {
      router.push(addr)
    },
    clean_data: function (item) {
      item.p_paragraph = item.p_paragraph.replace(/(?:\r\n|\r|\n)/g, "<br>")
      if (item.p_title === null || item.p_title === "") {
        if (item.r_name === null || item.r_name === "") {
          item.poem_title = "无题"
        } else {
          item.poem_title = item.r_name
        }
      } else {
        item.poem_title = item.p_title
      }
      if (item.a_name === null || item.a_name === "") {
        item.a_name = "无名"
      }
      if (item.p_note === null || item.p_note === "") {
        item.p_note = ""
      }
      if (item.r_name === null || item.r_name === "") {
        item.has_rhythmic = false
      } else {
        item.has_rhythmic = true
      }
      item.to_poem_link = "/poem/" + item.p_id
    },
    prev_page: function () {
      if (this.curr_page > 1) {
        this.curr_page -= 1
        this.next_disabled = false
        axios
          .get(
            this.BASE_URL +
              "/search/poem?query_type=" +
              this.query_method +
              "&query_str=" +
              this.query_text +
              "&items_per_page=" +
              this.items_per_pag +
              "&curr_page=" +
              this.curr_page
          )
          .then((response) => {
            this.poem_list = response.data.result
            this.poem_list_length = response.data.num_res
            for (let i = 0; i < this.poem_list_length; i++) {
              this.clean_data(this.poem_list[i])
            }
            if (this.poem_list_length < this.items_per_pag) {
              this.next_disabled = true
            }
            if (this.curr_page <= 1) {
              this.prev_disabled = true
            }
            this.loaded = true
            this.scroll_up()
          })
          .catch(function (error) {
            console.log(error)
          })
      }
    },
    next_page: function () {
      this.curr_page += 1
      axios
        .get(
          this.BASE_URL +
            "/search/poem?query_type=" +
            this.query_method +
            "&query_str=" +
            this.query_text +
            "&items_per_page=" +
            this.items_per_pag +
            "&curr_page=" +
            this.curr_page
        )
        .then((response) => {
          this.poem_list = response.data.result
          this.poem_list_length = response.data.num_res
          for (let i = 0; i < this.poem_list_length; i++) {
            this.clean_data(this.poem_list[i])
          }
          if (this.poem_list_length < this.items_per_pag) {
            this.next_disabled = true
          }
          if (this.curr_page > 1) {
            this.prev_disabled = false
          }
          this.loaded = true
          this.scroll_up()
        })
        .catch(function (error) {
          console.log(error)
        })
    },
    scroll_up: function () {
      window.scrollTo(0, 0)
    }
  },
  mounted() {
    axios
      .get(
        this.BASE_URL +
          "/search/poem?query_type=" +
          this.query_method +
          "&query_str=" +
          this.query_text +
          "&items_per_page=" +
          this.items_per_pag +
          "&curr_page=" +
          this.curr_page
      )
      .then((response) => {
        this.poem_list = response.data.result
        this.poem_list_length = response.data.num_res
        for (let i = 0; i < this.poem_list_length; i++) {
          this.clean_data(this.poem_list[i])
        }
        if (this.poem_list_length < this.items_per_pag) {
          this.next_disabled = true
        }
        if (/Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent)) {
          this.is_mobile = true
          this.n_col = 1
        }
        this.loaded = true
      })
      .catch(function (error) {
        console.log(error)
      })
    if (this.query_method === "title") {
      this.query_text_han = "标题"
    } else if (this.query_method === "author") {
      this.query_text_han = "作者"
    } else if (this.query_method === "para") {
      this.query_text_han = "正文"
    } else if (this.query_method === "rhy") {
      this.query_text_han = "词牌/韵律"
    }
  },
  data() {
    return {
      items_per_pag: 50,
      curr_page: 1,
      poem_list: [],
      poem_list_length: 0,
      loaded: false,
      query_text_han: "",
      prev_disabled: true,
      next_disabled: false,
      is_mobile: false,
      n_col: 5,
      isGridCollapsed: false, // 控制网格是否折叠
      collapsedRows: 2 // 控制折叠的行数
    }
  },
  setup() {
    return {}
  },
  computed: {
    gridCollapsed() {
      return this.isGridCollapsed
    },
    gridCollapsedRows() {
      return this.isGridCollapsed ? this.collapsedRows : null
    }
  },
  props: {
    // eslint-disable-next-line vue/require-prop-type-constructor
    query_method: "",
    // eslint-disable-next-line vue/require-prop-type-constructor
    query_text: ""
  }
})
</script>

<style scoped>
.n-card:hover {
  transition-duration: 0.7s;
  border-color: #18a058;
  cursor: pointer;
}
</style>
