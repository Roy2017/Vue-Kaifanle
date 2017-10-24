
<template>
    <div class="container">
        <myHeader/>
        <div class="form-group has-feedback search">
            <input type="text"
                   class="form-control"
                   v-model="kw"
                   placeholder="输入菜品或者原材料的名称" />
            <span class="form-control-feedback glyphicon glyphicon-search"></span>

        </div>

        <div>
            <div class="list-group">
                <a v-on:click="jump(dish.did)"
                   v-for="dish in dishList"
                   class="list-group-item">
                    <div class="media">
                        <div class="media-left media-middle">
                            <img class="media-object"
                                 :src="'./static/img/'+dish.img_sm"
                                 alt="" />
                        </div>

                        <div class="media-body">
                            <h3>{{dish.name}}</h3>
                            <p>{{dish.material}}</p>
                            <p>价格：{{dish.price}}</p>
                        </div>

                    </div>
                </a>
            </div>
        </div>
        <button v-if="hasMore&&noSearching"
                v-on:click="loadMore()"
                class="btn btn-success btn-block">
            加载更多
        </button>

        <p v-if="!hasMore&&noSearching">没有更多数据可以加载了</p>
        <myFooter/>
    </div>
</template>

<script>

import kflHeader from '@/components/Header'
import kflFooter from '@/components/Footer'
export default {
    name: 'main',
    components: {
        'myHeader': kflHeader,
        'myFooter': kflFooter
    },
    data() {
        return {
            oldDishList:[],
            dishList: [],
            kw: '',
            hasMore: true,
            noSearching:true
        }
    },
    methods: {
        jump: function (did) {
            this.$parent.jump('/detail/' + did);
        },
        loadMore: function () {
          this.$http.jsonp(
            'http://'+serverUrl+'/PHP/data/dish_getbypage.php', {params:{start:this.dishList.length}}
          )
          .then(function (response) {
              this.dishList = this.dishList.concat(response.data);
              if (response.data.length < 5) {
                  this.hasMore = false;
              }
          }, function (error) {
              console.log(error);
          })
        }
    }
    ,
    watch: {
        kw: function (newValue, oldValue) {
          //先保存当前的列表
          if(!oldValue){
            this.oldDishList = this.dishList;
            this.noSearching = false;
          }
          //kw为空后，重新出现列表
          if(!newValue){
            this.dishList = this.oldDishList;
            this.noSearching = true;
          }
          else{
          //发送请求
            this.$http
                .jsonp(
                'http://'+serverUrl+'/PHP/data/dish_getbykw.php',{params:{kw:this.kw},jsonp: 'callback' }
                )
                .then(function (response) {
                    console.log(this.kw,newValue,oldValue);
                    this.dishList = response.data;
                }, function (error) {
                    console.log(error);
                })
          }

        }
    },
    created() {
        console.log(serverUrl);
        this.loadMore();


    }
}
</script>

<style scoped>
.redColor {
    color: red
}
</style>
