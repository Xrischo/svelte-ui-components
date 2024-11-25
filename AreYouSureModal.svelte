<script lang="ts">
  import { submitSVG, cancelExitSVG } from ''

  interface Props {
    text: string
    confirmButtonText?: string
    onClose: () => void
    onConfirm: () => void
  }

  let { text, confirmButtonText = 'Confirm', onClose, onConfirm }: Props = $props()
</script>

<div class="modal-content text-center flex flex-col items-center justify-center w-full">
  <div class="modal-header">
    <h5 class="text-lg modal-title mb-8">{text}</h5>
  </div>
  <div class="modal-footer flex flex-row justify-center gap-4">
    <button class="white-button" onclick={onClose}>
      {@html cancelExitSVG}
      Cancel
    </button>
    <button class="success-button" onclick={onConfirm}>
      {@html submitSVG}
      {confirmButtonText}
    </button>
  </div>
</div>
