<script setup lang="ts">
import { computed, ref } from "vue";
import { SecLabIcon } from "@seclab-dev/vue";
import type {
  HostScanResult,
  HostState,
  HostVisualState,
} from "@/types/scanner";
import { t } from "@/i18n";

const props = defineProps<{
  ipList: string[];
  ipStates: Record<string, HostState>;
  results: HostScanResult[];
  interactive: boolean;
  collapsed?: boolean;
}>();

const emit = defineEmits<{
  select: [ip: string];
  toggle: [];
}>();

const hostStateVisuals = computed<
  Array<{
    status: HostVisualState;
    icon: string;
    label: string;
  }>
>(() => [
  { status: "pending", icon: "host-idle", label: t.value.legendIdle },
  { status: "scanning", icon: "host-scanning", label: t.value.legendScanning },
  {
    status: "alive-no-port",
    icon: "host-alive",
    label: t.value.legendAliveNoPort,
  },
  {
    status: "alive-with-port",
    icon: "host-open-ports",
    label: t.value.legendAliveWithPort,
  },
  {
    status: "offline",
    icon: "host-unresponsive",
    label: t.value.legendOffline,
  },
]);

const hostStateVisualMap = computed(
  () =>
    Object.fromEntries(
      hostStateVisuals.value.map((item) => [item.status, item]),
    ) as Record<HostVisualState, (typeof hostStateVisuals.value)[number]>,
);

const hoveredIp = ref<string | null>(null);
const hoveredIpDetails = computed(() => {
  if (!hoveredIp.value || !props.interactive) return null;
  const state = props.ipStates[hoveredIp.value];
  const result = props.results.find((item) => item.host === hoveredIp.value);
  return {
    ip: hoveredIp.value,
    status: state?.status ?? "offline",
    ports: state?.ports ?? [],
    detail:
      result?.detail ||
      (state?.status === "offline" ? t.value.unresponsive : ""),
  };
});

/**
 * @description 返回主机状态对应的图标与展示文案。
 */
function getHostVisual(status?: HostVisualState) {
  return hostStateVisualMap.value[status ?? "pending"];
}

/**
 * @description 首行主机悬浮提示改向下展开，避免被面板顶部裁切。
 */
function isTooltipBelow(index: number) {
  return index < 16;
}

function tooltipColumnClass(index: number) {
  if (index % 16 === 0) return "is-tooltip-left";
  if (index % 16 === 15) return "is-tooltip-right";
  return "";
}
</script>

<template>
  <div class="host-grid-panel" :class="{ collapsed }" data-ui="host-grid">
    <button
      type="button"
      class="host-grid-toolbar"
      data-slot="header"
      :aria-expanded="!collapsed"
      @click="emit('toggle')"
    >
      <div class="host-grid-title">
        <SecLabIcon class="panel-chevron" name="chevron-down" :size="16" />
        <SecLabIcon name="network" :size="18" />
        <span>{{ t.segmentHostStatus }}</span>
        <span class="host-grid-count">{{ ipList.length }} IP</span>
      </div>
      <div class="host-grid-legend" :aria-label="t.legendTitle">
        <span
          v-for="item in hostStateVisuals"
          :key="item.status"
          class="legend-item"
          :class="item.status"
        >
          <span class="legend-state-icon-box">
            <SecLabIcon
              class="legend-state-icon"
              :name="item.icon"
              :size="14"
            />
          </span>
          {{ item.label }}
        </span>
      </div>
    </button>

    <div v-if="!collapsed" class="host-grid-scroll" data-slot="content">
      <div class="host-grid">
        <button
          v-for="(ip, index) in ipList"
          :key="ip"
          type="button"
          class="host-grid-cell"
          :class="[
            ipStates[ip]?.status ?? 'pending',
            { 'is-tooltip-below': isTooltipBelow(index) },
            tooltipColumnClass(index),
          ]"
          :aria-label="`${ip} ${getHostVisual(ipStates[ip]?.status).label}`"
          @mouseenter="hoveredIp = ip"
          @mouseleave="hoveredIp = null"
          @focus="hoveredIp = ip"
          @blur="hoveredIp = null"
          @click="emit('select', ip)"
        >
          <SecLabIcon
            class="host-grid-cell-icon"
            :name="getHostVisual(ipStates[ip]?.status).icon"
            :size="16"
          />
          <span
            v-if="hoveredIp === ip && hoveredIpDetails"
            class="host-grid-tooltip"
          >
            <span class="tooltip-ip">{{ hoveredIpDetails.ip }}</span>
            <span class="tooltip-status">
              {{ getHostVisual(hoveredIpDetails.status).label }}
            </span>
            <span v-if="hoveredIpDetails.detail" class="tooltip-detail">
              {{ hoveredIpDetails.detail }}
            </span>
            <span
              v-if="hoveredIpDetails.ports.length > 0"
              class="tooltip-ports"
            >
              {{ t.portListHover(hoveredIpDetails.ports.join(", ")) }}
            </span>
          </span>
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.host-grid-panel {
  flex: 1 1 0;
  min-width: 0;
  min-height: 0;
  display: flex;
  flex-direction: column;
  background: var(--sdl-bg-card);
  border: 1px solid var(--sdl-border-default);
  border-radius: var(--sdl-radius-md);
  overflow: hidden;
}

.host-grid-panel.collapsed {
  flex: 0 0 auto;
}

.host-grid-toolbar {
  width: 100%;
  min-height: 44px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--sdl-space-4);
  padding: var(--sdl-space-2) var(--sdl-space-4);
  background: var(--sdl-bg-panel);
  border-bottom: 1px solid var(--sdl-border-subtle);
  border-top: 0;
  border-left: 0;
  border-right: 0;
  color: inherit;
  font: inherit;
  text-align: left;
  cursor: pointer;
}

.host-grid-title {
  display: flex;
  align-items: center;
  gap: var(--sdl-space-2);
  color: var(--sdl-text-primary);
  font-size: var(--sdl-font-body-sm);
  font-weight: 650;
  white-space: nowrap;
}

.host-grid-title > :not(.panel-chevron):first-of-type {
  color: var(--sdl-primary);
}

.panel-chevron {
  color: var(--sdl-text-muted);
  transition: transform 0.18s ease;
}

.host-grid-toolbar[aria-expanded="false"] .panel-chevron {
  transform: rotate(-90deg);
}

.host-grid-count {
  padding: 2px 6px;
  border-radius: var(--sdl-radius-sm);
  color: var(--sdl-text-muted);
  background: var(--sdl-bg-muted);
  font-family: var(--sdl-font-mono);
  font-size: var(--sdl-font-caption);
  font-weight: 500;
}

.host-grid-legend {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  flex-wrap: wrap;
  gap: var(--sdl-space-3);
}

.legend-item {
  height: 18px;
  display: inline-flex;
  align-items: center;
  gap: var(--sdl-space-1);
  font-size: var(--sdl-font-caption);
  line-height: 18px;
  white-space: nowrap;
}

.legend-item.pending,
.legend-item.pending .legend-state-icon-box {
  color: var(--sdl-text-muted);
}

.legend-item.scanning,
.legend-item.scanning .legend-state-icon-box {
  color: var(--sdl-info);
}

.legend-item.alive-no-port,
.legend-item.alive-no-port .legend-state-icon-box {
  color: var(--sdl-success);
}

.legend-item.alive-with-port,
.legend-item.alive-with-port .legend-state-icon-box {
  color: var(--sdl-primary);
}

.legend-item.offline,
.legend-item.offline .legend-state-icon-box {
  color: var(--sdl-text-subtle);
}

.legend-state-icon-box {
  width: 14px;
  height: 14px;
  flex: 0 0 auto;
  display: grid;
  place-items: center;
  line-height: 1;
  overflow: hidden;
}

.legend-state-icon-box :deep(.sl-icon),
.legend-state-icon-box :deep(svg) {
  display: block;
  width: 14px;
  height: 14px;
  transform: none !important;
  transition: none !important;
  animation: none !important;
}

.host-grid-scroll {
  flex: 1;
  min-height: 0;
  overflow: auto;
  padding: var(--sdl-space-4);
}

.host-grid {
  display: grid;
  grid-template-columns: repeat(16, minmax(18px, 1fr));
  gap: 4px;
  min-width: 420px;
}

.host-grid-cell {
  aspect-ratio: 1;
  min-width: 0;
  display: grid;
  place-items: center;
  position: relative;
  padding: 0;
  border-radius: var(--sdl-radius-xs);
  border: 1px solid var(--sdl-border-subtle);
  background: var(--sdl-bg-card);
  color: var(--sdl-text-muted);
  cursor: pointer;
  transition:
    transform 0.18s ease,
    border-color 0.18s ease,
    background-color 0.18s ease,
    box-shadow 0.18s ease;
}

.host-grid-cell:hover,
.host-grid-cell:focus-visible {
  z-index: 10;
  transform: scale(1.16);
  outline: none;
  box-shadow: var(--sdl-shadow-panel);
}

.host-grid-cell.pending {
  background: var(--sdl-bg-muted);
  border-color: var(--sdl-border-default);
}

.host-grid-cell.scanning {
  color: var(--sdl-info);
  background: var(--sdl-info-soft);
  border-color: var(--sdl-info);
  animation: host-cell-scanning 0.9s ease-in-out infinite alternate;
}

.host-grid-cell.scanning .host-grid-cell-icon {
  animation: host-scanning-pulse 0.9s ease-in-out infinite alternate;
}

.host-grid-cell.alive-no-port {
  color: var(--sdl-success);
  background: var(--sdl-success-soft);
  border-color: color-mix(
    in srgb,
    var(--sdl-success) 58%,
    var(--sdl-border-default)
  );
}

.host-grid-cell.alive-with-port {
  color: var(--sdl-primary);
  background: color-mix(in srgb, var(--sdl-primary) 10%, var(--sdl-bg-card));
  border-color: color-mix(
    in srgb,
    var(--sdl-primary) 58%,
    var(--sdl-border-default)
  );
}

.host-grid-cell.offline {
  color: var(--sdl-text-subtle);
  background: var(--sdl-bg-muted);
  border-style: dashed;
}

.host-grid-cell-icon {
  width: min(68%, 16px);
  height: min(68%, 16px);
}

.host-grid-tooltip {
  position: absolute;
  left: 50%;
  bottom: calc(100% + 8px);
  width: 208px;
  display: flex;
  flex-direction: column;
  gap: var(--sdl-space-1);
  padding: var(--sdl-space-2) var(--sdl-space-3);
  transform: translateX(-50%);
  border: 1px solid var(--sdl-border-default);
  border-radius: var(--sdl-radius-sm);
  background: var(--sdl-bg-panel);
  color: var(--sdl-text-secondary);
  box-shadow: var(--sdl-shadow-panel);
  font-size: var(--sdl-font-caption);
  line-height: 1.45;
  text-align: left;
  pointer-events: none;
  z-index: 100;
}

.host-grid-cell.is-tooltip-below .host-grid-tooltip {
  top: calc(100% + 8px);
  bottom: auto;
}

.host-grid-cell.is-tooltip-left .host-grid-tooltip {
  left: 0;
  transform: none;
}

.host-grid-cell.is-tooltip-right .host-grid-tooltip {
  right: 0;
  left: auto;
  transform: none;
}

.tooltip-ip {
  color: var(--sdl-text-primary);
  font-family: var(--sdl-font-mono);
  font-weight: 700;
}

.tooltip-status {
  color: currentColor;
  font-weight: 650;
}

.tooltip-ports {
  color: var(--sdl-primary);
  font-family: var(--sdl-font-mono);
}

@keyframes host-scanning-pulse {
  from {
    opacity: 0.45;
    transform: scale(0.86);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

@keyframes host-cell-scanning {
  from {
    box-shadow: 0 0 0 0 color-mix(in srgb, var(--sdl-info) 20%, transparent);
    transform: scale(0.94);
  }
  to {
    box-shadow: 0 0 0 3px color-mix(in srgb, var(--sdl-info) 26%, transparent);
    transform: scale(1.03);
  }
}

@media (max-width: 920px) {
  .host-grid-toolbar {
    align-items: flex-start;
    flex-direction: column;
  }

  .host-grid-legend {
    justify-content: flex-start;
  }
}
</style>
