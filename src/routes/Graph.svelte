<script lang="ts">
  import { onMount } from "svelte";

  let viz: any;
  let selectedNode: {
    raw: any;
    caption: string;
    group: string;
    id: string;
    properties?: Record<string, any>;
  } | null = null;

  type NonFlatLabelConfig = {
    caption: string;
    size?: number;
  };

  type NodeInfo = {
    caption: string;
    group: string;
    id: string;
    properties?: Record<string, any>;
  };

  type NeovisConfig = {
    containerId: string;
    neo4j: {
      serverUrl: string;
      serverUser: string;
      serverPassword: string;
    };
    labels: Record<string, NonFlatLabelConfig>;
    relationships: Record<string, { thickness: string; caption: boolean }>;
    initialCypher: string;
    arrows: boolean;
    hierarchical: boolean;
    interaction: {
      zoomView: boolean;
      dragNodes: boolean;
    };
    nonFlat: true;
  };

  onMount(async () => {
    const Neovis = (await import("neovis.js/dist/neovis.js")).default;

    const config: NeovisConfig = {
      containerId: "viz",
      neo4j: {
        serverUrl: import.meta.env.VITE_NEO4J_SERVER_URL,
        serverUser: import.meta.env.VITE_NEO4J_SERVER_USER,
        serverPassword: import.meta.env.VITE_NEO4J_SERVER_PASSWORD,
      },
      labels: {
        Movie: { caption: "title" },
        Person: { caption: "name" },
      },
      relationships: {
        ACTED_IN: {
          thickness: "roles",
          caption: true,
        },
      },
      initialCypher: "MATCH (p:Person)-[r:ACTED_IN]->(m:Movie) RETURN p,r,m",
      arrows: true,
      hierarchical: false,
      interaction: {
        zoomView: true,
        dragNodes: true,
      },
      nonFlat: true,
    };

    try {
      viz = new Neovis(config);

      viz.registerOnEvent(
        "clickNode",
        (event: {
          node: { caption: string; group: string; id: string; raw: string };
        }) => {
          selectedNode = event.node;
        }
      );

      viz.render();
    } catch (error) {
      console.error("Error creating NeoVis instance:", error);
    }
  });
</script>

<div
  class="flex flex-row flex-nowrap content-between justify-center items-center gap-3"
>
  <div>
    <div id="viz" class="w-[800px] h-[600px] border border-black"></div>
  </div>
  <div class="w-[200px]">
    <h2 class="text-lg font-semibold mb-2">Node Details</h2>
    {#if selectedNode !== null}
      <div>
        <h3 class="text-md font-semibold mb-1">Group:</h3>
        <p>{selectedNode.group}</p>
        <h3 class="text-md font-semibold mb-1">ID:</h3>
        <p>{selectedNode.id}</p>
        {#if selectedNode.raw.properties}
          {#each Object.entries(selectedNode.raw.properties) as [key, value]}
            <h3 class="text-md font-semibold mb-1">{key}:</h3>
            <p>{value}</p>
          {/each}
        {/if}
      </div>
    {:else}
      <p class="italic text-gray-600">
        Select a node in the graph to view details.
      </p>
    {/if}
  </div>
</div>
