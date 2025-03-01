<script lang="ts">
    import { marked } from "marked";
    import DOMPurify from "dompurify";
    import { Segment } from "@skeletonlabs/skeleton-svelte";
    import Prompt1 from "./prompt1.svelte";
    import Prompt2 from "./prompt2.svelte";

    let { workflow } = $props();

    let md_to_html = (md: string) =>
        DOMPurify.sanitize(marked.parse(md.replace(/^[\u200B\u200C\u200D\u200E\u200F\uFEFF]/, "")) as string);

    let workflow_yaml = $derived.by(() =>
        workflow ? md_to_html(workflow["llm_response"]) : "",
    );
    let workflow_yaml_answer = $derived.by(() =>
        workflow ? md_to_html(workflow["answer"]) : "",
    );

    // Prompt 1
    let score = $state("");

    // Prompt 2
    let validity = $state("");
    let events = $state("");
    let jobs = $state("");
    let steps = $state("");
    let dependencies = $state("");

    const total: any = JSON.parse(localStorage.getItem("total") || "{}");

    export const commit = () => {
        total[workflow["id"]] = {
            score: score,
            validity: validity,
            events: events,
            jobs: jobs,
            steps: steps,
            dependencies: dependencies,
            level: workflow["level"],
        };
        localStorage.setItem("total", JSON.stringify(total));
    };

    export const updateView = () => {
        score = total[workflow["id"]]?.score;
        validity = total[workflow["id"]]?.validity;
        events = total[workflow["id"]]?.events;
        jobs = total[workflow["id"]]?.jobs;
        steps = total[workflow["id"]]?.steps;
        dependencies = total[workflow["id"]]?.dependencies;
    }

    export const getTotal = () => {
        return total;
    }
</script>

{#if workflow}
    <form class="grid grid-cols-2 gap-8 p-4" oninput={commit}>
        <div>
            <p>{workflow["prompt"]}</p>
        </div>
        <div>
            <p>{@html workflow_yaml}</p>
        </div>
        <div class="p-4 card border-surface-200-800 border-[1px]">
            <Prompt1 />
        </div>
        <div class="flex items-center pt-8">
            <div class="mr-4">
                <p>Strongly Disagree</p>
            </div>
            <Segment name="align" bind:value={score}>
                <Segment.Item value="1">1</Segment.Item>
                <Segment.Item value="2">2</Segment.Item>
                <Segment.Item value="3">3</Segment.Item>
                <Segment.Item value="4">4</Segment.Item>
                <Segment.Item value="5">5</Segment.Item>
            </Segment>
            <div class="ml-4">
                <p>Strongly Agree</p>
            </div>
        </div>
        <!-- <div class="card border-surface-200-800 border-[1px]">
            <div class="m-4">
                <Prompt2 />
            </div>
        </div>
        <div>
            <div class="grid">
                <div class="flex flex-row items-center my-2">
                    <div class="mr-4">
                        <p>Validity:</p>
                    </div>
                    <Segment name="align" bind:value={validity}>
                        <Segment.Item value="1">Valid</Segment.Item>
                        <Segment.Item value="2"
                            >Valid with minor errors</Segment.Item
                        >
                        <Segment.Item value="3"
                            >Mostly valid with major errors</Segment.Item
                        >
                        <Segment.Item value="4">Invalid</Segment.Item>
                    </Segment>
                </div>
                <div class="flex flex-row items-center my-2">
                    <div class="mr-4">
                        <p>Requested events present:</p>
                    </div>
                    <Segment name="align" bind:value={events}>
                        <Segment.Item value="1">Yes</Segment.Item>
                        <Segment.Item value="0">No</Segment.Item>
                    </Segment>
                </div>
                <div class="flex flex-row items-center my-2">
                    <div class="mr-4">
                        <p>Requested jobs present:</p>
                    </div>
                    <Segment name="align" bind:value={jobs}>
                        <Segment.Item value="1">Yes</Segment.Item>
                        <Segment.Item value="0">No</Segment.Item>
                    </Segment>
                </div>
                <div class="flex flex-row items-center my-2">
                    <div class="mr-4">
                        <p>Requested steps present:</p>
                    </div>
                    <Segment name="align" bind:value={steps}>
                        <Segment.Item value="1">Yes</Segment.Item>
                        <Segment.Item value="0">No</Segment.Item>
                        <Segment.Item value="-">Not requested</Segment.Item>
                    </Segment>
                </div>
                <div class="flex flex-row items-center my-2">
                    <div class="mr-4">
                        <p>Requested dependencies present:</p>
                    </div>
                    <Segment name="align" bind:value={dependencies}>
                        <Segment.Item value="1">Yes</Segment.Item>
                        <Segment.Item value="0">No</Segment.Item>
                        <Segment.Item value="-">Not requested</Segment.Item>
                    </Segment>
                </div>
            </div>
        </div> -->
    </form>
    <div>
        <p>{@html workflow_yaml_answer}</p>
    </div>
{/if}