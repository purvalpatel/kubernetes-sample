🔵 1. 𝗣𝗲𝗻𝗱𝗶𝗻𝗴 – 𝗧𝗵𝗲 𝗕𝗶𝗿𝘁𝗵 👶
Kubernetes is setting up everything for your Pod—finding the right node, allocating resources, and preparing the environment.

🔧 𝟮. 𝗖𝗼𝗻𝘁𝗮𝗶𝗻𝗲𝗿 𝗖𝗿𝗲𝗮𝘁𝗶𝗻𝗴 – The Setup Phase 🏗️
Your container images are pulled, networking is configured, and dependencies are set. Think of it as the pre-launch preparations!

💡 3. 𝗥𝘂𝗻𝗻𝗶𝗻𝗴 – The Spotlight Moment 🌟
The Pod is now live, serving requests like a champ. This is where your application operates at its peak. Monitoring is key to maintaining performance!

🎯 4. 𝗦𝘂𝗰𝗰𝗲𝗲𝗱𝗲𝗱 – 𝗠𝗶𝘀𝘀𝗶𝗼𝗻 𝗔𝗰𝗰𝗼𝗺𝗽𝗹𝗶𝘀𝗵𝗲𝗱 🎉
For Pods that run once (like batch jobs), this is their final stage—they complete the task and exit successfully.

❌ 5. 𝗙𝗮𝗶𝗹𝗲𝗱 – 𝗢𝗼𝗽𝘀, 𝗦𝗼𝗺𝗲𝘁𝗵𝗶𝗻𝗴 𝗪𝗲𝗻𝘁 𝗪𝗿𝗼𝗻𝗴 🚨
The Pod crashed due to an error it couldn’t recover from. Time to dig into logs and investigate!

♻️ 6. 𝗖𝗿𝗮𝘀𝗵𝗟𝗼𝗼𝗽𝗕𝗮𝗰𝗸𝗢𝗳𝗳 – 𝗦𝘁𝘂𝗰𝗸 𝗶𝗻 𝗮 𝗥𝗲𝘀𝘁𝗮𝗿𝘁 𝗟𝗼𝗼𝗽 🔄
The Pod keeps restarting due to repeated failures. It’s a sign that something deeper is wrong—time for some debugging!

🔚 7. 𝗧𝗲𝗿𝗺𝗶𝗻𝗮𝘁𝗶𝗻𝗴 – A Graceful Goodbye 🛑
When a Pod is no longer needed, Kubernetes shuts it down while ensuring no abrupt failures or data loss.
