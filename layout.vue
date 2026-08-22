<template>
    <div class="horizon horizon-skin" :class="rootClasses" :style="skinConfig">
        <div id="top"></div>

        <div class="nav-wrapper" :class="{ 'navbar-fixed-top': fixedNavbar }">
            <nav class="navbar navbar-dark">
                <nuxt-link class="navbar-brand" to="/">
                    <template v-if="navbarLogoShow">
                        <i v-if="logoIcon && logoIcon.length" class="navbar-logo-icon" :class="['fa-' + logoIcon[0], 'fa-' + logoIcon[1]]" :style="logoImageStyle"></i>
                        <img v-else-if="navbarLogo" class="navbar-logo-img" :src="navbarLogo" :style="logoImageStyle" alt="logo">
                    </template>
                    <span v-if="navbarBrandTextShow" class="navbar-brand-text">{{ navbarBrandText }}</span>
                </nuxt-link>

                <ul class="nav navbar-nav">
                    <li class="nav-item">
                        <nuxt-link class="nav-link" to="/RecentChanges">
                            <span class="fa-solid fa-pen"></span><span class="hide-title">최근 변경</span>
                        </nuxt-link>
                    </li>
                    <li class="nav-item">
                        <nuxt-link class="nav-link" to="/RecentDiscuss">
                            <span class="fa-solid fa-comments"></span><span class="hide-title">최근 토론</span>
                        </nuxt-link>
                    </li>
                    <li class="nav-item">
                        <dropdown>
                            <template #toggle>
                                <a class="nav-link dropdown-toggle dropdown-toggle-fix" href="#" @click.prevent>
                                    <span class="fa-solid fa-wrench"></span><span class="hide-title">특수 기능</span>
                                    <i class="fa-solid fa-caret-down"></i>
                                </a>
                            </template>
                            <div class="dropdown-menu dropdown-menu-right nav-dropdown-menu" role="menu">
                                <nuxt-link to="/Upload" class="dropdown-item"><i class="fa-solid fa-upload"></i> 파일 올리기</nuxt-link>
                                <div class="dropdown-divider"></div>
                                <nuxt-link to="/NeededPages" class="dropdown-item"><i class="fa-solid fa-file-circle-exclamation"></i> 작성이 필요한 문서</nuxt-link>
                                <nuxt-link to="/OrphanedPages" class="dropdown-item"><i class="fa-solid fa-file-circle-xmark"></i> 고립된 문서</nuxt-link>
                                <nuxt-link to="/OrphanedCategories" class="dropdown-item"><i class="fa-solid fa-filter-circle-xmark"></i> 고립된 분류</nuxt-link>
                                <nuxt-link to="/UncategorizedPages" class="dropdown-item"><i class="fa-solid fa-folder-open"></i> 분류가 되지 않은 문서</nuxt-link>
                                <nuxt-link to="/OldPages" class="dropdown-item"><i class="fa-solid fa-file-circle-question"></i> 편집된 지 오래된 문서</nuxt-link>
                                <nuxt-link to="/ShortestPages" class="dropdown-item"><i class="fa-solid fa-magnifying-glass-minus"></i> 내용이 짧은 문서</nuxt-link>
                                <nuxt-link to="/LongestPages" class="dropdown-item"><i class="fa-solid fa-magnifying-glass-plus"></i> 내용이 긴 문서</nuxt-link>
                                <nuxt-link to="/BlockHistory" class="dropdown-item"><i class="fa-solid fa-ban"></i> 차단 내역</nuxt-link>
                                <nuxt-link to="/Terms" class="dropdown-item"><i class="fa-solid fa-file-invoice"></i> 약관</nuxt-link>
                                <nuxt-link to="/License" class="dropdown-item"><i class="fa-solid fa-copyright"></i> 라이선스</nuxt-link>
                            </div>
                        </dropdown>
                    </li>
                    <li class="nav-item" v-if="customMenus.length">
                        <dropdown>
                            <template #toggle>
                                <a class="nav-link dropdown-toggle dropdown-toggle-fix" href="#" @click.prevent>
                                    <span class="fa-solid fa-hammer"></span><span class="hide-title">관리 기능</span>
                                    <i class="fa-solid fa-caret-down"></i>
                                </a>
                            </template>
                            <div class="dropdown-menu dropdown-menu-right nav-dropdown-menu" role="menu">
                                <nuxt-link v-for="menu in customMenus" :key="menu.to" :to="menu.to" class="dropdown-item">
                                    <i :class="menu.icon"></i> {{ menu.title }}
                                </nuxt-link>
                            </div>
                        </dropdown>
                    </li>
                </ul>

                <div class="navbar-login">
                    <dropdown class="login-menu">
                        <template #toggle>
                            <a id="login-menu" class="dropdown-toggle" type="button">
                                <img v-if="gravatarUrl" class="profile-img" :src="gravatarUrl" alt="profile">
                                <span v-else class="fa-solid fa-user-large"></span>
                            </a>
                        </template>
                        <div class="dropdown-menu dropdown-menu-right login-dropdown-menu">
                            <div class="login-dropdown-name">
                                <div class="login-dropdown-name1">{{ accountName }}</div>
                                <div class="login-dropdown-name2">{{ accountLabel }}</div>
                            </div>
                            <div v-if="isLoggedIn" class="dropdown-divider"></div>
                            <button v-if="isLoggedIn" class="dropdown-item" @click="$router.push('/member/mypage')"><i class="fa-solid fa-circle-user"></i> 내 정보</button>
                            <template v-if="isLoggedIn">
                                <button class="dropdown-item" @click="$router.push(doc_action_link(user_doc(accountName), 'w'))"><i class="fa-solid fa-file-lines"></i> 사용자 문서</button>
                                <button v-if="isLoggedIn" class="dropdown-item" @click="$router.push(accountContributionDocumentLink)"><i class="fa-solid fa-rectangle-list"></i> 내 기여 목록</button>
                            </template>
                            <div v-if="isLoggedIn" class="dropdown-divider"></div>
                            <template v-if="isLoggedIn">
                                <button class="dropdown-item" @click="$router.push('/member/starred_documents')"><i class="fa-solid fa-star"></i> 내 문서함</button>
                                <button class="dropdown-item" @click="$router.push('/member/notifications')"><i class="fa-solid fa-bell bell-icon"></i> 알림</button>
                            </template>
                            <div class="dropdown-divider"></div>
                            <button v-if="isDarkMode" class="dropdown-item" type="button" @click.prevent="setTheme('light')">
                                <i class="fa-solid fa-sun"></i><span>라이트 모드</span>
                            </button>
                            <button v-else class="dropdown-item" type="button" @click.prevent="setTheme('dark')">
                                <i class="fa-solid fa-moon"></i><span>다크 모드</span>
                            </button>
                            <button class="dropdown-item" type="button" @click.prevent="openSettingModal">
                                <i class="fa-solid fa-gear"></i><span>설정</span>
                            </button>
                            <div class="dropdown-divider"></div>

                            <template v-if="isLoggedIn">
                                <template v-if="secondaryAccounts.length">
                                    <div v-for="(account, index) in secondaryAccounts" :key="account.uuid || account.name || index" class="dropdown-item account-switch-item">
                                        <nuxt-link :to="switchAccountRoute(account)" class="account-switch-link">
                                            <img v-if="account.avatar" class="account-switch-avatar" :src="account.avatar" :alt="account.name || 'account'">
                                            <span v-else class="account-switch-avatar account-switch-avatar--fallback">{{ (account.name || '?').charAt(0).toUpperCase() }}</span>
                                            <span class="account-switch-name">{{ account.name }}</span>
                                        </nuxt-link>
                                        <button type="button" class="account-switch-logout" @click.stop.prevent="logoutOtherAccount(account.uuid)">
                                            <i class="fa-solid fa-xmark"></i>
                                        </button>
                                    </div>
                                </template>
                                <button v-else class="dropdown-item account-empty" type="button" @click.prevent>
                                    <i class="fa-solid fa-user-xmark"></i>
                                    전환할 계정이 없습니다.
                                </button>
                                <button class="dropdown-item" @click="$router.push({path:'/member/login',query:{redirect:$route.fullPath}})"><i class="fa-solid fa-user-plus"></i> 계정 추가</button>
                                <div class="dropdown-divider"></div>
                            </template>

                            <button v-if="isLoggedIn" class="dropdown-item" @click="$router.push('/member/logout')"><i class="fa-solid fa-arrow-right-from-bracket"></i> 로그아웃</button>
                            <template v-else>
                                <button class="dropdown-item" @click="$router.push('/member/login')"><i class="fa-solid fa-arrow-right-to-bracket"></i> 로그인</button>
                                <button class="dropdown-item" @click="$router.push('/member/signup')"><i class="fa-solid fa-user-plus"></i> 회원가입</button>
                            </template>
                        </div>
                    </dropdown>
                </div>

                <div class="navbar-alarm">
                    <nuxt-link class="navbar-alarm-item" :class="{ 'navbar-alarm-item--unread': alarmCount !== '0' }" to="/member/notifications">
                        <i class="fa-solid fa-bell"></i>
                    </nuxt-link>
                </div>

                <search-form />
            </nav>
        </div>

        <div v-if="siteNotice" class="horizon-top-notice-bg">
            <div class="horizon-top-notice" v-html="siteNotice"></div>
        </div>

        <div class="menu-buttons">
            <a class="scroll-menu" href="#sidebar"><i class="fa-solid fa-bars"></i></a>
        </div>

        <div class="scroll-buttons">
            <a v-if="hasToc" class="scroll-toc" href="#toc" id="scroll-button-toc"><i class="fa-solid fa-list-ul" aria-hidden="true"></i></a>
            <a class="scroll-button" :style="!hasToc ? 'border-radius: 20px 0 0 20px;' : ''" href="#top" id="scroll-button-left"><i class="fa-solid fa-angle-up" aria-hidden="true"></i></a>
            <a class="scroll-bottom" href="#bottom" id="scroll-button-right"><i class="fa-solid fa-angle-down" aria-hidden="true"></i></a>
        </div>

        <div class="content-wrapper">
            <div class="container-fluid horizon-content">
                <div class="horizon-content-header">
                    <content-tool @onClickEditBtn="showEditMessage" />

                    <div class="title">
                        <h1 id="main_title">
                            <nuxt-link v-if="pageDocument" class="main_title_a" :to="doc_action_link(pageDocument, 'w')">
                                <template v-if="pageTitleNamespace">
                                    <span class="page-title_namespace">{{ pageTitleNamespace }}</span><span class="page-title_text">{{ pageTitleText }}</span>
                                </template>
                                <template v-else>
                                    {{ pageTitle }}
                                </template>
                            </nuxt-link>
                            <span v-else class="main_title_a">
                                <template v-if="pageTitleNamespace">
                                    <span class="page-title_namespace">{{ pageTitleNamespace }}</span><span class="page-title_text">{{ pageTitleText }}</span>
                                </template>
                                <template v-else>
                                    {{ pageTitle }}
                                </template>
                            </span>
                            <sub v-if="PageRevision"> r{{ PageRevision }}</sub>
                        </h1>
                        <div v-if="pageDate" class="last-edit">마지막 편집: {{ pageDate }}</div>
                    </div>
                </div>

                <div class="horizon-content-main wiki-article">
                    <alert v-if="isShowACLMessage && pageAclMessage" error closable @close="isShowACLMessage = false">
                        <span v-html="pageAclMessage"></span>
                    </alert>
                    <nuxt />
                    <div v-if="pageViewName === 'license'">
                        <h2>Horizon Skin License</h2>
                        <pre>{{ License }}</pre>
                    </div>
                    <div v-if="pageCopyrightText" class="wiki-copyright" v-html="pageCopyrightText"></div>
                </div>
            </div>

            <div class="horizon-sidebar" id="sidebar">
                <div class="sidebar-0">
                    <nuxt-link class="sidebar-0-title" to="/member/mypage"><i class="fa-solid fa-user-large"></i> 사용자 <i class="sidebar-angle fa-solid fa-angle-right"></i></nuxt-link>
                    <div class="sidebar-0-account">
                        <div class="sidebar-0-profile">
                            <img v-if="gravatarUrl" class="profile-img" :src="gravatarUrl" alt="profile">
                            <i v-else class="fa-solid fa-circle-user"></i>
                        </div>
                        <div class="sidebar-0-name">
                            <div id="sidebar-0-name1">{{ accountName }}</div>
                            <div id="sidebar-0-name2">{{ accountLabel }}</div>
                            <div id="sidebar-0-name3"></div>
                        </div>
                    </div>
                    <div class="sidebar-0-tool">
                            <nuxt-link v-if="isLoggedIn" class="sidebar-0-item" to="/member/mypage"><i class="fa-solid fa-circle-user"></i></nuxt-link>
                            <nuxt-link class="sidebar-0-item" to="/member/starred_documents"><i class="fa-solid fa-star"></i></nuxt-link>
                            <nuxt-link class="sidebar-0-item" to="/member/notifications"><i class="fa-solid fa-bell"></i></nuxt-link>
                            <a href="#" class="sidebar-0-item" @click.prevent="openSettingModal"><i class="fa-solid fa-gear"></i></a>
                            <nuxt-link class="sidebar-0-item" to="/member/logout"><i class="fa-solid fa-arrow-right-from-bracket"></i></nuxt-link>
                    </div>
                </div>

                <div class="sidebar-1" v-if="sidebarDocument.length > 0">
                    <nuxt-link class="sidebar-1-title" to="/RecentChanges"><i class="fa-solid fa-pen"></i> 최근 변경 <i class="sidebar-angle fa-solid fa-angle-right"></i></nuxt-link>
                    <recent-card :limit="14" />
                </div>

                <div class="sidebar-2" v-if="recentDiscuss.length > 0">
                    <nuxt-link class="sidebar-2-title" to="/RecentDiscuss"><i class="fa-solid fa-comments"></i> 최근 토론 <i class="sidebar-angle fa-solid fa-angle-right"></i></nuxt-link>
                    <ul class="sidebar-2-list">
                        <li v-for="item in recentDiscuss" :key="item.key">
                            <nuxt-link class="recent-item" :to="item.to">
                                <span class="recent-time">{{ item.time }}</span>
                                <span class="recent-title">{{ item.title }}</span>
                            </nuxt-link>
                        </li>
                    </ul>
                </div>

                <div class="sidebar-5" v-html="sidebarFooterHtml"></div>
            </div>
        </div>

        <div class="horizon-footer" id="bottom">
            <p class="footer-p" v-html="footerTailHtml"></p>
            <p class="footer-p" v-html="footerText"></p>
            <div class="footer-p" id="powered">
                Powered by <a href="https://github.com/wjdgustn/thetree" target="_blank" rel="noopener noreferrer">the tree</a>,
                Designed by <a href="https://github.com/Clounet-Dev/Horizon" target="_blank" rel="noopener noreferrer">Horizon</a>.
            </div>
        </div>
    </div>
</template>

<style>
@import "./css/bootstrap.min.css";
@import "./css/jquery-ui.min.css";
@import url("https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.6.0/css/all.min.css");
@import url("https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/static/pretendard.min.css");
@import "./css/default.css";
@import "./css/default_mobile.css";
@import "./css/style.css";
@import "./css/tool.css";
@import "./css/dark.css";

.horizon-skin {
    font-family: var(--horizon-font) !important;
}

.vfm, .vfm__container, .vfm__overlay {
    z-index: 9999 !important;
}
</style>

<script>
import Common from '~/mixins/common';
import Alert from '~/components/alert';
import Dropdown from './components/dropdown';
import SearchForm from './layouts/searchForm';
import RecentCard from './layouts/recentCard';
import ContentTool from './layouts/contentTool';
import SettingModal from './components/settingModal';
import License from './LICENSE.md?raw';

export default {
    mixins: [Common],
    components: {
        Alert,
        Dropdown,
        SearchForm,
        RecentCard,
        ContentTool
    },
    data() {
        return {
            isShowACLMessage: false,
            recentDiscuss: [],
            sidebarDocument: [],
            sidebarDiscuss: [],
            License,
            hasToc: false,
            tocObserver: null
        };
    },
    head() {
        return {
            htmlAttrs: {
                class: this.isDarkMode ? 'theseed-dark-mode' : ''
            },
            meta: [
                {
                    name: 'theme-color',
                    content: this.isDarkMode ? this.darkBrandColor : this.brandColor
                }
            ]
        };
    },
    computed: {
        isDarkMode() {
            return this.$store.state.currentTheme === 'dark';
        },
        rootClasses() {
            return {
                'theseed-dark-mode': this.isDarkMode
            };
        },
        fixedNavbar() {
            return this.$store.state.localConfig?.['horizon.fixed_navbar'] === true;
        },
        navbarBrandText() {
            return this.$store.state.config?.['skin.horizon.navbar_logo_text'] || this.$store.state.config?.['wiki.site_name'] || 'horizon';
        },
        navbarLogo() {
            return this.$store.state.config?.['skin.horizon.navbar_logo'] || '';
        },
        navbarLogoShow() {
            return this.$store.state.config?.['skin.horizon.navbar_logo_show'] !== false;
        },
        navbarBrandTextShow() {
            return this.$store.state.config?.['skin.horizon.navbar_logo_text_show'] !== false;
        },
        logoImageStyle() {
            return this.$store.state.config?.['skin.horizon.logo_image_style'] || '';
        },
        logoIcon() {
            const icon = this.$store.state.config?.['skin.horizon.logo_icon'];
            return Array.isArray(icon) && icon.length >= 2 ? icon : null;
        },
        brandColor() {
            return this.$store.state.config?.['skin.horizon.brand_color'] || '#696fff';
        },
        subColor() {
            return this.$store.state.config?.['skin.horizon.sub_color'] || '#A8ACFF';
        },
        darkBrandColor() {
            return this.$store.state.config?.['skin.horizon.dark_brand_color'] || '#2d2f34';
        },
        darkSubColor() {
            return this.$store.state.config?.['skin.horizon.dark_sub_color'] || '#383b40';
        },
        navbarLogoPadding() {
            return this.$store.state.config?.['skin.horizon.navbar_logo_padding'] || '0';
        },
        navbarLogoWidth() {
            return this.$store.state.config?.['skin.horizon.navbar_logo_width'] || 'auto';
        },
        gravatarUrl() {
            return this.$store.state.session?.gravatar_url || '';
        },
        isLoggedIn() {
            return this.$store.state.session?.account?.type === 1;
        },
        accountName() {
            return this.$store.state.session?.account?.name || 'IP User';
        },
        accountLabel() {
            if (!this.$store.state.session?.account) return 'IP User';
            return this.$store.state.session.account.type === 1 ? 'Member' : 'IP User';
        },
        alarmCount() {
            return this.$store.state.session?.alarm || '0';
        },
        accountUuid() {
            return this.$store.state.session?.account?.uuid || '';
        },
        accountContributionDocumentLink() {
            return this.accountUuid ? this.contribution_link(this.accountUuid) : '/member/mypage';
        },
        accountContributionDiscussLink() {
            return this.accountUuid ? this.contribution_link_discuss(this.accountUuid) : '/member/mypage';
        },
        accountContributionEditRequestLink() {
            return this.accountUuid ? this.contribution_link_edit_request(this.accountUuid) : '/member/mypage';
        },
        accountContributionAcceptedEditRequestLink() {
            return this.accountUuid ? this.contribution_link_accepted_edit_request(this.accountUuid) : '/member/mypage';
        },
        customMenus() {
            const icons = [
                'fa-solid fa-user-check',
                'fa-solid fa-user-gear',
                'fa-solid fa-user-shield',
                'fa-solid fa-user-plus',
                'fa-solid fa-clock-rotate-left',
                'fa-solid fa-user-tag',
                'fa-solid fa-gear',
                'fa-solid fa-code'
            ];
            return this.$store.state.session?.menus?.map((menu, index) => ({
                title: menu.t,
                to: menu.l,
                icon: icons[index] || 'fa-solid fa-hammer'
            })) || [];
        },
        siteNotice() {
            return this.$store.state.config?.['skin.horizon.sitenotice'] || this.$store.state.config?.['wiki.sitenotice'] || '';
        },
        sidebarFooterHtml() {
            return this.$store.state.config?.['wiki.sidebar_html'] || '';
        },
        footerText() {
            return this.$store.state.config?.['wiki.footer_text'] || '';
        },
        footerTailHtml() {
            return this.$store.state.config?.['skin.horizon.footer_html'] || '';
        },
        pageData() {
            return this.$store.state.page?.data || {};
        },
        pageDocument() {
            return this.pageData.document || null;
        },
        pageTitle() {
            return this.$store.state.page?.title || this.pageDocument?.title || 'horizon';
        },
        pageTitleParts() {
            const title = this.pageTitle;
            const colonIndex = title.indexOf(':');
            if (colonIndex > 0) {
                return {
                    namespace: title.substring(0, colonIndex + 1),
                    text: title.substring(colonIndex + 1)
                };
            }
            return {
                namespace: '',
                text: title
            };
        },
        pageTitleNamespace() {
            return this.pageTitleParts.namespace;
        },
        pageTitleText() {
            return this.pageTitleParts.text;
        },
        pageViewName() {
            return this.$store.state.page?.viewName || '';
        },
        PageRevision() {
            return this.pageData.rev || '';
        },
        pageDate() {
            return this.formatRelativeTime(this.pageData.date, true) || '';
        },
        pageViews() {
            if (typeof this.pageData.views === 'number') return this.pageData.views;
            if (typeof this.pageData.view_count === 'number') return this.pageData.view_count;
            return null;
        },
        pageAclMessage() {
            return this.pageData.edit_acl_message || '';
        },
        pageCopyrightText() {
            const pageData = this.$store.state.page && this.$store.state.page.data;
            return pageData && typeof pageData.copyright_text === 'string' && pageData.copyright_text.trim()
                ? pageData.copyright_text
                : '';
        },
        secondaryAccounts() {
            const session = this.$store.state.session || {};
            const currentUuid = session.account && session.account.uuid;
            return (Array.isArray(session.otherAccounts) ? session.otherAccounts : [])
                .filter(account => account && account.uuid && account.uuid !== currentUuid)
                .map(account => ({
                    uuid: account.uuid,
                    name: account.name || account.uuid,
                    avatar: account.gravatar_url || ''
                }));
        },
        skinConfig() {
            const fontSetting = this.$store.state.localConfig?.['horizon.font'] || 'auto';
            const fontFamily = fontSetting === 'browser' ? 'sans-serif' : '"Pretendard", sans-serif';

            const lightVars = {
                '--horizon-font': fontFamily,
                '--main-color': this.brandColor,
                '--nav-bg-color': this.$store.state.config?.['skin.horizon.nav_bg_color'] || this.brandColor,
                '--menu-button-color': this.brandColor,
                '--sub-color': this.subColor,
                '--light-sub-color': this.brandColor,
                '--border-color': '#e1e8ed',
                '--sub-border-color': '#ccc',
                '--body-color': '#f3f3f3',
                '--text-color': '#373a3c',
                '--text-color-1': '#000',
                '--text-color-2': '#666',
                '--a-color': '#2c6ee8',
                '--nav-color': '#fcfcfc',
                '--bg-color-0': 'white',
                '--bg-color-1': '#fbfbfb',
                '--bg-color-2': '#f7f7f7',
                '--bg-color-3': '#f0f0f0',
                '--button-bg-color': '#fff',
                '--topic-color': '#bbeabb',
                '--link-color': '#090',
                '--light-text-secondary-color': 'var(--text-color-2)',
                '--dark-text-secondary-color': 'var(--text-color-2)',
                '--article-background-color': 'var(--bg-color-0)',
                '--light-article-background-color': 'var(--bg-color-0)',
                '--dark-article-background-color': 'var(--bg-color-0)',
                '--brand-bright-color-1': 'var(--main-color)',
                '--brand-bright-color-2': 'var(--sub-color)',
                '--brand-color-1': 'var(--main-color)',
                '--brand-color-2': 'var(--sub-color)',
                '--light-text-color': 'var(--text-color)',
                '--dark-text-color': 'var(--text-color)',
                '--horizon-ns-bg-color': this.$store.state.config?.['skin.horizon.ns_bg_color'] || '#a8acff',
                '--horizon-ns-text-color': this.$store.state.config?.['skin.horizon.ns_text_color'] || 'inherit',
                '--horizon-ns-size': this.$store.state.config?.['skin.horizon.ns_size'] || '12px',
                '--navbar-logo-size': this.$store.state.config?.['skin.horizon.navbar_logo_size'] || 'auto',
                '--navbar-logo-padding': this.navbarLogoPadding,
                '--navbar-logo-width': this.navbarLogoWidth
            };

            const darkVars = {
                '--horizon-font': fontFamily,
                '--main-color': this.darkBrandColor,
                '--nav-bg-color': this.darkBrandColor,
                '--menu-button-color': '#777',
                '--sub-color': this.darkSubColor,
                '--light-sub-color': '#4d5158',
                '--border-color': '#555',
                '--sub-border-color': '#444',
                '--body-color': '#000',
                '--text-color': '#E0E0E0',
                '--text-color-1': '#ddd',
                '--text-color-2': '#bbb',
                '--a-color': '#A7C8FF',
                '--nav-color': '#ddd',
                '--bg-color-0': '#1c1d1f',
                '--bg-color-1': '#111',
                '--bg-color-2': '#272727',
                '--bg-color-3': '#393939',
                '--button-bg-color': '#000',
                '--topic-color': '#325a56',
                '--link-color': '#0b0',
                '--light-text-secondary-color': 'var(--text-color-2)',
                '--dark-text-secondary-color': 'var(--text-color-2)',
                '--article-background-color': 'var(--bg-color-0)',
                '--light-article-background-color': 'var(--bg-color-0)',
                '--dark-article-background-color': 'var(--bg-color-0)',
                '--brand-bright-color-1': 'var(--main-color)',
                '--brand-bright-color-2': 'var(--sub-color)',
                '--brand-color-1': 'var(--main-color)',
                '--brand-color-2': 'var(--sub-color)',
                '--light-text-color': 'var(--text-color)',
                '--dark-text-color': 'var(--text-color)',
                '--horizon-ns-bg-color': this.$store.state.config?.['skin.horizon.dark.ns_bg_color'] || '#383b40',
                '--horizon-ns-text-color': this.$store.state.config?.['skin.horizon.dark.ns_text_color'] || 'inherit',
                '--horizon-ns-size': this.$store.state.config?.['skin.horizon.ns_size'] || '12px',
                '--navbar-logo-size': this.$store.state.config?.['skin.horizon.navbar_logo_size'] || 'auto',
                '--navbar-logo-padding': this.navbarLogoPadding,
                '--navbar-logo-width': this.navbarLogoWidth
            };

            return this.isDarkMode ? darkVars : lightVars;
        }
    },
    watch: {
        $route() {
            this.isShowACLMessage = false;
            this.$nextTick(() => {
                setTimeout(() => {
                    this.checkToc();
                }, 100);
            });
        },
        '$store.state.page.data': {
            handler() {
                this.$nextTick(() => {
                    setTimeout(() => {
                        this.checkToc();
                    }, 100);
                });
            },
            deep: true
        }
    },
    mounted() {
        this.loadSidebarData();
        this.checkToc();
        this.tocObserver = new MutationObserver(() => {
            this.checkToc();
        });
        this.tocObserver.observe(document.body, { childList: true, subtree: true });
    },
    beforeUnmount() {
        if (this.tocObserver) {
            this.tocObserver.disconnect();
        }
    },
    methods: {
        checkToc() {
            this.hasToc = !!document.getElementById('toc') || !!document.querySelector('.wiki-macro-toc');
        },
        openSettingModal() {
            this.$vfm.show({ component: SettingModal });
        },
        formatRelativeTime(ts, isAbsolute = false) {
            if (!ts) return '';
            
            const now = Math.floor(Date.now() / 1000);
            const diff = now - ts;
            const date = new Date(ts * 1000);

            const Y = date.getFullYear();
            const M = String(date.getMonth() + 1).padStart(2, '0');
            const D = String(date.getDate()).padStart(2, '0');
            const hh = String(date.getHours()).padStart(2, '0');
            const mm = String(date.getMinutes()).padStart(2, '0');
            const ss = String(date.getSeconds()).padStart(2, '0');
            
            const absolute = `${Y}-${M}-${D} ${hh}:${mm}:${ss}`;

            if (isAbsolute) {
                let relative = '';
                if (diff < 10) relative = '방금 전';
                else if (diff < 60) relative = `${diff}초 전`;
                else if (diff < 3600) relative = `${Math.floor(diff / 60)}분 전`;
                else if (diff < 86400) relative = `${Math.floor(diff / 3600)}시간 전`;
                else if (diff < 2678400) {
                    const day = Math.floor(diff / 86400);
                    relative = day === 1 ? '어제' : `${day}일 전`;
                } else {
                    const months = Math.floor(diff / 2592000);
                    const years = Math.floor(diff / 31536000);
                    if (years > 0) relative = `${years}년 전`;
                    else if (months > 0) relative = `${months}개월 전`;
                    else relative = `${Math.floor(diff / 86400)}일 전`;
                }
                return `${absolute} (${relative})`;
            }

            if (diff < 10) return '방금 전';
            if (diff < 60) return `${diff}초 전`;
            
            const min = Math.floor(diff / 60);
            if (min < 60) return `${min}분 전`;
            
            const hour = Math.floor(min / 60);
            if (hour < 24) return `${hour}시간 전`;
            
            const day = Math.floor(hour / 24);
            if (day === 1) return '어제';
            if (day < 31) return `${day}일 전`;
            
            return `${Y}-${M}-${D}`;
        },
        setTheme(theme) {
            this.$store.commit('localConfigSetValue', { key: 'wiki.theme', value: theme });
        },
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
        showEditMessage() {
            if (this.isShowACLMessage) {
                return;
            }

            if (this.pageAclMessage) {
                this.isShowACLMessage = true;
            }
        },
        async loadSidebarData() {
            try {
                const json = await this.internalRequest('/sidebar', { noProgress: true });
                this.sidebarDocument = json?.document || [];
                this.sidebarDiscuss = json?.discuss || [];
                this.recentDiscuss = this.sidebarDiscuss.slice(0, 6).map((item, index) => ({
                    key: `${index}-${item.url}`,
                    to: `/thread/${encodeURIComponent(item.url)}`,
                    time: this.formatSidebarDate(item.date),
                    title: item.topic || ''
                }));
            } catch {
                this.recentDiscuss = [];
                this.sidebarDocument = [];
                this.sidebarDiscuss = [];
            }
        },
        switchAccountRoute(account) {
            const uuid = account && account.uuid;
            return uuid ? `/member/switch_account/${encodeURIComponent(uuid)}` : '/member/login';
        },
        async logoutOtherAccount(uuid) {
            if (!uuid) return;
            await this.internalRequestAndProcess(`/member/logout_other/${encodeURIComponent(uuid)}`, { method: 'POST' });
            if (typeof window !== 'undefined') window.location.reload();
        }
    }
};
</script>