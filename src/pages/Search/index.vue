<template>
  <div>
    
    <!-- 商品分类三级列表 -->
    <TypeNav />
    <div class="main">
      <div class="py-container">
        <!--bread 面包屑-->
        <div class="bread">
          <ul class="fl sui-breadcrumb">
            <li>
              <a href="#">全部结果</a>
            </li>
          </ul>
          <ul class="fl sui-tag">
            <!-- 分类的面包屑 -->
            <li class="with-x" v-if="searchParams.categoryName">
              {{ searchParams.categoryName
              }}<i @click="removeCategoryName">×</i>
            </li>
            <!-- 关键字面包屑 -->
            <li class="with-x" v-if="searchParams.keyword">
              {{ searchParams.keyword }}<i @click="removeKeyword">×</i>
            </li>
            <!-- 品牌信息面包屑 -->
            <li class="with-x" v-if="searchParams.trademark">
              {{ searchParams.trademark.split(":")[1]
              }}<i @click="removeTrademark">×</i>
            </li>
            <!-- 商品属性面包屑 -->
            <li
              class="with-x"
              v-for="(attrValue, index) in searchParams.props"
              :key="index"
            >
              {{ attrValue.split(":")[1] }}<i @click="removeAttr(index)">×</i>
            </li>
          </ul>
          
        </div>

        <!--selector-->
        <SearchSelector
          @trademarkInfo="trademarkInfo"
          @attrInfo="attrInfo"
        ></SearchSelector>

        <!--details-->
        <div class="details clearfix">
          <!-- 导航条 -->
          <div class="sui-navbar">
            <div class="navbar-inner filter">
              <!-- 排序结构 -->
              <ul class="sui-nav">
                <li :class="{ active: isOne }" @click="changeOder('1')">
                  <a
                    >综合<span
                      v-show="isOne"
                      class="iconfont"
                      :class="{ 'icon-UP': isEsc, 'icon-DOWN': isDesc }"
                    ></span
                  ></a>
                </li>
                <li :class="{ active: isTwo }" @click="changeOder('2')">
                  <a
                    >价格<span
                      v-show="isTwo"
                      class="iconfont"
                      :class="{ 'icon-UP': isEsc, 'icon-DOWN': isDesc }"
                    ></span
                  ></a>
                </li>
              </ul>
            </div>
          </div>
          <!-- 销售产品列表 -->
          <div class="goods-list">
            <ul class="yui3-g">
              <li
                class="yui3-u-1-5"
                v-for="goodList in goodsList"
                :key="goodList.id"
              >
                <div class="list-wrap">
                  <div class="p-img">
                    <!-- 在路由跳转的时候切记别忘了带id（params）参数 -->
                    <router-link :to="`/detail/${goodList.id}`"
                      ><img v-lazy="goodList.defaultImg"
                    /></router-link>
                  </div>
                  <div class="price">
                    <strong>
                      <em>¥ </em>
                      <i>{{ goodList.price }}</i>
                    </strong>
                  </div>
                  <div class="attr">
                    <a
                      target="_blank"
                      href="item.html"
                      title="促销信息，下单即赠送三个月CIBN视频会员卡！【小米电视新品4A 58 火爆预约中】"
                      >{{ goodList.title }}</a
                    >
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
          <!-- 分页器:测试分页器阶段 -->
          <Pagination
            :pageNo="searchParams.pageNo"
            :pageSize="searchParams.pageSize"
            :total="total"
            :continues="5"
            @getPageNo="getPageNo"
          ></Pagination>
        </div>
      </div>
    </div>
    
  </div>
</template>

<script>
import { mapGetters, mapState } from "vuex";
import SearchSelector from "./SearchSelector/SearchSelector";
export default {
  name: "Search",
  data() {
    return {
      
      //带给服务器的参数
      searchParams: {
        //一级分类的ID
        category1Id: "",
        //二级分类的ID
        category2Id: "",
        //三级分类的ID
        category3Id: "",
        //分类名字
        categoryName: "",
        //关键字
        keyword: "",
        //排序:初始化为综合|降序
        order: "1:desc",
        //分页器参数，代表当前是第几页
        pageNo: 1,
        //每页展示多少数据
        pageSize: 10,
        //平台售卖属性
        props: [],
        //品牌
        trademark: "",
      },
    };
  },
  components: {
    SearchSelector,
  },
  //当前组件挂载完毕之前执行一次【在mounted之前】
  beforeMount() {
    //在发请求之前，把接口需要传递的参数进行整理（把参数整理好，服务器就会返回要查询的数据）
    Object.assign(this.searchParams, this.$route.query, this.$route.params);
  },
  //组件挂载完毕z执行一次【仅仅执行一次】
  mounted() {
    this.getData();
  },
  computed: {
    ...mapGetters(
      //mapGetters里的写法：传递的数组，因为getters计算时没有划分模块【home，search】
      ["goodsList", "trademarkList", "attrsList"]
    ),
    isOne() {
      return this.searchParams.order.indexOf("1") != -1;
    },
    isTwo() {
      return this.searchParams.order.indexOf("2") != -1;
    },
    isEsc() {
      return this.searchParams.order.indexOf("asc") != -1;
    },
    isDesc() {
      return this.searchParams.order.indexOf("desc") != -1;
    },
    //获取search模块展示产品一共多少数据
    ...mapState({ total:state=>state.search.searchList.total }),
  },
  methods: {
    //向服务器获取Search模块的数据（根据参数不同，返回不同的数据进行展示）
    //把这次请求封装成一个函数：当需要调用的时候调用即可
    getData() {
      this.$store.dispatch("getSearchInfo", this.searchParams);
    },
    //删除分类的名字
    removeCategoryName() {
      //给服务器的参数清空之后，还需要再发一次请求
      //带给服务器参数说明可有可无的：如果属性为空字符串还是会把相应的字段带给服务器
      //但是变为undefined就不会把字段带给服务器
      this.searchParams.category1Id = undefined;
      this.searchParams.category2Id = undefined;
      this.searchParams.category3Id = undefined;
      this.searchParams.categoryName = undefined;
      this.getData();
      //地址栏也需要修改:进行路由跳转（search页面更新，干掉参数，自己挑自己）
      //本意是删除query，如果路径中出现params不应该删除，路由跳转的时候应该带着
      if (this.$route.params) {
        this.$router.push({ name: "search", params: this.$route.params });
      }
    },
    //删除关键字
    removeKeyword() {
      //给服务器带的参数searchParams的keyword清空
      this.searchParams.keyword = undefined;
      this.getData();
      //通知兄弟组件Header清除关键字
      this.$bus.$emit("clear");
      //进行路由跳转
      if (this.$route.query) {
        this.$router.push({ name: "search", query: this.$route.query });
      }
    },
    //删除品牌信息
    removeTrademark() {
      this.searchParams.trademark = undefined;
      this.getData();
    },
    //删除商品属性
    removeAttr(index) {
      //整理参数
      this.searchParams.props.splice(index, 1);
      //再次发请求
      this.getData();
    },
    //自定义事件的回调
    trademarkInfo(trademark) {
      //整理品牌字段的参数 “ID:品牌名称”
      this.searchParams.trademark = `${trademark.tmId}:${trademark.tmName}`;
      this.getData();
    },
    //收集商品属性的自定义事件回调
    attrInfo(attrs, attrValue) {
      //整理好格式 示例["属性ID:属性值:属性名"]
      let props = `${attrs.attrId}:${attrValue}:${attrs.attrName}`;
      //数组去重,不然会无限返回
      if (this.searchParams.props.indexOf(props) == -1) {
        this.searchParams.props.push(props);
        //再次发请求
        this.getData();
      }
    },
    //排序操作
    changeOder(flag) {
      //flag形参：它是一个标记，代表用户点击的是综合（1）还是价格（2）【用户点击的时候传递进来的】
      //这里获取的是初始化的状态【需要通过初始状态去判断接下来要做什么】
      let originFlag = this.searchParams.order.split(":")[0];
      let originSort = this.searchParams.order.split(":")[1];
      //准备一个新的oder属性值
      let newOrder = "";
      //判断的是多次点击的是不是同一个按钮
      //如果用户点击的参数和初始化参数相等，则传入（初始化的状态）：（再做判断，如果初始化的是降序的话改成升序，否则降序）
      //不相等的话，则传入（flag）：（默认为desc）
      if (flag == originFlag) {
        newOrder = `${originFlag}:${originSort == "desc" ? "asc" : "desc"}`;
      } else {
        //点击的不是一个按钮
        newOrder = `${flag}:${"desc"}`;
      }
      //将新的order赋予searchParams
      this.searchParams.order = newOrder;
      this.getData();
    },
    //自定义事件回调---获取当前第几页
    getPageNo(pageNo){
      //整理带给服务器的参数
      this.searchParams.pageNo = pageNo
      this.getData()
      console.log(pageNo);
    }
  },
  //数据监听：监听组件实例身上的属性值的变化
  watch: {
    //监听属性
    $route: {
      handler() {
        //再次发送请求之前需要再整理参数发给服务器
        Object.assign(this.searchParams, this.$route.query, this.$route.params);
        //再次发起ajax请求
        this.getData();
        //每一次请求完毕，应该把相应的1、2、3级分类的id清空，准备接收下一次的1、2、3的ID
        //分类名字与关键字不用清理：因为每一次路由发生变化的时候，都会给予重新赋值
        this.searchParams.category1Id = undefined;
        this.searchParams.category2Id = undefined;
        this.searchParams.category3Id = undefined;
      },
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