<template>
  <div class="hello">
  <div  class="page-wrapper default-theme sidebar-bg bg1" :class="{ active : isActive, 'toggled' : acting}" v-if="isAdmin">
            
        <nav id="sidebar" class="sidebar-wrapper">

            <div class="sidebar-content">
                <!-- sidebar-brand  -->
                <div class=" sidebar-brand">
                    <a>Vue-Commerce</a>
                    <b-button variant="light" class="p-1 mt-0 float-right" @click="acting = !acting">
                      <i class="fas fa-times fa-md"></i>
                    </b-button>
                </div>
                <!-- sidebar-header  -->
                <div class="sidebar-item sidebar-header d-flex flex-nowrap">
                    <div class="user-pic">
                        <img class="img-responsive img-rounded" src="https://image.flaticon.com/icons/svg/236/236832.svg" alt="User picture">
                    </div>
                    <div class="user-info">
                        <span class="user-name">
                            <strong>dynamic username</strong>
                        </span>
                        <span class="user-role">{{email}}</span>
                        <span class="user-status">
                            <i class="fa fa-circle"></i>
                            <span class="mx-2">{{ $t('adminPage.menu.status') }}</span>
                        </span>
                    </div>
                </div>
                <!-- sidebar-search  -->
                <div class="sidebar-item sidebar-search">
                    <div>
                        <div class="input-group">
                            <input type="text" class="form-control search-menu" :placeholder="$t('adminPage.menu.search')">
                            <div class="input-group-append">
                                <span class="input-group-text">
                                    <i class="fa fa-search" aria-hidden="true"></i>
                                </span>
                            </div>
                        </div>
                    </div>
                </div>
                
                <!-- sidebar-menu  -->
                <div class=" sidebar-item sidebar-menu">
                    <ul>
                        <li class="header-menu">
                            <span>{{ $t('adminPage.menu.general') }}</span>
                        </li>
                        <!-- overview -->
                        <li class="sidebar" v-if="isAdmin">
                            <router-link :to="`/${$i18n.locale}/admin/overview`">
                                <i class="fa fa-tachometer-alt"></i>
                                <span class="menu-text mx-2">{{ $t('adminPage.menu.overview') }}</span>
                                <mdb-badge color="grey darken-3">{{ $t('alerts.graph-construction.title') }}</mdb-badge>
                            </router-link>
                        </li>
                        <!-- products -->
                        <li class="sidebar" v-if="isAdmin">
                            <router-link :to="`/${$i18n.locale}/admin/products`">
                                <i class="fa fa-shopping-cart"></i>
                                <span class="menu-text mx-2">{{ $t('adminPage.menu.products') }}</span>
                                <span class="badge badge-pill badge-primary">{{ products.length }}</span>
                            </router-link>
                        </li>
                        <!-- categories -->
                        <li class="sidebar" v-if="isAdmin">
                            <router-link :to="`/${$i18n.locale}/admin/categories`">
                                <i class="fa fa-shopping-cart"></i>
                                <span class="menu-text mx-2">categories</span>
                                <mdb-badge color="grey darken-3">{{ $t('alerts.graph-construction.title') }}</mdb-badge>
                            </router-link>
                        </li>
                        <!-- orders -->
                        <li class="sidebar" v-if="isAdmin">
                            <router-link :to="`/${$i18n.locale}/admin/orders`">
                                <i class="far fa-gem"></i>
                                <span class="menu-text mx-2">{{ $t('adminPage.menu.orders') }}</span>
                                <mdb-badge color="grey darken-3">{{ $t('alerts.graph-construction.title') }}</mdb-badge>
                            </router-link>
                        </li>
                        <!-- profile -->
                        <li class="sidebar-dropdown">
                            <router-link :to="`/${$i18n.locale}/admin/profile`">
                                <i class="far fa-user"></i>
                                <span class="menu-text mx-2">{{ $t('adminPage.menu.profile') }}</span>
                            </router-link>
                        </li>
                        <!-- users -->
                        <li class="sidebar-dropdown">
                            <router-link :to="`/${$i18n.locale}/admin/users`">
                                <i class="fas fa-users"></i>
                                <span class="menu-text mx-2" v-if="isAdmin">{{ $t('adminPage.menu.users') }}</span>
                                 <span class="badge badge-pill badge-primary">{{ users.length }}</span>
                            </router-link>
                        </li>
                        <li class="sidebar">
                            <a href="#" @click.prevent="logout">
                                <i class="fa fa-power-off"></i>
                                <span class="menu-text mx-2">{{ $t('adminPage.menu.logout') }}</span>
                            </a>
                        </li>
                    </ul>
                </div>
                <!-- sidebar-menu  -->
            </div>
        </nav>
        
        <!-- page-content  -->
        <main class="page-content pt-2">
          <div class="container-fluid">
            <div class="row">
              <div class="form-group col-md-12">
                <!-- toggler -->
                <transition name="fade">
                    <b-button class="btn btn-sm btn-dark position-fixed d-block float-left" 
                    v-if="!acting" @click="acting = !acting" style="left: 10px; opacity: 0.5; z-index: 1; top: 30px;">
                        <i class="fas fa-bars"></i>
                    </b-button>
                </transition>
            
            <!-- <transition name="fade"> -->
                <router-view></router-view>
            <!-- </transition> -->
              </div>
            </div>
          </div>
        </main>
        <!-- page-content" -->
    </div>
    <!-- page-wrapper -->
  </div>
</template>

<script>
// global {$, jQuery}
import { db, fbAuth } from '@/assets/js/firebase';
import { mdbBadge } from 'mdbvue';
import {
        mapGetters,
        mapState
    } from 'vuex';
export default {
  name: "admin",
  props: {
    msg: String
  },
  components: {
      mdbBadge,
  },
  data() {
      return {
          isActive: true,
          acting: true,
          name: null,
          email: null,
          isAdmin: null,
          products: [],
      }
  },
    methods: {
        logout() {
            fbAuth.auth().signOut()
                .then(() => {
                this.$router.replace('/');
                })
            .catch((response, err) => {
                console.log(err);
            });
        },
    },
    computed: 
            mapState({ users: state => state.users }),

    created() {
        fbAuth.auth().onAuthStateChanged(user => {
          if(user) {
              user.getIdTokenResult().then(idTokenResult => {
                  user.admin = idTokenResult.claims.admin;
                  this.isAdmin = user.admin
                  if (user.admin != true) {
                      this.$router.push('/')
                  } else {
                      var userInfo = fbAuth.auth().currentUser;
                      this.email = userInfo.email;
                  }
              });
            }
        });

        db.collection("products").onSnapshot(snapShot => {
            snapShot.forEach(doc => {
                this.products.push(doc.data())
            });
        });

        this.$store.dispatch('listAllUsers')
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.custom {
    left: 0;
}

</style>
