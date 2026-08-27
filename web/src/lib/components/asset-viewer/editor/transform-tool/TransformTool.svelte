<script lang="ts">
  import { shortcuts } from '$lib/actions/shortcut';
  import { type ColorAdjustment, transformManager } from '$lib/managers/edit/transform-manager.svelte';
  import { Button, HStack, IconButton } from '@immich/ui';
  import { mdiFlipHorizontal, mdiFlipVertical, mdiRotateLeft, mdiRotateRight } from '@mdi/js';
  import { t } from 'svelte-i18n';

  interface AspectRatioOption {
    label: string;
    value: string;
    width?: number;
    height?: number;
    isFree?: boolean;
  }

  type ColorAdjustmentLabel =
    | 'editor_brightness'
    | 'editor_contrast'
    | 'editor_saturation'
    | 'editor_exposure'
    | 'editor_temperature'
    | 'editor_tint'
    | 'editor_black_point'
    | 'editor_white_point'
    | 'editor_sharpness';

  const aspectRatios: AspectRatioOption[] = [
    { label: $t('crop_aspect_ratio_free'), value: 'free', isFree: true },
    { label: $t('crop_aspect_ratio_original'), value: 'original', width: 24, height: 18 },
    { label: '5:4', value: '5:4', width: 22, height: 18 },
    { label: '4:5', value: '4:5', width: 18, height: 22 },
    { label: '4:3', value: '4:3', width: 24, height: 18 },
    { label: '3:4', value: '3:4', width: 18, height: 24 },
    { label: '3:2', value: '3:2', width: 24, height: 16 },
    { label: '2:3', value: '2:3', width: 16, height: 24 },
    { label: '16:9', value: '16:9', width: 24, height: 14 },
    { label: '9:16', value: '9:16', width: 14, height: 24 },
    { label: $t('crop_aspect_ratio_square'), value: '1:1', width: 20, height: 20 },
  ];

  const colorAdjustments: { id: ColorAdjustment; label: ColorAdjustmentLabel; min: number; max: number }[] = [
    { id: 'brightness', label: 'editor_brightness', min: -100, max: 100 },
    { id: 'contrast', label: 'editor_contrast', min: -100, max: 100 },
    { id: 'saturation', label: 'editor_saturation', min: -100, max: 100 },
    { id: 'exposure', label: 'editor_exposure', min: -100, max: 100 },
    { id: 'temperature', label: 'editor_temperature', min: -100, max: 100 },
    { id: 'tint', label: 'editor_tint', min: -100, max: 100 },
    { id: 'blackPoint', label: 'editor_black_point', min: -100, max: 100 },
    { id: 'whitePoint', label: 'editor_white_point', min: -100, max: 100 },
    { id: 'sharpness', label: 'editor_sharpness', min: 0, max: 100 },
  ];

  let isRotated = $derived(transformManager.normalizedRotation % 180 !== 0);

  function rotatedRatio(ratio: AspectRatioOption): string {
    if (ratio.value === 'free') {
      return ratio.value;
    }

    if (isRotated) {
      let [width, height] = ratio.value.split(':', 2);
      return `${height}:${width}`;
    }
    return ratio.value;
  }

  function ratioSelected(ratio: AspectRatioOption): boolean {
    const currentRatioRotated = rotatedRatio(ratio);

    return transformManager.cropAspectRatio === currentRatioRotated;
  }

  function selectAspectRatio(ratio: AspectRatioOption) {
    let appliedRatio;
    if (ratio.value === 'original') {
      const { width, height } = transformManager.cropImageSize;
      appliedRatio = `${width}:${height}`;
    } else {
      appliedRatio = rotatedRatio(ratio);
    }

    transformManager.setAspectRatio(appliedRatio);
  }

  async function rotateImage(degrees: number) {
    await transformManager.rotate(degrees);
  }

  function mirrorImage(axis: 'horizontal' | 'vertical') {
    transformManager.mirror(axis);
  }

  function setColorAdjustment(type: ColorAdjustment, event: Event) {
    transformManager.setColorAdjustment(type, Number((event.currentTarget as HTMLInputElement).value));
  }
</script>

<svelte:document
  use:shortcuts={[
    { shortcut: { key: ']' }, onShortcut: () => rotateImage(90) },
    { shortcut: { key: '[' }, onShortcut: () => rotateImage(-90) },
  ]}
/>

<div class="mt-3 px-4">
  <div class="mt-2 flex h-10 w-full items-center justify-between text-sm">
    <h2>{$t('editor_orientation')}</h2>
  </div>
  <HStack>
    <IconButton
      class="w-full"
      size="small"
      aria-label={$t('editor_rotate_left')}
      icon={mdiRotateLeft}
      onclick={() => rotateImage(-90)}
    />
    <IconButton
      class="w-full"
      size="small"
      aria-label={$t('editor_rotate_right')}
      icon={mdiRotateRight}
      onclick={() => rotateImage(90)}
    />
    <IconButton
      class="w-full"
      size="small"
      aria-label={$t('editor_flip_horizontal')}
      icon={mdiFlipHorizontal}
      onclick={() => mirrorImage('horizontal')}
    />
    <IconButton
      class="w-full"
      size="small"
      aria-label={$t('editor_flip_vertical')}
      icon={mdiFlipVertical}
      onclick={() => mirrorImage('vertical')}
    />
  </HStack>

  <div class="mt-6 flex h-10 w-full items-center justify-between text-sm">
    <h2>{$t('crop')}</h2>
  </div>

  <!-- Aspect Ratio Grid -->
  <div class="mb-4 grid grid-cols-2">
    {#each aspectRatios as ratio (ratio.value)}
      <HStack>
        <Button
          class="m-2 size-14"
          shape="round"
          onclick={() => selectAspectRatio(ratio)}
          aria-label={ratio.label}
          color={ratioSelected(ratio) ? 'primary' : 'secondary'}
          variant={ratioSelected(ratio) ? 'filled' : 'outline'}
        >
          {#if ratio.isFree}
            <!-- Free crop icon with dashed border -->
            <div
              class="size-6 shrink-0 rounded-xs border-2 border-dashed {ratioSelected(ratio)
                ? 'border-black'
                : 'border-white'}"
            ></div>
          {:else}
            <!-- Aspect ratio box -->
            <div
              class="shrink-0 rounded-xs border-2 {ratioSelected(ratio) ? 'border-black' : 'border-white'}"
              style="width: {ratio.width}px; height: {ratio.height}px;"
            ></div>
          {/if}
        </Button>
        <span class="text-sm text-white">{ratio.label}</span>
      </HStack>
    {/each}
  </div>

  <div class="mt-6 flex h-10 w-full items-center justify-between text-sm">
    <h2>{$t('editor_adjustments')}</h2>
  </div>
  {#each colorAdjustments as adjustment (adjustment.id)}
    <label class="mb-3 block text-sm text-white" for={adjustment.id}>
      {$t(adjustment.label)}: {transformManager[adjustment.id]}
      <input
        class="mt-2 w-full"
        id={adjustment.id}
        type="range"
        min={adjustment.min}
        max={adjustment.max}
        value={transformManager[adjustment.id]}
        oninput={(event) => setColorAdjustment(adjustment.id, event)}
      />
    </label>
  {/each}
</div>
