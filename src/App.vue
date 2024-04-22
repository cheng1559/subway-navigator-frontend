<template>
  <div class="flex flex-col h-screen w-screen">
    <div class="flex items-center justify-center bg-[#2148C0] dark:bg-gray-800 h-32 min-h-32 transition-colors">

      <h1 class="text-lg font-bold text-white"> 出发车站： </h1>
      <select class="select select-bordered w-36 max-w-xs mr-5" v-model="station_from" @change="query">
        <option disabled selected>请选择出发车站</option>
        <option v-for="(item, index) in data" :key="index">{{item}}</option>
      </select>

      <h1 class="text-lg font-bold text-white"> 到达车站： </h1>
      <select class="select select-bordered w-36 max-w-xs mr-5" v-model="station_to" @change="query">
        <option disabled selected>请选择到达车站</option>
        <option v-for="(item, index) in data" :key="index">{{item}}</option>
      </select>
      
      <div class="flex flex-col mr-5">
        <label class="label cursor-pointer">
          <span class="text-md font-medium text-white mr-2">最短时间</span> 
          <input type="radio" name="radio-10" class="radio " value='time' v-model="query_type" @change="query" checked />
        </label>
        <label class="label cursor-pointer">
          <span class="text-md font-medium text-white mr-2">最少换乘</span> 
          <input type="radio" name="radio-10" class="radio 0" value='transfer' v-model="query_type" @change="query" />
        </label>
      </div>

      <button class="btn text-lg mr-5" @click="invert">反向</button>
      <button class="btn text-lg mr-5" @click="lineDataInit(), add_line.showModal()">添加线路</button>
      <button class="btn text-lg mr-5" @click="remove_line.showModal(), fetchLine()">删除线路</button>
      <button class="btn text-lg mr-5" @click="confirm.showModal()">重新载入</button>

      <label class="swap swap-rotate hover:bg-gray-100 dark:hover:bg-gray-800 text-white dark:hover:text-gray-200 hover:text-gray-700 transition-all w-10 h-10 rounded-full hover:scale-110 absolute right-0 mr-5">
        <input type="checkbox" class="theme-controller" value="synthwave" :checked="isDark" @change="toggleDark"/>
        <svg class="swap-off fill-current w-7 h-7" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path d="M5.64,17l-.71.71a1,1,0,0,0,0,1.41,1,1,0,0,0,1.41,0l.71-.71A1,1,0,0,0,5.64,17ZM5,12a1,1,0,0,0-1-1H3a1,1,0,0,0,0,2H4A1,1,0,0,0,5,12Zm7-7a1,1,0,0,0,1-1V3a1,1,0,0,0-2,0V4A1,1,0,0,0,12,5ZM5.64,7.05a1,1,0,0,0,.7.29,1,1,0,0,0,.71-.29,1,1,0,0,0,0-1.41l-.71-.71A1,1,0,0,0,4.93,6.34Zm12,.29a1,1,0,0,0,.7-.29l.71-.71a1,1,0,1,0-1.41-1.41L17,5.64a1,1,0,0,0,0,1.41A1,1,0,0,0,17.66,7.34ZM21,11H20a1,1,0,0,0,0,2h1a1,1,0,0,0,0-2Zm-9,8a1,1,0,0,0-1,1v1a1,1,0,0,0,2,0V20A1,1,0,0,0,12,19ZM18.36,17A1,1,0,0,0,17,18.36l.71.71a1,1,0,0,0,1.41,0,1,1,0,0,0,0-1.41ZM12,6.5A5.5,5.5,0,1,0,17.5,12,5.51,5.51,0,0,0,12,6.5Zm0,9A3.5,3.5,0,1,1,15.5,12,3.5,3.5,0,0,1,12,15.5Z"/></svg>
        <svg class="swap-on fill-current w-7 h-7" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path d="M21.64,13a1,1,0,0,0-1.05-.14,8.05,8.05,0,0,1-3.37.73A8.15,8.15,0,0,1,9.08,5.49a8.59,8.59,0,0,1,.25-2A1,1,0,0,0,8,2.36,10.14,10.14,0,1,0,22,14.05,1,1,0,0,0,21.64,13Zm-9.5,6.69A8.14,8.14,0,0,1,7.08,5.22v.27A10.15,10.15,0,0,0,17.22,15.63a9.79,9.79,0,0,0,2.1-.22A8.11,8.11,0,0,1,12.14,19.73Z"/></svg>
      </label>
    </div>

    <div class="flex-1 flex overflow-y-hidden">
      <div class="p-5" @dblclick="my_modal.showModal()">
        <img src="@/assets/image.png" class="h-full"/>
      </div>
      
      <div class="flex-1 p-5 flex flex-col">
        <h1 class="font-medium text-xl" v-if="pathData.length > 1"> 用时 {{ time }}，途径 {{ pass }} 站，换乘 {{ transfer }} 次</h1>
        <ul class="timeline timeline-vertical overflow-y-auto w-full max-w-full">
          <li v-for="(item, index) in pathData" :key="index">
            <hr v-if="index != 0"/>
            <div class="timeline-start" v-if="!item.isTransfer">{{formatTime(item.time)}}</div>
            <div class="timeline-middle">
              <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" class="w-5 h-5"><path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.857-9.809a.75.75 0 00-1.214-.882l-3.483 4.79-1.88-1.88a.75.75 0 10-1.06 1.061l2.5 2.5a.75.75 0 001.137-.089l4-5.5z" clip-rule="evenodd" /></svg>
            </div>
            <div class="timeline-end timeline-box"
            :class="!item.isTransfer ? 'bg-primary text-primary-content' : 'bg-warning text-warning-content'">{{item.name}}</div>
            <hr v-if="index != pathData.length - 1"/>
          </li>
        </ul>
      </div>
    </div>

    <dialog ref="my_modal" class="modal">
      <div class="modal-box max-w-full overflow-auto">
        <form method="dialog">
          <button class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2">✕</button>
        </form>
        <img src="@/assets/image.png" @dblclick="my_modal.showModal()"/>
      </div>
      <form method="dialog" class="modal-backdrop">
        <button>close</button>
      </form>
    </dialog>

    <dialog ref="add_line" class="modal">
      <div class="modal-box w-[410px]">
        <form method="dialog">
          <button class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2">✕</button>
        </form>
        <div class="text-2xl font-bold mb-5"> 线路信息： </div>
        <form class="space-y-1.5" @submit.prevent="addLine">
          <div class="grid grid-cols-[3fr_2fr] gap-3 mb-3">
            <div>
              <div class="label-text text-lg font-bold mb-2"> 线路名称： </div>
              <input type="text" class="input input-bordered w-full" v-model="lineName" placeholder="请输入线路名称" required/>
            </div>
            <div>
              <div class="label-text text-lg mb-2 font-bold"> 时速 (km/h)： </div>
              <input type="number" min=10 max=500 class="input input-bordered w-full" v-model="lineSpeed" placeholder="80" required />
            </div>
          </div>

          <div class="label-text text-lg mb-3 font-bold"> 站点数量： </div>
          <div class="flex">
            <span class="countdown font-mono text-xl mx-3">
              <span :style="{'--value': stationNum}"></span>
            </span>
            <div class="flex-1">
              <input type="range" min="2" max="30" v-model="stationNum" class="range range-sm" step="1" />
              <div class="w-full flex justify-between text-xs px-2 -mt-1">
                <span v-for="index in 29" :key="index">|</span>
              </div>
            </div>
          </div>
          
          <div class="label-text text-lg font-bold"> 站点信息： </div>
          <div class="overflow-y-auto max-h-72">
            <div class="flex flex-col">
              <div id="stations" v-for="index in parseInt(stationNum)" :key="index">
                <div class="mb-1 font-bold "> 第 {{ index }} 站：</div>
                <div class="grid grid-cols-[5fr_4fr] gap-4 mb-2 px-3">
                  <div class="w-full">
                    <label for="name" class="block mb-1 label-text">站点名称：</label>
                    <input id="name" class="input input-bordered w-full" :placeholder="'第' + index + '站'"
                    v-model="stationsInfo[index - 1].name" required />
                  </div>
                  <div class="w-full">
                    <label for="distance" class="block mb-1 label-text">到上一站的距离 (m)：</label>
                    <input type="number" id="distance" min=1 max=999999 class="input input-bordered w-full" 
                    v-model="stationsInfo[index - 1].distance"
                    :placeholder="!isLoop && index == 1 ? 0 : 1000"
                    :disabled="!isLoop && index == 1" required />
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="flex items-center">
            <h1 class="label-text text-lg mr-1 font-bold"> 是否为环线： </h1>
            <input class="checkbox" type="checkbox" v-model="isLoop" @change="stationsInfo[0].distance = isLoop ? null : 0"/>
          </div>

          <div class="text-error" v-if="errorInfo != ''"> 错误：{{ errorInfo }} </div>

          <div class="flex justify-end">
            <button class="btn text-lg mr-3">添加</button>
            <button class="btn text-lg mr-3" @click="fillInTestData" type="button">填写测试数据</button>
            <form method="dialog">
              <button class="btn text-lg">取消</button>
            </form>
          </div>
        </form>
        
      </div>
      <form method="dialog" class="modal-backdrop">
        <button>close</button>
      </form>
    </dialog>

    <dialog ref="remove_line" class="modal">
      <div class="modal-box w-96">
        <form method="dialog">
          <button class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2">✕</button>
        </form>
        <h3 class="text-2xl font-bold mb-5"> 要删除的线路： </h3>
        <div class="w-full flex">
          <select class="select select-bordered max-w-lg flex-1 mr-5" v-model="removedLine" @change="query">
            <option disabled selected>请选择要删除的线路</option>
            <option v-for="(item, index) in lineData" :key="index">{{item}}</option>
          </select>
          <form method="dialog">
            <button class="btn btn-primary text-lg" :class="{'btn-disabled': removedLine == ''}"
            @click="deleteLine()">删除</button>
          </form>
        </div>
      </div>
      <form method="dialog" class="modal-backdrop">
        <button>close</button>
      </form>
    </dialog>

    <dialog ref="confirm" class="modal">
      <div class="modal-box max-w-lg">
        <h3 class="font-bold text-2xl">确定要重新载入所有线路吗？</h3>
        <p class="pt-4 text-lg">该操作将从文件中重新载入所有线路，之前手动添加的线路将全部被删除！</p>
        <div class="modal-action">
          <form method="dialog">
            <button class="btn mr-5 text-lg" @click="reload()">确定</button>
            <button class="btn text-lg">取消</button>
          </form>
        </div>
      </div>
      <form method="dialog" class="modal-backdrop">
        <button>close</button>
      </form>
    </dialog>
  </div>

</template>

<script setup>
  import axios from 'axios';
  import { ref } from 'vue';
  const data = ref([]);
  const station_from = ref('');
  const station_to = ref('');
  const pathData = ref([]);
  const lineData = ref([]);

  const my_modal = ref(null);
  const add_line = ref(null);
  const confirm = ref(null);
  const remove_line = ref(null);
  const query_type = ref('time');

  const time = ref(0);
  const transfer = ref(0);
  const pass = ref(0);

  const removedLine = ref('');

  const stationNum = ref(2);
  const lineName = ref('');
  const lineSpeed = ref(null);
  const isLoop = ref(false);
  const stationsInfo = ref([]);

  const errorInfo = ref('');

  function lineDataInit() {
    errorInfo.value = '';
    lineName.value = '';
    lineSpeed.value = null;
    isLoop.value = false;
    stationNum.value = 2;
    stationsInfo.value = [];
    for (let i = 0; i < 30; i++) {
      stationsInfo.value.push({
        name: '',
        distance: null
      });
    }
    stationsInfo.value[0].distance = 0;
  }

  lineDataInit();

  function formatTime(seconds) {
    if (seconds == null) {
      return '';
    }
    const hours = Math.floor(seconds / 3600);
    const minutes = Math.floor((seconds % 3600) / 60);
    const remainingSeconds = Math.floor(seconds % 60);

    const hoursDisplay = hours > 0 ? (hours < 10 ? "0" + hours : hours) + ":" : "";
    const minutesDisplay = minutes > 0 ? (minutes < 10 ? "0" + minutes : minutes) + ":" : "00:";
    const secondsDisplay = remainingSeconds > 0 ? (remainingSeconds < 10 ? "0" + remainingSeconds : remainingSeconds) : "00";

    return hoursDisplay + minutesDisplay + secondsDisplay;
  }

  async function fetch() {
    const response = await axios.get('http://127.0.0.1:8080/fetch');
    data.value = response.data;
  }

  async function fetchLine() {
    const response = await axios.get('http://127.0.0.1:8080/fetchLine');
    lineData.value = response.data;
  }

  async function query() {
    const params = {
      type: query_type.value,
      station_from: station_from.value,
      station_to: station_to.value
    };
    const response = await axios.get('http://127.0.0.1:8080/query', {
      params
    });
    pathData.value = response.data;
    if (pathData.value.length == 0) {
      time.value = 0;
      transfer.value = 0;
      pass.value = 0;
      return;
    }
    time.value = formatTime(pathData.value[pathData.value.length - 1].time);
    transfer.value = pathData.value[pathData.value.length - 1].transfer;
    pass.value = pathData.value.length - transfer.value - 3;
  }

  async function invert() {
    const temp = station_from.value;
    station_from.value = station_to.value;
    station_to.value = temp;
    query();
  }

  async function deleteLine() {
    const response = await axios.post('http://127.0.0.1:8080/deleteLine', {
      line: removedLine.value
    });
    if (response.data.msg == 'ok') {
      fetch();
      fetchLine();
      query();
    } else {
      console.log('delete failed', response.data.error);
    }
    removedLine.value = '';
  }

  function fillInTestData() {
    lineName.value = '测试线路';
    lineSpeed.value = 80;
    isLoop.value = false;
    for (let i = 0; i < stationNum.value; i++) {
      stationsInfo.value[i] = { name: '测试站点' + (i + 1), distance: i == 0 ? 0 : 1000 };
    }
  }

  async function addLine() {
    var distances = [];
    var stations = [];
    for (let i = 0; i < stationNum.value; i++) {
      distances.push(stationsInfo.value[i].distance);
      stations.push(stationsInfo.value[i].name);
    }

    const response = await axios.post('http://127.0.0.1:8080/addLine', {
      name: lineName.value,
      speed: lineSpeed.value,
      distances: distances,
      stations: stations,
      loop: isLoop.value
    });

    if (response.data.status == 200) {
      fetch();
      fetchLine();
      query();
      add_line.value.close();
    } else {
      console.log('add failed', response.data.error);
      errorInfo.value = response.data.error;
    }
  }

  async function reload() {
    const response = await axios.get('http://127.0.0.1:8080/reload');
    if (response.data.msg == 'ok') {
      fetch();
      fetchLine();
      query();
    } else {
      console.log('reload failed', response.data.error);
    }
  }

  const isDark = ref(false);
  const systemTheme = window.matchMedia('(prefers-color-scheme: dark)').matches;
  if (systemTheme) {
    isDark.value = true;
    document.documentElement.classList.add('dark');
  }

  function toggleDark() {
    isDark.value = !isDark.value;
    if (document.documentElement.classList.contains('dark')) {
      document.documentElement.classList.remove('dark');
      document.documentElement.setAttribute('data-theme', 'light');
      localStorage.setItem('theme', 'light');
    } else {
      document.documentElement.classList.add('dark');
      document.documentElement.setAttribute('data-theme', 'dark');
      localStorage.setItem('theme', 'dark');
    }
  }

  fetch();
</script>

<style>

</style>
