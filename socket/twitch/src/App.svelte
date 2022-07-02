<script>
	import * as tmi from 'tmi.js';
	import { msgs, channels } from './store/global.js';

	const BOT_NAME = 'account name';
	const BOT_KEY = 'oauth: https://twitchapps.com/tmi/';

	let client;
	let messageBox;
	let autoscroll = true;
	let msgInput = new String();

	client = new tmi.Client({
		identity: {
			username: BOT_NAME,
			password: BOT_KEY
		},
		channels: $channels,
		options: {
			debug: false,
			messagesLogLevel: "info"
		},
		connection: {
			reconnect: true,
			secure: true
		},
	});

	client.connect().catch('ERROR', console.error);

	client.on('message', (channel, tags, message, self) => {

		if (message) {
			msgs.update(msg => msg = [...msg, { 'message': message, 'tags': tags } ]);
		}
		
		/* debug
		console.log('tags', tags);
		console.log(`${tags.username}: ${message}`);
		*/

		// don't react to self
		if (self) return;

		let parseMsg = message.toLowerCase().trim().split(" "); 
		
		if (parseMsg.some( m => m.includes(BOT_NAME)) ) {
			console.log('someone mentioned me!');
		}

		if (parseMsg[0] === '!kill') {
			// TODO:: killable while window unfocused
			alert(`${tags.username} killed your bot, refresh to revive`); 
		} else if (parseMsg[0] === '!hello') {
			client.say($channels[0], `@${tags.username} hello world!`);
		}
		
		if (autoscroll && messageBox.scrollHeight) {
			messageBox.scrollTop = messageBox.scrollHeight;
		}
	});

	const hdlSendMsg = () => {
		client.say($channels[0], msgInput);
		msgInput = new String();
	}

	// sendMsg on enter
	document.onkeypress = (e) =>  {
		if (msgInput.length) {
			if (e.code === 'Enter') {
				hdlSendMsg();
			}
		}
	}
</script>


<header>
	<h3>Chat ({$msgs.length})</h3>
	<button on:click={() => autoscroll = !autoscroll} class:Autoscroll={autoscroll} >
		Autoscroll
	</button>
	<section>
		<input type='text' bind:value={msgInput} placeholder='enter message' />
		<button on:click={hdlSendMsg}>Send</button>
	</section>
</header>

<main bind:this={messageBox}>
	<section>	
		{#each $msgs as msg}
			<div class='Row' title={msg.tags['client-nonce']} >
				<h5>{msg.tags.username}</h5>
				<p>{msg.message}</p>
			</div>
		{/each}
	</section>
</main>


