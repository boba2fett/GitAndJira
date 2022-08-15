<script lang="ts">
    import Plus from "svelte-material-icons/Plus.svelte";
    import Pause from "svelte-material-icons/Pause.svelte";
    import Play from "svelte-material-icons/Play.svelte";
    import Delete from "svelte-material-icons/Delete.svelte";
    import ContentCopy from "svelte-material-icons/ContentCopy.svelte";
	import type { Settings, TimeRecord } from "./global";
    import Time from "./Time.svelte";
    import { onMount } from 'svelte';
    import TimeSum from "./TimeSum.svelte";
    import Tooltip from "./Tooltip.svelte";

    const defaultUrlRule = [
        {
            type: "DL21-05299",
            baseUri: "https://jira.d-velop.de",
            regex: ".*/browse/([^/\?]+).*"
        },
        {
            type: "DL21-05301",
            baseUri: "https://dvelop.lightning.force.com/",
            regex: ".*/Case/([^/\?]+).*"
        }
    ];

    let settings: Settings;
    async function getSettings(): Promise<void> {
        settings = await browser.storage.local.get() as any as Settings;
        if (!settings.urlRules) {
            settings.urlRules = defaultUrlRule;
        }
        if (!settings.timeRecordings) {
            settings.timeRecordings = [];
        }
        settings.timeRecordings.sort((a,b) => (a.lastEndTime?.getTime() || a.lastStartTime.getTime()) - (b.lastEndTime?.getTime() || b.lastStartTime.getTime()));
    };
    getSettings();

    async function addTask(task: string): Promise<void> {
        let type = settings.fallbackType;
        if (!task) {
            [task, type] = await getUrlMatch();
        }
        if (!task) {
            return;
        }

        if (settings.timeRecordings.find(x => x.task === task)) {
            return;
        }
        settings.timeRecordings = settings.timeRecordings.concat([{
            task: task,
            timeSeconds: 0,
            lastStartTime: new Date(),
            lastEndTime: null,
            type: type,
        }]);
        await browser.storage.local.set(settings as any as browser.storage.StorageObject);
        taskName = "";
    }

    async function removeTask(index: number): Promise<void> {
		settings.timeRecordings.splice(index, 1);
        settings.timeRecordings = settings.timeRecordings;
        await browser.storage.local.set(settings as any as browser.storage.StorageObject);
	}

    async function pauseTask(index: number): Promise<void> {
		settings.timeRecordings[index].lastEndTime = new Date();
		settings.timeRecordings[index].timeSeconds += (settings.timeRecordings[index].lastEndTime.getTime() - settings.timeRecordings[index].lastStartTime.getTime()) / 1000;
        settings.timeRecordings = settings.timeRecordings;
        await browser.storage.local.set(settings as any as browser.storage.StorageObject);
	}

    async function playTask(index: number): Promise<void> {
		settings.timeRecordings[index].lastEndTime = null;
		settings.timeRecordings[index].lastStartTime = new Date();
        settings.timeRecordings = settings.timeRecordings;
        await browser.storage.local.set(settings as any as browser.storage.StorageObject);
	}

    function generateClipBoardText(timeRecord: TimeRecord): string {
        if (settings?.navisionSupport)
        {
            return (timeRecord.type || settings.fallbackType) + "\t" + timeRecord.task + "\t" + getHours(timeRecord);
        }
        return timeRecord.task;
    }

    function getHours(timeRecord: TimeRecord): string {
        const timeSeconds = timeRecord.timeSeconds + (timeRecord.lastEndTime ? 0 : (new Date().getTime() - timeRecord.lastStartTime.getTime()) / 1000);
        const hours = Math.floor(timeSeconds / 360) / 10;
        return hours.toLocaleString();
    }

    async function executeScript(code: string): Promise<any>
    {
        const tabs = await browser.tabs.query({active: true, currentWindow: true});
		const tab = tabs[0];
        return await browser.tabs.executeScript(tab.id ,{code: code});
    }

	async function getUrlMatch(): Promise<[string, string]> {
		const result = await executeScript("window.location.href;");
		let url = result[0] as any as string;
        if (!url)
        {
            return null;
        }
        const urlRules = settings.urlRules.filter(urlRule => url.startsWith(urlRule.baseUri));
        if (urlRules.length === 0) {
            console.log("No matching url rule found.", urlRules, url);
            return;
        }
        const urlRegex = new RegExp(urlRules[0].regex);
        const matches = url.match(urlRegex);
        if (matches?.length > 1) {
            return [matches[1], urlRules[0].type];
        }
        return null;
    }

    function onKeyPress(e: KeyboardEvent):void {
        if (e.key === "Enter") {
            addTask(taskName);
        }
    }

    let taskName = "";
    let input: HTMLInputElement;
    onMount(function() {
        input.focus();
    });

</script>

<main>
    <div class="task">
        <label class="first" for={"task"}>Task:</label>
        <input id={"task"} bind:value={taskName} bind:this={input} on:keypress={onKeyPress}>
        <button class="last" on:click={() => addTask(taskName)}><Plus /></button>
    </div>
    {#if settings?.timeRecordings}
        <div class="timerecordings">
            {#each settings.timeRecordings as timeRecording, index}
                <div class="timerecording">
                    {#if !timeRecording.lastEndTime}
                        <button class="first" on:click={() => pauseTask(index)}><Pause /></button>
                    {:else}
                        <button class="first" on:click={() => playTask(index)}><Play /></button>
                    {/if}
                    <span class="description">{timeRecording.task}</span>
                    <span class="spacer"></span>
                    <button class="first" on:click={() => navigator.clipboard.writeText(generateClipBoardText(timeRecording))}><ContentCopy /></button>
                    <Tooltip title={getHours(timeRecording)}>
                        <span><Time {timeRecording}/></span>
                    </Tooltip>
                    <button class="last" on:click="{() => removeTask(index)}"><Delete /></button>
                </div>
            {/each}
        </div>
        <div class="sum">
            <span class="spacer"></span>
            <TimeSum timeRecordings={settings.timeRecordings}/>
        </div>
    {/if}
</main>

<style lang="scss">
	main {
		padding: 1em;
		width: 400px;
		height: 280px;
        overflow: hidden;
	}

    .spacer {
        margin: 0 auto;
    }

    .task {
        display: flex;
        flex-direction: row;
        align-items: center;
        vertical-align: baseline;
        input {
            width: 80%;
        }
    }

    .description {
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
    }

    .first {
        margin-right: 1em;
    }

    .last {
        margin-left: 1em;
    }

    .timerecording {
        display: flex;
        flex-direction: row;
        align-items: center;
        vertical-align: baseline;
    }

    .timerecordings {
        overflow-y: scroll;
        height: 80%;
    }

    .sum {
        display: flex;
        flex-direction: row;
        align-items: center;
        vertical-align: baseline;
    }
</style>