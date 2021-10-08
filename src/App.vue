<template>
	<div id="app" class="d-flex align-items-center justify-content-center">

        <div 
			class="section d-flex flex-column align-items-center justify-content-center flex-wrap" 
			style="max-width: 1300px;" >

            <p class="header-1 text-muted py-0 px-0 mb-4"> Containers </p>
		
			<b-list-group
				v-if="meta" 
				class="d-flex flex-row align-items-center justify-content-center flex-wrap">
				
				<b-list-group-item 
					v-for="(value, key) in meta" 
					@click="changeSelected(key)"
					:key="key" 
					:active="selected==key" 
					:class="{'bg-primary': selected==key}" 
					class="btn d-flex flex-row align-items-center m-2 bg-transparent border border-primary" 
					style="border-radius: 1em" >
					<div class="d-flex align-items-center mr-lg-2" style="width:4em; height:4em; border-radius:50%;">
						<i class="fa fa-microchip w-100" :class="{'text-light': selected==key, 'text-primary': selected!=key}" style="font-size: 2.5em;"></i>
					</div>
					<span class="h5 my-auto font-weight-bold" :class="{'text-muted': selected!=key}"> {{toTitleCase(value.name)}} </span>
				</b-list-group-item>

			</b-list-group>
		</div>

		<transition name="bounce" >
			<div v-if="selected" :class="{'maximized': maximized}" class="terminal mx-5 d-flex flex-column shadow" style="min-width:50vw; max-width:800px; max-height:600px; overflow:hidden; border-radius:.75em;" >
				<div class="d-flex shadow" style="background:#181818; padding: .85em;">
					<div @click="changeSelected(null)" class="mx-1 rounded-circle bg-danger d-flex" style="height:1em;width:1em" >
						<i class="fa fa-close font-weight-bold text-white mx-auto my-auto" style="font-size:.5em;"></i>
					</div>
					<div @click="maximized=!maximized" class="mx-1 rounded-circle bg-success d-flex" style="height:1em;width:1em" >
						<i class="fa fa-expand font-weight-bold text-white mx-auto my-auto" style="font-size:.5em;"></i>
					</div>
					<span class="ml-auto mr-3 my-auto text-white" > Service log - <span class="font-weight-bold"> {{ toTitleCase(meta[selected].name) }} </span> </span>
				</div>
				<pre
					id="console"
					style="min-width:50vw; max-width:800px; max-height:600px; overflow:auto; list-style:none; background:#1c1c1c;"
					class="p-5 m-0 text-white text-left d-block" >
				</pre>
			</div>
		</transition>

	</div>
</template>

<script>
/* eslint-disable */ 
import Convert from 'ansi-to-html';
import axios from 'axios';
export default {
	name: 'App',
	data: ()=>{
		return {
			host: 'localhost:3000',
			meta: null,
			selected: null,
			parser: new Convert(),
			maximized: false,
		}
	},
	mounted() {
		this.start();
	},
	methods: {
		start() {
			// console.log("Starting client...");
			// this.socket = io.connect(this.host+'/containers');

			// this.socket.on('connect', ()=>{
			// 	console.log('Connected to log server...');
			// 	this.socket.emit('meta', {});
			// });

			// this.socket.on("meta", (meta)=>{
			// 	console.log("Meta received: ", meta);
			// 	this.meta = meta;
			// });
			axios
			.get("http://localhost:3000/meta")
			.then(res => {
				console.log(res.data);
				this.meta = res.data;
			})
			.catch(err => {
				console.log("Error: ", err);
			})
		},
		subscribe() {
			console.log("Starting client...");
			this.socket = io.connect(this.host+'/'+this.selected);

			this.socket.on('connect', ()=>{
				console.log('Connected to log server...');
			});

			this.socket.on(this.selected+"-init", (data) => {
				document.getElementById('console').innerHTML = "";
				data.forEach(line => {
					var item = document.createElement('li');
					var log;
					try {
						log = JSON.parse(line).log;
					}catch (err) {
						log = line;
					}
					item.innerHTML = this.parser.toHtml(log);
					document.getElementById('console').appendChild(item);
					document.getElementById('console').scrollTop = document.getElementById('console').scrollHeight;
				});
			});

			this.socket.on(this.selected+"-line", (line) => {
				var item = document.createElement('li');
				item.innerHTML = this.parser.toHtml(line);
				document.getElementById('console').appendChild(item);
				document.getElementById('console').scrollTop = document.getElementById('console').scrollHeight;
			});

			this.socket.on(this.selected+"-error", (err) => {
				var item = document.createElement('li');
				item.innerHTML = this.parser.toHtml(err);
				document.getElementById('console').appendChild(item);
				document.getElementById('console').scrollTop = document.getElementById('console').scrollHeight;
				this.socket.emit('unsubscribe');
			});
		},
		// subscribe() {
			// this.socket.emit('subscribe', {containerID: this.selected});
			
			// this.socket.on(this.selected+"-init", (data) => {
			// 	document.getElementById('console').innerHTML = "";
			// 	data.forEach(line => {
			// 		var item = document.createElement('li');
			// 		var log;
			// 		try {
			// 			log = JSON.parse(line).log;
			// 		}catch (err) {
			// 			log = line;
			// 		}
			// 		item.innerHTML = this.parser.toHtml(log);
			// 		document.getElementById('console').appendChild(item);
			// 		document.getElementById('console').scrollTop = document.getElementById('console').scrollHeight;
			// 	});
			// });

			// this.socket.on(this.selected+"-line", (line) => {
			// 	var item = document.createElement('li');
			// 	item.innerHTML = this.parser.toHtml(line);
			// 	document.getElementById('console').appendChild(item);
			// 	document.getElementById('console').scrollTop = document.getElementById('console').scrollHeight;
			// });

			// this.socket.on(this.selected+"-error", (err) => {
			// 	var item = document.createElement('li');
			// 	item.innerHTML = this.parser.toHtml(err);
			// 	document.getElementById('console').appendChild(item);
			// 	document.getElementById('console').scrollTop = document.getElementById('console').scrollHeight;
			// 	this.socket.emit('unsubscribe', {containerID: this.selected});
			// });
		// },
		// changeSelected(key) {
			// if(this.selected) {
			// 	this.socket.emit('unsubscribe', {containerID: this.selected});
			// 	this.selected = null;
			// 	this.socket.on('unsubscribed', (containerID) => {
			// 		console.log("Unsubscribed: ", containerID);
			// 		this.socket.removeAllListeners(containerID+'-init');
			// 		this.socket.removeAllListeners(containerID+'-line');
			// 		if(this.selected != key) {
			// 			this.selected = key;
			// 			this.subscribe();
			// 		} else {
			// 			this.selected = null;
			// 		}
			// 	});
			// }else {
			// 	this.selected = key;
			// 	this.subscribe();
			// }
        // },
		changeSelected(key) {
			if(this.selected) {
				let selected = this.selected;
				this.selected = null;
				this.socket.emit('unsubscribe', {containerID: selected});
				this.socket.removeAllListeners(selected+'-init');
				this.socket.removeAllListeners(selected+'-line');
				setTimeout(()=>{
					this.selected = key;
					if(this.selected)
						this.subscribe();
				}, 50);
			}else {
				this.selected = key;
				this.subscribe();
			}
        },
		toTitleCase(str) {
			return str.replace(/\w\S*/g, (txt) => { return txt.charAt(0).toUpperCase() + txt.substr(1).toLowerCase() });
		}
	}
}
</script>

<style>
html,
body {
	background-color: #121212;
	/* color: #1F1B24 !important; */
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
<style>
/* body,
html {
  width: 100%;
}
#app {
  font-family: 'Roboto', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  font-size: 1.125em;
} */
.section {
  min-height: 80vh;
}
.leaf-card {
  border-radius: 1.75em;
  border-top-right-radius: .5em;
  border-bottom-left-radius: .5em;
  transition: all .25s;
}
.leaf-card:hover {
  transform: translateY(-.5em);
}
</style>

<style>
/* contact us button  */
.rounded-custom-5 {
    border-radius: 5em;
}
.rounded-custom-1 {
    border-radius: 1em;
}
.outline-primary {
    color: #5874c9;
    border: 2px solid #5874c9;
}
.outline-primary:hover {
    color: white;
    background: #5874c9;
}
.outline-light {
    color: white;
    border: 2px solid white;
}
.outline-light:hover {
    color: #5874c9;
    background-color: white;
}
.contact-btn {
    height: 45px;
    width: 130px;
    color: #5874c9;
    border-radius: 5em;
    align-self: flex-end;
    border: 2px solid #5874c9;
}
.contact-btn:hover {
    color: white;
    background: #5874c9;
}
.contact-bar {
    max-width: 600px;
    width: 100%;
}
</style>
<style>
/* Fonts */
.header-1 {
  font-family: 'Lato';
  font-weight: 900;
  font-size: 2.5em;
}
.header-2 {
  font-family: 'Lato';
  font-weight: 900;
  font-size: 1.25em;
}
</style>
<style>
/* Colors */
.text-primary {
  color: #5874c9 !important;
}
.bg-primary {
  background-color: #5874c9 !important;
}
.bg-primary-gradient {
  background: linear-gradient(120deg, #5874c9 30%, #90CAF9);
}
.border-light-custom{
  border-color: #2c3e50;

}
</style>
<style> 
/* Animation */
.bounce-enter-active {
  animation: bounce-in .5s;
}
.bounce-leave-active {
  animation: bounce-in .5s reverse;
}
@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.1);
  }
  100% {
    transform: scale(1);
  }
}
.bounce-modal.show {
    animation: bounce-in .5s;
}
.maximized * {
	max-width: unset !important; 
	max-height: unset !important;
}
.maximized {
	position: fixed;
	z-index: 10;
	max-width: unset !important; 
	max-height: unset !important;
	width: 70vw !important;
	height: 70vh !important;
}
</style>