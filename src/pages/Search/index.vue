<template>
  <div>
    <TypeNav />
    <div class="main">
      <div class="py-container">
        <!--bread-->
        <div class="bread">
          <ul class="fl sui-breadcrumb">
            <li>
              <a href="#">全部结果</a>
            </li>
          </ul>
          <ul class="fl sui-tag">
            <!-- 分类的面包屑 -->
            <li class="with-x" v-if="searchParams.categoryName">
              {{ searchParams.categoryName }}
              <i @click="removeCategoryName">×</i>
            </li>
            <!-- 关键字的面包屑 -->
            <li class="with-x" v-if="searchParams.keyword">
              {{ searchParams.keyword }}
              <i @click="removeKeyword">×</i>
            </li>
            <!-- 品牌信息的面包屑 -->
            <li class="with-x" v-if="searchParams.trademark">
              {{ searchParams.trademark.split(":")[1] }}
              <i @click="removeTrademark">×</i>
            </li>
            <!-- 平台的售卖的属性值面包屑 -->
            <li
              class="with-x"
              v-for="(item, index) in searchParams.props"
              :key="index"
            >
              {{ item.split(":")[1] }}
              <i @click="removeAttr(index)">×</i>
            </li>
          </ul>
        </div>

        <!--selector-->
        <SearchSelector @trademarkInfo="trademarkInfo" @attrInfo="attrInfo" />

        <!--details-->
        <div class="details clearfix">
          <div class="sui-navbar">
            <div class="navbar-inner filter">
              <!-- 排序结构 -->
              <ul class="sui-nav">
                <li
                  :class="{ active: isInclude('1') }"
                  @click="changeOrder('1')"
                >
                  <a href="#"
                    >综合
                    <span v-show="isInclude('1')">
                      <up-two
                        theme="outline"
                        size="14"
                        fill="white"
                        v-show="isInclude('a')"
                      />
                      <down-two
                        theme="outline"
                        size="14"
                        fill="white"
                        v-show="isInclude('d')"
                      />
                    </span>
                  </a>
                </li>
                <li
                  :class="{ active: isInclude('2') }"
                  @click="changeOrder('2')"
                >
                  <a href="#"
                    >价格
                    <span v-show="isInclude('2')">
                      <up-two
                        theme="outline"
                        size="14"
                        fill="white"
                        v-show="isInclude('a')"
                      />
                      <down-two
                        theme="outline"
                        size="14"
                        fill="white"
                        v-show="isInclude('d')"
                      />
                    </span>
                  </a>
                </li>
                <!-- <li>
                  <a href="#">销量</a>
                </li>
                <li>
                  <a href="#">新品</a>
                </li>
                <li>
                  <a href="#">评价</a>
                </li>
                <li>
                  <a href="#">价格⬆</a>
                </li>
                <li>
                  <a href="#">价格⬇</a>
                </li> -->
              </ul>
            </div>
          </div>
          <!-- 销售产品列表 -->
          <div class="goods-list">
            <ul class="yui3-g">
              <li 
                class="yui3-u-1-5" 
                v-for="good in goodsList" 
                :key="good.id"
              >
                <div class="list-wrap">
                  <div class="p-img">
                    <router-link :to="`/detail/${good.id}`">
                      <img v-lazy="good.defaultImg" />
                    </router-link>
                  </div>
                  <div class="price">
                    <strong>
                      <em>¥ </em>
                      <i>{{ good.price }}.00</i>
                    </strong>
                  </div>
                  <div class="attr">
                    <a target="_blank" href="item.html" title="114514">{{
                      good.title
                    }}</a>
                  </div>
                  <div class="commit">
                    <i class="command">已有<span>2000</span>人评价</i>
                  </div>
                  <div class="operate">
                    <a
                      href="success-cart.html"
                      target="_blank"
                      class="sui-btn btn-bordered btn-danger"
                      >加入购物车</a
                    >
                    <a href="javascript:void(0);" class="sui-btn btn-bordered"
                      >收藏</a
                    >
                  </div>
                </div>
              </li>
            </ul>
          </div>
          <!-- 分页器 -->
          <Pagination 
            :pageNo="searchParams.pageNo" 
            :pageSize="searchParams.pageSize" 
            :total="total" 
            :continues="5"
            @getPageNo="getPageNo"
          />
        </div>

      </div>
    </div>
  </div>
</template>

<script>
import { mapGetters, mapState } from "vuex";
import { UpTwo, DownTwo } from "@icon-park/vue";
import "@icon-park/vue/styles/index.css";

import SearchSelector from "./SearchSelector/SearchSelector";
import Pagination from "../../components/Pagination/index.vue";
export default {
  name: "Search",
  // props: ['pageNo'], 
  data() {
    return {
      //带给服务器的参数
      searchParams: {
        //一级分类的id
        category1Id: "",
        //二级分类的id
        category2Id: "",
        //三级分类的id
        category3Id: "",
        //分类名
        categoryName: "",
        //关键字
        keyword: "",
        //排序（初始状态应该是综合且降序）
        order: "1:desc",
        //页码
        pageNo: 1,
        //每页展示数据个数
        pageSize: 10,
        //平台售卖属性的参数
        props: [],
        //品牌
        trademark: "",
      },
    };
  },
  components: {
    SearchSelector,
    UpTwo,
    DownTwo,
    Pagination
},
  //在发请求之前，把接口需要传递的参数进行整理
  beforeMount() {
    //复杂的写法
    // this.searchParams.category1Id = this.$route.query.category1Id
    // this.searchParams.category2Id = this.$route.query.category2Id
    // this.searchParams.category3Id = this.$route.query.category3Id
    // this.searchParams.categoryName = this.$route.query.categoryName
    // this.searchParams.keyword = this.$route.params.keyword
    //Object.assign合并对象写法
    Object.assign(this.searchParams, this.$route.query, this.$route.params);
  },
  mounted() {
    //在发请求之前带给服务器参数（searchParams参数发生变化有数值带给服务器）
    this.getData();
  },
  computed: {
    //mapGetters传递数组，因为getters计算是没有划分模块的
    ...mapGetters(["goodsList"]),
    //获取search模块产品的数量
    ...mapState({
      total: state=> state.search.searchList.total
    })
  },
  methods: {
    //向服务器发请求获取search模块数据（根据参数不同返回不同展示数据）
    getData() {
      this.$store.dispatch("getSearchList", this.searchParams);
    },
    removeId() {
      this.searchParams.category1Id = undefined;
      this.searchParams.category2Id = undefined;
      this.searchParams.category3Id = undefined;
    },
    isEmptyObj(data) {
      if (JSON.stringify(data) === "{}") return true;
    },
    //删除分类的名字
    removeCategoryName() {
      //把带给服务器的参数置空
      //带给服务器的参数是可有可无的，但属性值为空的字符串还是会被带给服务器
      //但是把相应的字段变为undefined，此字段就不会被带给服务器
      this.searchParams.categoryName = undefined;
      this.removeId();
      //向服务器发请求
      this.getData();

      //地址栏也需要修改：进行路由跳转（现在的路由跳转只是跳转到自己这里）
      //本意是删除query，如果路径中有params，不应该删除，带着params进行跳转
      if (this.isEmptyObj(this.$route.params))
        this.$router.push({ name: "home" });
      else this.$router.push({ name: "search", params: this.$route.params });
    },
    //删除关键字
    removeKeyword() {
      //把带给服务器的参数置空
      this.searchParams.keyword = undefined;
      //再次发请求
      this.getData();
      //通知Header删除文本框
      this.$bus.$emit("clear");
      //进行路由跳转
      if (this.isEmptyObj(this.$route.query))
        this.$router.push({ name: "home" });
      else this.$router.push({ name: "search", query: this.$route.query });
    },
    //删除品牌信息
    removeTrademark() {
      //将品牌信息清空
      this.searchParams.trademark = undefined;
      //再次发请求
      this.getData();
    },
    removeAttr(index) {
      //再次整理参数
      this.searchParams.props.splice(index, 1);
      //再次发请求
      this.getData();
    },
    //自定义事件回调
    trademarkInfo(trademark) {
      //整理品牌字段的参数
      this.searchParams.trademark = `${trademark.tmId}:${trademark.tmName}`;
      //再次发请求
      this.getData();
    },
    attrInfo(attr, item) {
      //参数格式
      let props = `${attr.attrId}:${item}:${attr.attrName}`;
      //数组去重
      if (this.searchParams.props.indexOf(props) === -1)
        //推入数组
        this.searchParams.props.push(props);
      //再次发请求
      this.getData();
    },
    isInclude(value) {
      return this.searchParams.order.indexOf(value) !== -1;
    },
    //flag：标记，代表用户点击的选项
    changeOrder(flag) {
      //获得最开始的状态
      let originFlag = this.searchParams.order.split(':')[0]
      let originSort = this.searchParams.order.split(':')[1]
      //准备一个新的order属性值
      let newOrder = ''
      if(flag === originFlag) {
          newOrder = `${originFlag}:${originSort='desc'? 'asc': 'desc'}`
      } else newOrder = `${flag}:desc`
      //赋新值
      this.searchParams.order = newOrder
      //再发送请求
      this.getData()
    },
    //获取当前页码
    getPageNo(pageNo) {
      // //整理带给服务器的参数
      // this.searchParams.pageNo = pageNo
      // //再次发请求
      // this.getData()
      this.searchParams.pageNo = pageNo
      this.getData()
    }
  },
  watch: {
    //监听组件的$route属性
    //$route，是vue-router提供的，不必用深度监听
    $route() {
      //再次整理最新的商品名字参数
      // this.searchParams.category1Id = this.$route.query.category1Id;
      // this.searchParams.category2Id = this.$route.query.category2Id;
      // this.searchParams.category3Id = this.$route.query.category3Id;
      // this.searchParams.categoryName = this.$route.query.categoryName;

      //先把用户前面存储的1|2|3级别ID清除
      this.removeId();

      //再次发请求之前整理给服务器的参数
      Object.assign(this.searchParams, this.$route.query, this.$route.params);
      //再次发起ajax请求
      this.getData();
    },
  },
};
</script>

<style lang="less" scoped>
.main {
  margin: 10px 0;

  .py-container {
    width: 1200px;
    margin: 0 auto;

    .bread {
      margin-bottom: 5px;
      overflow: hidden;

      .sui-breadcrumb {
        padding: 3px 15px;
        margin: 0;
        font-weight: 400;
        border-radius: 3px;
        float: left;

        li {
          display: inline-block;
          line-height: 18px;

          a {
            color: #666;
            text-decoration: none;

            &:hover {
              color: #4cb9fc;
            }
          }
        }
      }

      .sui-tag {
        margin-top: -5px;
        list-style: none;
        font-size: 0;
        line-height: 0;
        padding: 5px 0 0;
        margin-bottom: 18px;
        float: left;

        .with-x {
          font-size: 12px;
          margin: 0 5px 5px 0;
          display: inline-block;
          overflow: hidden;
          color: #000;
          background: #f7f7f7;
          padding: 0 7px;
          height: 20px;
          line-height: 20px;
          border: 1px solid #dedede;
          white-space: nowrap;
          transition: color 400ms;
          cursor: pointer;

          i {
            margin-left: 10px;
            cursor: pointer;
            font: 400 14px tahoma;
            display: inline-block;
            height: 100%;
            vertical-align: middle;
          }

          &:hover {
            color: #28a3ef;
          }
        }
      }
    }

    .details {
      margin-bottom: 5px;

      .sui-navbar {
        overflow: visible;
        margin-bottom: 0;

        .filter {
          min-height: 40px;
          padding-right: 20px;
          background: #fbfbfb;
          border: 1px solid #e2e2e2;
          padding-left: 0;
          border-radius: 0;
          box-shadow: 0 1px 4px rgba(0, 0, 0, 0.065);

          .sui-nav {
            position: relative;
            left: 0;
            display: block;
            float: left;
            margin: 0 10px 0 0;

            li {
              float: left;
              line-height: 18px;

              a {
                display: block;
                cursor: pointer;
                padding: 11px 15px;
                color: #777;
                text-decoration: none;
              }

              &.active {
                a {
                  background: #e1251b;
                  color: #fff;
                }
              }
            }
          }
        }
      }

      .goods-list {
        margin: 20px 0;

        ul {
          display: flex;
          flex-wrap: wrap;

          li {
            height: 100%;
            width: 20%;
            margin-top: 10px;
            line-height: 28px;

            .list-wrap {
              .p-img {
                padding-left: 15px;
                width: 215px;
                height: 255px;

                a {
                  color: #666;

                  img {
                    max-width: 100%;
                    height: auto;
                    vertical-align: middle;
                  }
                }
              }

              .price {
                padding-left: 15px;
                font-size: 18px;
                color: #c81623;

                strong {
                  font-weight: 700;

                  i {
                    margin-left: -5px;
                  }
                }
              }

              .attr {
                padding-left: 15px;
                width: 85%;
                overflow: hidden;
                margin-bottom: 8px;
                min-height: 38px;
                cursor: pointer;
                line-height: 1.8;
                display: -webkit-box;
                -webkit-box-orient: vertical;
                -webkit-line-clamp: 2;

                a {
                  color: #333;
                  text-decoration: none;
                }
              }

              .commit {
                padding-left: 15px;
                height: 22px;
                font-size: 13px;
                color: #a7a7a7;

                span {
                  font-weight: 700;
                  color: #646fb0;
                }
              }

              .operate {
                padding: 12px 15px;

                .sui-btn {
                  display: inline-block;
                  padding: 2px 14px;
                  box-sizing: border-box;
                  margin-bottom: 0;
                  font-size: 12px;
                  line-height: 18px;
                  text-align: center;
                  vertical-align: middle;
                  cursor: pointer;
                  border-radius: 0;
                  background-color: transparent;
                  margin-right: 15px;
                }

                .btn-bordered {
                  min-width: 85px;
                  background-color: transparent;
                  border: 1px solid #8c8c8c;
                  color: #8c8c8c;

                  &:hover {
                    border: 1px solid #666;
                    color: #fff !important;
                    background-color: #666;
                    text-decoration: none;
                  }
                }

                .btn-danger {
                  border: 1px solid #e1251b;
                  color: #e1251b;

                  &:hover {
                    border: 1px solid #e1251b;
                    background-color: #e1251b;
                    color: white !important;
                    text-decoration: none;
                  }
                }
              }
            }
          }
        }
      }

      .page {
        width: 733px;
        height: 66px;
        overflow: hidden;
        float: right;

        .sui-pagination {
          margin: 18px 0;

          ul {
            margin-left: 0;
            margin-bottom: 0;
            vertical-align: middle;
            width: 490px;
            float: left;

            li {
              line-height: 18px;
              display: inline-block;

              a {
                position: relative;
                float: left;
                line-height: 18px;
                text-decoration: none;
                background-color: #fff;
                border: 1px solid #e0e9ee;
                margin-left: -1px;
                font-size: 14px;
                padding: 9px 18px;
                color: #333;
              }

              &.active {
                a {
                  background-color: #fff;
                  color: #e1251b;
                  border-color: #fff;
                  cursor: default;
                }
              }

              &.prev {
                a {
                  background-color: #fafafa;
                }
              }

              &.disabled {
                a {
                  color: #999;
                  cursor: default;
                }
              }

              &.dotted {
                span {
                  margin-left: -1px;
                  position: relative;
                  float: left;
                  line-height: 18px;
                  text-decoration: none;
                  background-color: #fff;
                  font-size: 14px;
                  border: 0;
                  padding: 9px 18px;
                  color: #333;
                }
              }

              &.next {
                a {
                  background-color: #fafafa;
                }
              }
            }
          }

          div {
            color: #333;
            font-size: 14px;
            float: right;
            width: 241px;
          }
        }
      }
    }
  }
}
</style>