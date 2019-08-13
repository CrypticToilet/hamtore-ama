<template>
  <v-dialog v-model="dialog" max-width="1000">
    <v-btn slot="activator" color="green lighten-4"><v-icon class="mr-2">help</v-icon>公開設定の方法</v-btn>
    <v-card class="pa-4">
      <v-card-title>
        <h2>テーブルの公開設定</h2>
        <v-btn color="success" @click="dialog = false" class="mx-4">閉じる</v-btn>
      </v-card-title>

      <h3>1. "File" メニューより "Share..." を選択</h3>
      <img src="img/1.jpg" class="grey pa-1" />

      <h3 class="mt-4">2. "共有設定" が開くので "変更..." を選択</h3>
      <img src="img/2.jpg" class="grey pa-1" />

      <h3 class="mt-4">3. "リンクの共有" が開くので "オン - リンクを知っている全員" を選択</h3>
      <p>保存をクリックします</p>
      <img src="img/3.jpg" class="grey pa-1" />

      <h3 class="mt-4">4. "共有設定" が開きリンクが表示されたら "完了" をクリックして終了です</h3>
      <p>リンクをコピーする必要はありません</p>
      <img src="img/4.jpg" class="grey pa-1" />
      <img src="img/5.jpg" class="grey pa-1" />

      <h3 class="red--text">※URLは他人に教えないように注意してください。この設定によりURLが漏洩するとTableが外部から参照される可能性があることをご理解ください。</h3>
      <p>第三者がこのURLにアクセスした場合、更新や削除ができないことを確認しております(参照のみ)</p>

      <div class="my-2">
        <v-btn color="success" @click="dialog = false">閉じる</v-btn>
      </div>
    </v-card>
  </v-dialog>
</template>
<script>
export default {
  name: "HowToPermit",
  data() {
    return {
      dialog: false
    };
  }
};
</script>

