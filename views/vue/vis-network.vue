<template>
  <div>
		<div id="vis-network"></div>
  </div>
</template>

<script>
export default {
	mounted() {
		var driver = neo4j.driver('bolt://10.30.4.134:7687',
				neo4j.auth.basic('neo4j', 'Password12345'),
				{/* encrypted: 'ENCRYPTION_OFF' */ });

		const query =
			`
			MATCH (n)-[d]->(n1)
			WITH [n,d,n1] AS arrayData
			RETURN arrayData limit 50
		`;

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
				createNetwork(nodes, edges)
				session.close();
				driver.close();
			})
			.catch((error) => {
				console.error(error);
			});

		function createNetwork(nodes, edges) {		
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

