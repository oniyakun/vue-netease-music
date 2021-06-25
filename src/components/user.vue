<template>
  <div class="user">
    <!-- 登录前 -->
    <div @click="onOpenModal" class="login-trigger" v-if="!isLogin">
      <i class="user-icon iconfont icon-yonghu" />
      <p class="user-name">未登录</p>
    </div>
    <!-- 登录后 -->
    <div @click="onLogout" class="logined-user" v-else>
      <img v-lazy="$utils.genImgUrl(user.avatarUrl, 80)" class="avatar" />
      <p class="user-name">{{ user.nickname }}</p>
    </div>

    <!-- 登录框 -->
    <el-dialog
      :modal="false"
      :visible.sync="visible"
      :width="$utils.toRem(320)"
    >
      <p slot="title">登录</p>
      <div class="login-body">
        <el-input
          class="input"
          placeholder="手机"
          v-model="phone"
          @input="change($event)"
        />
        <el-input
          class="input"
          placeholder="密码"
          v-model="passwd"
          show-password
          @input="change($event)"
        />
      </div>
      <div class="login-help">
          <p class="help">如账密无误但提示账号密码错误时再按一次登录即可</p>
      </div>
      <span class="dialog-footer" slot="footer">
        <el-button
          :loading="loading"
          @click="onLogin(phone, passwd)"
          class="login-btn"
          type="primary"
          >登 录</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script type="text/ecmascript-6">
var request = require('request');
var md5 = require('md5-node');
import storage from "good-storage"
import { UID_KEY, isDef } from "@/utils"
import { confirm } from "@/base/confirm"
import {
  mapActions as mapUserActions,
  mapState as mapUserState,
  mapGetters as mapUserGetters
} from "@/store/helper/user"
if (window.require) {
  var ipc = window.require('electron').ipcRenderer
}
export default {
  // 自动登录
  created() {
    const uid = storage.get(UID_KEY)
    if (isDef(uid)) {
      this.onAutoLogin(uid)
    }
  },
  data() {
    return {
      visible: false,
      loading: false,
      uid: ""
    }
  },
  methods: {
    onOpenModal() {
      this.visible = true
    },
    onCloseModal() {
      this.visible = false
    },
    async onLogin(phone, passwd) {
      var md5_passwd = md5(passwd)
      var options = {
        'method': 'GET',
        'url': 'https://netease-music-api.fe-mm.com/login/cellphone?phone=' + phone + '&md5_password=' + md5_passwd,
        'headers': {
          'Content-Type': 'application/json',
        }
      };
      await request(options, function (error, response) {
        if (error) throw new Error(error);
        var obj = JSON.parse(response.body);
        var uid = obj.account.id;
        storage.set('uid', uid)
      });
      this.loading = true
      let uid = storage.get('uid')
      const success = await this.login(uid).finally(() => {
        this.loading = false
      })
      if (success) {
        this.onCloseModal()
      }
      storage.remove('uid')
    },
    async onAutoLogin(uid) {
      this.loading = true
      const success = await this.login(uid).finally(() => {
        this.loading = false
      })
      if (success) {
        this.onCloseModal()
      }
    },
    onLogout() {
      confirm("确定要注销吗？", () => {
        this.logout()
      })
    },
    onClickLink() {
      if (window.require) {
        ipc.send('openLink');
      }
    },
    change(e) {
      this.$forceUpdate()
    },
    ...mapUserActions(["login", "logout"])
  },
  computed: {
    ...mapUserState(["user"]),
    ...mapUserGetters(["isLogin"])
  },
  components: {}
}
</script>

<style lang="scss" scoped>
.user {
  padding: 16px;
  padding-bottom: 0;
  margin-bottom: 12px;

  .login-trigger {
    display: flex;
    align-items: center;
    cursor: pointer;
  }
  .user-icon {
    font-size: 24px;
  }

  .user-name {
    margin-left: 8px;
  }

  .logout {
    transform: translateY(1px);
    margin-left: 8px;
    color: var(--font-color-shallow-grey);
  }

  .login-body {
    .input {
      margin-bottom: 16px;
    }

    .login-help {
      .help {
        margin-bottom: 4px;
      }
    }
  }

  .login-btn {
    width: 100%;
    padding: 8px 0;
  }

  .logined-user {
    display: flex;
    align-items: center;
    cursor: pointer;

    .avatar {
      @include round(40px);
    }
  }
}
</style>
