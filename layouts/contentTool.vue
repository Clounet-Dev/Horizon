<template>
    <div v-if="main.length" class="content-tools">
        <div class="btn-group" role="group" aria-label="content-tools">
            <template v-for="item in main" :key="item.key">
                <a v-if="item.onclick" href="#" class="btn btn-secondary tools-btn" :class="item.class" @click.prevent="item.onclick">
                    <span v-html="item.html || item.title"></span>
                </a>
                <nuxt-link v-else :to="item.to" class="btn btn-secondary tools-btn" :class="item.class">
                    <span v-html="item.html || item.title"></span>
                </nuxt-link>
            </template>
            <template v-if="menu.length">
                <dropdown class="btn btn-secondary tools-btn tool-more-btn" :class="{'d-md-none': !showAdminMenu}">
                    <template #toggle>
                        <div class="dropdown-toggle"><span class="fa-solid fa-ellipsis-vertical"></span></div>
                    </template>
                    <div class="dropdown-menu dropdown-menu-right" role="menu">
                        <template v-for="item in menu" :key="item.key">
                            <a v-if="item.onclick" href="#" class="dropdown-item" :class="item.class" @click.prevent="item.onclick">
                                <span v-html="item.html || item.title"></span>
                            </a>
                            <nuxt-link v-else :to="item.to" class="dropdown-item" :class="item.class">
                                <span v-html="item.html || item.title"></span>
                            </nuxt-link>
                        </template>
                    </div>
                </dropdown>
            </template>
        </div>
    </div>
</template>

<script>
import Common from '~/mixins/common';
import Dropdown from '../components/dropdown';
import { toast } from 'vue-sonner';

export default {
    mixins: [Common],
    emits: ['onClickEditBtn'],
    components: {
        Dropdown
    },
    data() {
        return {
            main: [],
            menu: []
        };
    },
    computed: {
        data() {
            return this.$store.state.page?.data || {};
        },
        showAdminMenu() {
            const doc = this.data?.document;
            if (!doc) return false;
            const isValidUserDocument = doc.namespace === '사용자' || (doc.namespace && doc.namespace.replace(/\s+/g, '') === '아이피사용자');
            return Boolean(this.$store.state.session?.quick_block && isValidUserDocument);
        }
    },
    mounted() {
        this.$watch(() => this.$store.state.page?.viewName, () => this.calculate(), { immediate: true });
        this.$watch(() => this.$store.state.page?.data, () => this.calculate(), { deep: true });
    },
    methods: {
        closeAllDropdowns() {
            document.body.click();
        },
        async copyUserDocumentTarget() {
            const user = this.data?.user;
            const doc = this.data?.document;
            if (!user || !user.uuid) return;
            
            try {
                await navigator.clipboard.writeText(user.uuid);
                if (typeof toast !== 'undefined') {
                    toast(this.$t('components.author_span.copied_uuid', { accountName: doc?.title || '사용자' }));
                }
            } catch (e) {
                console.error(e);
            }
            this.closeAllDropdowns();
        },
        onUserDocumentBlockButtonClick() {
            const doc = this.data?.document;
            if (!doc) return;
            
            const isIp = doc.namespace && doc.namespace.replace(/\s+/g, '') === '아이피사용자';
            const note = this.$t('components.author_span.quick_block_note', { pos: this.$store.state.page?.title || '' }).trim();

            this.openQuickACLGroup({
                ...(isIp ? { ip: doc.title } : { username: doc.title }),
                note,
            });
            this.closeAllDropdowns();
        },
        calculate() {
            this.main = [];
            this.menu = [];

            if (!this.data.document) {
                return;
            }

            const viewName = this.$store.state.page?.viewName;
            const uuid = this.data.uuid;

            if (viewName === 'wiki' || !viewName) {
                if (!uuid) {
                    if (this.data.starred) {
                        this.main.push({
                            key: 'unstar',
                            to: this.doc_action_link(this.data.document, 'member/unstar'),
                            html: `<span class="fa-solid fa-star"></span> ${this.data.star_count}`,
                            class: 'd-none d-md-inline-block'
                        });
                    } else if (this.data.star_count >= 0) {
                        this.main.push({
                            key: 'star',
                            to: this.doc_action_link(this.data.document, 'member/star'),
                            html: `<span class="fa-regular fa-star"></span> ${this.data.star_count}`,
                            class: 'd-none d-md-inline-block'
                        });
                    }
                    this.main.push({
                        key: 'backlink',
                        to: this.doc_action_link(this.data.document, 'backlink'),
                        title: '<i class="fa-solid fa-link"></i> 역링크'
                    });
                    this.main.push({
                        key: 'discuss',
                        to: this.doc_action_link(this.data.document, 'discuss'),
                        class: this.data.discuss_progress ? 'btn-discuss-progress' : null,
                        title: '<i class="fa-solid fa-comments"></i> 토론'
                    });
                    if (this.data.editable === true && this.data.edit_acl_message) {
                        this.main.push({
                            key: 'edit-request',
                            onclick: () => this.$emit('onClickEditBtn'),
                            html: '<span class="fa-solid fa-code-pull-request"></span> 편집 요청'
                        });
                    } else if (this.data.editable === false && this.data.edit_acl_message) {
                        this.main.push({
                            key: 'edit-lock',
                            onclick: () => this.$emit('onClickEditBtn'),
                            html: '<span class="fa-solid fa-lock"></span> 편집'
                        });
                    } else {
                        this.main.push({
                            key: 'edit',
                            to: this.doc_action_link(this.data.document, 'edit'),
                            html: '<span class="fa-solid fa-pen-to-square"></span> 편집'
                        });
                    }
                    this.main.push({
                        key: 'history',
                        to: this.doc_action_link(this.data.document, 'history', this.data.rev ? { from: this.data.rev } : undefined),
                        title: '<i class="fa-solid fa-clock-rotate-left"></i> 역사'
                    });
                    this.main.push({
                        key: 'acl',
                        to: this.doc_action_link(this.data.document, 'acl'),
                        title: '<i class="fa-solid fa-shield-halved"></i> ACL'
                    });

                    if (this.data.user) {
                        this.menu.push({
                            key: 'contribution',
                            to: this.contribution_link(this.data.user.uuid),
                            html: '<i class="fa-solid fa-list"></i> 기여 목록'
                        });

                        if (this.showAdminMenu) {
                            this.menu.push({
                                key: 'block',
                                onclick: this.onUserDocumentBlockButtonClick,
                                html: '<i class="fa-solid fa-ban"></i> 사용자 차단'
                            });
                            
                            if (this.data.user.uuid) {
                                this.menu.push({
                                    key: 'block-history',
                                    to: {
                                        path: '/BlockHistory',
                                        query: { query: this.data.user.uuid, target: 'text' }
                                    },
                                    html: '<i class="fa-solid fa-user-lock"></i> 차단 내역 조회'
                                });
                            }
                            
                            this.menu.push({
                                key: 'copy-uuid',
                                onclick: this.copyUserDocumentTarget,
                                html: '<i class="fa-solid fa-copy"></i> UUID 복사'
                            });
                        }
                    }
                    this.menu.push({
                        key: 'move',
                        to: this.doc_action_link(this.data.document, 'move'),
                        html: '<i class="fa-solid fa-arrow-right-arrow-left"></i> 이동',
                        class: 'd-md-none'
                    });
                    this.menu.push({
                        key: 'delete',
                        to: this.doc_action_link(this.data.document, 'delete'),
                        html: '<i class="fa-solid fa-trash-can"></i> 삭제',
                        class: 'd-md-none'
                    });
                }
                return;
            }

            if (viewName === 'raw' || viewName === 'blame' || viewName === 'revert' || viewName === 'diff') {
                this.main.push({
                    key: 'history',
                    to: this.doc_action_link(this.data.document, 'history', this.data.rev ? { from: this.data.rev } : undefined),
                    class: 'btn-info',
                    title: '역사'
                });
                this.main.push({
                    key: 'wiki',
                    to: this.doc_action_link(this.data.document, 'w', uuid ? { uuid } : undefined),
                    title: '보기'
                });
                this.main.push({
                    key: 'raw',
                    to: this.doc_action_link(this.data.document, 'raw', uuid ? { uuid } : undefined),
                    class: viewName === 'raw' ? 'disabled' : null,
                    title: 'RAW'
                });
                this.main.push({
                    key: 'blame',
                    to: this.doc_action_link(this.data.document, 'blame', uuid ? { uuid } : undefined),
                    class: viewName === 'blame' ? 'disabled' : null,
                    title: 'Blame'
                });
                if (viewName === 'diff') {
                    this.main.push({
                        key: 'diff',
                        title: '비교'
                    });
                }
            }
        }
    }
};
</script>