<template>
  <v-container
    v-if="visible && playbackManager.currentItem && playbackManager.nextItem"
    class="up-next-dialog pointer-events-none pa-lg-6">
    <v-row>
      <v-col
        cols="12"
        offset-md="6"
        md="6"
        offset-lg="8"
        lg="4"
        offset-xl="9"
        xl="3">
        <v-card class="pointer-events-all">
          <v-card-title class="text-h6">
            <i18n path="dialog.upNext.nextItemPlayingIn" tag="span">
              <template #time>
                <span class="text-primary darken-2">
                  {{ $tc('units.time.seconds', currentItemTimeLeft) }}
                </span>
              </template>
            </i18n>
          </v-card-title>
          <v-card-subtitle class="text-truncate text-subtitle-1">
            <span v-if="playbackManager.currentItem.Type === 'Episode'">
              {{ playbackManager.nextItem.SeriesName }} -
              {{
                $t('seasonEpisodeAbbrev', {
                  seasonNumber: playbackManager.nextItem.ParentIndexNumber,
                  episodeNumber: playbackManager.nextItem.IndexNumber
                })
              }}
              <span v-if="$vuetify.display.smAndUp"> - </span> <br v-else />
              {{ playbackManager.nextItem.Name }}
            </span>
            <span v-if="playbackManager.currentItem.Type === 'Movie'">
              {{ playbackManager.nextItem.Name }}
            </span>
          </v-card-subtitle>
          <v-card-text v-if="playbackManager.nextItem?.RunTimeTicks">
            <span>
              {{ getRuntimeTime(playbackManager.nextItem.RunTimeTicks) }}
              <span class="pl-4">
                {{
                  $t('endsAt', {
                    time: getEndsAtTime(playbackManager.nextItem.RunTimeTicks)
                  })
                }}
              </span>
            </span>
          </v-card-text>
          <v-card-actions>
            <v-spacer />
            <v-btn
              class="bg-primary-darken-2"
              variant="flat"
              @click="playbackManager.setNextTrack">
              {{ $t('dialog.upNext.startNow') }}
            </v-btn>
            <v-btn @click="isHiddenByUser = true">
              {{ $t('dialog.upNext.hide') }}
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup lang="ts">
import { computed, ref, watch } from 'vue';
import { playbackManagerStore } from '@/store';
import { ticksToMs, getEndsAtTime, getRuntimeTime } from '@/utils/time';

const emit = defineEmits<{
  (e: 'change', isVisible: boolean): void;
}>();

const playbackManager = playbackManagerStore();

const isHiddenByUser = ref(false);

const currentItemDuration = computed(
  (): number => ticksToMs(playbackManager.currentItem?.RunTimeTicks) / 1000
);
const currentItemTimeLeft = computed(() =>
  Math.round(currentItemDuration.value - (playbackManager.currentTime || 0))
);
const nextUpDuration = computed(() => {
  // If longer than 5 hours, set the duration to 9 minutes
  if (currentItemDuration.value >= 5 * 60 * 60) {
    return 540;
  }
  // If longer than 2 hours, set the duration to 3.5 minutes
  else if (currentItemDuration.value >= 2 * 60 * 60) {
    return 210;
  }
  // If longer than 45 minutes, set the duration to 2 minutes
  else if (currentItemDuration.value >= 45 * 60) {
    return 120;
  }

  return 45;
});
const visible = computed(
  () =>
    !isHiddenByUser.value &&
    playbackManager.currentlyPlayingMediaType === 'Video' &&
    currentItemTimeLeft.value <= nextUpDuration.value
);

watch(
  () => playbackManager.currentItemIndex,
  () => {
    isHiddenByUser.value = false;
  }
);
watch(visible, () => emit('change', visible.value));
</script>
<style lang="scss" scoped>
.up-next-dialog {
  position: absolute;
  bottom: 0;
  right: 0;
  z-index: 9999;
}
</style>
