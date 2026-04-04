<script lang="ts">
  import Subject from "./lib/Subject.svelte";
  type SubjectData = {
    id: number;
    name?: string;
    credit: number;
    grade: string;
  };
  const gpaToCreditMapping: Record<string, number> = {
    A: 4,
    "A-": 3.75,
    "B+": 3.25,
    B: 3,
    "B-": 2.75,
    "C+": 2.25,
    C: 2,
    "C-": 1.75,
    D: 1,
    F: 0,
  };
  let subjects: SubjectData[] = $state([]);
  let countOfSubjects = $state(0);

  function addSubject() {
    countOfSubjects += 1;
    let x = countOfSubjects;
    subjects.push({
      id: x,
      name: "",
      credit: 3,
      grade: "A",
    });
  }
  function removeSubject(id: number) {
    subjects = subjects.filter((x) => x.id != id);
    countOfSubjects = subjects.length;
  }

  let totalCredits = $derived(
    subjects.reduce((acc, cv) => {
      const credit = Number(cv.credit);
      return Number.isFinite(credit) && credit > 0 ? acc + credit : acc;
    }, 0),
  );

  let calculationString = $derived.by(() => {
    const terms: string[] = [];
    let countedCredits = 0;

    for (const subject of subjects) {
      const credit = Number(subject.credit);
      const point = gpaToCreditMapping[subject.grade];

      if (!Number.isFinite(credit) || credit <= 0 || point === undefined) {
        continue;
      }

      terms.push(`(${point} * ${credit})`);
      countedCredits += credit;
    }

    return countedCredits > 0
      ? `[${terms.join(" + ")}] / ${countedCredits}`
      : "";
  });

  let gpa = $derived.by(() => {
    // Kahan summation reduces floating-point accumulation error for long subject lists.
    let totalGradePoints = 0;
    let gradePointsCompensation = 0;
    let countedCredits = 0;
    let creditsCompensation = 0;

    const addGradePoints = (value: number) => {
      const y = value - gradePointsCompensation;
      const t = totalGradePoints + y;
      gradePointsCompensation = t - totalGradePoints - y;
      totalGradePoints = t;
    };

    const addCredits = (value: number) => {
      const y = value - creditsCompensation;
      const t = countedCredits + y;
      creditsCompensation = t - countedCredits - y;
      countedCredits = t;
    };

    for (const subject of subjects) {
      const credit = Number(subject.credit);
      const point = gpaToCreditMapping[subject.grade];

      if (!Number.isFinite(credit) || credit <= 0 || point === undefined) {
        continue;
      }

      addGradePoints(point * credit);
      addCredits(credit);
    }

    return countedCredits > 0 ? totalGradePoints / countedCredits : 0;
  });

  let gpaDisplay = $derived(gpa > 0 ? gpa.toFixed(2) : "0.00");
</script>

<main>
  <h1>GPA calculator</h1>
  <p>An easy to use, customizable GPA calculator</p>
  <div class="subject-input">
    <ul>
      {#each subjects as sub (sub.id)}
        <li>
          <Subject
            id={sub.id}
            bind:name={sub.name}
            bind:credit={sub.credit}
            bind:grade={sub.grade}
          /><button
            onclick={() => {
              removeSubject(sub.id);
            }}>Delete</button
          >
        </li>
      {/each}
      <button onclick={addSubject}>+</button>
    </ul>
  </div>
  <div class="result">
    <p>Total credits = {totalCredits}</p>
    {#if gpa > 0}
      <p>GPA = {gpaDisplay}</p>
      <p class="formula-show">
        How it is calculated: <br /><span>{calculationString}</span>
      </p>
    {/if}
  </div>
</main>

<style>
  main {
    display: flex;
    flex-direction: column;
    align-items: center;
    height: 100%;
  }

  .subject-input {
    margin: 1.5rem 0rem;
    padding: 1.25rem 0.5rem;
    border-radius: 0.5rem;
    border: 1px solid black;
  }

  ul {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 0;
  }
  li {
    list-style: none;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  ul button {
    font-size: 1.35rem;
    padding: 0.25rem 8rem;
    border: none;
  }
  li button {
    font-size: 1rem;
    padding: 0.25rem 0.5rem;
    margin: 0.5rem 1rem;
  }

  .result {
    padding: 1rem;
    background-color: rgb(14, 126, 255);
    border-radius: 0.5rem;
    color: white;
    font-size: 1.5rem;
  }

  .formula-show {
    margin-top: 0.5rem;
  }
  .formula-show span {
    font-size: 1.25rem;
  }
  span {
    width: 10ch;
    overflow: auto;
  }
</style>
