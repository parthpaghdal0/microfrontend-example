<script lang="ts">
  export let preview; // If preview==='true', display dummy data for itemdetail contents
  export let issueid; // must be present unless preview===true
  export let baseapiurl = ""; // Not required for our own app, but is for Salesforce
  export let theme = "light"; // 'light' or 'dark'

  type Issue = {
    id: string;
    name: string;
    description: string;
  };

  let data: Promise<Issue>;

  const getApiEndpoint = (baseUrl, id) => `${baseUrl}/issues/${id}`;

  const getDummyData = async (): Promise<Issue> => {
    return new Promise((resolve) => {
      setTimeout(() => {
        resolve({
          id: "1234",
          name: "Write Your Talk",
          description: "If you don't you'll regret it.",
        });
      }, 3000);
    });
  };

  const getData = async (baseUrl): Promise<Issue> => {
    return await fetch(getApiEndpoint(baseUrl, issueid)).then((response) => {
      if (response.ok) {
        return response.json().then((json) => {
          if (!json.issue) {
            throw Error(json.error || "Response Malformed");
          }
          return json.issue;
        });
      } else {
        console.log("Network response was not ok.");
        throw Error(response.statusText);
      }
    });
  };

  $: {
    if (typeof preview !== "undefined") {
      data =
        preview === "false" && !!issueid ? getData(baseapiurl) : getDummyData();
    }
  }
</script>

<style lang="scss">
  *,
  *:before,
  *:after {
    box-sizing: border-box;
  }

  .background {
    --bg-color: transparent;
    --text-color: white;
    --primary-color: #002e6e;

    &.dark {
      --bg-color: #0a3056;
      --text-color: #333;
      --primary-color: #c0e2ec;
    }

    background-color: var(--bg-color);
    color: var(--text-color);
    padding: 20px;
  }
  .issue {
    background-color: var(--primary-color);
    border: 1px solid;
    border-color: var(--text-color);
    padding: 10px;
    border-radius: 6px;
    margin: 0 auto;
    max-width: 500px;

    &__title {
      margin-bottom: 20px;
      font-size: 23px;
    }
    &__description {
      font-size: 18px;
    }
  }
</style>

<!--- Configure SPA ---->
<svelte:options tag="itemdetail-spa" />

<div class="background {theme}">
  {#await data}
    <div class="issue issue--loading">...loading...</div>
  {:then issue}
    {#if issue}
      <div class="issue">
        <h1 class="issue__title">{issue.name}</h1>
        <p class="issue__description">{issue.description}</p>
      </div>
    {/if}
  {:catch error}
    <div class="issue">
      <p class="issue__description">{error}</p>
    </div>
  {/await}
</div>
