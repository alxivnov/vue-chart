<template>
	<div>
		<form-textarea v-model="query" :placeholder="query" rows="3" />
		<btn @click="refresh">Query</btn>
		<div id="vis-network" class="mt-3" style="width: 100%; height: 500px; border: 1px solid lightgray;"></div>
	</div>
</template>

<script>
export default {
	data() {
		return {
			query: `MATCH (n)-[d]->(n1)
WITH [n,d,n1] AS arrayData
RETURN arrayData limit 50`
		}
	},
	mounted() {
		this.refresh();
	},
	methods: {
		refresh() {
			this.apiSetup();
		},
		apiSetup() {
//			let url = '/agents/neo4j/query';
//			let url = 'http://localhost:5000/functions/d07d11ded09ef1c3ec0f073b060439fa';
			let url = '';
			return fetch(url, { method: 'POST', headers: { 'Content-Type': 'text/plain' }, body: this.query})
				.then(res => res.json())
				.then(arr => {
					let nodes = {};
					let edges = {};

					arr.flatMap(record => record.arrayData).forEach(x => {
						let id = Number(x.identity);

						if (x.start && x.end) {
							if (!edges[id])
								edges[id] = {
									id,
									from: Number(x.start),
									to: Number(x.end),
									label: x.label
								};
						} else {
							if (!nodes[id])
								nodes[id] = {
									id,
									label: x.properties.name || x.properties.id,
									group: x.labels[0]
								};
						}
					});

					this.createNetwork(Object.values(nodes), Object.values(edges));
				})
				.catch(err => console.log(err));
		},
		neoSetup() {
			var driver = neo4j.driver('bolt://10.30.4.134:7687',
					neo4j.auth.basic('neo4j', 'Password12345'),
					{/* encrypted: 'ENCRYPTION_OFF' */ });

			const query =
				this.query;

			const params = {}
			const session = driver.session({ database: "neo4j" });

			var nodes = []
			var edges = []
			session.run(query, params)
				.then((result) => {
					result.records.forEach((record) => {
						record.get(0).map(function(node) {
							if (typeof node['type'] !== "undefined")  {
								if (!edges.find(val => val.id === node.identity.low)) {
									edges.push({
										id: node.identity.low,
										from: node.start.low,
										to: node.end.low,
										label: node['type']
									})
								}
							} else if (!nodes.find(val => val.id === node.identity.low)) {
								nodes.push({
									id: node.identity.low,
									label:  node.properties.name ? node.properties.name : node.labels[0],
									group: node.labels[0]
								})
							}
						});
					});
					this.createNetwork(nodes, edges)
					session.close();
					driver.close();
				})
				.catch((error) => {
					console.error(error);
				});
		},
		createNetwork(nodes, edges) {
			var container = document.getElementById("vis-network");
			var data = {
				nodes: nodes,
				edges: edges,
			};
			var options = {
				nodes: {
					shape: "circle",
					size: 30,
					font: {
						size: 10,
						color: "#000",
					},
					borderWidth: 2,
				},
				edges: {
					arrows: 'from',
					width: 2,
					font: {
						size: 10,
						align: "bottom"
					},
				},
				physics:{
					enabled: true
				}
			};
			var network = new vis.Network(container, data, options);
		}
	}
}
</script>

