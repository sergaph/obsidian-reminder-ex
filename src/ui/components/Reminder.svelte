<script lang="typescript">
  import type { Component } from "obsidian";
  import type { Reminder } from "../../model/reminder";
  import { DateTime, inHours, Later, tomorrow } from "../../model/time";
  import Icon from "./Icon.svelte";
  import Markdown from "./Markdown.svelte";

  export let reminder: Reminder;
  export let component: Component;
  export let onRemindMeLater: (time: DateTime) => void;
  export let onDone: () => void;
  export let onOpenFile: () => void;
  export let onMute: () => void;
  // Do not set initial value so that svelte can render the placeholder `Remind Me Later`.
  let selectedIndex: number;

  export let laters: Array<Later> = [];

  function remindMeLater() {
    const selected = laters[selectedIndex];
    if (selected == null) {
      return;
    }
    onRemindMeLater(selected.later());
  }

  const handlePressKey = (e: KeyboardEvent) => {
    switch (e.key.toLowerCase()) {
      case "d":
        onDone();
        removePressKeyListener();
        break;
      case "m":
        onMute();
        removePressKeyListener();
        break;
      case "t":
        onRemindMeLater(tomorrow()());
        removePressKeyListener();
        break;
      case "1":
        onRemindMeLater(inHours(1)());
        removePressKeyListener();
        break;
      case "3":
        onRemindMeLater(inHours(3)());
        removePressKeyListener();
        break;
      default:
        break;
    }
  };

  function removePressKeyListener() {
    window.removeEventListener("keypress", handlePressKey);
  }

  window.addEventListener("keypress", handlePressKey);

  console.log("OK");
</script>

<main id="reminder-modal">
  <h1>
    <Markdown
      markdown={reminder.title}
      sourcePath={reminder.file}
      {component}
    />
  </h1>
  <span class="reminder-file" on:click={onOpenFile}>
    <Icon icon="link" />
    {reminder.file}
  </span>
  <div class="reminder-actions">
    <button class="mod-cta" on:click={onDone}>
      <Icon icon="check-small" /> Mark as Done
    </button>
    <button on:click={onMute}>
      <Icon icon="minus-with-circle" /> Mute
    </button>
    <select
      class="dropdown later-select"
      bind:value={selectedIndex}
      on:change={remindMeLater}
    >
      <!-- placeholder -->
      <option selected disabled hidden>Remind Me Later</option>
      <!-- options -->
      {#each laters as later, i}
        <option value={i} selected={selectedIndex === i}>{later.label}</option>
      {/each}
    </select>
  </div>
  <div class="reminder-hotkeys-info">
    <span>t — tomorrow,</span>
    <span>1 - 1 hour,</span>
    <span>3 - 3 hour,</span>
    <span>d - done,</span>
    <span>m - mute</span>
  </div>
</main>

<style>
  main {
    padding: 1em;
    margin: 0 auto;
  }

  .reminder-actions {
    margin-top: 1rem;
    display: flex;
    gap: 0.5rem;
  }

  .reminder-file {
    color: var(--text-muted);
    cursor: pointer;
  }

  .reminder-file:hover {
    color: var(--text-normal);
    text-decoration: underline;
  }

  .later-select {
    font-size: 14px;
  }

  .reminder-hotkeys-info {
    color: var(--text-faint);
    font-size: xx-small;
    margin-top: 1rem;
    display: flex;
    gap: 0.5rem;
  }
</style>
