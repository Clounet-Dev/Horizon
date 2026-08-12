<template>
    <setting>
        <setting-item-select label="폰트" ckey="horizon.font" default="auto">
            <option value="auto">자동</option>
            <option value="browser">브라우저 기본</option>
        </setting-item-select>
    </setting>
</template>

<script>
import Common from '~/mixins/common';

import Setting from '~/components/setting';
import SettingItemCheckbox from '~/components/settingItemCheckbox';
import SettingItemInput from '~/components/settingItemInput';
import SettingItemSelect from '~/components/settingItemSelect';

import { isMobile } from '~/utils';

export default {
    mixins: [Common],
    components: {
        Setting,
        SettingItemCheckbox,
        SettingItemInput,
        SettingItemSelect,
    },
    computed: {
        sidebar() {
            return isMobile ? "hide" : "right";
        }
    }
}
</script>