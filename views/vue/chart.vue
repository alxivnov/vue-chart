<template>
  <div>
    <div id="chart"></div>
		<div id="mynetwork"></div>
  </div>
</template>

<script>
export default {
	mounted() {
		var driver = neo4j.driver('bolt://3.235.46.32:7687',
				neo4j.auth.basic('neo4j', 'massed-extenuations-desk'),
				{/* encrypted: 'ENCRYPTION_OFF' */ });
				
		const query =
			`
			MERGE (p:Person {name:$name})
			ON MATCH SET p.lastAt = timestamp()
			ON CREATE SET p.createAt = timestamp()
			RETURN p.name AS name
		`;

		const params = { "name": "Victoryia Struk" };

		const session = driver.session({ database: "neo4j" });

		session.run(query, params)
			.then((result) => {
				result.records.forEach((record) => {
					console.log(record.get('name'));
				});
				session.close();
				driver.close();
			})
			.catch((error) => {
				console.error(error);
			});

		var color = "gray";
		var len = undefined;

		var nodes = [
			{ id: 0, label: "Cloud Atlas", group: 0 },
			{ id: 1, label: "Tom Tykwer", group: 1 },
			{ id: 2, label: "Lana Wachowski", group: 1 },
			{ id: 3, label: "Lilly Wachowski", group: 1 },
		];
		var edges = [
			{ from: 1, to: 0, label: 'DIRECTED' },
			{ from: 2, to: 0, label: 'DIRECTED' },
			{ from: 3, to: 0, label: 'DIRECTED' }
		];

		// create a network
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
		};
		var network = new vis.Network(container, data, options);

		/*
			var options = {
				chart: {
					type: 'bar'
				},
				series: [{
					name: 'sales',
					data: [30,40,45,50,49,60,70,91,125]
				}],
				xaxis: {
					categories: [1991,1992,1993,1994,1995,1996,1997, 1998,1999]
				}
			}

			var chart = new ApexCharts(document.querySelector("#chart"), options);

			chart.render();
		*/
	}
}
</script>

