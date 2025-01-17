<template>
  <div class="search">
    <div class="search-img" flex-center>
      <q-carousel
        swipeable
        style="background: transparent"
        height="100%"
        animated
        infinite
        transition-prev="scale"
        transition-next="scale"
        class="rounded-borders"
        ref="carousel"
        v-model="slide"
        @before-transition="
          (e) => {
            changeSearchMethod(e);
          }
        "
      >
        <template v-slot:control>
          <q-carousel-control
            position="top-left"
            :offset="[18, 18]"
            class="q-gutter-xs control"
          >
            <q-btn
              push
              dense
              class="search_method_btn"
              :text-color="currentSearch === 'baidu' ? 'white' : 'white'"
              :icon="currentSearch === 'baidu' ? 'shopping_cart' : 'search'"
              @click="$refs.carousel.next()"
            >
              <div class="q-ml-sm">
                {{ currentSearch === "baidu" ? "淘宝搜索" : "百度搜索" }}
              </div>
            </q-btn>
          </q-carousel-control>
        </template>
        <q-carousel-slide name="baidu" class="column no-wrap flex-center">
          <img src="../../../assets/imgs/baidu.png" />
        </q-carousel-slide>
        <q-carousel-slide name="taobao" class="column no-wrap flex-center">
          <img src="../../../assets/imgs/taobao.png" />
        </q-carousel-slide>
      </q-carousel>
    </div>
    <div class="search-container">
      <div class="baidu-search">
        <q-input
          v-model="search_input"
          clearable
          class="text-h5 search_input"
          :outlined="!!!search_input"
          :filled="!!search_input"
          rounded
          @blur="hideSearchContent"
          @focus="showSearchContent = true"
          placeholder="请输入内容"
          @input="onInput"
          @keydown.40.native="nextOption"
          @keydown.38.native="lastOption"
          @keydown.13.native="submit"
          @keydown.ctrl.67="copy_translate_result"
          @keydown.meta.67="copy_translate_result"
        />
        <div class="search-content" v-show="showSearchContent">
          <q-card
            square
            v-for="(item, idx) in arr"
            :key="idx"
            :data-content="item"
            :data-id="idx"
            @click="itemClick($event)"
            :class="idx == index ? 'active' : ''"
          >
            <q-card-section class="text-primary">
              <div class="text-h6">{{ item }}</div>
            </q-card-section>
          </q-card>
        </div>
      </div>
      <div
        class="translate-search q-ml-md"
        :style="{
          width: translate_result.length > 10 ? '20%' : '10%',
        }"
      >
        <q-input
          class="translate-input"
          v-model="translate_result"
          rounded
          outlined
          placeholder="翻译结果"
          :disable="!translate_result"
        >
          <template v-slot:append>
            <q-icon name="content_copy" @click="copy_translate_result">
              <q-tooltip
                :value="!!translate_result"
                anchor="center right"
                self="center left"
              >
                ctrl+c
              </q-tooltip>
            </q-icon>
          </template>
        </q-input>
      </div>
    </div>
  </div>
</template>
<script>
import md5 from "js-md5";
import Api from "../../../api/api";
export default {
  name: "Search",
  data() {
    return {
      timer: {
        translate: null, //节流定时器
      },
      translate_result: "",
      slide: "baidu", //图片切换
      search_input: "", //搜索内容
      arr: [], //搜索后结果
      index: -1, //当前选中项
      showSearchContent: true, //是否显示搜索后选框
      currentSearch: "baidu", //当前搜索方式baidu/taobao
    };
  },
  computed: {
    hasContent() {
      return this.arr.length > 0;
    },
  },

  watch: {
    index() {
      if (this.index >= this.arr.length - 1) {
        this.index = this.arr.length - 1;
      } else if (this.index <= -1) {
        this.index = -1;
      }
      this.search_input = this.arr[this.index];
    },
    search_input(val) {
      var val = val || "";
      if (val.length === 0) {
        this.translate_result = "";
        this.arr = [];
      }
    },
  },
  methods: {
    hideSearchContent() {
      setTimeout(() => {
        this.arr = [];
        this.showSearchContent = false;
      }, 150);
    },
    onInput() {
      if (!!this.search_input) {
        this.getData(); //开始请求百度搜索候选栏
        this.translate(); //百度翻译
      }
    },
    nextOption(event) {
      // 按的下键
      this.index = this.index + 1;
    },
    lastOption(event) {
      // 按的下键
      this.index = this.index - 1;
    },
    render_candidate(data) {
      if (this.currentSearch === "baidu") {
        this.arr = data.s;
      } else if (this.currentSearch === "taobao") {
        let newdata = [];
        data.result.forEach((item, index) => {
          newdata[index] = item[0];
        });
        this.arr = newdata;
      }
    },
    getData() {
      switch (this.currentSearch) {
        case "baidu":
          Api.searchBaidu({
            wd: this.search_input,
            callback_method: this.render_candidate,
          });
          break;
        case "taobao":
          Api.searchTaobao({
            q: this.search_input,
            callback_method: this.render_candidate,
          });
          break;
        default:
          break;
      }
    },
    itemClick(event) {
      if (this.currentSearch === "baidu") {
        window.open(
          "https://www.baidu.com/s?wd=" + event.currentTarget.dataset.content
        );
      } else if (this.currentSearch === "taobao") {
        window.open(
          "https://s.taobao.com/search?q=" + event.currentTarget.dataset.content
        );
      }
    },
    submit() {
      if (this.currentSearch === "baidu") {
        window.open("https://www.baidu.com/s?wd=" + this.search_input);
      } else if (this.currentSearch === "taobao") {
        window.open("https://s.taobao.com/search?q=" + this.search_input);
      }
    },
    changeSearchMethod(e) {
      // 改变了搜索内容
      this.currentSearch = e;
    },
    translate() {
      let query = this.search_input;
      if (this.timer.translate) {
        clearTimeout(this.timer.translate);
      }
      this.timer.translate = setTimeout(() => {
        let appid = "20201013000588468";
        let key = "S31B040thI1pprUuNIXM";
        let salt = new Date().getTime();
        Api.translate({
          q: query,
          appid,
          salt,
          from: "zh",
          to: "en",
          sign: md5(appid + query + salt + key),
        }).then((res) => {
          let { trans_result, error_code } = res;
          this.translate_result = trans_result[0]?.dst || "访问过快";
          this.timer.translate = null;
        });
      }, 700);
    },
    copy_translate_result() {
      this.$copyText(this.translate_result);
      this.$q.notify({ color: "orange-6", message: "复制成功" });
      this.wd = "";
      this.translate_result = "";
    },
  },
  mounted() {
    window.addEventListener("blur", () => {
      document.title = "导航在这里哦 Navigation";
      this.search_input = "";
    });
  },
};
</script>

<style lang="stylus" scoped="scoped">
.search {
  .search-img {
    &:hover {
      .control {
        opacity: 1;
      }
    }
  }

  .search-img {
    width: 100%;
    height: 200px;

    .control {
      opacity: 0;
      transition: 0.3s;

      .search_method_btn {
        background-color: var(--main-color-1);
        opacity: var(--main-opacity);
      }
    }

    img {
      display: block;
      height: 200px;
      margin: 0 auto;
      opacity: var(--main-opacity);
    }
  }

  .search-container {
    display: flex;
    height: 56px;
    margin-top: 10px;

    ::v-deep .q-input {
      opacity: var(--main-opacity);

      .q-field__control {
        background-color: var(--input-background-color);

        input {
          color: var(--input-font-color);
        }
      }

      .q-field__control:before {
        border: var(--input-border);
      }
    }

    ::v-deep .material-icons {
      color: var(--input-font-color);
    }

    .baidu-search {
      flex: 1;
      position: relative;
      transition: 0.3s;

      .search-content {
        position: absolute;
        z-index: 9;
        width: 100%;

        .q-card {
          cursor: pointer;
          background-color: var(--default-background-color);
          opacity: var(--main-opacity);

          &:hover, &.active {
            background-color: var(--main-color-3);
          }
        }
      }
    }

    .translate-search {
      height: 100%;
      transition: 0.3s;
      padding: 0;
    }
  }
}

@media screen and (max-width: 768px) {
  .search {
    .search-img {
      img {
        width: 100%;
        height: auto;
      }
    }
  }
}
</style>