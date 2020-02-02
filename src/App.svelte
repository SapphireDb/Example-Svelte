<script>
    import {SapphireDb} from 'sapphiredb';
    import {map} from "rxjs/operators";

    const db = new SapphireDb({
        serverBaseUrl: 'sapphiredb-demo.azurewebsites.net',
        useSsl: true,
        connectionType: 'websocket'
    });

    /* Basic example */
    let exampleCollection = db.collection('basic.examples');
    let examples$ = exampleCollection.values();
    let examples = [];

    examples$.subscribe((v) => examples = v);

    function addExample() {
        exampleCollection.add({
            content: prompt('Content of new example:')
        });
    }

    function updateExample(example) {
        exampleCollection.update(Object.assign(example, {content: prompt('New content of example:')}))
    }

    function removeExample(example) {
        exampleCollection.remove(example);
    }

    /* Chat */
    let userCollection = db.collection('chat.users');
    let users$ = userCollection.values();
    let users = [];

    users$.subscribe((v) => users = v);

    function createUser() {
        userCollection.add({
            username: prompt('New username: ') || 'default'
        });
    }

    let currentUser;
    let chatPartner;
    let chatPartners$;
    let chatPartners;

    let chatPartnerSubscription;

    function setUser(user) {
        currentUser = user;
        chatPartner = null;
        chatPartners$ = users$.pipe(
                map((users) => users.filter(u => u.id !== currentUser.id))
        );

        if (chatPartnerSubscription) {
        	chatPartnerSubscription.unsubscribe();
		}

        chatPartnerSubscription = chatPartners$.subscribe(v => chatPartners = v);
    }

    let messages;
	let messageSubscription;
	let messageCollection;

    function setChatPartner(user) {
		chatPartner = user;

		if (messageSubscription) {
			messageSubscription.unsubscribe();
		}

		messageCollection = db.collection('chat.messages').where([
			[['ownerId', '==', currentUser.id], 'and', ['receiverId', '==', chatPartner.id]],
			'or',
			[['ownerId', '==', chatPartner.id], 'and', ['receiverId', '==', currentUser.id]]
		]);
		messageSubscription = messageCollection.values().subscribe(v => messages = v);
	}

	let newMessageContent;

    function sendMessage() {
		messageCollection.add({
			ownerId: currentUser.id,
			receiverId: chatPartner.id,
			content: newMessageContent
		});
		newMessageContent = '';
	}
</script>

<h1>Example Svelte application using SapphireDb</h1>

<h2>Basic example</h2>

{#each examples as value}
    <div>
        {JSON.stringify(value)}
        <button on:click={updateExample(value)}>Update</button>
        <button on:click={removeExample(value)}>Remove</button>
    </div>
{/each}

<button on:click={addExample}>Add</button>

<h2>Chat</h2>

<h3>Users</h3>

{#each users as user}
    <button on:click={setUser(user)}>{user.username}</button>
{/each}

<button on:click={createUser}>Create user</button>

{#if currentUser}
	<h3>You are: {currentUser.username}</h3>
	<h4>Select chat partner</h4>
	{#each chatPartners as user}
		<button on:click={setChatPartner(user)}>{user.username}</button>
	{/each}

	{#if chatPartner}
		<h3>Chat between '{currentUser.username}' and '{chatPartner.username}'</h3>
		<h4>Message</h4>

		{#if messages}
			{#each messages as message}
				<div style="text-align: {message.ownerId === currentUser.id ? 'right' : 'left'}">
					{message.content}({message.createdOn})
				</div>
			{/each}
		{/if}

		<label>
			New Message:
			<input bind:value={newMessageContent} />
		</label>
		<button on:click="{sendMessage}">Send</button>
	{/if}
{/if}
