<script>
  import { getContext, onDestroy } from "svelte";
  import { PowerTable } from "./muonw2/powertable";
  import "./muonw2/powertable/package/styles/power-table.scss";

  const { styleable, rowSelectionStore } = getContext("sdk");
  const component = getContext("component");
  // let selectedRows = [];

  // $: {
  //   rowSelectionStore.actions.updateSelection(
  //     $component.id,
  //     selectedRows.length ? selectedRows[0].tableId : "",
  //     selectedRows.map((row) => row)
  //   );
  // }

  export let dataProvider;
  export let columns;
  export let rowSelection;
  export let headerText;
  export let footerText;
  export let headerFilters;
  export let footerFilters;
  export let onRowClick;
  export let onRowDblClick;
  export let noDataText;

  onDestroy(() => {
    rowSelectionStore.actions.updateSelection($component.id, []);
  });


  function uuid() {
    return "cxxxxxxxxxxxx4xxxyxxxxxxxxxxxxxxx".replace(/[xy]/g, (c) => {
      const r = (Math.random() * 16) | 0;
      const v = c === "x" ? r : (r & 0x3) | 0x8;
      return v.toString(16);
    });
  }

  $: ptData =
    dataProvider?.rows.map((v) => (v._id ? v : { ...v, _id: uuid() })) ?? [];

  $: ptInstructs = modifyColumns(columns);

  const modifyColumns = (columns) => {
    let newList = [];
    if (typeof(columns) == 'undefined'){
      return []
    } 
    for (var i = 0; i < columns.length; i++) {
      let col = { key: columns[i]["name"], title: columns[i]["displayName"] };
      newList.push(col);
      newList[i]["key"] = columns[i]["name"];
      newList[i]["title"] = columns[i]["name"];
      columns[i]["key"] = columns[i]["name"];
      columns[i]["title"] = columns[i]["displayName"];
    }

    return newList;
  };

  $: ptOptions = {
    segments: {
      topBar: ["settings", "search", "pagination"],
      pTable: ["table"],
      bottomBar: ["dropdown", "stats", "pagination"],
    },
    rowsPerPageOptions: [5,10, 20 , 100, 200],
    footerText: footerText,
    headerText: headerText,
    footerFilters: footerFilters,
    headerFilters: headerFilters,
    checkboxColumn: rowSelection,
  };

  function exportCsv() {
    let json = ptData;
    var fields = Object.keys(json[0]);
    var replacer = function (key, value) {
      return value === null ? "" : value;
    };
    var csv = json.map(function (row) {
      return fields
        .map(function (fieldName) {
          return JSON.stringify(row[fieldName], replacer);
        })
        .join(",");
    });
    csv.unshift(fields.join(",")); // add header column
    csv = csv.join("\r\n");
    let csvContent = "data:text/csv;charset=utf-8," + csv;
    var downloadLink = document.createElement("a");
    downloadLink.href = csvContent;
    downloadLink.download = "data.csv";

    document.body.appendChild(downloadLink);
    downloadLink.click();
    document.body.removeChild(downloadLink);
  }
</script>

<div use:styleable={$component.styles}>
  <div class="MuonW PowerTable">
    <PowerTable
      {ptInstructs}
      {ptData}
      {ptOptions}
      on:rowClicked={onRowClick}
      on:rowDblClicked={onRowDblClick}
    >
      <div slot="noResults">{noDataText}</div>
      <div slot="settings">
        <button data-name="item" on:click={exportCsv}
          >Export all data as CSV</button
        >
      </div>
    </PowerTable>
  </div>
</div>
