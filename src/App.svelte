<script>
  import { onMount } from "svelte";
  import { Circle3 } from "svelte-loading-spinners";

  const courses = ["CPSC-120", "CPSC-121", "CPSC-131", "CPSC-240", "CPSC-375"];

  let cwid_inp = "";
  let course_inp = "";

  const click = () => {
    signin(cwid_inp, course_inp);
  };

  const signin = (cwid, course) => {
    if (!cwid.length || !course.length) return;
    loading = true;
    response = -1;
    fetch(
      `https://si-attendance-api.vercel.app/signin?cwid=${cwid}&course=${course}`,
      {
        method: "GET",
        headers: { "Content-Type": "application/json" },
      }
    ).then((res) => {
      console.log("response:", res);
      if (res.ok) {
        response = 1;
        // save to localStorage
        localStorage.setItem("cwid", cwid);
        localStorage.setItem("course", course);
      } else response = 0;
      loading = false;
    });
  };

  let response = -1;
  let prefetched = false;
  let loading = false;

  onMount(async () => {
    // check browser storage
    let stored_cwid = localStorage.getItem("cwid");
    let stored_course = localStorage.getItem("course");
    if (!stored_cwid && !stored_course) return;
    // otherwise both populated
    loading = true;
    prefetched = true;
    signin(stored_cwid, stored_course);
  });
</script>

<main>
  <h1>SI Attendance</h1>
  {#if !prefetched}
    <div class="form" style:visibility={prefetched ? "hidden" : "visible"}>
      <input placeholder="CWID" type="text" bind:value={cwid_inp} />
      <select bind:value={course_inp}>
        {#each courses as some}
          <option value={some}>{some}</option>
        {/each}
      </select>
      <button on:click={click}>Submit</button>
    </div>
  {/if}
  {#if loading}
    <span class="loading">
      <Circle3 size="60" color="#FF3E00" unit="px" duration="1s" />
    </span>
    {#if prefetched}
      <h3>Grabbing stored credentials...</h3>
    {/if}
  {/if}
  {#if !loading && response == 1}
    <div class="good">✅</div>
  {/if}
  {#if !loading && response == 0}
    <div class="good">❌</div>
  {/if}
  {#if response != -1}
    {#if response == 1}
      <div class="response">Successfully signed in!</div>
    {:else}
      <div class="response">Something went wrong!</div>
    {/if}
  {/if}
</main>

<style>
  .good {
    font-size: 5em;
  }
  .loading {
    align-self: center;
    margin-top: 20px;
  }

  h3 {
    color: aliceblue;
  }

  main {
    display: flex;
    align-items: center;
    flex-direction: column;
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  .form {
    display: flex;
    flex-direction: column;
    margin-top: 80px;
    width: 100%;
  }

  .response {
    color: aliceblue;
    font-size: 1.5em;
    font-weight: 500;
  }

  h1 {
    color: aliceblue;
    text-transform: uppercase;
    font-size: 2.5em;
    font-weight: 700;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
    .form {
      width: 40%;
    }
  }

  button {
    background-color: rgb(122, 166, 128);
    color: aliceblue;
  }
</style>
