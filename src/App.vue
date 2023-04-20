<template>
  <header>
    <nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark">
      <div class="container-fluid">
        <a class="navbar-brand" href="#">sshauto</a>
        <div class="collapse navbar-collapse" id="navbarCollapse">
          <ul class="navbar-nav me-auto mb-2 mb-md-0">
            <input type="text" class="form-control me-2" placeholder="Suche" data-toggle="tooltip" title="Filtern der Nodes (* für Wildcards)" @input="filterNodes" />
            <input type="text" class="form-control me-2" placeholder="Kommando" v-model="command" />
            <button type="button" :class="{'btn':true, 'btn-primary':!copied, 'btn-success':copied, 'me-2':true}" @click="generateCommand">{{ copyText }}</button>
          </ul>
        </div>
      </div>
    </nav>
  </header>

  <main style="margin-top: 70px">

    <div class="container">
      <table class="table table-striped table-hover">
        <thead>
          <th scope="col">Hostname</th>
          <th scope="col">Version</th>
          <th scope="col">Gluon Version</th>
        </thead>
        <tbody>
          <tr v-for="node in filteredNodes">
            <td scope="row">{{ node.hostname }}</td>
            <td scope="row">{{ node.firmware.release }}</td>
            <td scope="row">{{ node.firmware.base }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </main>
</template>

<script>
export default {
  data() {
    return {
      nodes: [],
      filteredNodes: [],
      filterText: "",
      command: "",
      copied: false,
      copyText: "Generieren"
    }
  },
  methods: {
    filterNodes(event) {
      if(event.target.value.toLowerCase().includes('*')) {
        this.filteredNodes = this.nodes.filter(
        item => new RegExp('^' + event.target.value.toLowerCase().replace(/\*/g, '.*') + '$').test(item.hostname.toLowerCase())
      );
      }else{
        this.filteredNodes = this.nodes.filter((node) => {
            return node.hostname.toLowerCase().includes(event.target.value.toLowerCase())
        });
      }
    },
    generateCommand() {
      var commands = "";

      this.filteredNodes.forEach(node => {
        var address;
        node.addresses.forEach(currentAddress => {
          if (currentAddress.startsWith("fdca")) {
            address = currentAddress;
          }
        });

        commands += `ssh ${address} "${this.command}" &\n`;
      });

      const self = this;
      navigator.clipboard.writeText(commands).then(function () {
        console.log('Async: Copying to clipboard was successful!');

        self.copied = true;
        self.copyText = "Kopiert!";

        setTimeout(()=> {
          self.copied = false;
          self.copyText = "Generieren";
        }, 5*1000);
      }, function (err) {
        console.error('Async: Could not copy text: ', err);
      });
    }
  },
  mounted() {
    fetch('https://harvester.ffh.zone/api/meshviewer.json')
      .then((response) => response.json())
      .then((data) => {
        this.nodes = data.nodes;
        this.filter = this.nodes;
      });
  }
}
</script>