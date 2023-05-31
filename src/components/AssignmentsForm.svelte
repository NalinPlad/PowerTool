<script>
  import { onMount } from "svelte";
  import { bind } from "svelte/internal";

  // onMount(async () => {
  //   const file_sample = await fetch("/sample.html");
  //   const file_data = await file_sample.blob();
  //   files = file_data;
  // });

  let files;
  // let files = fetch("/sample.txt");
  // for debugging, set file to a sample file 'sample.html' using file api
  // let file = new File([""], "sample.html", {type: "text/html"});

  let file_content = localStorage.getItem("file_content")
    ? localStorage.getItem("file_content")
    : "";
  let table_json = localStorage.getItem("table_json")
    ? JSON.parse(localStorage.getItem("table_json"))
    : [];
  let headers = localStorage.getItem("headers")
    ? JSON.parse(localStorage.getItem("headers"))
    : [];

  let showHelp = false;

  const toggleHelp = () => {
    showHelp = !showHelp;
  };

  const process_content = () => {};

  const update_percent_and_grade = () => {
    console.log("hello")
    console.table(table_json);
    // calculate the percentage column based on the score and points columns and validate the calculate against the original percentage column
    table_json = table_json.map((row) => {
      const row_json = {};
      for (const header of Object.keys(row)) {
        if (header === "%") {
          // round to nearest hundredth
          // const percentage = (row["score"] / row["points"]) * 100;
          const percentage = Math.round((row["score"] / row["points"]) * 10000) / 100;

          // if (percentage !== parseFloat(row[header])) {
          //   console.log(
          //     `Error: Calculated percentage ${percentage} does not match original percentage ${row[header]}`
          //   );
          // }
          row_json["%"] = percentage;
          continue;
        }
        row_json[header] = row[header];
      }
      return row_json;
    });

    //calculate the letter grade from the percentage column (ew fix this please it was generated)
    table_json = table_json.map((row) => {
      const row_json = {};
      for (const header of Object.keys(row)) {
        if (header === "Grade") {
          const percentage = row["%"];
          if (percentage >= 97) {
            row_json["Grade"] = "A+";
          } else if (percentage >= 93) {
            row_json["Grade"] = "A";
          } else if (percentage >= 90) {
            row_json["Grade"] = "A-";
          } else if (percentage >= 87) {
            row_json["Grade"] = "B+";
          } else if (percentage >= 83) {
            row_json["Grade"] = "B";
          } else if (percentage >= 80) {
            row_json["Grade"] = "B-";
          } else if (percentage >= 77) {
            row_json["Grade"] = "C+";
          } else if (percentage >= 73) {
            row_json["Grade"] = "C";
          } else if (percentage >= 70) {
            row_json["Grade"] = "C-";
          } else if (percentage >= 67) {
            row_json["Grade"] = "D+";
          } else if (percentage >= 63) {
            row_json["Grade"] = "D";
          } else if (percentage >= 60) {
            row_json["Grade"] = "D-";
          } else {
            row_json["Grade"] = "F";
          }
          continue;
        }
        row_json[header] = row[header];
      }
      return row_json;
    });
  };

  const process = () => {
    console.log(files);
    if (files) {
      const reader = new FileReader();
      // console.log(files[0]);
      reader.addEventListener(
        "load",
        () => {
          file_content = reader.result;
          localStorage.setItem("file_content", file_content);
          const parser = new DOMParser();
          const html_doc = parser.parseFromString(file_content, "text/html");
          const table = html_doc.querySelector("#scoreTable");

          // Convert table to JSON, remember that the Flags section has 5 columns in the table
          headers = [];
          for (const header of table.querySelectorAll("thead tr th")) {
            // if Flags, add 5 columns [collected, late, missing, exempt from final grade, absent, incomplete, excluded from final grade]
            if (
              header.textContent.replace(/^\s+|\s+$|\s+(?=\s)/g, "") === "Flags"
            ) {
              headers.push(
                "collected",
                "late",
                "missing",
                "exempt from final grade",
                "absent",
                "incomplete",
                "excluded from final grade"
              );
              continue;
            }
            headers.push(
              header.textContent.replace(/^\s+|\s+$|\s+(?=\s)/g, "")
            );
          }

          // console.log(headers);
          table_json = [];
          for (const row of table.querySelectorAll("tbody tr")) {
            const row_json = {};
            for (const [i, cell] of row.querySelectorAll("td").entries()) {
              // strip all newlines and white space(not including spaces between words)
              row_json[headers[i]] = cell.textContent.replace(
                /^\s+|\s+$|\s+(?=\s)/g,
                ""
              );
            }
            table_json.push(row_json);
          }

          // filter out any flag columns and comments column
          table_json = table_json.map((row) => {
            const row_json = {};
            for (const header of headers) {
              if (
                header === "collected" ||
                header === "late" ||
                header === "missing" ||
                header === "exempt from final grade" ||
                header === "absent" ||
                header === "incomplete" ||
                header === "excluded from final grade" ||
                header === "View comments and descriptions" ||
                header === "Category"
              ) {
                continue;
              }
              row_json[header] = row[header];
            }
            return row_json;
          });

          // remove last row (last updated) and assign the date to variable instead
          let last_row = table_json.pop();

          // parse the score column [score/points] and assign the values to two new columns, score and points
          table_json = table_json.map((row) => {
            const row_json = {};
            for (const header of Object.keys(row)) {
              if (header === "Score") {
                const score = row[header].split("/");
                row_json["score"] = score[0];
                row_json["points"] = score[1];
                continue;
              }
              row_json[header] = row[header];
            }
            return row_json;
          });

          // convert the score / points columns to json numbers instead of strings
          table_json = table_json.map((row) => {
            const row_json = {};
            for (const header of Object.keys(row)) {
              if (header === "score" || header === "points") {
                row_json[header] = Number(row[header]);
                continue;
              }
              row_json[header] = row[header];
            }
            return row_json;
          });

          update_percent_and_grade();

          localStorage.setItem("headers", JSON.stringify(headers));
          localStorage.setItem("table_json", JSON.stringify(table_json));
          // console.log(table_json);
        },
        false
      );
      reader.readAsText(files[0]);

      console.log(table_json);
    }
  };

  $: if (files) {
    // Note that `files` is of type `FileList`, not an Array:
    // https://developer.mozilla.org/en-US/docs/Web/API/FileList
    // console.log(files);

    for (const file of files) {
      // console.log(`${file.name}: ${file.size} bytes`);
    }
  }
</script>

<div class="overflow-hidden">
  <header class="bg-slate-50 p-5 w-screen rounded-lg">
    <div class="text-2xl">
      PowerTool v0.3 by <span class="text-blue-900 tracking-wide">notmysql</span
      >
    </div>

    <div>
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <h1
        on:click={toggleHelp}
        class="my-1 bg-slate-200 w-fit p-1 px-2 hover:bg-slate-300 hover:scale-105 transition-all cursor-default rounded-md italic text-slate-900 shadow select-none"
      >
        How to use PowerTool <span class="not-italic">🤔</span>
      </h1>
      {#if showHelp}
        <p
          class="text-slate-800 bg-amber-100 w-fit p-3 rounded-md before:content-['->'] before:text-slate-500 before:px-3"
        >
          To get your PowerSchool file, log into PowerSchool, navigate to your
          class, and right click on the page and select <code
            class="bg-slate-100 p-1 text-sm shadow-sm">Save As</code
          >
        </p>
      {/if}
    </div>
  </header>

  <main class="p-5 w-fit m-0">
    <section class="my-5 flex items-center" preventDefault>
      <div class="flex flex-col bg-blue-100 p-2 w-fit rounded-md shadow-sm">
        <label for="pf" class="text-sm m-1 underline"
          >Click here to add a new PowerSchool file</label
        >
        <input
          accept="text/html"
          bind:files
          id="pf"
          name="powerschoolfile"
          type="file"
          class="hidden"
        />
        {#if files}
          {#each Array.from(files) as file}
            <p>{file.name} ({file.size}B)</p>
          {/each}
        {/if}
      </div>

      {#if files}
        <button
          class="mx-2 bg-green-100 p-3 rounded-md transition-all h-fit hover:scale-105 shadow-sm"
          on:click={process}>😎 Process →</button
        >
      {/if}
    </section>
    {#if file_content}
      <!-- make table with table header and table content -->
      <table class="table-auto">
        <thead>
          <tr>
            {#each Object.keys(table_json[0]) as header}
              <th class="">{header}</th>
            {/each}
          </tr>
        </thead>
        <tbody>
          {#each table_json as row, i1 (i1)}
            <tr>
              {#each Object.values(row) as cell, i (i)}
                <!-- if the column is score, make it an input -->
                {#if Object.keys(table_json[0])[i] === "score"}
                  <td class="flex"
                    ><input
                      type="number"
                      class=" mx-1 w-20"
                      bind:value={table_json[i1]["score"]}
                      step="0.01"
                      on:input={update_percent_and_grade}
                    /></td
                  >
                {:else if Object.keys(table_json[0])[i] === "points"}
                  <td class=""
                    ><input
                      type="number"
                      class=" mx-1 w-20"
                      bind:value={table_json[i1]["points"]}
                      step="0.01"
                      on:input={update_percent_and_grade}
                    /></td
                  >
                {:else if Object.keys(table_json[0])[i] === "%"}
                  <td class="flex">
                    {table_json[i1]["%"]}
                  </td>
                {:else}
                  <td class="">{cell}</td>
                {/if}
              {/each}
            </tr>
          {/each}
        </tbody>
      </table>

      <!-- <p>{last_row}</p> -->
    {/if}
  </main>
</div>

<style>
  th,
  tr,
  td {
    @apply px-2 py-[5px];
  }

  table {
    border: 1px solid #ddd;
    border-collapse: separate;
    border-left: 0;
    border-radius: 4px;
    border-spacing: 0px;
  }

  thead {
    display: table-header-group;
    vertical-align: middle;
    border-color: inherit;
    border-collapse: separate;
    border-bottom: 2px solid #ddd;
  }

  tr {
    display: table-row;
    vertical-align: inherit;
    border-color: inherit;
    padding: 5px;
  }

  tr:nth-child(odd) {
    background-color: #f9f9f9;
  }

  th {
    border-bottom: 1px solid #ddd;
  }

  th,
  td {
    /* padding: 5px 4px 6px 4px; */
    text-align: left;
    vertical-align: top;
    border-left: 1px solid #ddd;
  }

  td {
    /* border-top: 1px solid #dddddd83; */
  }

  thead:first-child tr:first-child th:first-child,
  tbody:first-child tr:first-child td:first-child {
    border-radius: 4px 0 0 0;
  }

  thead:last-child tr:last-child th:first-child,
  tbody:last-child tr:last-child td:first-child {
    border-radius: 0 0 0 4px;
  }
</style>
