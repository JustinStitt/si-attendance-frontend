<script>
  import { onMount } from "svelte";
  import { Circle3 } from "svelte-loading-spinners";

  import BurgerMenu from "./components/svelte-burger-menu/src/BurgerMenu.svelte";
  // TODO:justinstitt fetch these from backend (requires getting CWID first, then displaying courses)
  const courses = ["CPSC-120", "CPSC-121", "CPSC-131", "CPSC-240", "MATH-115", "MATH-250A"];

  let cwid_inp = "";
  let course_inp = "";

  const click = () => {
    signin(cwid_inp, course_inp);
  };

  let can_submit = true;

  const signin = (cwid, course) => {
    if (!cwid.length || !course.length) return;
    if (!can_submit) return;
    console.log("SIGNED IN");
    can_submit = false;
    loading = true;
    response = -1;
    fetch(
      `https://si-attendance-api.vercel.app/signin?cwid=${cwid}&course=${course}`,
      // `http://127.0.0.1:5000/signin?cwid=${cwid}&course=${course}`,
      {
        method: "GET",
        headers: { "Content-Type": "application/json" },
      }
    )
      .then((res) => res.json())
      .then((res) => {
        console.log("response:", res);
        if (res.errmessage.length == 0) {
          response = 1;
          // save to localStorage
          localStorage.setItem("cwid", cwid);
          localStorage.setItem("course", course);
          localStorage.setItem("last-sign-in", Date.now());
        } else {
          response = 0;
          response_message = res["errmessage"];
          console.log(response_message);
        }
        loading = false;
        can_submit = true;
      });
  };
  let response_message = "";
  let response = -1;
  let prefetched = false;
  let loading = false;
  let signed_in_today = false;

  onMount(async () => {
    // check browser storage
    let stored_cwid = localStorage.getItem("cwid");
    let stored_course = localStorage.getItem("course");
    let last_sign_in = localStorage.getItem("last-sign-in") || Date.now();
    let hour_delta = Math.abs(last_sign_in - Date.now()) / 36e5; // 60*60*1000
    if (!stored_cwid && !stored_course) {
      return;
    }

    // otherwise both populated
    console.log(hour_delta);
    if (hour_delta < 18) {
      signed_in_today = true;
      return;
    }
    loading = true;
    prefetched = true;
    can_submit = true;
    signin(stored_cwid, stored_course);
  });

  let show_cache_help_text = false;
  let cache_help_text;
  const clearCache = () => {
    localStorage.clear();
    show_cache_help_text = true;
    setTimeout(() => {
      show_cache_help_text = false;
    }, 2500);
  };

  const signInDifferentCourse = () => {
    signed_in_today = false;
    localStorage.clear(); // theoretically the student can proceed to sign back
    // into the course that we just warned them about, but I CBA solving this
  };

  let nonCWIDStudent = false; // assume student has a CWID
  let non_cwid_status = "";
  let non_cwid_name = "";

  const noCWID = () => {
    nonCWIDStudent = !nonCWIDStudent;
    menu_open = false;
  };

  const nonCWIDSignIn = () => {
    console.log(non_cwid_name, non_cwid_status);
    loading = true;
    fetch(
      // `http://127.0.0.1:5000/noncwidsignin?nonCWIDStatus=${non_cwid_status}&course=${course_inp}&name=${non_cwid_name}`,
      `https://si-attendance-api.vercel.app/noncwidsignin?nonCWIDStatus=${non_cwid_status}&course=${course_inp}&name=${non_cwid_name}`,
      {
        method: "GET",
        headers: { "Content-Type": "application/json" },
      }
    )
      .then((res) => res.json())
      .then((res) => {
        console.log("response:", res);
        response = 1;
        prefetched = true;
        loading = false;
      });
  };

  let cwid_message = "";

  // as per http://www.fullerton.edu/cwid/
  const checkCWIDValid = (_cwid) => {
    can_submit = false;
    if (!_cwid.length) {
      cwid_message = "";
      return;
    }
    if (_cwid.length != 9) {
      cwid_message = "CWID should be 9 digits long!";
      return;
    }
    for (let d of _cwid) {
      if (d < "0" || d > "9") {
        cwid_message = "CWID should only contain digits!";
        return;
      }
    }
    if (_cwid[0] != "8") {
      cwid_message = "CWID must start with 8";
      return;
    }
    cwid_message = "";
    can_submit = true;
  };

  let menu_open;

  let cwid_focused = false;
  const toggleCWIDFocus = () => {
    cwid_focused = !cwid_focused;
  };

  let cwid_warning_icon;
  let show_cwid_warning_message = false;
  $: show_cwid_warning_message = cwid_message.length != 0;
  $: cwid_valid = cwid_message.length == 0;
  $: course_valid = course_inp.length > 0;

  $: checkCWIDValid(cwid_inp);
</script>

<main>
  <div class="title">
    <h2>Supplemental Instruction</h2>
    <h1>Attendance</h1>
  </div>
  {#if !prefetched && !signed_in_today}
    <div class="form" style:visibility={prefetched ? "hidden" : "visible"}>
      {#if !nonCWIDStudent}
        <span>
          <input
            placeholder="CWID"
            type="text"
            bind:value={cwid_inp}
            pattern="\d*"
            style="border: 1px solid aliceblue"
            on:focus={toggleCWIDFocus}
            on:blur={toggleCWIDFocus}
          />
          {#if cwid_message.length == 0 && cwid_inp.length}
            <div class="cwid_warn_icon" class:noborder={1}>✅</div>
          {/if}
          {#if cwid_inp.length > 9 || (cwid_message.length != 0 && !cwid_focused)}
            <div
              bind:this={cwid_warning_icon}
              class="cwid_warn_icon"
              class:noborder={cwid_message.length == 0}
            >
              {cwid_message.length ? "❗" : "✅"}
            </div>
            <div class="cwid_warn">{cwid_message}</div>
          {/if}
        </span>
      {/if}
      {#if nonCWIDStudent}
        <input
          type="text"
          placeholder="Name"
          style="border: 1px solid aliceblue"
          bind:value={non_cwid_name}
        />
        <div class="select-box" style="border: 1px solid aliceblue">
          <select bind:value={non_cwid_status}>
            <option value="" selected hidden required>Status</option>
            <option value="open-enrollment">Open Enrollment Student</option>
            <option value="transfer">Transfer Student</option>
            <option value="other">Other</option>
          </select>
        </div>
      {/if}
      <div class="select-box" style="border: 1px solid aliceblue">
        <select bind:value={course_inp}>
          <!-- placeholder text -->
          <option value="" selected hidden required> SI COURSE </option>
          {#each courses as some}
            <option value={some}>{some}</option>
          {/each}
        </select>
      </div>
      {#if !nonCWIDStudent}
        <button
          on:click={click}
          disabled={!course_valid || !cwid_valid || !can_submit}>Submit</button
        >
      {:else}
        <button
          on:click={nonCWIDSignIn}
          disabled={!course_valid ||
            non_cwid_name == "" ||
            non_cwid_status == ""}>Submit</button
        >
      {/if}
    </div>
  {/if}
  {#if signed_in_today}
    <h3>
      You've Already Signed in Today For <big
        >{localStorage.getItem("course")}</big
      >
    </h3>
    <button on:click={signInDifferentCourse} class="different"
      >Sign in For a Different Course</button
    >
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
      <div class="response">Successfully signed in to {course_inp}!</div>
    {:else}
      <div class="response">{response_message}!</div>
    {/if}
  {/if}
</main>

<BurgerMenu
  class="menu"
  burgerColor="white"
  backgroundColor="#ff7900"
  bind:open={menu_open}
>
  <span class="bmenu-flex">
    {#if nonCWIDStudent}
      <button class="burger-button" on:click={noCWID}>Sign In With CWID</button>
    {:else}
      <button class="burger-button" on:click={noCWID}
        >I Don't Have a CWID!</button
      >
    {/if}
    <button class="burger-button" on:click={clearCache}>Clear Cache</button>
    {#if show_cache_help_text}
      <h4 class="cache-help-text" bind:this={cache_help_text}>cleared✅</h4>
    {/if}
    <h5 class="burger-content" meta="utf-8">
      💙 Made with love by Justin Stitt and Aaron Lieberman. 💜
      <a
        href="https://github.com/JustinStitt/si-attendance-frontend"
        style="color: #705380;">Source Code</a
      >
    </h5>
  </span>
</BurgerMenu>

<style>
  /* title styles */
  .title {
    /* width + 60px padding left and right === 350px width */
    width: 280px;
    padding: 20px 35px;
    margin-top: 20px;
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
    margin-top: 45px;
    gap: 55px;
  }

  .response {
    color: #f8f8f8;
    font-size: 1.5em;
    font-weight: 500;
    text-overflow: show;
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
  button:hover:enabled {
    background-color: #303030;
    color: #ff7900;
  }
  button:disabled {
    border: #303030 3px dotted;
    filter: opacity(0.25);
  }

  .cwid_warn {
    color: aliceblue;
    position: absolute;
    top: 30%;
    left: 50%;
    z-index: 999;
    background-color: rgb(60, 58, 57);
    border: 1px solid aliceblue;
    border-radius: 4px;
    padding: 5px;
  }

  .cwid_warn_icon {
    left: 85%;
    position: relative;
    width: 33px;
    height: 30px;
    margin-top: -13%;
    font-size: 1.3em;
    border: 2px solid aliceblue;
    border-radius: 50%;
    background-color: rgb(58, 58, 58);
  }

  .cwid_warn_icon:hover {
    background-color: rgb(88, 86, 86);
    transform: rotateZ(7deg);
  }

  .noborder {
    border: none;
    background: none;
  }

  .cache-help-text {
    color: aliceblue;
    position: absolute;
    width: 40%;
    top: 55px;
    left: 66%;
  }

  .bmenu-flex {
    height: 93vh;
    display: flex;
    flex-direction: column;
  }

  big {
    color: orange;
  }

  .different {
    font-size: 1.3rem;
  }

  .burger-content {
    color: aliceblue;
    position: bottom;
    margin-top: auto;
  }

  .burger-button {
    background-color: #303030;
    font-size: 1rem;
    width: 100%;
    margin-bottom: 2vh;
    transition: none !important;
  }

  .burger-button:hover {
    background-color: #303040 !important;
    transform: scale(1.1);
  }

  .burger-button:active {
    background-color: darkgray !important;
    transform: scale(1.15);
  }
</style>
