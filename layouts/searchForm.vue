<template>
    <form id="searchform" class="form-inline" @submit.prevent="onClickSearch">
        <div class="input-group">
            <a class="btn btn-secondary" id="navbar-random" href="/random" @click.prevent="goRandomPage"><i class="fa-solid fa-shuffle"></i></a>
            <input
                ref="searchInput"
                type="search"
                name="search"
                placeholder="검색하려면 여기에 입력하세요."
                accesskey="f"
                class="form-control"
                id="searchInput"
                autocomplete="off"
                v-model="searchText"
                @keydown.enter.prevent="onClickMove"
                @keydown.esc.prevent="searchText = ''"
            >
            <span class="input-group-btn">
                <button type="submit" name="fulltext" value="search" id="searchSearchButton" class="btn btn-secondary"><span class="fa-solid fa-magnifying-glass"></span></button>
                <button type="button" name="fulltext" value="view" id="searchGoButton" class="btn btn-secondary" @click.prevent="onClickMove"><span class="fa-solid fa-arrow-right"></span></button>
            </span>
        </div>
    </form>
</template>

<script>
import Common from '~/mixins/common';

export default {
    mixins: [Common],
    data() {
        return {
            searchText: ''
        };
    },
    methods: {
        onClickSearch() {
            if (!this.searchText) return;
            this.$router.push(`/Search?q=${encodeURIComponent(this.searchText)}`);
        },
        onClickMove() {
            if (!this.searchText) return;
            this.$router.push(`/Go?q=${encodeURIComponent(this.searchText)}`);
        },
        goRandomPage() {
            this.$router.push('/random');
        }
    }
};
</script>