<script lang="ts">
import { Suspense, createSuspense } from "@svelte-drama/suspense";
const suspend = createSuspense();
import { Settings, settings } from "@utils/settings/index";
import Parent from "./Parent.svelte";
async function getItem(item) {
    try {
        const response = await fetch(`/api/packages/${item.name}`);
        const data = await response.json();
        return {
            ...data,
            package_name: item.name
        };
    } catch (error) {
        console.error("error: failed to fetch", error);
        return null;
    }
}
async function getAssets() {
    const items = JSON.parse(localStorage.getItem(Settings.PluginSettings.plugins)) || [];
    const promises = items.map(getItem);
    const dataArray = await Promise.all(promises);
    let accumulatedData = dataArray.filter((data) => data !== null);
    accumulatedData = accumulatedData.filter(({ remove }) => remove !== true);
    console.log(JSON.stringify(accumulatedData));
    return accumulatedData;
}
let assets = getAssets();
let compRef = [];
</script>
<Suspense>
  {#snippet children(suspend)}
    {#await suspend(assets) then data}
        {#each Object.entries(data) as [key, asset]}
            <Parent bind:this={compRef[key]}>
                <div class="rounded-3xl bg-navbar-color w-64 flex flex-col cursor-pointer">
                    <div class="w-full">
                        <img src={`/packages/${asset.package_name}/${asset.image}`} alt="plugin" class="aspect-[16/9] rounded-t-3xl"/>
                    </div>
                    <div class="h-2/6 text-center content-center p-3 font-semibold items-center flex flex-col">
                        <div class="text-2xl"> {asset.title} </div>
                        <div class="flex flex-row">
                            <div class="h-8 w-8 cursor-pointer" on:click={() => {settings.marketPlaceSettings.uninstall(asset.type === "page" ? "plugin-page" : "plugin-sw", asset.package_name); compRef[key].$destroy()}}>
                                <svg xmlns="http://www.w3.org/2000/svg" width="100%" height="100%" viewBox="0 0 256 256" {...$$props}>
                                    <path fill="currentColor" d="M216 48h-40v-8a24 24 0 0 0-24-24h-48a24 24 0 0 0-24 24v8H40a8 8 0 0 0 0 16h8v144a16 16 0 0 0 16 16h128a16 16 0 0 0 16-16V64h8a8 8 0 0 0 0-16M112 168a8 8 0 0 1-16 0v-64a8 8 0 0 1 16 0Zm48 0a8 8 0 0 1-16 0v-64a8 8 0 0 1 16 0Zm0-120H96v-8a8 8 0 0 1 8-8h48a8 8 0 0 1 8 8Z" />
                                </svg>
                            </div>
                            <a class="h-8 w-8 cursor-pointer" href={`../catalog/package/${asset.package_name}`}>
                                <svg xmlns="http://www.w3.org/2000/svg" width="100%" height="100%" viewBox="0 0 256 256" {...$$props}>
                                    <path fill="currentColor" d="M192 136v72a16 16 0 0 1-16 16H48a16 16 0 0 1-16-16V80a16 16 0 0 1 16-16h72a8 8 0 0 1 0 16H48v128h128v-72a8 8 0 0 1 16 0m32-96a8 8 0 0 0-8-8h-64a8 8 0 0 0-5.66 13.66L172.69 72l-42.35 42.34a8 8 0 0 0 11.32 11.32L184 83.31l26.34 26.35A8 8 0 0 0 224 104Z" />
                                </svg>
                            </a>
                        </div>
                    </div>
                </div>
            </Parent>
        {/each}
    {/await}
    {/snippet}
</Suspense>
