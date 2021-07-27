<template>
  <div>
    <div id="chart"></div>
		<div id="mynetwork"></div>
  </div>
</template>

<script>
export default {
	mounted() {
		var driver = neo4j.driver('bolt://54.211.109.0:7687',
				neo4j.auth.basic('neo4j', 'ages-synthetics-vehicle'),
				{/* encrypted: 'ENCRYPTION_OFF' */ });
				
		const query =
			`
			MATCH (n1)-[r]->(n2) 
			WITH
				[n1] AS n1,
				[n2] AS n2,
				[r] AS r
			UNWIND (n1 + n2) AS node
			UNWIND r AS edge
			RETURN {  	
				nodes: 
          CASE
					WHEN node.name IS NOT NULL
					THEN {
						name: node.name,
						group: labels(node),
						id: id(node)
					} ELSE {
							name: node.title,
							group: labels(node),
							id: id(node)
					} END,
					edges: {
						id: id(edge),
						type: type(edge),
						from: startNode(edge),
						to: endNode(edge)
				}
			} AS arrayData limit 10
		`;

		// const params = { "name": "" };
		const params = {}
		const session = driver.session({ database: "neo4j" });

		var nodes = []
		var edges = []
		session.run(query, params)
			.then((result) => {
				result.records.forEach((record) => {
					nodes.push({
						id: record.get('arrayData').nodes.id.low,
						label: record.get('arrayData').nodes.name,
						group: record.get('arrayData').nodes.group[0]
					})
					edges.push({
						id: record.get('arrayData').edges.id.low,
						from: record.get('arrayData').edges.from.identity.low,
						to: record.get('arrayData').edges.to.identity.low,
						label: record.get('arrayData').edges.type
					})
				});
				
				nodes = nodes.slice().reverse().filter((v,i,a)=>a.findIndex(t=>(t.id === v.id))===i).reverse()
				edges = edges.slice().reverse().filter((v,i,a)=>a.findIndex(t=>(t.id === v.id))===i).reverse()
				createNetwork(nodes, edges)
				session.close();
				driver.close();
			})
			.catch((error) => {
				console.error(error);
			});

		function createNetwork(nodes, edges) {		
			var container = document.getElementById("mynetwork");
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

