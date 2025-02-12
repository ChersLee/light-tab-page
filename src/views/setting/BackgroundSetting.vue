<template>
  <div class="background-setting">
    <setting-item :lable="t('background.wallpaper.setting')">
      <a-radio-group v-model:value="background.type" button-style="solid">
        <a-radio :value="BackgroundType.None">{{ t("background.wallpaper.none") }}</a-radio>
        <a-radio :value="BackgroundType.Local">{{ t("background.wallpaper.local") }}</a-radio>
        <a-radio :value="BackgroundType.Bing" :disabled="!isExtension" v-permis="Permis.bing">
          {{ t("background.wallpaper.bing") }}
        </a-radio>
      </a-radio-group>
    </setting-item>

    <setting-item
      v-if="background.type === BackgroundType.Local"
      class="upload-layout"
      :lable="t('background.wallpaper.upload')"
    >
      <a-upload
        class="background-uploader"
        list-type="picture-card"
        :show-upload-list="false"
        :customRequest="uploadBackgroundImage"
        accept="image/*"
        style="width: 100%"
      >
        <img v-if="background.url" :src="background.url" alt="avatar" />
        <div v-else>
          <plus-outlined />
        </div>
      </a-upload>
    </setting-item>

    <template v-if="background.type !== BackgroundType.None">
      <setting-item :lable="t('background.blur')">
        <a-slider v-model:value="background.blur" :max="48" :tipFormatter="toPixel" />
      </setting-item>

      <setting-item :lable="t('background.maskOpacity')">
        <a-slider
          v-model:value="background.maskOpacity"
          :step="0.01"
          :max="1"
          :tipFormatter="toPercent"
        />
      </setting-item>

      <setting-item :lable="t('background.wallpaperDark')" horizontal>
        <a-switch v-model:checked="background.autoOpacity" />
      </setting-item>
    </template>
  </div>
</template>

<script lang="ts" setup>
import { watch } from "vue"
import { PlusOutlined } from "@ant-design/icons-vue"
import { BackgroundSetting, BackgroundType } from "@/types"
import { isExtension, Permis } from "@/plugins/extension"
import { toPixel, toPercent } from "@/utils/format"
import { useStore } from "@/store"
import { SettingActions, SettingMutations } from "@/store/setting"
import { deepComputed } from "@/utils/common"
import { useI18n } from "vue-i18n"

const { t } = useI18n()
const store = useStore()
const background = deepComputed(() => store.state.setting.background, updateBackgroundSetting)

function uploadBackgroundImage(e) {
  store.dispatch(SettingActions.uploadBackgroundImage, e.file)
}

function updateBackgroundSetting(value: BackgroundSetting) {
  store.commit(SettingMutations.updateBackgroundSetting, value)
}

// 监听到背景类型为Bing则拉取壁纸
watch(
  () => background.type,
  type => {
    if (type === BackgroundType.Bing) {
      store.dispatch(SettingActions.loadBingDailyWallpaper)
    }
  }
)
</script>

<style lang="less">
.background-setting {
  .upload-layout {
    margin-top: 8px;

    .background-uploader {
      .ant-upload,
      img {
        width: 100%;
        height: 128px;

        object-fit: cover;
        object-position: center;
      }
    }
  }
}
</style>
