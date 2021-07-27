<template>
  <div>
		<div id="viz"></div> 
  </div>
</template>

<script>
export default {
	mounted() {		
		let viz;
		function draw() {
				const query =`
					MATCH (n)-[r]->(m) RETURN * limit 10
				`;
				let config = {
						container_id: "viz",
						server_url: "bolt://10.30.4.134:7687",
						server_user: "neo4j",
						server_password: "Password12345",
						labels: {
								"Character": {
										"caption": "name",
										"size": "pagerank",
										"community": "community",
										"title_properties": [
												"name",
												"pagerank"
										]
								}
						},
						relationships: {
								"INTERACTS": {
										"thickness": "weight",
										"caption": false
								}
						},
						arrows: true,
						initial_cypher: query
				};

				viz = new NeoVis.default(config);
				viz.render();

		}
		draw()
	}
}
</script>

