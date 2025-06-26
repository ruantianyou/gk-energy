<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8">
    <title>高考能量站</title>
    <script src="https://cdn.jsdelivr.net/npm/vue@3/dist/vue.global.prod.js"></script>
    <style>
        body {
            margin: 0;
            background: linear-gradient(45deg, #1a237e, #4a148c);
            height: 100vh;
            overflow: hidden;
            /* 页面进场动效 */
            opacity: 0;
            transform: translateY(30px) scale(0.98);
            animation: pageFadeIn 1s ease forwards;
        }
        @keyframes pageFadeIn {
            to {
                opacity: 1;
                transform: translateY(0) scale(1);
            }
        }

        #app {
            display: flex;
            flex-direction: column;
            align-items: center;
            color: white;
            min-height: 100vh;
            justify-content: center;
            /* 容器淡入 */
            opacity: 0;
            animation: appFadeIn 1s 0.5s cubic-bezier(.68,-0.55,.27,1.55) forwards;
        }
        @keyframes appFadeIn {
            to { opacity: 1; }
        }

        h1 {
            font-size: 2.8em;
            margin-bottom: 0.5em;
            letter-spacing: 2px;
            text-shadow: 0 4px 24px rgba(0,0,0,0.25);
            opacity: 0;
            transform: translateY(-30px) scale(0.95);
            animation: titleIn 1s 1s cubic-bezier(.68,-0.55,.27,1.55) forwards;
        }
        @keyframes titleIn {
            to {
                opacity: 1;
                transform: translateY(0) scale(1);
            }
        }

        .time-now {
            font-size: 1.5em;
            margin-bottom: 1em;
            text-shadow: 0 0 5px rgba(255,255,255,0.3);
            opacity: 0;
            transform: translateY(-30px) scale(0.95);
            animation: fadeInUp 0.9s 1.3s cubic-bezier(.68,-0.55,.27,1.55) forwards;
        }

        .countdown {
            font-size: 3em;
            text-shadow: 0 0 10px rgba(255, 255, 255, 0.5);
            margin-bottom: 1em;
            opacity: 0;
            transform: translateY(-20px) scale(0.98);
            animation: fadeInUp 0.9s 1.5s cubic-bezier(.68,-0.55,.27,1.55) forwards;
        }

        @keyframes fadeInUp {
            to {
                opacity: 1;
                transform: translateY(0) scale(1);
            }
        }

        .btn {
            padding: 15px 30px;
            background: linear-gradient(90deg,#00e676,#1de9b6);
            border: none;
            border-radius: 25px;
            font-size: 1.2em;
            cursor: pointer;
            font-weight: bold;
            color: #212121;
            box-shadow: 0 4px 18px 0 rgba(0,230,118,0.18);
            transition:
                background 0.4s cubic-bezier(.68,-0.55,.27,1.55),
                box-shadow 0.25s,
                transform 0.15s;
            position: relative;
            overflow: hidden;
            margin-bottom: 1.5em;
            opacity: 0;
            transform: scale(0.92);
            animation: btnIn 0.7s 1.7s cubic-bezier(.68,-0.55,.27,1.55) forwards;
        }
        @keyframes btnIn {
            to {
                opacity: 1;
                transform: scale(1);
            }
        }
        .btn:hover {
            background: linear-gradient(90deg,#43e97b,#38f9d7);
            box-shadow: 0 8px 32px 0 rgba(0,230,118,0.24);
            transform: scale(1.06) translateY(-2px);
        }
        .btn:active {
            background: linear-gradient(90deg,#00b168,#00e676);
            box-shadow: 0 2px 6px 0 rgba(0,230,118,0.12);
            transform: scale(0.96);
        }
        .btn::after {
            content: "";
            display: block;
            position: absolute;
            left: 50%;
            top: 50%;
            width: 0;
            height: 0;
            background: rgba(255,255,255,0.24);
            border-radius: 100%;
            transform: translate(-50%, -50%);
            transition: width 0.2s, height 0.2s;
            z-index: 1;
        }
        .btn:active::after {
            width: 150%;
            height: 300%;
        }

        canvas {
            position: fixed;
            top: 0;
            left: 0;
            pointer-events: none;
        }
    </style>
</head>

<body>
    <div id="app">
        <h1>高考能量补给站</h1>
        <div class="time-now">{{ now }}</div>
        <div class="countdown">{{ days }}天{{ hours }}时{{ minutes }}分{{ seconds }}秒</div>
        <button class="btn" @click="sendConfetti" @mouseenter="btnHover = true" @mouseleave="btnHover = false">
            {{ btnHover ? "好运来啦！" : "获取好运" }}
        </button>
        <canvas ref="canvas"></canvas>
    </div>

    <script>
        const { createApp, ref, onMounted } = Vue;

        createApp({
            setup() {
                const canvas = ref(null)
                const days = ref(0)
                const hours = ref(0)
                const minutes = ref(0)
                const seconds = ref(0)
                const now = ref('')
                const btnHover = ref(false)
                let ctx = null

                // 动态粒子数组
                let particles = []

                // 粒子类
                class Particle {
                    constructor(x, y, vx, vy, color) {
                        this.x = x
                        this.y = y
                        this.vx = vx
                        this.vy = vy
                        this.radius = 3 + Math.random() * 2
                        this.color = color
                        this.life = 60 + Math.random() * 40
                        this.alpha = 1
                    }
                    update() {
                        this.x += this.vx
                        this.y += this.vy
                        this.vy += 0.06
                        this.life--
                        if (this.life < 20) this.alpha = this.life / 20
                    }
                    draw(ctx) {
                        ctx.save()
                        ctx.globalAlpha = this.alpha
                        ctx.beginPath()
                        ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2)
                        ctx.fillStyle = this.color
                        ctx.fill()
                        ctx.restore()
                    }
                }

                // 动画循环
                const animate = () => {
                    ctx.clearRect(0, 0, canvas.value.width, canvas.value.height)
                    for (let i = particles.length - 1; i >= 0; i--) {
                        particles[i].update()
                        particles[i].draw(ctx)
                        if (particles[i].life <= 0) {
                            particles.splice(i, 1)
                        }
                    }
                    requestAnimationFrame(animate)
                }

                // 倒计时计算 & 实时时间
                const updateTime = () => {
                    const target = new Date('2025-06-07T09:00:00')
                    const nowDate = new Date()
                    const diff = target - nowDate

                    if (diff > 0) {
                        days.value = Math.floor(diff / (1000 * 60 * 60 * 24))
                        hours.value = Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60))
                        minutes.value = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60))
                        seconds.value = Math.floor((diff % (1000 * 60)) / 1000)
                    } else {
                        days.value = 0
                        hours.value = 0
                        minutes.value = 0
                        seconds.value = 0
                    }

                    now.value = nowDate.getFullYear() + '-' +
                        String(nowDate.getMonth() + 1).padStart(2, '0') + '-' +
                        String(nowDate.getDate()).padStart(2, '0') + ' ' +
                        String(nowDate.getHours()).padStart(2, '0') + ':' +
                        String(nowDate.getMinutes()).padStart(2, '0') + ':' +
                        String(nowDate.getSeconds()).padStart(2, '0')
                }

                // 彩纸特效动效（多角度随机）
                const sendConfetti = () => {
                    const w = canvas.value.width
                    const h = canvas.value.height
                    for (let i = 0; i < 50; i++) {
                        const angle = Math.random() * Math.PI * 2
                        const speed = 3 + Math.random() * 3
                        const vx = Math.cos(angle) * speed
                        const vy = Math.sin(angle) * speed
                        const x = w / 2
                        const y = h / 3
                        const color = `hsl(${Math.random() * 360}, 70%, 60%)`
                        particles.push(new Particle(x, y, vx, vy, color))
                    }
                }

                onMounted(() => {
                    ctx = canvas.value.getContext('2d')
                    canvas.value.width = window.innerWidth
                    canvas.value.height = window.innerHeight

                    setInterval(updateTime, 1000)
                    updateTime()
                    animate()
                })

                return { days, hours, minutes, seconds, now, canvas, sendConfetti, btnHover }
            }
        }).mount('#app')
    </script>
</body>

</html>
