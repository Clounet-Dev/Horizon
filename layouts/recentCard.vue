<template>
    <div class="sidebar-1-content">
        <ul class="sidebar-1-list" id="sidebar-1-list">
            <li v-for="item in recent.slice(0, limit)" :key="item.key">
                <nuxt-link class="recent-item" :to="doc_action_link(item.document, 'w')">
                    <span class="recent-time">{{ item.time }}</span>
                    <span class="recent-title">{{ item.document }}</span>
                </nuxt-link>
            </li>
        </ul>
    </div>
</template>

<script>
import Common from '~/mixins/common';

export default {
    mixins: [Common],
    props: {
        limit: {
            type: Number,
            default: 14
        }
    },
    data() {
        return {
            recent: []
        };
    },
    mounted() {
        this.loadSidebar();
    },
    methods: {
        formatSidebarDate(dateValue) {
            if (dateValue == null) return '';

            const date = new Date(Number(dateValue) * 1000);
            if (Number.isNaN(date.getTime())) return '';

            const now = Date.now();
            const isRecent = (now - date.getTime()) < 1000 * 60 * 60 * 24;
            if (isRecent) {
                return date.toLocaleTimeString('ko-KR', {
                    hour: '2-digit',
                    minute: '2-digit'
                });
            }

            return date.toLocaleDateString('ko-KR', {
                year: '2-digit',
                month: '2-digit',
                day: '2-digit'
            });
        },
        async loadSidebar() {
            try {
                const json = await this.internalRequest('/sidebar', { noProgress: true });
                this.recent = (json?.document || []).map((item, index) => ({
                    key: `${index}-${item.document}`,
                    document: item.document || '',
                    time: this.formatSidebarDate(item.date)
                }));
            } catch {
                this.recent = [];
            }
        }
    }
};
</script>