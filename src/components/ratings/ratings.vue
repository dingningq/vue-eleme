<template>
  <div class="ratings-wrap" ref="scroll_hook">
    <div>
      <header class="head">
        <div class="left">
          <div class="score">{{ seller.score }}</div>
          <div class="txt">综合评分</div>
          <div class="than">高于周边商家{{ seller.rankRate }}%</div>
        </div>
        <div class="right">
          <div class="item">
            <span class="txt">服务态度</span>
            <stars class="star" :sizes="36" :scores="seller.serviceScore" v-if="seller.serviceScore"></stars>
            <span class="score">{{ seller.serviceScore }}</span>
          </div>
          <div class="item">
            <span class="txt">商品评分</span>
            <stars class="star" :sizes="36" :scores="seller.foodScore" v-if="seller.foodScore"></stars>
            <span class="score">{{ seller.foodScore }}</span>
          </div>
          <div class="item">
            <span class="txt">送达时间</span>
            <span class="time">{{ seller.deliveryTime }}分钟</span>
          </div>
        </div>
      </header>
      <splits></splits>
      <div class="ratings-box">
        <feeds @onlyHave="toggle" :txts="txtArr"></feeds>
        <ul class="ratings-list">
          <li class="item bd" v-for="(item, index) in newRatings">
            <img class="avatar" :src="item.avatar" width="28" height="28" alt="">
            <div class="cont">
              <div class="info">
                <div class="user">
                  <span class="name">{{ item.username }}</span>
                  <span class="time">{{ item.rateTime | formatDate }}</span>
                </div>
                <div class="delivery">
                  <stars :scores="item.score"></stars>
                  <span class="time">{{ item.deliveryTime }}分钟送达</span>
                </div>
              </div>
              <p class="feed">{{ item.text }}</p>
              <div class="kinds">
                <i class="icon" :class="icon[item.rateType]"></i>
                <span class="txt" v-for="items in item.recommend">{{ items }}</span>
              </div>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
  import stars from '../../components/stars/stars.vue';
  import splits from '../../components/splits/splits.vue';
  import feeds from '../../components/feeds/feeds.vue';
  import {formatDate} from '../../common/js/util';
  import BScroll from 'better-scroll';

  export default {
    data() {
      return {
        seller: {},
        ratings: [],
        icon: ['icon-thumb_up', 'icon-thumb_down'],
        curIndex: 0,
        flag: false
      };
    },
    computed: {
      newRatings() {
        let arr = this.ratings.filter((item) => {
          if (!this.curIndex) {
            return item;
          } else {
            return item.rateType === this.curIndex - 1;
          }
        });
        let temp = arr.filter((item) => {
          if (!this.flag) {
            return item;
          } else {
            return item.text.length > 0;
          }
        });
        return temp;
      },
      txtArr() {
        let all = this.ratings.length;
        let good = this.ratings.filter((item) => {
          return item.rateType === 0;
        }).length;
        let bad = this.ratings.filter((item) => {
          return item.rateType === 1;
        }).length;
        return [
          {title: '全部', num: all},
          {title: '满意', num: good},
          {title: '不满意', num: bad}
        ];
      }
    },
    mounted() {
      this.initData();
    },
    methods: {
      initData() {
        this.$http.get('api/seller')
          .then((res) => {
            res = res.data;
            if (res.errno === 0) {
              this.seller = res.data;
            }
          })
          .catch((err) => {
            console.log(err);
          });
        this.$http.get('api/ratings')
          .then((res) => {
            res = res.data;
            if (res.errno === 0) {
              this.ratings = res.data;
              this.initScroll();
            }
          })
          .catch((err) => {
            console.log(err);
          });
      },
      toggle(index, bool) {
        this.curIndex = index;
        this.flag = bool;
        this.initScroll();
      },
      initScroll() {
        this.$nextTick(() => {
          if (!this.scroll) {
            this.scroll = new BScroll(this.$refs.scroll_hook, {
              click: true,
              probeType: 3
            });
          } else {
            this.scroll.refresh();
          }
        });
      }
    },
    filters: {
      formatDate
    },
    components: {
      stars,
      splits,
      feeds
    }
  };
</script>

<style scoped lang="scss">
  .ratings-wrap {
    position: fixed;
    left: 0;
    top: 175px;
    right: 0;
    bottom: 48px;
    overflow: hidden;
    .head {
      display: flex;
      height: 106px;
      padding: 18px 0;
      box-sizing: border-box;
      .left {
        flex: 0 0 120px;
        width: 120px;
        text-align: center;
        .score {
          line-height: 1.2;
          font-size: 24px;
          color: rgb(255, 153, 0);
        }
        .txt {
          margin-top: 6px;
          font-size: 12px;
          color: rgb(7, 17, 27);
        }
        .than {
          margin-top: 8px;
          font-size: 10px;
          color: rgb(147, 153, 159);
        }
      }
      .right {
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        padding: 0 5px;
        border-left: 1px solid rgba(147, 153, 159, .1);
        .item {
          display: flex;
          align-items: center;
          line-height: 1.5;
          font-size: 12px;
          .txt {
            color: rgb(7, 17, 27);
          }
          .star {
            margin-left: 3px;
          }
          .score {
            margin-left: 3px;
            color: rgb(255, 153, 0);
          }
          .time {
            margin-left: 3px;
            color: rgb(147, 153, 159);
          }
        }
      }
      @media all and (min-width: 375px) {
        .left {
          flex: 0 0 135px;
          width: 135px;
        }
        .right {
          padding: 0 24px;
          .item {
            .star, .score, .time {
              margin-left: 6px;
            }
          }
        }
      }
    }
    .ratings-box {
      padding-top: 6px;
      .ratings-list {
        padding: 0 18px;
        border-top: 1px solid rgba(147, 153, 159, .2);
        .item {
          display: flex;
          padding: 18px 0;
          &:last-child:after {
            display: none;
          }
          .avatar {
            flex: 0 0 28px;
            width: 28px;
            border-radius: 50%;
          }
          .cont {
            flex: 1;
            margin-left: 12px;
            .info {
              line-height: 1.2;
              font-size: 10px;
              .time {
                color: rgb(147, 153, 159);
              }
              .user {
                display: flex;
                justify-content: space-between;
                .name {
                  color: rgb(7, 17, 27);
                }
              }
              .delivery {
                display: flex;
                align-items: center;
                .time {
                  margin-left: 6px;
                }
              }
            }
            .feed {
              line-height: 1.5;
              text-align: justify;
              margin-top: 6px;
              font-size: 12px;
              color: rgb(7, 17, 27);
            }
            .kinds {
              display: flex;
              flex-wrap: wrap;
              font-size: 0;
              .icon {
                line-height: 16px;
                margin: 8px 8px 0 0;
                font-size: 12px;
                &.icon-thumb_up {
                  color: rgb(0, 160, 220);
                }
                &.icon-thumb_down {
                  color: rgb(183, 187, 191);
                }
              }
              .txt {
                max-width: 62px;
                line-height: 16px;
                margin: 8px 8px 0 0;
                padding: 0 6px;
                white-space: nowrap;
                overflow: hidden;
                text-overflow: ellipsis;
                font-size: 9px;
                color: rgb(147, 153, 159);
                border: 1px solid rgba(7, 17, 27, .1);
                border-radius: 1px;
                box-sizing: border-box;
              }
            }
          }
        }
      }
    }
  }
</style>
