<template>
	<div id="app" class="d-flex flex-column align-items-center">
		{{meta}}
	</div>
</template>

<script>
/* eslint-disable */ 
import Box from './components/box.vue'
import Chat from './components/chat.vue'
export default {
	name: 'App',
	components: {
		Box,
		Chat
	},
	data: ()=>{
		return {
			host: '',
			meta: null,
		}
	},
	mounted() {
		
	},
	methods: {
		start() {
			this.socket = io.connect(this.host+'/containers');

			this.socket.on('connect', ()=>{
				console.log('Connected to log server...');
				this.socket.emit('meta', {});
			});

			this.socket.on("meta", (meta)=>{
				console.log("Meta received: ", meta);
				this.meta = meta;
			});

			this.socket.on("line", (line) => {
				console.log(line);
			});
		},
	}
}
</script>

<style>
html,
body {
	background-color: #121212;
	color: #1F1B24 !important;
}
#app {
	font-family: Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
	color: #2c3e50;
	margin-top: 60px;
}
.title {
	font-family: 'Mitr', sans-serif;
}
</style>