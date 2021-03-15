<template>
   <v-app id="app">
     <!-- Navigation bar -->
     <TopNavBar></TopNavBar>
     <!-- Side navigation bar -->
     <SideNavBar></SideNavBar>
     <!-- Content -->
     <v-content>
       <router-view :key="$route.fullPath"></router-view>
     </v-content>
     <!-- Footer -->
     <Footer></Footer>
     <!-- Back to top -->
     <BackTop></BackTop>
     <!-- Search modal box -->
     <searchModel></searchModel>
     <!-- Login modal box -->
     <LoginModel></LoginModel>
     <!-- Register modal box -->
     <RegisterModel></RegisterModel>
     <!-- Forgot password modal box -->
     <ForgetModel></ForgetModel>
   </v-app>
</template>

<script>
import TopNavBar from "./components/layout/TopNavBar";
import SideNavBar from "./components/layout/SideNavBar";
import Footer from "./components/layout/Footer";
import BackTop from "./components/BackTop";
import searchModel from "./components/model/SearchModel";
import LoginModel from "./components/model/LoginModel";
import RegisterModel from "./components/model/RegisterModel";
import ForgetModel from "./components/model/ForgetModel";
export default {
  components: {
    TopNavBar,
    SideNavBar,
    Footer,
    BackTop,
    searchModel,
    LoginModel,
    RegisterModel,
    ForgetModel
  }
};
</script>
