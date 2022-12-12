<script>
  import { getContext, onDestroy } from "svelte";
  //import { PowerTable } from "./muonw/powertable/src/lib/components/PowerTable.svelte";
  //import "@muonw/powertable/powertable/package/dist/power-table.scss";
  import { PowerTable } from "./muonw2/powertable";
  import "./muonw2/powertable/package/styles/power-table.scss";
  const { styleable, getAction, ActionTypes, routeStore, rowSelectionStore } =
    getContext("sdk");
  const component = getContext("component");
  let selectedRows = [];

  rowSelectionStore.subscribe((value) => {
    console.log(value);
  });

  console.log(rowSelectionStore);

  $: {
    rowSelectionStore.actions.updateSelection(
      $component.id,
      selectedRows.length ? selectedRows[0].tableId : "",
      selectedRows.map((row) => row._id)
    );
  }

  export let dataProvider;
  export let columns;
  export let isActive = false;
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

  const toggleSelectRow = (row) => {
    console.log("test")
    console.log(JSON.stringify(selectedRows))
    if (selectedRows.some((selectedRow) => selectedRow._id === row._id)) {
      selectedRows = selectedRows.filter(
        (selectedRow) => selectedRow._id !== row._id
      );
    } else {
      selectedRows = [...selectedRows, row];
    }
  };

  $: dataContext = {
    isActive,
  };

  let ptData = [
    { id: 1, name: "Fay" },
    { id: 2, name: "Luca" },
  ];
  $: ptData = dataProvider?.rows ?? [];

  $: ptInstructs = modifyColumns(columns);

  const modifyColumns = (columns) => {
    let newList = [];
    for (var i = 0; i < columns.length; i++) {
      let col = { key: columns[i]["name"], title: columns[i]["displayName"] };
      newList.push(col);
      newList[i]["key"] = columns[i]["name"];
      newList[i]["title"] = columns[i]["name"];
      columns[i]["key"] = columns[i]["name"];
      columns[i]["title"] = columns[i]["displayName"];
      //delete columns[i]['name']
      //delete columns[i]['displayName']
      //delete columns[i]['id']
    }

    return newList;
  };

  $: ptOptions = {
    segments: {
      topBar: ["settings", "search", "pagination"],
      pTable: ["table"],
      bottomBar: ["dropdown", "stats", "pagination"],
    },
    rowsPerPageOptions: [10, 100, 200],
    footerText: footerText,
    headerText: headerText,
    footerFilters: footerFilters,
    headerFilters: headerFilters,
    checkboxColumn: rowSelection,
  };

  function handleMessage(event) {
		alert("gotcha");
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
      on:checkBoxToggled={(e) => toggleSelectRow(e.detail.data)}
    >
      <div slot="noResults">{noDataText}</div>
      <div slot="settings">
        <button data-name="item">Export current data</button>
      </div>
    </PowerTable>
    {selectedRows}
  </div>
</div>
