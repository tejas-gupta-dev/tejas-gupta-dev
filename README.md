<!-- ======================= HEADER ======================= -->
<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&height=240&color=0:0f2027,50:203a43,100:2c5364&text=Tejas%20Gupta&fontColor=ffffff&fontSize=60&fontAlignY=38&animation=fadeIn&desc=Full-Stack%20%C2%B7%20Real-Time%20Systems%20%C2%B7%20Machine%20Learning&descAlignY=58&descSize=20" alt="header" width="100%" />

<a href="https://github.com/tejas-gupta-dev">
  <img src="https://readme-typing-svg.demolab.com?font=Fira+Code&weight=600&size=24&duration=3000&pause=1000&color=58A6FF&center=true&vCenter=true&width=760&height=50&lines=Software+Engineer+who+ships+end-to-end;Real-Time+%26+Distributed+Systems;Full-Stack+Developer;Machine+Learning+Enthusiast;Building+apps+that+see%2C+sync+%26+recommend" alt="Typing animation" />
</a>

<br/><br/>

<img src="https://img.shields.io/github/followers/tejas-gupta-dev?label=Followers&style=for-the-badge&logo=github&color=2c5364" alt="Followers" />
<img src="https://img.shields.io/badge/Seeking-SDE_Roles-2ea44f?style=for-the-badge" alt="Seeking SDE roles" />

<br/><br/>

**[Flagship Project](#-flagship-project-watch-party)** &nbsp;·&nbsp; **[More Projects](#-more-projects)** &nbsp;·&nbsp; **[Skills](#%EF%B8%8F-skills)** &nbsp;·&nbsp; **[Engineering Practices](#-engineering-practices)** &nbsp;·&nbsp; **[Contact](#-lets-connect)**

</div>

<br/>

<!-- ======================= ABOUT ======================= -->
## 👨‍💻 About Me

I build software end to end: React front ends, Node.js back ends, real-time systems, and ML features that run in the browser. I care about how things work under the hood, from clock synchronization between devices to how a service behaves when it's scaled across servers.

```js
const tejas = {
  role: "Software Engineer (full-stack, real-time systems, ML)",
  languages: ["TypeScript", "JavaScript", "Python", "C++"],
  strongestAt: ["Real-time systems", "Full-stack web apps", "Applied ML"],
  currentlyBuilding: "Real-time, emotion-aware, AI-powered web apps",
  exploring: ["Distributed Systems", "Computer Vision", "Databases"],
  funFact: "I taught a webcam to pick my playlist 🎵",
};
```

<br/>

<!-- ======================= FLAGSHIP ======================= -->
## ⭐ Flagship Project: Watch Party

<div align="center">

### 🎬 Synchronized video rooms, built like a production system

*Create a room, share a 6-character code, and every viewer's player stays in lockstep.*

<a href="https://github.com/tejas-gupta-dev/Watch-Party">
  <img src="https://github-readme-stats.vercel.app/api/pin/?username=tejas-gupta-dev&repo=Watch-Party&theme=tokyonight&hide_border=true" alt="Watch Party" />
</a>

<br/>

<img src="https://img.shields.io/badge/React-18-61DAFB?style=for-the-badge&logo=react&logoColor=black" alt="React" />
<img src="https://img.shields.io/badge/TypeScript-strict-3178C6?style=for-the-badge&logo=typescript&logoColor=white" alt="TypeScript" />
<img src="https://img.shields.io/badge/Node.js-20-339933?style=for-the-badge&logo=nodedotjs&logoColor=white" alt="Node.js" />
<img src="https://img.shields.io/badge/Socket.IO-realtime-010101?style=for-the-badge&logo=socketdotio&logoColor=white" alt="Socket.IO" />
<img src="https://img.shields.io/badge/PostgreSQL-16-4169E1?style=for-the-badge&logo=postgresql&logoColor=white" alt="PostgreSQL" />
<img src="https://img.shields.io/badge/Redis-7-DC382D?style=for-the-badge&logo=redis&logoColor=white" alt="Redis" />
<img src="https://img.shields.io/badge/Docker-compose-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker" />

</div>

<br/>

**The problem:** keeping several people's video players in sync is hard. Network latency differs per device, clocks disagree, and users pause, seek and reconnect at unpredictable times.

**The approach:** the server is the single source of truth, and every client corrects itself against it.

| Engineering challenge | How I solved it |
|:--|:--|
| ⏱️ **Clock differences between devices** | NTP-style offset estimation: 8 ping round trips per client, keeping the lowest-latency sample |
| 🎯 **Staying in sync without stutter** | Playback is stored as an anchor `{position, updatedAt, playing}`. Small drift is fixed by nudging playback speed (max ±8%), large drift by seeking |
| 🔐 **Who can do what** | One shared permission matrix (host, moderator, participant) used by both the UI and the server, with every handler checking it before changing state |
| ✋ **Controlled access to playback** | Participants *request* changes, and hosts and moderators approve or reject them from a live queue |
| 📡 **Scaling past one server** | Multiple Node instances behind nginx, rooms pinned to a node by hashing the room code, events fanned out through the Redis Socket.IO adapter |
| 🔁 **Reconnects and late joiners** | Full state snapshot on join, version numbers to discard stale updates, graceful connection draining on shutdown |
| 🎞️ **Large video files** | Browser uploads directly to S3-compatible storage with presigned URLs (up to 4 GB), so media never touches the realtime path |
| 📈 **Knowing it works** | Prometheus metrics and a Grafana dashboard (sockets, rooms, latency, drift), 22 unit and integration tests, GitHub Actions CI, and a k6 load test |

<details>
<summary><b>🏗️ System architecture</b></summary>

<br/>

```mermaid
flowchart LR
  B["Browser<br/>React + Socket.IO"] -->|HTTPS / WSS| N["nginx<br/>hash on room code"]
  N --> S1["Node server 1"]
  N --> S2["Node server 2"]
  S1 <-->|"pub/sub, snapshots, chat"| R[("Redis")]
  S2 <-->|"pub/sub, snapshots, chat"| R
  S1 --> P[("PostgreSQL")]
  S2 --> P
  B -->|"presigned upload / signed download"| M[("S3 / MinIO")]
  S1 -->|metrics| PR["Prometheus"] --> G["Grafana"]
  S2 -->|metrics| PR
```

</details>

<details>
<summary><b>🔄 How one sync event flows</b></summary>

<br/>

```mermaid
sequenceDiagram
  participant H as Host
  participant S as Server
  participant V as Viewer
  H->>S: play {position: 12}
  S->>S: check role, update state
  S-->>H: playback state, version + 1
  S-->>V: playback state, version + 1
  V->>V: target = position + elapsed server time
  loop every 1 s
    V->>V: measure drift, then nudge speed or seek
  end
```

</details>

<div align="center">

<a href="https://github.com/tejas-gupta-dev/Watch-Party"><img src="https://img.shields.io/badge/Read_the_Code_and_Architecture_Docs-181717?style=for-the-badge&logo=github&logoColor=white" alt="Watch Party repository" /></a>

</div>

<br/>

<!-- ======================= MORE PROJECTS ======================= -->
## 🚀 More Projects

<div align="center">
<table>
  <tr>
    <td align="center" width="50%">
      <a href="https://github.com/tejas-gupta-dev/EmotionMusic">
        <img src="https://github-readme-stats.vercel.app/api/pin/?username=tejas-gupta-dev&repo=EmotionMusic&theme=tokyonight&hide_border=true" alt="EmotionMusic" />
      </a>
    </td>
    <td align="center" width="50%">
      <a href="https://github.com/tejas-gupta-dev/migrane">
        <img src="https://github-readme-stats.vercel.app/api/pin/?username=tejas-gupta-dev&repo=migrane&theme=tokyonight&hide_border=true" alt="Migraine Analysis" />
      </a>
    </td>
  </tr>
  <tr>
    <td align="center" width="50%">
      <a href="https://github.com/tejas-gupta-dev/movie_recommender_system">
        <img src="https://github-readme-stats.vercel.app/api/pin/?username=tejas-gupta-dev&repo=movie_recommender_system&theme=tokyonight&hide_border=true" alt="Movie Recommender System" />
      </a>
    </td>
    <td align="center" width="50%">
      <a href="https://github.com/tejas-gupta-dev/tinydb">
        <img src="https://github-readme-stats.vercel.app/api/pin/?username=tejas-gupta-dev&repo=tinydb&theme=tokyonight&hide_border=true" alt="tinydb" />
      </a>
    </td>
  </tr>
</table>
</div>

<br/>

| Project | What it does | Stack |
| :-- | :-- | :-- |
| 🎵 **[EmotionMusic](https://github.com/tejas-gupta-dev/EmotionMusic)** ([live demo](https://emotion-music-beta.vercel.app)) | Reads your facial expression through the webcam in real time and recommends music that matches your mood | `React` `Node.js` `Express` `MongoDB` `MediaPipe` |
| 🧬 **[Migraine Analysis](https://github.com/tejas-gupta-dev/migrane)** | Classifies migraine types, predicts intensity, mines symptom associations and ranks key features, with a Streamlit prediction app | `Python` `Jupyter` `Streamlit` `Random Forest` `SVM` `DNN` |
| 🎞️ **[Movie Recommender](https://github.com/tejas-gupta-dev/movie_recommender_system)** | Content-based movie recommendations on the TMDB dataset using cosine similarity | `Python` `Jupyter` |
| 🗄️ **[tinydb](https://github.com/tejas-gupta-dev/tinydb)** | A lightweight database project in C++, containerized with Docker | `C++` `CMake` `Docker` |

<br/>

<!-- ======================= SKILLS ======================= -->
## 🛠️ Skills

<div align="center">

| | |
|:--|:--|
| **💬 Languages** | <img src="https://skillicons.dev/icons?i=ts,js,py,cpp&theme=dark" alt="Languages" /> |
| **🎨 Frontend** | <img src="https://skillicons.dev/icons?i=react,css,vite&theme=dark" alt="Frontend" /> <img src="https://img.shields.io/badge/Framer_Motion-0055FF?style=flat-square&logo=framer&logoColor=white" alt="Framer Motion" /> |
| **🧩 Backend and Real-Time** | <img src="https://skillicons.dev/icons?i=nodejs,express&theme=dark" alt="Backend" /> <img src="https://img.shields.io/badge/Socket.IO-010101?style=flat-square&logo=socketdotio&logoColor=white" alt="Socket.IO" /> |
| **🗄️ Data** | <img src="https://skillicons.dev/icons?i=postgres,mongodb,redis&theme=dark" alt="Databases" /> |
| **🚢 DevOps and Observability** | <img src="https://skillicons.dev/icons?i=docker,nginx,git,vercel&theme=dark" alt="DevOps" /> <img src="https://img.shields.io/badge/Prometheus-E6522C?style=flat-square&logo=prometheus&logoColor=white" alt="Prometheus" /> <img src="https://img.shields.io/badge/Grafana-F46800?style=flat-square&logo=grafana&logoColor=white" alt="Grafana" /> |
| **🤖 ML and Data Science** | <img src="https://img.shields.io/badge/Jupyter-F37626?style=flat-square&logo=jupyter&logoColor=white" alt="Jupyter" /> <img src="https://img.shields.io/badge/Streamlit-FF4B4B?style=flat-square&logo=streamlit&logoColor=white" alt="Streamlit" /> <img src="https://img.shields.io/badge/MediaPipe-0097A7?style=flat-square&logo=google&logoColor=white" alt="MediaPipe" /> |
| **⚙️ Systems** | <img src="https://skillicons.dev/icons?i=cpp,cmake&theme=dark" alt="Systems" /> |

</div>

<br/>

<!-- ======================= PRACTICES ======================= -->
## 🧭 Engineering Practices

- ✅ **Testing:** unit and integration tests with real sockets, plus load testing with k6
- 🔒 **Security basics:** input validation on every endpoint and event, hashed passwords, signed tokens, rate limiting, permission checks before any state change
- 📊 **Observability:** metrics, dashboards and health checks built in from the start
- 🔁 **CI/CD:** automated typecheck, tests and Docker builds on every push
- 🧱 **Clean structure:** shared types between client and server, so a contract change is a compile error instead of a runtime bug
- 📝 **Documentation:** architecture notes, event contracts and design trade-offs written down, including known limits and what I'd change at 100× scale

<br/>

<!-- ======================= STATS ======================= -->
## 📊 GitHub Stats

<div align="center">

<img height="170" src="https://github-readme-stats.vercel.app/api?username=tejas-gupta-dev&show_icons=true&theme=tokyonight&hide_border=true&count_private=true" alt="GitHub stats" />
<img height="170" src="https://github-readme-stats.vercel.app/api/top-langs/?username=tejas-gupta-dev&layout=compact&theme=tokyonight&hide_border=true" alt="Top languages" />

<br/>

<img src="https://streak-stats.demolab.com?user=tejas-gupta-dev&theme=tokyonight&hide_border=true" alt="GitHub streak" />

</div>

<br/>

<!-- ======================= CONNECT ======================= -->
## 📫 Let's Connect

<div align="center">

<!-- Remove the comment markers and add your details:
<a href="https://linkedin.com/in/YOUR-HANDLE"><img src="https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn" /></a>
<a href="mailto:YOUR-EMAIL"><img src="https://img.shields.io/badge/Email-D14836?style=for-the-badge&logo=gmail&logoColor=white" alt="Email" /></a>
-->
<a href="https://github.com/tejas-gupta-dev?tab=repositories"><img src="https://img.shields.io/badge/Browse_My_Repositories-181717?style=for-the-badge&logo=github&logoColor=white" alt="Repositories" /></a>
<a href="https://emotion-music-beta.vercel.app"><img src="https://img.shields.io/badge/Try_EmotionMusic-000000?style=for-the-badge&logo=vercel&logoColor=white" alt="EmotionMusic demo" /></a>

<br/><br/>

<i>⭐ If you like something you see, a star goes a long way. Thanks for stopping by!</i>

</div>

<!-- ======================= FOOTER ======================= -->
<img src="https://capsule-render.vercel.app/api?type=waving&section=footer&height=120&color=0:0f2027,50:203a43,100:2c5364" alt="footer" width="100%" />
