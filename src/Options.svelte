<script lang="ts">
    import Plus from "svelte-material-icons/Plus.svelte";
    import ContentSave from "svelte-material-icons/ContentSave.svelte";
    import Delete from "svelte-material-icons/Delete.svelte";
    import type { Predefined, Settings } from "./global";

    const predefineds: Predefined[] = [
        {
            name: "jira",
            regex: ".*/browse/([^/\?]+).*"
        },
        {
            name: "salesforce",
            regex: ".*/Case/([^/\?]+).*"
        },
        {
            name: "stackoverflow",
            regex: ".*/questions/([^/\?]+).*"
        },
    ];

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
    async function getSettings() {
        settings = await browser.storage.local.get() as any as Settings;
        if (!settings.urlRules) {
            settings.urlRules = defaultUrlRule;
        }
        if (!settings.fallbackType)
        {
            settings.fallbackType = "DL21-05303";
        }
        if (!settings.navisionSupport)
        {
            settings.navisionSupport = false;
        }
    };
    getSettings();

	async function setSettings() {
		browser.storage.local.set(settings as any as browser.storage.StorageObject);
        await getSettings();
	}

    function addSetting() {
		settings.urlRules = settings.urlRules.concat({
            baseUri: "",
            regex: "",
            type: ""
        });
	}

    function removeSetting(index: number) {
		settings.urlRules.splice(index, 1);
        settings.urlRules = settings.urlRules;
	}
</script>
{#if settings}
    <h3>Navision Support (Experimental)</h3>
    <input type="checkbox" class="navision" bind:checked={settings.navisionSupport} />

    {#if settings.navisionSupport}
        <h4>Fallback Type</h4>
        <input bind:value={settings.fallbackType}>
    {/if}

    <h3>BaseUri To Regex</h3>

    {#if settings?.urlRules}
        {#each settings.urlRules as urlRule, index}
            <div class="urlregex">
                <label for={"url_" + index}>URL</label>
                <input id={"url_" + index} bind:value={urlRule.baseUri}>
                <label for={"regex_" + index}>REGEX</label>
                <input id={"regex_" + index} bind:value={urlRule.regex}>
                <select bind:value={urlRule.regex}>
                    {#each predefineds as predefined}
                        <option value={predefined.regex}>
                            {predefined.name}
                        </option>
                    {/each}
                </select>
                {#if settings.navisionSupport}
                    <label for={"type_" + index}>TYPE</label>
                    <input id={"type_" + index} bind:value={urlRule.type}>
                {/if}

                <div class="buttons">
                    <button on:click={() => removeSetting(index)}><Delete /></button>
                </div>
            </div>
        {/each}
    {/if}
    
    <div class="buttons">
        <button on:click={addSetting}><Plus /></button>
        <button on:click={setSettings}><ContentSave /></button>
    </div>
{/if}
<style>
    .buttons {
        display: flex;
    }

    .navision {
        align-self: start;
        margin-bottom: 1em;
    }

    .urlregex {
        display: flex;
        flex-direction: column;
        margin-bottom: 1em;
    }
</style>