<template>
  <div>
    <div class="user_icons hidden-sm-and-down">
      <el-row type="flex" justify="center">
        <el-col :xs="24" :sm="7" :md="7" :lg="7" :xl="7">
          <router-link to="/favorites" exact>
            <div>
              <span v-if="this.$store.getters.user &&
                          this.$store.getters.user.favorites &&
              Object.keys(this.$store.getters.user.favorites).length">
                <img v-if="this.$route.path === '/favorites'"
                     src="@/assets/icons/user/favorite_full_white.svg" alt=""
                     class="favorite_icon"
                     height="40px" width="40px">
                <img v-else-if="color === 'red'"
                     src="@/assets/icons/user/favorite_full.svg"
                     alt=""
                     height="40px">
                <img v-else-if="color === 'black'"
                     src="@/assets/icons/user/favorite_full_black.svg"
                     alt=""
                     height="40px">
              </span>
              <span v-else>
                <img v-if="this.$route.path === '/favorites'"
                     src="@/assets/icons/user/favorite_white.svg" alt=""
                     class="favorite_icon"
                     height="40px" width="40px">
                <img v-else-if="color === 'red'"
                     src="@/assets/icons/user/favorite.svg"
                     alt=""
                     height="40px">
                <img v-else-if="color === 'black'"
                     src="@/assets/icons/user/favorite_black.svg"
                     alt=""
                     height="40px">
              </span>
            </div>
          </router-link>
        </el-col>
        <el-col :xs="24" :sm="7" :md="7" :lg="7" :xl="7">
          <router-link to="/account" exact>
            <div>
              <img v-if="this.$route.path === '/account'"
                   src="@/assets/icons/user/user_white.svg" alt=""
                   class="user_icon"
                   height="41px">
              <img v-else-if="color === 'red'"
                   src="@/assets/icons/user/user_red.svg" alt=""
                   class="user_icon"
                   height="41px">
              <img v-else
                   src="@/assets/icons/user/user_black.svg"
                   alt=""
                   height="41px">
            </div>
          </router-link>
        </el-col>
        <el-col :xs="24" :sm="7" :md="7" :lg="7" :xl="7">
          <router-link to="/cart" exact>
            <div>
              <img v-if="this.$route.path === '/cart'"
                   src="@/assets/icons/user/bag_white.svg"
                   alt=""
                   height="38px">
              <img v-else-if="color === 'red'"
                   src="@/assets/icons/user/bag.svg"
                   alt=""
                   height="38px">
              <img v-else
                   src="@/assets/icons/user/bag_black.svg"
                   alt=""
                   height="38px">
            </div>
          </router-link>
        </el-col>
        <el-col
          v-if="this.$store.getters.user.cart"
          :xs="24" :sm="3" :md="3" :lg="3" :xl="3" align="left">
          <transition name="bounce">
            <cart-product-count :color="color"/>
          </transition>
        </el-col>
      </el-row>
    </div>
    <!--Mobile version-->
  </div>
</template>

<script>
import CartProductCount from './CartProductCount'

export default {
  name: 'UserIcons',
  components: {CartProductCount},
  props: ['color']
}
</script>

<style scoped lang="scss">
  .user_icons {
    position: absolute;
    top: 40px;
    right: 40px;
    width: 200px;
    z-index: 2;
  }

  .mobile_user_icons {
    margin-top: 20px;
    margin-bottom: 30px;
  }

  img:hover {
    filter: drop-shadow(2px 2px 10px #888);
  }

</style>
