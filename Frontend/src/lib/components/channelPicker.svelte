<script lang="ts">
    import { createBubbler } from 'svelte/legacy';

    const bubble = createBubbler();
    import { currentChannels, type Channel } from "$lib/scripts/servers";
    import { onDestroy, onMount } from "svelte";
    import { fade, scale } from "svelte/transition";

    interface Props {
        current: Channel;
        message: string;
        type?: string;
        zIndex?: number;
        callback: (id: Channel | undefined) => void;
    }

    let {
        current,
        message,
        type = "TEXT",
        zIndex = 100,
        callback
    }: Props = $props();

    let channels: Channel[] = []
    let sub = currentChannels.subscribe((entities) => {
        for(let entity of entities) {
            if(entity.type != type) continue;
            channels.push(entity);
        }
    })

    onDestroy(() => sub());

    function close() {
        callback(undefined);
    }

</script>

<div out:fade={{duration: 250}} in:fade={{duration: 250}} class="dialog-outer" style="z-index: {zIndex};">
    <div out:scale={{start: 0.8, duration: 250}} in:scale={{start: 0.8, duration: 250}} class="dialog">

        <div class="header">
            <h2>{message}</h2>
            <span onclick={close} onkeydown={bubble('keydown')} class="material-icons icon-medium clickable hover-primary">close</span>
        </div>

        <div class="content">
            <div class="channels">
                {#each channels as channel}
                <div onclick={() => callback(channel)} onkeydown={bubble('keydown')} 
                    class="channel clickable {current.id == channel.id ? 'selected' : ''}">
                    <span class="material-icons icon-primary icon-small">{channel.type == "TEXT" ? "tag" : channel.type == "CATEGORY" ? "folder" : "graphic_eq"}</span>
                    <div class="name">{channel.name}</div>
                </div>
                {/each}
                <div onclick={() => callback({
                    id: "-1",
                    name: "hi",
                    type: "TEXT"
                })} onkeydown={bubble('keydown')} 
                    class="channel clickable {current.id == null ? 'selected' : ''}">
                    <span class="material-icons icon-primary icon-small">close</span>
                    <div class="name">None</div>
                </div>
            </div>
        </div>
    </div>
</div>

<style lang="scss">
    @import '$lib/default.scss';
    @import '$lib/styles/dialog.scss';

    .channels {
        display: flex;
        flex-direction: column;
        gap: 0.5rem;

        .channel {
            padding: 0.2rem;
            background-color: var(--onyx);
            border-radius: 0.5rem;
            display: flex;
            align-items: center;
            transition: 250ms ease;
            gap: 0.2rem;

            &:hover {
                background-color: var(--outer-space);
            }
        }

        .selected {
            background-color: var(--outer-space);
        }
    }

</style>
