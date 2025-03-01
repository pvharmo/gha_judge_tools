<script lang="ts">
    import { onDestroy } from "svelte";
    import { AppBar, FileUpload, Segment } from "@skeletonlabs/skeleton-svelte";
    import IconUpload from 'lucide-svelte/icons/upload';
    import IconDownload from 'lucide-svelte/icons/download';
    import RatingView from "$lib/components/RatingView.svelte";
    import CompareView from "$lib/components/CompareView.svelte";

    interface Workflow {
        id: string;
        level: string;
        judgement: string;
        answer: string;
        prompt: string;
        llm_response: string;
        bleu_score: number;
        judge_score: number;
    }

    let workflows: undefined | Workflow[] = $state(
        JSON.parse(localStorage.getItem("workflows") || "{}"),
    );

    let all_workflows: undefined | Workflow[] = $state(
        JSON.parse(localStorage.getItem("workflows") || "{}"),
    );

    let page = $state(0);
    let workflow: Workflow = $derived(
        workflows
            ? workflows[page]
            : {
                id: "",
                level: "",
                judgement: "",
                answer: "",
                prompt: "",
                llm_response: "",
                bleu_score: 0,
                judge_score: 0,
            },
    );

    let load_json = async (e: any) => {
        const file = e.acceptedFiles[0];
        const text = await file.text();
        const json = JSON.parse(text);

        json.sort(() => Math.random() - 0.5)

        localStorage.setItem("workflows", JSON.stringify(json.slice(0, 100)));

        const filtered_json = json.slice(950, 950*2)
        .filter((w: any) => w["llm_response"].trim().endsWith("```"))
        .sort((a: any, b: any) => {
            if (a["id"] < b["id"]) return 1;
            if (a["id"] > b["id"]) return -1;
            if (a["level"] < b["level"]) return 1;
            if (a["level"] > b["level"]) return -1
            return 0;
        });

        workflows = filtered_json;
        all_workflows = filtered_json;
    };

    function handleShortcut(event: KeyboardEvent) {
        const direction = event.key === "+" ? 1 : event.key === "-" ? -1 : 0;
        event.preventDefault();
        // commit();
        if (
            workflows &&
            page + direction < workflows.length &&
            page + direction >= 0
        ) {
            view == "rating" && rating_view?.commit();
            page += direction;
            view == "rating" && rating_view?.updateView();
        }
    }
    document.addEventListener("keyup", handleShortcut);

    onDestroy(() => {
        document.removeEventListener("keyup", handleShortcut);
    });

    const downloadScores = () => {
        const blob = new Blob([JSON.stringify(rating_view?.getTotal())], {
            type: "application/json",
        });
        const url = window.URL.createObjectURL(blob);
        window.open(url);
        URL.revokeObjectURL(url);
    };

    let rating_view: RatingView | null = $state(null);

    // svelte-ignore state_referenced_locally
    let workflow_id = $state(workflow["id"]);

    let edit_id = $state(false);

    let find_id = () => {
        page = workflows?.findIndex((w) => w["id"] === workflow_id) || page;
        edit_id = false;
    };

    let filter_level = (level: string) => {
        if (level === "") {
            workflows = all_workflows;
        } else {
            workflows = all_workflows?.filter((w) => w["level"] === level);
        }
    };

    let view = $state("rating");
</script>

<AppBar>
    {#snippet lead()}
    {#if edit_id}
    <input class="input" type="text" bind:value={workflow_id} />
    <button class="btn preset-filled" onclick={find_id}>
        Find
    </button>
    {:else}
        <button class="btn preset-tonal-surface" onclick={() => edit_id = true}>
            {workflow ? workflow["id"] : ""}
        </button>
    {/if}
    <select class="select" onchange={(e) => filter_level((e.target as HTMLSelectElement).value)}>
        <option value="">All</option>
        <option value="level1">1</option>
        <option value="level2">2</option>
        <option value="level3">3</option>
        <option value="level4">4</option>
        <option value="level5">5</option>
    </select>
    {/snippet}
    <div class="flex items-center justify-center pt-2" style="font-size: 1.2em;">
        {workflow && workflow["level"]} -- BLEU score {workflow && workflow["bleu_score"].toFixed(2)}
        {#if view != "rating"}
        -- LLM Judge {workflow && workflow["judge_score"]}
        {/if}
    </div>

    {#snippet trail()}
        <div class="flex items-center">
            <p style="font-size: 1.2em;">{page + 1} / {workflows?.length}</p>
        </div>
        <div class="btn-group preset-outlined-surface-200-800 flex-col p-2 md:flex-row">
            <button class="btn btn-sm {view == "rating" && "preset-filled"}" onclick={() => view = "rating"}>
                Rating
            </button>
            <button class="btn btn-sm {view == "compare" && "preset-filled"}" onclick={() => view = "compare"}>
                Compare
            </button>
        </div>
        <FileUpload name="example-button" accept="application/json" onFileChange={load_json} maxFiles={1}>
          <button class="btn btn-icon">
            <IconUpload class="size-4" />
          </button>
        </FileUpload>
        <button class="btn btn-icon " onclick={downloadScores}>
            <IconDownload class="size-4" />
        </button>
    {/snippet}
</AppBar>

{#if view == "rating"}
<RatingView bind:this={rating_view} {workflow} />
{:else}
<CompareView workflow={workflow} />
{/if}