<template>
  <div>
    <v-layout row justify-center>
      <v-flex lg6 class="my-4">
        <v-card class="white ma-2" height="100%">
          <v-flex dark flat class="teal pa-1">
            <v-icon color="white">play_circle_filled</v-icon>
            <span class="title white--text px-2">Enter Password</span>
            <span class="caption mx-2 white--text">パスワードを入力してください。</span>
          </v-flex>
          <v-form class="ma-4" ref="form">
            <v-text-field
              class="mx-1"
              v-model="password"
              label="Password"
              required
              hint="はむとれ ama 専用パスワードを入力します。"
              :append-icon="e1 ? 'visibility' : 'visibility_off'"
              @click:append="() => (e1 = !e1)"
              :type="e1 ? 'password' : 'text'"
            ></v-text-field>

            <v-layout class="my-2">
              <v-btn color="success" type="submit" class="mr-4" @click.stop="validate()">確認</v-btn>
              <v-btn color="error" class="mr-4" @click="reset()">リセット</v-btn>
            </v-layout>
          </v-form>
        </v-card>
      </v-flex>
    </v-layout>

    <!-- スナックバー -->
    <v-snackbar class="ma-2" :top="true" v-model="snack" :timeout="timeout">
      <v-icon color="yellow" class="ma-2">warning</v-icon>
      {{ text }}
      <v-btn color="blue" text @click="snackbar = false">Close</v-btn>
    </v-snackbar>
  </div>
</template>

<script>
import CryptoJS from "crypto-js";

export default {
  name: "LoginComponent",
  props: ["passconfig"], //user_configの親から子へ受け渡し用。受け渡しの制約で"_"が使用できないため別の名前を指定している
  data() {
    return {
      password: "",
      snack: false,
      text: "エラー",
      timeout: 1000,
      e1: true,
      user_config: {}
    };
  },
  created(){
    this.user_config = this.passconfig; //親から渡ってきたデータの名前を変更(アンダーバーは認識しないのでpassconfigとして受け渡し)
  },
  methods: {
    reset() {
      this.password = "";
    },
    validate() {
      // 復号化
      try {
        var bytes = CryptoJS.AES.decrypt(
          this.user_config.encrypted_config.toString(),
          this.password
        );
        var decrypted_user_config = JSON.parse(
          bytes.toString(CryptoJS.enc.Utf8)
        );
        this.$emit('decrypted',decrypted_user_config);

      } catch {
        this.snack = !this.snack;
        this.text = "パスワードが違います。";
      }
    }
  }
};
</script>

<style>
</style>