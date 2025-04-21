<script setup lang="ts">
const appConfig = useAppConfig()
const timeEstablished = appConfig.timeEstablished

const blogLog = [
    { date: '2026-09-18', content: '域名到期，非必要情况会保持续费' },
    { date: '2025-03-16', content: '基于纸鹿的 Nuxt Content 重构博客上线' },
    { date: '2024-12-24', content: '更换框架为 WordPress，升级博客' },
    { date: '2024-06-20', content: '更换到当前域名' },
    { date: '2022-09-01', content: '拥有自己第一个博客，后续部分更新记录与部分文章已丢失。' },
]
</script>

<template>
    <h3 class="widget-title">
        更新日志
    </h3>
    <!-- TODO: 优化时间线样式 -->
    <div class="widget-card">
        <ul>
            <li v-for="(item, index) in blogLog" :key="index">
                <small>{{ item.date }}</small><br>
                {{ item.content }}
            </li>
        </ul>
    </div>
</template>
