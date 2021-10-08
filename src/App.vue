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
			<div v-if="selected" class="mx-5 d-flex flex-column shadow-lg" style="min-width:50vw; max-width:800px; max-height:600px; overflow:hidden; border-radius:.75em;" >
				<div class="d-flex shadow-lg" style="background:#181818; padding: .85em;">
					<div class="mx-1 rounded-circle bg-danger" style="height:1em;width:1em" ></div>
					<div class="mx-1 rounded-circle bg-warning" style="height:1em;width:1em" ></div>
					<div class="mx-1 rounded-circle bg-success" style="height:1em;width:1em" ></div>
					<span class="ml-auto mr-3 my-auto text-white" > Service log - <span class="font-weight-bold"> {{ toTitleCase(meta[selected].name) }} </span> </span>
				</div>
				<pre
					id="console"
					style="min-width:50vw; max-width:800px; max-height:600px; overflow:auto; list-style:none; background:#1c1c1c;"
					class="p-5 text-white text-left d-block" >
				</pre>
			</div>
		</transition>

	</div>
</template>

<script>
/* eslint-disable */ 
export default {
	name: 'App',
	data: ()=>{
		return {
			host: 'localhost:3000',
			meta: null,
			selected: null,
		}
	},
	mounted() {
		this.start();
	},
	methods: {
		start() {
			console.log("Starting client...");
			this.socket = io.connect(this.host+'/containers');

			this.socket.on('connect', ()=>{
				console.log('Connected to log server...');
				this.socket.emit('meta', {});
			});

			this.socket.on("meta", (meta)=>{
				console.log("Meta received: ", meta);
				this.meta = meta;
			});
		},
		subscribe() {
			this.socket.emit('subscribe', {containerID: this.selected});
			
			this.socket.on("init", (data) => {
				data.forEach(line => {
					document.getElementById('console').append(JSON.parse(line.replace(/(\r\n|\n|\r)/gm, "")).log);
					document.getElementById('console').scrollTop = document.getElementById('console').scrollHeight;
				});
			});

			this.socket.on("line", (line) => {
				var item = document.createElement('li');
				item.innerText = line;
				item.style.color = '#fff';
				document.getElementById('console').appendChild(item);
				document.getElementById('console').scrollTop = document.getElementById('console').scrollHeight;
			});
		},
		changeSelected(key) {
			this.socket.emit('unsubscribe');
			this.socket.disconnect();
			this.start();
			setTimeout(()=>{
				if(this.selected != key) {
					this.selected = null;
					setTimeout(() => {
						this.selected = key;
						this.subscribe();
					}, 50);
				} else {
					this.selected = null;
				}
			}, 500);
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
</style>
