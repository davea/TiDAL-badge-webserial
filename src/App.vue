<template>
  <div class="flexible-content">
    <mdb-modal size="lg" style="cursor: pointer; z-index: 9999;" v-show="false && !is_connected" @click.native="connect()">
      <mdb-modal-header>
        <mdb-modal-title>Let's get started</mdb-modal-title>
      </mdb-modal-header>
      <mdb-modal-body v-if="has_serial">Click anywhere to connect to your badge via USB</mdb-modal-body>
      <mdb-modal-body v-else>It appears your browser does not support WebSerial. Make sure to use Chrome (at least v89).</mdb-modal-body>
    </mdb-modal>
    <!-- Sidebar -->
    <div class="sidebar-fixed position-fixed">
      <a class="logo-wrapper"><img alt="" class="img-fluid" src="./assets/EMF-W-coral-LogoText-2022-colour.png"/></a>
      <mdb-list-group class="list-group-flush">
          <mdb-list-group-item
            :action="true"
            @click.native="connectClicked"
            :disabled="! has_serial"
            >
            <mdb-icon :icon="!is_connected ? 'phone' : 'phone-slash'" class="mr-3" />{{ !is_connected ? 'Connect' : 'Disconnect' }}
          </mdb-list-group-item>
          <mdb-list-group-item
            :action="true"
            :class="activeItem === 1 && 'active'"
            @click.native="$router.push('/apps')"
            :disabled="!is_connected"
            ><mdb-icon icon="table" class="mr-3" />Apps</mdb-list-group-item>
          <mdb-list-group-item
            :action="true"
            :class="activeItem === 2 && 'active'"
            @click.native="$router.push('/programming')"
            :disabled="!is_connected"
            ><mdb-icon icon="i-cursor" class="mr-3" style="width:16px;text-align:center" />Programming & Files</mdb-list-group-item>
          <mdb-list-group-item
            :action="true"
            :class="activeItem === 3 && 'active'"
            @click.native="$router.push('/update')"
            :disabled="!is_connected"
          ><mdb-icon icon="bolt" class="mr-3" style="width:16px;text-align:center" />Update</mdb-list-group-item>
          <mdb-list-group-item
            :action="true"
            :class="activeItem === 4 && 'active'"
            @click.native="$router.push('/settings')"
            :disabled="!is_connected"
            ><mdb-icon icon="cog" class="mr-3" />Settings</mdb-list-group-item>
      </mdb-list-group>
    </div>
    <!-- notifications -->
    <div class="container">
        <div class="placement">
          <mdb-toast-notification v-for="message in messages" :key="message.id" :title="message.title" :message="message.message" :icon="message.icon" :iconColor="message.color" :show="show"/>
        </div>
      </div>
    <!-- /Sidebar  -->
    <main class="d-flex flex-column min-vh-100">
      <div class="flex-grow-1 p-5">
        <router-view v-on:genNotification="onNotification">
        </router-view>
      </div>
      <ftr color="primary-color-dark" class="text-center font-small darken-2">
        <div class="pb-4 pt-4">
          <a href="https://twitter.com/emfcamp"><mdb-icon fab icon="twitter" class="mr-3"/></a>
          <!-- <a href="https://www.youtube.com/watch?v=G0mEW8f2vac"><mdb-icon fab icon="youtube" class="mr-3"/></a> -->
          <a href="https://github.com/emfcamp"><mdb-icon fab icon="github" class="mr-3"/></a>
        </div>
      </ftr>
    </main>
  </div>
</template>

<script>
import {
  mdbModal,
  mdbModalHeader,
  mdbModalTitle,
  mdbModalBody,
  mdbIcon,
  mdbListGroup,
  mdbListGroupItem,
  mdbFooter,
  mdbToastNotification,
  waves
} from "mdbvue";

import {device, connect, disconnect} from './badgecomm';

let component = undefined;
setInterval(() => {
  component.is_connected = device !== undefined && device.opened;
}, 1000);

setInterval(() => {
  let delete_list = [];
  for(let i=0; i < component.messages.length; i++) {
    component.messages[i].lifetime--;
    if(component.messages[i].lifetime <= 0) {
      delete_list.unshift(i);
    }
  }
  for(let i=0; i < delete_list.length; i++) {
    component.messages.splice(delete_list[i], 1);
  }
}, 100);

export default {
  name: "AdminTemplate",
  components: {
    mdbModal,
    mdbModalHeader,
    mdbModalTitle,
    mdbModalBody,
    mdbListGroup,
    mdbListGroupItem,
    mdbIcon,
    mdbToastNotification,
    ftr: mdbFooter
  },
  methods: {
    connectClicked() {
      if (! component.is_connected) {
        connect();
      } else {
        disconnect();
        if (component.$route.name != 'Welcome') {
          component.$router.push('/');
        }
      }
    },
    connect:connect,
    disconnect:disconnect,
    onNotification: (message, title='', icon='info', color='elegant', lifetime=30) => {
      let newmessage = {id: component.message_id, title:title, message:message, icon:icon, color:color, lifetime:lifetime};
      component.message_id++;
      component.messages.push(newmessage);
    },
  },
  data() {
    return {
      show: true,
      message_id: 1,
      messages: [],
      is_connected: device !== undefined && device.opened
    };
  },
  computed:{
    activeItem() {
      return component.$route.meta.page;
    }
  },
  beforeMount() {
    component = this;
    this.has_serial = !!navigator.serial;
  },
  mixins: [waves]
};
</script>


<style>
@import url("https://fonts.googleapis.com/css?family=Roboto:300,400,500,700&display=swap");
.navbar-light .navbar-brand {
  margin-left: 15px;
  color: #2196f3 !important;
  font-weight: bolder;
}
.placement {
  position: fixed;
  right: 10px;
  top: 10px;
  z-index:8000;
}
</style>

<style scoped>
main {
  background-color: #3c3c3c;
}

.flexible-content {
  transition: padding-left 0.3s;
  padding-left: 270px;
  height: 100%;
}

.flexible-navbar {
  transition: padding-left 0.5s;
  padding-left: 270px;
}

.sidebar-fixed {
  left: 0;
  top: 0;
  height: 100vh;
  width: 270px;
  box-shadow: 0 2px 5px 0 rgba(0, 0, 0, 0.16), 0 2px 10px 0 rgba(0, 0, 0, 0.12);
  z-index: 1050;
  background-color: #171123;
  padding: 1.5rem;
  padding-top: 0;
}

.sidebar-fixed .logo-wrapper img {
  width: 100%;
  padding-top: 0.5rem;
  padding-bottom: 0.5rem;
}

.sidebar-fixed .list-group-item {
  display: block !important;
  transition: background-color 0.3s;
}

.sidebar-fixed .list-group .active {
  box-shadow: 0 2px 5px 0 rgba(0, 0, 0, 0.16), 0 2px 10px 0 rgba(0, 0, 0, 0.12);
  border-radius: 5px;
}
</style>
