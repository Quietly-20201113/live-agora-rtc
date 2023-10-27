<template>
  <div class="left-side">
    <h3>Pleat check you camera / microphone!</h3>
    <div class="buttons">
      <button @click="turnOnCamera" :class="{ 'button-on': isVideoOn }">
        Turn {{ isVideoOn ? "off" : "on" }} camera
      </button>
      <button @click="turnOnMicrophone" :class="{ 'button-on': isAudioOn }">
        Turn {{ isAudioOn ? "off" : "on" }} Microphone
      </button>
    </div>
    <h3>Please input the channel name</h3>
    <input v-model="channel" />
    <div class="buttons">
      <button @click="joinChannel" :class="{ 'button-on': isJoined }">
        Join Channel
      </button>
      <button @click="publishVideo" :class="{ 'button-on': isVideoPubed }">
        Publish Video
      </button>
      <button @click="publishAudio" :class="{ 'button-on': isAudioPubed }">
        Publish Audio
      </button>
      <button @click="leaveChannel">Leave Channel</button>
    </div>
  </div>
  <div class="right-side">
    <video v-show="isVideoOn" id="camera-video"></video>
    <video v-show="isVideoSubed" id="remote-video"></video>
    <video id="video_remote"></video>
    <div v-if="isJoined && !isVideoSubed" class="waiting">
      You can shared channel {{ channel }} to others.....
    </div>
    <div id="player-con"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import type {
  IAgoraRTCClient,
  IAgoraRTCRemoteUser,
  ICameraVideoTrack,
  IMicrophoneAudioTrack,
  IRemoteAudioTrack,
} from "agora-rtc-sdk-ng";

import AgoraRTC from "agora-rtc-sdk-ng";


onMounted(() => {
  const player = new Aliplayer({
  "id": "player-con",
  "source": "//player.alicdn.com/video/aliyunmedia.mp4",
  "width": "100%",
  "height": "500px",
  "autoplay": true,
  "isLive": true,
  "rePlay": false,
  "playsinline": true,
  "preload": true,
  "controlBarVisibility": "hover",
  "useH5Prism": true
}, function (_player: any) {
    console.log("The player is created");
  }
);
})

let track: ICameraVideoTrack;
let audioTrack: IMicrophoneAudioTrack;
let client: IAgoraRTCClient;

let isJoined = ref(false);
let isAudioPubed = ref(false);
let isVideoPubed = ref(false);
let isAudioSubed = ref(false);
let isVideoSubed = ref(false);

let isVideoOn = ref(false);
let isAudioOn = ref(false);

async function turnOnCamera() {
  isVideoOn.value = !isVideoOn.value;

  if (track) {
    return track.setEnabled(!track.enabled);
  }
  track = await AgoraRTC.createCameraVideoTrack();
  track.play("camera-video");
}

async function turnOnMicrophone() {
  isAudioOn.value = !isAudioOn.value;

  if (audioTrack) {
    return audioTrack.setEnabled(!audioTrack.enabled);
  }

  audioTrack = await AgoraRTC.createMicrophoneAudioTrack();
  // audioTrack.play();
}

const channel = ref("");
// you can apply appid follow the guide https://www.agora.io/en/blog/how-to-get-started-with-agora/
const APP_ID = "46d5a4ea1ece4d13ac2695e4584dd8bd";

const TOKEN = '007eJxTYNhosGFC+IXHE/dNyTP3LUuoCdnSJ7SLPaVympzTLJc9Gk8UGEzMUkwTTVITDVOTU01SDI0Tk43MLE1TTUwtTFJSLJJSPPZbpzYEMjKUB79lYIRCEJ+dwdDI2MTUzJyBAQCo8x9A'

async function joinChannel() {
  if (!channel.value) {
    channel.value = "vue-room";
  }

  if (isJoined.value) {
    await leaveChannel();
  }

  if (!client) {
    client = AgoraRTC.createClient({
      mode: "rtc",
      codec: "vp8",
    });
    console.log('*****************');
    
    client.on("user-published", onUserPublish);
  }

  await client.join(APP_ID, channel.value, TOKEN, null);
  isJoined.value = true;
  
}

async function leaveChannel() {
  isJoined.value = false;
  isAudioPubed.value = false;
  isVideoPubed.value = false;

  client && (await client.leave());
}

async function onUserPublish(
  user: IAgoraRTCRemoteUser,
  mediaType: "video" | "audio"
) {
  console.log('加入 = ', user);
  
  if (mediaType === "video") {
    const remoteTrack = await client.subscribe(user, mediaType);
    console.log('直播流地址 = ',remoteTrack);

      // 获取视频轨道
      // const videoTracks = remoteMediaStream.;

    const _element = document.getElementById('player-con');
    
    const videoElements= _element?.getElementsByTagName('video');
    console.log('地址 = ',videoElements);
    // remoteTrack.play("remote-video");
    if(videoElements != undefined){
      remoteTrack.play(videoElements[0]);
    }
    isVideoSubed.value = true;
  }
  if (mediaType === "audio") {
    const remoteTrack = await client.subscribe(user, mediaType);
    remoteTrack.play();
    isAudioSubed.value = true;
  }
}

async function publishVideo() {
  if (!isVideoOn.value) {
    await turnOnCamera();
  }

  if (!isJoined.value) {
    await joinChannel();
  }
  await client.publish(track);
  isVideoPubed.value = true;
}

async function publishAudio() {
  if (!isAudioOn.value) {
    await turnOnMicrophone();
  }
  if (!isJoined.value) {
    await joinChannel();
  }

  await client.publish(audioTrack);
  isAudioPubed.value = true;
}
</script>

<style></style>
