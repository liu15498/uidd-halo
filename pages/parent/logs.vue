<template>
  <div>
    <div>歷史紀錄</div>
    <div>
      <AudioLog
        v-for="audio in audios"
        :key="audio.id"
        :audio="audio"
        @edit-audio="onAudioEdit"
        @delete-audio="onAudioDelete"
      />
    </div>
    <b-modal ref="editModal" title="編輯" @ok="onEditOk">
      <b-form>
        <b-form-group label="內容" label-for="transcript-input">
          <b-form-input
            id="transcript-input"
            v-model="transcript"
            required
          ></b-form-input>
        </b-form-group>
      </b-form>
    </b-modal>
    <b-modal
      ref="deleteModal"
      title="刪除"
      ok-variant="danger"
      ok-title="刪除"
      @ok="onDeleteOk"
    >
      <h4 class="my-4">確認刪除錄音？</h4>
      <p v-if="audio">{{ audio.transcript }}</p>
    </b-modal>
  </div>
</template>

<script lang="ts">
import { Vue, Component, Ref } from 'vue-property-decorator';
import { BModal } from 'bootstrap-vue';
import AudioLog from '~/components/AudioLog.vue';
import { AudioData } from '~/assets/ts/AudioData';

@Component({
  components: {
    AudioLog,
  },
  layout: 'parent',
  async asyncData({ $axios, query }) {
    const audios = await $axios.$get('/api/parent/child_audio', {
      params: { c: query.c },
    });
    return { audios };
  },
})
export default class classname extends Vue {
  transcript: string = '';
  audio: AudioData | null = null;
  audios: AudioData[] = [];
  @Ref() readonly editModal!: BModal;
  @Ref() readonly deleteModal!: BModal;

  onAudioEdit(e: AudioData) {
    this.transcript = e.transcript;
    this.audio = e;
    this.editModal.show();
  }

  onAudioDelete(e: AudioData) {
    this.transcript = e.transcript;
    this.audio = e;
    this.deleteModal.show();
  }

  async onEditOk() {
    this.audio!.transcript = this.transcript;
    await this.$axios.$post('/api/parent/child_audio/edit', {
      id: this.audio!.id,
      transcript: this.transcript,
    });
    const index = this.audios.findIndex((x) => x.id === this.audio!.id);
    this.audios.splice(index, 1, Object.assign({}, this.audio));
  }

  async onDeleteOk() {
    await this.$axios.$post(`/api/parent/child_audio/delete/${this.audio!.id}`);
    const index = this.audios.findIndex((x) => x.id === this.audio!.id);
    this.audios.splice(index, 1);
  }
}
</script>

<style></style>