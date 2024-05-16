<script setup lang="ts">
import { onMounted, ref } from 'vue'
import SuperVizRoom from "../../sdk/dist";
import { VideoConference } from "../../sdk/dist";
import { CanvasPin, Comments, Realtime } from "../../sdk/dist";

const DEVELOPER_KEY = import.meta.env.VITE_DEVELOPER_API_KEY;

const urlParams = new URLSearchParams(window.location.search);

const room: any = ref(null)
const userId = urlParams.get("userId") || "<USER-ID>";
const userName = urlParams.get("userName") || "<USER-NAME>";
const channels = ref<any>([]);

const config = {
  video: false,
  comments: false,
  realtime: true
}

const initializeSuperVizRoom = async () => {
  room.value = await SuperVizRoom(DEVELOPER_KEY, {
    roomId: "<ROOM-ID>",
    group: {
      id: "<GROUP-ID>",
      name: "<GROUP-NAME>",
    },
    participant: {
      id: userId,
      name: userName,
      avatar: {
        "imageUrl": "https://<PATH>",
        "model3DUrl": "https://<PATH>",
      }
    },
    environment: 'local'
  });
}

const initComments = async () => {
  if (!config.comments) return
  const pinAdapter = new CanvasPin("my-id");
  const comments = new Comments(pinAdapter);
  await room.value.addComponent(comments);
} 

const initVideo = async () => {
  if (!config.video) return
  const video = new VideoConference({
    camsOff: false,
    chatOff: false,
    defaultAvatars: true,
    defaultToolbar: true,
    devices: {
      audioInput: true,
      audioOutput: true,
      videoInput: false
    },
    enableFollow: false,
    enableGather: false,
    enableGoTo: false,
    language: "en-US",
    locales: [ ],
    screenshareOff: false,
    skipMeetingSettings: false,
    participantType: "host",
    transcriptOff: false,
  });
  await room.value.addComponent(video);
}

const initRealtime = async () => {
  if (!config.realtime) return

  const realtime = new Realtime();
  await room.value.addComponent(realtime);
  await new Promise((resolve) => setTimeout(resolve, 5000));

  channels.value = [
    realtime.connect("channel-1"),
    realtime.connect("channel-2"),
    realtime.connect("channel-3"),
  ]

  console.log("Channels: ", channels.value)

  channels.value[0].subscribe("my-event-0", (data: any) => {
    speakEvent(data, 0);
  });

  channels.value[1].subscribe("my-event-1", (data: any) => {
    speakEvent(data, 1);
  });

  channels.value[2].subscribe("my-event-2", (data: any) => {
    speakEvent(data, 2);
  });
}

const speakEvent = (data: any, channelId: number) => {
  console.log(`Hey, we received one event on channel ${channelId}: `, data);
}

const emitEvent = (channelId: number) => {
  channels.value[channelId].publish(`my-event-${channelId}`, { message: "Hey! This message is coming from channel: " + channelId + "!" });
}

onMounted(async () => {
  await initializeSuperVizRoom();
  await initVideo();
  await initComments();
  await initRealtime();
})

</script>

<template>
  <div>
    <button @click="emitEvent(0)">Emit Event to channel 1</button>
    <button @click="emitEvent(1)">Emit Event to channel 2</button>
    <button @click="emitEvent(2)">Emit Event to channel 3</button>
   <!-- <canvas id="my-id" width="540" height="540"></canvas>  -->
  </div>
</template>
