<script setup lang="ts">
import { onMounted, ref } from 'vue'
import SuperVizRoom from "../../sdk/dist";
import { VideoConference } from "../../sdk/dist";
import { CanvasPin, Comments } from "../../sdk/dist";

const DEVELOPER_KEY = import.meta.env.VITE_DEVELOPER_API_KEY;

const room: any = ref(null)

const initializeSuperVizRoom = async () => {
  room.value = await SuperVizRoom(DEVELOPER_KEY, {
    roomId: "<ROOM-ID>",
    group: {
      id: "<GROUP-ID>",
      name: "<GROUP-NAME>",
    },
    participant: {
      id: "<USER-ID>",
      name: "Vitor",
      avatar: {
        "imageUrl": "https://<PATH>",
        "model3DUrl": "https://<PATH>",
      }
    },
    environment: 'local'
  });
}

const initComments = async () => {
  const pinAdapter = new CanvasPin("my-id");
  const comments = new Comments(pinAdapter);
  await room.value.addComponent(comments);
} 

const initVideo = async () => {
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

onMounted(async () => {
  await initializeSuperVizRoom();
  await initVideo();
  await initComments();
})

</script>

<template>
  <div>
   <canvas id="my-id" width="540" height="540"></canvas> 
  </div>
</template>
