<template>
  <div class="bottom-box" :class="wrapClass">
    <div class="left-wrap">
      <woot-button
        :title="$t('CONVERSATION.REPLYBOX.TIP_EMOJI_ICON')"
        icon="emoji"
        emoji="😊"
        color-scheme="secondary"
        variant="smooth"
        size="small"
        @click="toggleEmojiPicker"
      />

      <!-- ensure the same validations for attachment types are implemented in  backend models as well -->
      <file-upload
        ref="upload"
        :size="4096 * 4096"
        :accept="allowedFileTypes"
        :multiple="enableMultipleFileUpload"
        :drop="true"
        :drop-directory="false"
        :data="{ direct_upload_url: '/rails/active_storage/direct_uploads', direct_upload: true }"
        @input-file="onDirectFileUpload"
      >
        <woot-button
          v-if="showAttachButton"
          class-names="button--upload"
          :title="$t('CONVERSATION.REPLYBOX.TIP_ATTACH_ICON')"
          icon="attach"
          emoji="📎"
          color-scheme="secondary"
          variant="smooth"
          size="small"
        />
      </file-upload>
      <woot-button
        v-if="enableRichEditor && !isOnPrivateNote"
        icon="quote"
        emoji="🖊️"
        color-scheme="secondary"
        variant="smooth"
        size="small"
        :title="$t('CONVERSATION.REPLYBOX.TIP_FORMAT_ICON')"
        @click="toggleFormatMode"
      />
      <transition name="modal-fade">
        <div
          v-show="$refs.upload && $refs.upload.dropActive"
          class="modal-mask"
        >
          <fluent-icon icon="cloud-backup" />
          <h4 class="page-sub-title">
            {{ $t('CONVERSATION.REPLYBOX.DRAG_DROP') }}
          </h4>
        </div>
      </transition>
    </div>
    <div class="right-wrap">
      <div v-if="isFormatMode" class="enter-to-send--checkbox">
        <input
          :checked="enterToSendEnabled"
          type="checkbox"
          value="enterToSend"
          @input="toggleEnterToSend"
        />
        <label for="enterToSend">
          {{ $t('CONVERSATION.REPLYBOX.ENTER_TO_SEND') }}
        </label>
      </div>
      <woot-button
        size="small"
        :class-names="buttonClass"
        :is-disabled="isSendDisabled"
        @click="onSend"
      >
        {{ sendButtonText }}
      </woot-button>
    </div>
  </div>
</template>

<script>
import FileUpload from 'vue-upload-component';
import * as ActiveStorage from "activestorage";
import {
  hasPressedAltAndWKey,
  hasPressedAltAndAKey,
} from 'shared/helpers/KeyboardHelpers';
import eventListenerMixins from 'shared/mixins/eventListenerMixins';
import { ALLOWED_FILE_TYPES } from 'shared/constants/messages';

import { REPLY_EDITOR_MODES } from './constants';
export default {
  name: 'ReplyTopPanel',
  components: { FileUpload },
  mixins: [eventListenerMixins],
  props: {
    mode: {
      type: String,
      default: REPLY_EDITOR_MODES.REPLY,
    },
    onSend: {
      type: Function,
      default: () => {},
    },
    sendButtonText: {
      type: String,
      default: '',
    },
    showFileUpload: {
      type: Boolean,
      default: false,
    },
    onDirectFileUpload: {
      type: Function,
      default: () => {},
    },
    showEmojiPicker: {
      type: Boolean,
      default: false,
    },
    toggleEmojiPicker: {
      type: Function,
      default: () => {},
    },
    isSendDisabled: {
      type: Boolean,
      default: false,
    },
    setFormatMode: {
      type: Function,
      default: () => {},
    },
    isFormatMode: {
      type: Boolean,
      default: false,
    },
    isOnPrivateNote: {
      type: Boolean,
      default: false,
    },
    enableRichEditor: {
      type: Boolean,
      default: false,
    },
    enterToSendEnabled: {
      type: Boolean,
      default: true,
    },
    enableMultipleFileUpload: {
      type: Boolean,
      default: true,
    },
  },
  mounted() {
    ActiveStorage.start();
  },
  computed: {
    isNote() {
      return this.mode === REPLY_EDITOR_MODES.NOTE;
    },
    wrapClass() {
      return {
        'is-note-mode': this.isNote,
      };
    },
    buttonClass() {
      return {
        warning: this.isNote,
      };
    },
    showAttachButton() {
      return this.showFileUpload || this.isNote;
    },
    allowedFileTypes() {
      return ALLOWED_FILE_TYPES;
    },
  },
  methods: {
    handleKeyEvents(e) {
      if (hasPressedAltAndWKey(e)) {
        this.toggleFormatMode();
      }
      if (hasPressedAltAndAKey(e)) {
        this.$refs.upload.$children[1].$el.click();
      }
    },
    toggleFormatMode() {
      this.setFormatMode(!this.isFormatMode);
    },
    toggleEnterToSend() {
      this.$emit('toggleEnterToSend', !this.enterToSendEnabled);
    },
  },
};
</script>

<style lang="scss" scoped>
.bottom-box {
  display: flex;
  justify-content: space-between;
  padding: var(--space-slab) var(--space-normal);

  &.is-note-mode {
    background: var(--y-50);
  }
}

.left-wrap .button {
  margin-right: var(--space-small);
}

.left-wrap {
  align-items: center;
  display: flex;
}

.right-wrap {
  display: flex;

  .enter-to-send--checkbox {
    align-items: center;
    display: flex;

    input {
      margin: 0;
    }

    label {
      color: var(--s-500);
      font-size: var(--font-size-mini);
    }
  }
}

::v-deep .file-uploads {
  label {
    cursor: pointer;
  }
  &:hover .button {
    background: var(--s-100);
  }
}

.modal-mask {
  color: var(--s-600);
  background: var(--white-transparent);
  flex-direction: column;
}

.page-sub-title {
  color: var(--s-600);
}

.icon {
  font-size: 8rem;
}
</style>
