<template>
	<div id="app" class="d-flex align-items-center justify-content-center my-auto">

        <div 
			class="d-flex flex-column align-items-center justify-content-center flex-wrap" 
			style="max-width: 1300px;" >

            <p class="header-1 text-primary py-0 px-0 mb-4"> Containers </p>
		
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
					<span class="h5 my-auto font-weight-bold" :class="{'text-muted': selected!=key}"> {{parseName(value.name)}} </span>
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

					<span v-if="removing" class="ml-3 my-auto beat"> Container removed. Closing in ... {{ removeCounter }} </span>
					
					<span class="ml-auto mr-3 my-auto text-white" > Service log - <span class="font-weight-bold" v-if="meta[selected]"> {{ parseName(meta[selected].name) }} </span> </span>
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
export default {
	name: 'App',
	data: ()=>{
		return {
			host: 'localhost:3000',
			metaSocket: null,
			meta: null,
			selected: null,
			parser: new Convert(),
			maximized: false,
			removeCounter: 5,
			removing: null,
		}
	},
	mounted() {
		this.start();
	},
	methods: {
		start() {
			this.metaSocket = io.connect(this.host+'/meta');

			this.metaSocket.on('connect', () => {});

			this.metaSocket.on("meta", (meta)=>{
				this.meta = meta;
			});

			this.metaSocket.on("added", (container)=> {
				this.meta[container.id] = container;
				this.$forceUpdate();
			});

			this.metaSocket.on("removed", (containerID)=> {
				if(this.socket) {
					this.socket.emit('unsubscribe');
					this.socket.removeAllListeners(containerID+'-init');
					this.socket.removeAllListeners(containerID+'-line');
				}
				if(this.selected == containerID) {
					var handle = setInterval(() => {
						this.removing = this.selected;
						this.removeCounter--;
						if(this.removeCounter == 0) {
							clearInterval(handle);
							this.removeCounter = 5;
							this.removing = null;
							this.selected = null;
						}
					}, 1000);
				}
				delete this.meta[containerID];
				this.$forceUpdate();
			});
		},
		subscribe() {
			this.socket = io.connect(this.host+'/'+this.selected);

			this.socket.on('connect', () => {});

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

			this.socket.on(this.selected+"-down", () => {
				var item = document.createElement('li');
				item.style.color = "tomato";
				item.innerHTML = "Service is down...";
				document.getElementById('console').appendChild(item);
				document.getElementById('console').scrollTop = document.getElementById('console').scrollHeight;
			});
		},
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
		},
		parseName(fullname) {
			var name = fullname;
			if(fullname.split('_').length > 1) {
				var parts = fullname.split('_');
				if(parts.length == 2)
					name = parts[0];
				else if(parts.length == 3)
					name = parts[1];
				else(parts.length > 3)
					name = parts.slice(1, parts.length-1);
			}
			return name.join("-");
		}
	}
}
</script>

<style>
html,
body {
	background-color: #121212;
	min-height: 100vh;
	display: flex;
	flex-wrap: wrap;
	align-items: center;
	justify-content: center;
}
#app {
	font-family: Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
	color: #2c3e50;
	margin-top: 60px;
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
@keyframes heartbeat {
  0% {
	color: white;
	transform: scale(.99);
  }
  50% {
	color: tomato;
    transform: scale(1.01);
  }
  100% {
	color: white;
    transform: scale(.99);
  }
}
.beat {
    animation: heartbeat .75s infinite;
}
</style>