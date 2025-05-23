<script lang="ts">
    import { getModuleToggles } from "src/ts/process/modules";
    import { DBState, MobileGUI } from "src/ts/stores.svelte";
    import { parseToggleSyntax } from "src/ts/util";
    import CheckInput from "../UI/GUI/CheckInput.svelte";
    import { language } from "src/lang";
    import type { character, groupChat } from "src/ts/storage/database.svelte";
    import SelectInput from "../UI/GUI/SelectInput.svelte";
    import OptionInput from "../UI/GUI/OptionInput.svelte";
    import TextInput from "../UI/GUI/TextInput.svelte";

    interface Props {
        chara?: character|groupChat
        noContainer?: boolean
    }

    let { chara = $bindable(), noContainer }: Props = $props();

    let parsedKv = $derived(parseToggleSyntax(DBState.db.customPromptTemplateToggle + getModuleToggles()))

</script>

{#snippet toggles(reverse: boolean = false)}
    {#each parsedKv as toggle}
        {#if toggle.type === 'select'}
            <div class="flex gap-2 mt-2 items-center" class:flex-row-reverse={!reverse} class:justify-end={!reverse}>
                <span>{toggle.value}</span>

                <SelectInput className="w-32" bind:value={DBState.db.globalChatVariables[`toggle_${toggle.key}`]}>
                    {#each toggle.options as option, i}
                        <OptionInput value={i.toString()}>{option}</OptionInput>
                    {/each}
                </SelectInput>
            </div>
        {:else if toggle.type === 'text'}
            <div class="flex gap-2 mt-2 items-center" class:flex-row-reverse={!reverse} class:justify-end={!reverse}>
                <span>{toggle.value}</span>
                <TextInput className="w-32" bind:value={DBState.db.globalChatVariables[`toggle_${toggle.key}`]} />
            </div>
        {:else}
            <div class="flex mt-2 items-center">
                <CheckInput check={DBState.db.globalChatVariables[`toggle_${toggle.key}`] === '1'} reverse={reverse} name={toggle.value} onChange={() => {
                    DBState.db.globalChatVariables[`toggle_${toggle.key}`] = DBState.db.globalChatVariables[`toggle_${toggle.key}`] === '1' ? '0' : '1'
                }} />
            </div>
        {/if}
    {/each}
{/snippet}

{#if !noContainer && parsedKv.length > 4}
    <div class="h-48 border-darkborderc p-2 border rounded flex flex-col items-start mt-2 overflow-y-auto">
        <div class="flex mt-2 items-center w-full" class:justify-end={$MobileGUI}>
            <CheckInput bind:check={DBState.db.jailbreakToggle} name={language.jailbreakToggle} reverse />
        </div>
        {@render toggles(true)}
        {#if DBState.db.supaModelType !== 'none' || DBState.db.hanuraiEnable || DBState.db.hypaV3}
            <div class="flex mt-2 items-center w-full" class:justify-end={$MobileGUI}>
                <CheckInput bind:check={chara.supaMemory} reverse name={DBState.db.hypaV3 ? language.ToggleHypaMemory : DBState.db.hanuraiEnable ? language.hanuraiMemory : DBState.db.hypaMemory ? language.ToggleHypaMemory : language.ToggleSuperMemory}/>
            </div>
        {/if}
    </div>
{:else}
    <div class="flex mt-2 items-center">
        <CheckInput bind:check={DBState.db.jailbreakToggle} name={language.jailbreakToggle}/>
    </div>
    {@render toggles()}
    {#if DBState.db.supaModelType !== 'none' || DBState.db.hanuraiEnable || DBState.db.hypaV3}
        <div class="flex mt-2 items-center">
            <CheckInput bind:check={chara.supaMemory} name={DBState.db.hypaV3 ? language.ToggleHypaMemory : DBState.db.hanuraiEnable ? language.hanuraiMemory : DBState.db.hypaMemory ? language.ToggleHypaMemory : language.ToggleSuperMemory}/>
        </div>
    {/if}
{/if}