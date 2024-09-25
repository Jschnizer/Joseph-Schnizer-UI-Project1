<script defer>
  import { onMount, tick } from "svelte";

  let currentTime = "";
  let uptime = "0s";
  let startTime = Date.now();
  let showSettings = false;
  let themeColor = "green";

  let title = "";
  let score = "";
  let result = "Win";
  let opponent = "";
  let surface = "Hard";
  let notes = "";
  let opponentUTR = "";
  let myUTR = "";

  let submissions = [];
  let isSubmitted = false;
  let isSaved = false;

  let myStats = {
    totalWins: 0,
    totalLosses: 0,
    bestSurface: "",
    averageOpponentUTR: 0,
    matchesPlayed: 0,
    winPerc: 0,
  };

  let selectedSubmission = null;

  let selectedTab = "Entries";

  // Visibility states for each field
  let showOpponent = true;
  let showSurface = true;
  let showNotes = true;
  let showOpponentUTR = true;
  let showMyUTR = true;

  // Goal set value
  let goalValue = "";

  function updateTime() {
    const now = new Date();
    let hours = now.getHours();
    const minutes = now.getMinutes().toString().padStart(2, "0");
    const ampm = hours >= 12 ? "PM" : "AM";
    hours = hours % 12 || 12;
    currentTime = `${hours.toString().padStart(2, "0")}:${minutes} ${ampm} | `;

    const elapsed = Math.floor((Date.now() - startTime) / 1000);
    const hoursElapsed = Math.floor(elapsed / 3600);
    const minutesElapsed = Math.floor((elapsed % 3600) / 60);
    const secondsElapsed = elapsed % 60;
    uptime = `Uptime: ${hoursElapsed}h ${minutesElapsed}m ${secondsElapsed}s | `;
  }

  function submitForm() {
    const submission = {
      title,
      score,
      result,
      opponent: showOpponent ? opponent : "",
      surface: showSurface ? surface : "",
      notes: showNotes ? notes : "",
      opponentUTR: showOpponentUTR ? opponentUTR : "",
      myUTR: showMyUTR ? myUTR : "",
      timestamp: new Date().toLocaleString(),
    };
    submissions = [...submissions, submission];
    updateStatsSection();
    isSubmitted = true;
    resetForm();
    setTimeout(() => {
      isSubmitted = false;
    }, 1000);
  }

  function resetForm() {
    title = "";
    score = "";
    result = "Win";
    opponent = "";
    surface = "Hard";
    notes = "";
    opponentUTR = "";
    myUTR = "";
  }

  function selectSubmission(submission) {
    selectedSubmission = submission;
  }

  function saveChanges() {
    isSaved = true;
    updateStatsSection();
    setTimeout(() => {
      isSaved = false;
    }, 1000);
  }

  function toggleSettings() {
    showSettings = !showSettings;
  }

  function changeTheme(color) {
    themeColor = color;
    showSettings = false;
  }

  onMount(() => {
    updateTime();
    const interval = setInterval(updateTime, 1000);
    return () => clearInterval(interval);
  });

  // Function to close settings when clicked outside
  function handleClickOutside(event) {
    if (
      !event.target.closest(".settings-menu") &&
      !event.target.closest(".settings-icon")
    ) {
      showSettings = false;
    }
  }

  // Adding event listener for clicks outside settings
  onMount(() => {
    document.addEventListener("click", handleClickOutside);
    return () => document.removeEventListener("click", handleClickOutside);
  });

  function setGoal() {
    const value = parseInt(goalValue, 10);
    if (!isNaN(value) && value >= 0 && value <= 100) {
      console.log("Goal set to:", value);
    } else {
      alert("Please enter a number between 0 and 100.");
    }
  }

  function switchTab(tab) {
    selectedTab = tab;
  }

  function calculateStats(submissions) {
    const stats = {
      totalWins: 0,
      totalLosses: 0,
      bestSurface: "",
      averageOpponentUTR: 0,
      matchesPlayed: 0,
      winPerc: 0,
    };

    if (submissions.length === 0) {
      return stats;
    }

    const surfaceCounts = {};
    let totalUTR = 0;
    let numZeroes = 0;

    submissions.forEach((submission) => {
      if (submission.result.toLowerCase() === "win") {
        stats.totalWins++;
      } else if (submission.result.toLowerCase() === "loss") {
        stats.totalLosses++;
      }

      // Track surface counts
      const surface = submission.surface.toLowerCase();
      if (!surfaceCounts[surface]) {
        surfaceCounts[surface] = 0;
      }
      surfaceCounts[surface]++;

      // Accumulate total UTR for averaging
      const opponentUTR = Number(submission.opponentUTR);
      if (opponentUTR > 0) {
        totalUTR += opponentUTR;
      } else {
        numZeroes++;
      }
    });

    // Determine the best surface based on the most wins
    stats.bestSurface = Object.keys(surfaceCounts).reduce(
      (best, current) =>
        surfaceCounts[current] > (surfaceCounts[best] || 0) ? current : best,
      "",
    );

    // Calculate average UTR
    stats.averageOpponentUTR = Number(
      (totalUTR / (submissions.length - numZeroes)).toFixed(2),
    );

    if (!stats.averageOpponentUTR) {
      stats.averageOpponentUTR = 0;
    }

    stats.matchesPlayed = submissions.length;

    stats.winPerc = Math.round((stats.totalWins / stats.matchesPlayed) * 100);

    myStats = stats;
  }

  function updateStatsSection() {
    calculateStats(submissions);
  }

  updateStatsSection();
</script>

<main>
  <div class="info">
    <div class="nametime">
      <label class="name">Joseph Schnizer</label>
      <div class="tab-buttons">
        <button
          class:active={selectedTab === "Entries"}
          on:click={() => switchTab("Entries")}
        >
          Entries
        </button>
        <button
          class:active={selectedTab === "Goal Tracking"}
          on:click={() => switchTab("Goal Tracking")}
        >
          Goal Tracking
        </button>
      </div>
      <div class="time-uptime">
        <label class="time">{currentTime}</label>
        <label class="time">{uptime}</label>
        <div class="settings-icon" on:click={toggleSettings}>
          ⚙️
          {#if showSettings}
            <div class="settings-menu" on:click|stopPropagation>
              <h4>Change Theme</h4>
              <div class="theme-buttons">
                <button
                  class="theme-button"
                  on:click={() => changeTheme("#243AFF")}
                  style="background-color: #243AFF; border: {themeColor ===
                  '#243AFF'
                    ? '5px solid black'
                    : 'none'}"
                  aria-label="Blue Theme"
                ></button>
                <button
                  class="theme-button"
                  on:click={() => changeTheme("green")}
                  style="background-color: green; border: {themeColor ===
                  'green'
                    ? '5px solid black'
                    : 'none'}"
                  aria-label="Green Theme"
                ></button>
                <button
                  class="theme-button"
                  on:click={() => changeTheme("#D67009")}
                  style="background-color: #D67009; border: {themeColor ===
                  '#D67009'
                    ? '5px solid black'
                    : 'none'}"
                  aria-label="Orange Theme"
                ></button>
              </div>

              <h4>Field Visibility</h4>
              <div class="checkbox-grid">
                <label class="checkbox-label">
                  <input type="checkbox" bind:checked={showOpponent} />
                  Opponent
                </label>
                <label class="checkbox-label">
                  <input type="checkbox" bind:checked={showSurface} />
                  Surface
                </label>
                <label class="checkbox-label">
                  <input type="checkbox" bind:checked={showNotes} />
                  Match Notes
                </label>
                <label class="checkbox-label">
                  <input type="checkbox" bind:checked={showOpponentUTR} />
                  Opponent UTR
                </label>
                <label class="checkbox-label">
                  <input type="checkbox" bind:checked={showMyUTR} />
                  My UTR
                </label>
              </div>

              <h4>Set My Win % Goal</h4>
              <div class="goal-setting">
                <input
                  type="number"
                  min="0"
                  max="100"
                  bind:value={goalValue}
                  placeholder="0-100"
                  aria-label="Set Goal"
                  style="width: 80px; margin-right: 5px;"
                />
                %
                <button
                  on:click={setGoal}
                  class="submit-btn submit-group"
                  type="submit"
                  class:submitted={isSubmitted}>Set</button
                >
              </div>
            </div>
          {/if}
        </div>
      </div>
    </div>
  </div>

  {#if selectedTab === "Entries"}
    <div class="boxes-container">
      <div class="box" style="background-color: {themeColor};">
        <h1 class="entry">Match Information</h1>

        <form on:submit|preventDefault={submitForm} class="form">
          <fieldset class="form-group">
            <label for="title">Title</label>
            <input id="title" bind:value={title} placeholder="Enter title" />
          </fieldset>

          <fieldset class="form-group">
            <label for="score">Score</label>
            <input id="score" bind:value={score} placeholder="Enter score" />
          </fieldset>

          <fieldset class="form-group">
            <label for="result">Result</label>
            <select id="result" bind:value={result}>
              <option value="Win">Win</option>
              <option value="Loss">Loss</option>
            </select>
          </fieldset>

          {#if showOpponent}
            <fieldset class="form-group">
              <label for="opponent">Opponent</label>
              <input
                id="opponent"
                bind:value={opponent}
                placeholder="Enter opponent's name"
              />
            </fieldset>
          {/if}

          {#if showSurface}
            <fieldset class="form-group">
              <label>Surface</label>
              <div class="radio-group">
                <label>
                  <input
                    type="radio"
                    name="surface"
                    value="Hard"
                    bind:group={surface}
                  />
                  Hard
                </label>
                <label>
                  <input
                    type="radio"
                    name="surface"
                    value="Clay"
                    bind:group={surface}
                  />
                  Clay
                </label>
                <label>
                  <input
                    type="radio"
                    name="surface"
                    value="Grass"
                    bind:group={surface}
                  />
                  Grass
                </label>
                <label>
                  <input
                    type="radio"
                    name="surface"
                    value="Other"
                    bind:group={surface}
                  />
                  Other
                </label>
              </div>
            </fieldset>
          {/if}

          {#if showNotes}
            <fieldset class="form-group">
              <label for="notes">Match Notes</label>
              <textarea
                id="notes"
                bind:value={notes}
                placeholder="Enter match notes"
              ></textarea>
            </fieldset>
          {/if}

          {#if showOpponentUTR}
            <fieldset class="form-group">
              <label for="opponentUTR">Opponent UTR Rating</label>
              <input
                id="opponentUTR"
                bind:value={opponentUTR}
                placeholder="Enter opponent's UTR rating"
                type="number"
                min="1"
                max="16"
              />
            </fieldset>
          {/if}

          {#if showMyUTR}
            <fieldset class="form-group">
              <label for="myUTR">My Current UTR Rating</label>
              <input
                id="myUTR"
                bind:value={myUTR}
                placeholder="Enter your UTR rating"
                type="number"
                min="1"
                max="16"
              />
            </fieldset>
          {/if}

          <div class="form-group submit-group" style="padding-top: 20px;">
            <button
              class="submit-btn"
              type="submit"
              class:submitted={isSubmitted}
            >
              Submit
            </button>
          </div>
        </form>
      </div>

      <div class="resultsbox" style="background-color: {themeColor};">
        <h1 class="entry">Match Entries</h1>
        <div class="scrollable-list">
          <ul>
            {#each submissions as submission}
              <li
                on:click={() => selectSubmission(submission)}
                class:selected={selectedSubmission === submission}
              >
                <strong>{submission.title}</strong> - {submission.timestamp}
              </li>
            {/each}
          </ul>
        </div>

        {#if selectedSubmission}
          <div class="submission-details">
            <h3>Details for {selectedSubmission.title}</h3>
            <div class="submission-columns">
              <div class="left-column">
                <div class="form-group compact">
                  <label for="editScore">Score</label>
                  <input id="editScore" bind:value={selectedSubmission.score} />
                </div>

                <div class="form-group compact">
                  <label for="editResult">Result</label>
                  <select
                    id="editResult"
                    bind:value={selectedSubmission.result}
                  >
                    <option value="Win">Win</option>
                    <option value="Loss">Loss</option>
                  </select>
                </div>

                {#if showOpponent}
                  <div class="form-group compact">
                    <label for="editOpponent">Opponent</label>
                    <input
                      id="editOpponent"
                      bind:value={selectedSubmission.opponent}
                    />
                  </div>
                {/if}

                {#if showSurface}
                  <div class="form-group compact">
                    <label for="editSurface">Surface</label>
                    <div class="radio-group compact">
                      <label>
                        <input
                          type="radio"
                          name="editSurface"
                          value="Hard"
                          bind:group={selectedSubmission.surface}
                        />
                        Hard
                      </label>
                      <label>
                        <input
                          type="radio"
                          name="editSurface"
                          value="Clay"
                          bind:group={selectedSubmission.surface}
                        />
                        Clay
                      </label>
                      <label>
                        <input
                          type="radio"
                          name="editSurface"
                          value="Grass"
                          bind:group={selectedSubmission.surface}
                        />
                        Grass
                      </label>
                      <label>
                        <input
                          type="radio"
                          name="editSurface"
                          value="Other"
                          bind:group={selectedSubmission.surface}
                        />
                        Other
                      </label>
                    </div>
                  </div>
                {/if}
              </div>

              <div class="right-column">
                {#if showOpponentUTR}
                  <div class="form-group compact">
                    <label for="editOpponentUTR">Opponent UTR Rating</label>
                    <input
                      id="editOpponentUTR"
                      bind:value={selectedSubmission.opponentUTR}
                      type="number"
                      min="1"
                      max="16"
                    />
                  </div>
                {/if}

                {#if showMyUTR}
                  <div class="form-group compact">
                    <label for="editMyUTR">My Current UTR Rating</label>
                    <input
                      id="editMyUTR"
                      bind:value={selectedSubmission.myUTR}
                      type="number"
                      min="1"
                      max="16"
                    />
                  </div>
                {/if}

                {#if showNotes}
                  <div class="form-group compact">
                    <label for="editNotes">Match Notes</label>
                    <textarea
                      id="editNotes"
                      bind:value={selectedSubmission.notes}
                    ></textarea>
                  </div>
                {/if}
              </div>
            </div>

            <div class="form-group submit-group">
              <button
                class="submit-btn"
                on:click={saveChanges}
                class:saved={isSaved}
              >
                Save Changes
              </button>
            </div>
          </div>
        {/if}
      </div>
    </div>
  {:else if selectedTab === "Goal Tracking"}
    <div class="goal-box" style="background-color: {themeColor};">
      <div class="stats-section">
        <div class="stats-box">
          <h3>Stats Overview</h3>
          <div class="stats-item">Total Matches Won: {myStats.totalWins}</div>
          <div class="stats-item">
            Total Matches Lost: {myStats.totalLosses}
          </div>
          <div class="stats-item">
            Best Surface: {myStats.bestSurface || "N/A"}
          </div>
          <div class="stats-item">
            Average Opponent UTR: {myStats.averageOpponentUTR}
          </div>
          <div class="stats-item">Matches Played: {myStats.matchesPlayed}</div>
        </div>

        <div class="stats-box">
          <h3>Match Win Percentage</h3>
          <h1>{myStats.winPerc}%</h1>
          {#if myStats.winPerc >= Number(goalValue)}
            <p class="goal-message" style="color: green;">
              You are meeting your win percentage goal!
            </p>
          {:else}
            <p class="goal-message" style="color: red;">
              Win percentage goal not met, keep grinding!
            </p>
          {/if}
        </div>
      </div>
    </div>
  {/if}
</main>

<style>
  h4 {
    margin: 5px;
  }

  main {
    display: flex;
    flex-direction: column;
    height: 100vh;
    margin: 0;
    font-family: "Courier New", Courier, monospace;
  }

  .info {
    background-color: goldenrod;
    height: 45px;
    min-height: 45px;
    max-height: 45px;
    border-radius: 15px;
    margin: 10px;
    box-sizing: border-box;
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 10px;
  }

  .nametime {
    display: flex;
    width: 100%;
    justify-content: space-between;
    align-items: center;
  }

  .name {
    margin-right: auto;
  }

  .time-uptime {
    display: flex;
    align-items: center;
    gap: 10px;
  }

  .time {
    text-align: right;
  }

  .box {
    flex: 1;
    background-color: green;
    border-radius: 15px;
    margin: 10px;
    padding: 20px;
    box-sizing: border-box;
    width: 50vw;
  }

  .resultsbox {
    flex: 1;
    background-color: green;
    border-radius: 15px;
    margin: 10px;
    padding: 20px;
    box-sizing: border-box;
    position: relative;
    display: flex;
    flex-direction: column;
  }

  .boxes-container {
    display: flex;
    flex-direction: row;
    flex: 1;
    margin: 10px;
  }

  .scrollable-list {
    max-height: 50%;
    overflow-y: auto;
    background-color: #eee;
    padding: 10px;
    border-radius: 10px;
  }

  ul {
    list-style-type: none;
    padding: 0;
    height: 250px;
  }

  li {
    background-color: white;
    padding: 10px;
    margin-bottom: 5px;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.2s;
  }

  li:hover {
    background-color: lightgray;
  }

  li.selected {
    background-color: lightblue;
  }

  .submission-details {
    margin-top: 10px;
    color: white;
    max-height: 50%;
  }

  .form-group {
    display: flex;
    flex-direction: column;
    gap: 5px;
  }

  .form-group.compact input,
  .form-group.compact select,
  .form-group.compact textarea {
    padding: 5px;
    font-size: 0.9em;
  }

  .radio-group.compact label {
    margin-right: 10px;
    font-size: 0.9em;
  }

  label {
    font-weight: bold;

    color: white;
  }

  input,
  select,
  textarea {
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
    box-sizing: border-box;
  }

  textarea {
    resize: vertical;
    min-height: 100px;
  }

  .radio-group {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  .submit-group {
    padding-top: 20px;
    text-align: center;
  }

  .submit-btn {
    background-color: #007bff;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 5px;
    cursor: pointer;
    font-size: 16px;
    transition: background-color 0.3s;
  }

  .submit-btn.submitted,
  .submit-btn.saved {
    background-color: rgb(58, 212, 58);
  }

  .submission-columns {
    display: flex;
    justify-content: space-between;
  }

  .left-column,
  .right-column {
    width: 48%;
  }

  .form-group.compact input,
  .form-group.compact select,
  .form-group.compact textarea {
    padding: 5px;
    font-size: 0.9em;
  }

  .radio-group.compact label {
    margin-right: 10px;
    font-size: 0.9em;
  }

  .settings-icon {
    cursor: pointer;
    position: relative;
  }

  .settings-menu {
    position: absolute;
    top: 30px;
    right: 0;
    background-color: white;
    border: 5px solid goldenrod;
    border-radius: 5px;
    padding: 15px;
    display: flex;
    flex-direction: column;
    gap: 5px;
    z-index: 100;
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  .theme-buttons {
    display: flex;
    gap: 10px;
  }

  .theme-button {
    width: 30px;
    height: 30px;
    border-radius: 5px;
    cursor: pointer;
    transition: border 0.3s;
  }

  .checkbox-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 5px 15px;
  }

  .checkbox-label {
    display: flex;
    align-items: center;
    gap: 5px;
    color: black;
  }

  .settings-menu button {
    padding: 5px 10px;
    border: none;
    background-color: #f0f0f0;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  .settings-menu button:hover {
    background-color: #ddd;
  }

  .box,
  .resultsbox {
    transition: background-color 0.3s;
  }

  .resultsbox ul {
    padding: 0;
    list-style: none;
  }

  .resultsbox li {
    padding: 8px;
    border-bottom: 1px solid #ccc;
    cursor: pointer;
  }

  .resultsbox li.selected {
    background-color: #e0e0e0;
  }

  .resultsbox strong {
    font-weight: bold;
  }

  .tab-buttons {
    display: flex;
    gap: 10px;
    padding-right: 20px;
  }

  .tab-buttons button {
    padding: 5px 15px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    background-color: white;
    color: black;
    font-weight: bold;
  }

  .tab-buttons button.active {
    background-color: #007bff;
    color: white;
  }

  .goal-box {
    flex: 1;
    border-radius: 15px;
    margin: 10px;
    padding: 20px;
    box-sizing: border-box;
    display: flex;
    justify-content: center;
    text-align: center;
  }

  .stats-section {
    padding: 20px;
    background-color: #f9f9f9;
    border: 1px solid #ddd;
    border-radius: 5px;
    margin-top: 20px;
    height: 208px;
    display: flex;
    gap: 20px;
    justify-content: space-between;
    align-items: flex-start;
  }

  .stats-box {
    flex: 1;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 8px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
    height: 186px;
  }

  .stats-section h3 {
    margin-bottom: 10px;
    font-size: 20px;
    font-weight: bold;
  }

  .stats-item {
    margin: 5px 0;
    font-size: 16px;
  }
</style>
