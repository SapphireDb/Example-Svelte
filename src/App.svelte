<script>
	import { SapphireDb } from 'sapphiredb';

	const db = new SapphireDb({
		serverBaseUrl: 'localhost:5000',
		useSsl: false,
		apiKey: 'webapp',
		apiSecret: 'pw1234',
		connectionType: 'websocket'
	});

	let values = [];

	db.collection('demo.entries').values().subscribe((valuesUpdate) => {
		values = valuesUpdate;
	});
</script>

{#each values as value}
	<div>
		Id: {value.id}, Content: {value.content}
	</div>
{/each}
