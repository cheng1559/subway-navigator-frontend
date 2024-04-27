# Subway Navigator 前端仓库

本项目旨在实现一个北京地铁线路查询系统，支持的功能有：

- 浏览北京地铁路线图；

- 查询从给定起始站到终点站的最优方案，支持最短时间、最少换乘次数两种模式；

- 增加自定义线路及相关站点；

- 删除原有线路及相关站点；

前端部分基于 Vue3 框架开发，引入 Tailwind CSS 框架及 Daisy UI 组件库进行界面开发，引入 Axios 库实现和后端的 HTTP 通信。

**本项目已部署至 Cloudflare Pages，可通过 https://subway-navigator.org 访问。**

### 本地部署

1. 安装 18.20.2 以上版本的 [`Node.js`](https://nodejs.org/)；

2. 克隆该仓库到本地：

    ```
    git clone https://github.com/cheng1559/subway-navigator-frontend.git && cd./subway-navigator-frontend
    ```

3. 安装依赖项：

    ```
    npm install
    ```

4. 运行项目：

- 连接本地后端调试（默认使用 `8080` 端口，可在根目录下的`.env.local` 文件中更改）：

    ```
    npm run serve:local
    ```

- 连接云端服务（后端服务已部署至阿里云函数计算）：

    ```
    npm run serve:production
    ```