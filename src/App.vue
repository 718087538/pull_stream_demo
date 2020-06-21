<template>
  <div id="app">
    <h1>看直播</h1>
    <div id="show"></div>
  </div>
</template>

<script>
// import AgoraRTC from "agora-rtc-sdk";
import AgoraRTC from "agora-rtc-sdk";
export default {
  name: "App",
  data() {
    return {
      a: "ccc",
      rtc: {
        client: null,
        joined: false,
        published: false,
        localStream: null,
        remoteStreams: [],
        params: {}
      },
      option: {
        appID: "1e93df4848bf40119a2276cf5b9a3a5c",
        channel: "1",
        uid: null,
        token:
          "0061e93df4848bf40119a2276cf5b9a3a5cIADaupOXOCtyNTV00H2JP0JSoiViGlmo4xlyPvw/D8fkPbfv3IMAAAAAEAB2N7Fhcm3wXgEAAQBxbfBe"
      }
    };
  },
  components: {},
  beforeMount() {
    console.log("beforeMount");
    // 1，创建客户端
    this.rtc.client = AgoraRTC.createClient({ mode: "live", codec: "vp8" });
    // 2，初始化客户端
    let that = this;
    this.rtc.client.init(
      this.option.appID,
      function() {
        console.log("客户端init成功");

        that.rtc.client.setClientRole("audience");

        // console.log("UID", that.option.uid);
        // 3，加入频道
        // Join a channel
        that.rtc.client.join(
          that.option.token ? that.option.token : null,
          that.option.channel,
          that.option.uid ? +that.option.uid : null,

          function(uid) {
            //已经onSuccess
            // alert();
            that.$message({
              message:
                "加入频道: " + that.option.channel + " success, uid: " + uid,
              type: "success"
            });
            that.rtc.params.uid = uid;

            that.rtc.client.on("stream-added", function(evt) {
              console.log("流变动");
              var remoteStream = evt.stream;
              var id = remoteStream.getId();
              if (id !== that.rtc.params.uid) {
                that.rtc.client.subscribe(remoteStream, function(err) {
                  console.log("stream subscribe failed", err);
                });
              }
              console.log("stream-added remote-uid: ", id);
            });

            that.rtc.client.on("stream-subscribed", function(evt) {
              var remoteStream = evt.stream;
              var id = remoteStream.getId();
              // Add a view for the remote stream.
              // addView(id);
              // Play the remote stream.
              remoteStream.play("show");
              console.log("流订阅ID: ", id);
            });
          },
          function(err) {
            console.error("client join failed", err);
          }
        );
      },
      err => {
        console.error(err);
      }
    );
  }
};
</script>

<style>
#show {
  width: 300px;
  height: 200px;
}
</style>
