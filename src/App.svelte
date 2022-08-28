<script>
  import { onMount } from "svelte";
  import { Circle3 } from "svelte-loading-spinners";

  // TODO:justinstitt fetch these from backend (requires getting CWID first, then displaying courses)
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
  <div class="title">
    <h2>Supplemental Instruction</h2>
    <h1>Attendance</h1>
  </div>
  {#if !prefetched}
    <div class="form" style:visibility={prefetched ? "hidden" : "visible"}>
      <input
        placeholder="CWID"
        type="text"
        bind:value={cwid_inp}
        pattern="\d*"
      />
      <div class="select-box">
        <select bind:value={course_inp}>
          <!-- placeholder text -->
          <option value="" selected hidden required> SI COURSE </option>
          {#each courses as some}
            <option value={some}>{some}</option>
          {/each}
        </select>
      </div>
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
  /* title styles */
  .title {
    /* width + 60px padding left and right === 350px width */
    width: 280px;
    padding: 20px 35px;
    background-color: #303030;
    border-radius: 20px;
    display: flex;
    flex-direction: column;
  }
  .title h2,
  .title h1 {
    color: #f8f8f8;
    padding: 0;
    margin: 0;
  }
  .title h2 {
    font-weight: 500;
  }
  .title h1 {
    font-weight: 700;
  }
  /* status/processing styles */
  .good {
    font-size: 5em;
  }
  .loading {
    align-self: center;
    margin-top: 20px;
  }
  h3 {
    color: #f8f8f8;
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
    margin-top: 85px;
    gap: 55px;
  }
  .response {
    color: #f8f8f8;
    font-size: 1.5em;
    font-weight: 500;
  }
  h1 {
    color: #f8f8f8;
    text-transform: uppercase;
    font-size: 2.5em;
    font-weight: 700;
  }

  /* @media (min-width: 640px) {
    main {
      max-width: none;
    }
    .form {
      width: 100%;
    }
  } */

  button {
    border: #ff7900 3px solid;
    background-color: #ff7900;
    color: #f8f8f8;
  }
  button:hover {
    background-color: #303030;
    color: #ff7900;
  }
</style>
