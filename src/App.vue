<template>

	<div v-if="!unauthorized" id="app" class="d-flex flex-wrap align-items-center justify-content-center my-auto">

        <div 
			class="flex-column align-items-center justify-content-center flex-wrap" 
			:class="{'d-none d-lg-flex': selected, 'd-flex': !selected}"
			style="max-width:50em;" >

			<p class="header-2 text-primary py-0 px-0 my-3 logo-text"> 
				Watch..
				<svg class="mx-2"
					height="50px"
					width="34px"
					version="1.1"
					viewBox="0 0 234 250"
					xmlns="http://www.w3.org/2000/svg"
					xmlns:xlink="http://www.w3.org/1999/xlink" >
					<g fill="tomato" fill-rule="nonzero" id="logo">
						<path id="Shape" d="M232.237912,124.63958 L207.054086,87.2418902 L209.962149,5.64164223 C210.078472,3.66415939 209.08973,1.7448378 207.403054,0.756096381 C205.716377,-0.23264504 203.56441,-0.23264504 201.935895,0.814257642 C182.33555,14.540315 137.493219,48.3901684 122.720259,93.0580162 L145.984763,87.2418902 C145.984763,87.2418902 99.1649485,135.22493 81.6002479,210.369278 C81.6002479,210.369278 79.2156363,220.838305 78.5758624,227.875817 C79.4482813,229.911461 80.3788615,232.063428 81.1931191,234.273556 C81.2512804,234.44804 84.1593434,242.29981 85.3807298,246.312937 C86.078665,248.523064 88.1724703,250.093418 90.4989207,250.093418 L143.309345,250.093418 C145.635795,250.093418 147.729601,248.523064 148.427536,246.312937 C168.318687,181.230487 211.416181,170.528815 213.21918,170.179847 C214.847695,169.772718 216.243565,168.609493 216.9415,167.039139 L232.819524,129.757771 C233.517459,128.071095 233.342976,126.093612 232.237912,124.63958 Z M116.904133,221.012788 L99.4557548,191.932158 L134.352511,191.932158 L116.904133,221.012788 Z M160.17611,151.219276 C144.705215,151.219276 134.352511,151.219276 134.352511,151.219276 L180.881519,122.138646 C180.881519,122.138646 175.705167,151.219276 160.17611,151.219276 Z M87.8235028,52.3451341 C76.6565408,64.6753212 45.7147505,105.620848 36.9324002,151.219276 L53.1012305,151.219276 L49.4370711,173.262394 L47.4595883,185.185452 C43.3883,181.812099 35.5365299,176.752069 35.2457236,176.635747 C26.6960184,171.633879 20.7635699,170.238008 20.5890861,170.179847 C18.9605708,169.772718 17.5647006,168.609493 16.8667654,167.039139 L0.988741422,129.757771 C0.290806301,128.071095 0.465290081,126.093612 1.57035402,124.63958 L26.7541797,87.2418902 L23.8461166,5.64164223 C23.7297941,3.48967561 24.8930193,1.4540315 26.8705022,0.523451341 C28.7316625,-0.407128821 31.0581129,-0.0581612601 32.6866282,1.39587024 L87.8235028,52.3451341 Z M99.4557548,151.219276 C99.4557548,151.219276 89.1204989,151.219276 73.6088908,151.219276 C58.0972827,151.219276 52.9267467,122.138646 52.9267467,122.138646 L99.4557548,151.219276 Z" />
					</g>
				</svg>
				..dog
			</p>

			<b-list-group
				v-if="meta" 
				class="d-flex flex-row align-items-stretch justify-content-center flex-wrap">
				
				<b-list-group-item 
					v-for="(value, key) in meta" 
					@click="changeSelected(key)"
					:key="key" 
					:active="selected==key" 
					:class="{'bg-primary': selected==key}" 
					class="btn d-flex flex-row align-items-center m-2 " 
					style="border-radius: 1em; width: 15em; background-color:#1d1d1d;" >
					<div class="d-flex align-items-center mr-lg-2" style="width:2em; height:2em; border-radius:50%;">
						<i class="fa fa-microchip w-100" :class="{
							'text-light': value.state=='starting',
							'text-success': value.state=='running',
							'text-muted': ['exited', 'dead'].includes(value.state), 
							'text-warning': value.state=='restarting',
							'text-info': value.state=='paused',
							'text-danger': value.state=='removing',
						}" style="font-size: 1em;"></i>
					</div>
					<div class="d-flex flex-column text-left">
						<span class="h5 my-auto font-weight-bold text-white" style=""> {{parseName(value.name)}} </span>
						<span class="my-auto" :class="{'text-muted': selected!=key, 'text-white': selected==key}" v-text="value.state" > </span>
					</div>
				</b-list-group-item>

			</b-list-group>
		</div>

		<transition name="bounce" >
			<div 
				v-if="selected" 
				:class="{'maximized': maximized}" 
				class="mx-5 d-flex flex-column shadow" 
				style="min-width:50vw; max-width:650px; max-height:650px; overflow:hidden; border-radius:.75em;" >
				<b-input-group 
					style="width: unset;" 
					class="ml-auto mr-5 my-3" >
					<template #append>
						<span @click="caseSensitive=!caseSensitive;mark()" class="btn rounded-circle mx-1 font-weight-bold my-auto" :class="{'text-white': caseSensitive, 'text-muted': !caseSensitive}">Aa</span>
						<i v-if="highlight.length" @click="findNext" class="btn rounded-circle mx-1 fa fa-arrow-down font-weight-bold my-auto" :class="{'text-white': key}"></i>
						<i v-if="highlight.length" @click="findPrev" class="btn rounded-circle mx-1 fa fa-arrow-up font-weight-bold my-auto" :class="{'text-white': key}"></i>
					</template>
					<b-form-input @keypress.enter="mark()" v-model="key" placeholder="Search" class="bg-transparent border-0 mx-2" :class="{'text-white': key}"></b-form-input>
				</b-input-group>

				<div class="d-flex shadow" style="background:#181818; padding: .85em;">
					<div @click="changeSelected(null)" class="mx-1 rounded-circle bg-danger d-flex" style="height:1em;width:1em" >
						<i class="fa fa-close font-weight-bold text-white mx-auto my-auto" style="font-size:.5em;"></i>
					</div>
					<div @click="maximized=!maximized" class="mx-1 rounded-circle bg-success d-flex" style="height:1em;width:1em" >
						<i class="fa fa-expand font-weight-bold text-white mx-auto my-auto" style="font-size:.5em;"></i>
					</div>

					<span v-if="removing" class="ml-3 my-auto beat"> Container removed. Closing in ... {{ removeCounter }} </span>

					<div class="ml-auto mr-3">
						<div class="d-flex" v-if="meta[selected].state != 'restarting' && !removing">
							<span class="btn btn-outline-light mx-2" v-if="!['running', 'removing'].includes(meta[selected].state)" @click="Socket.emit(selected+'-start')">Start</span>
							<span class="btn btn-outline-light mx-2" v-if="meta[selected].state == 'running'" @click="Socket.emit(selected+'-restart')">Restart</span>
							<span class="btn btn-outline-light mx-2" v-if="meta[selected].state == 'running'" @click="Socket.emit(selected+'-stop')">Stop</span>
							<span class="btn btn-outline-light mx-2" v-if="meta[selected].state == 'exited'" @click="Socket.emit(selected+'-remove')">Remove</span>
						</div>
					</div>
					
					<span class="ml-3 mr-3 my-auto text-white" > Service log - <span class="font-weight-bold" v-if="meta[selected]"> {{ parseName(meta[selected].name) }} </span> </span>
				</div>
				<pre
					id="console"
					style="width:100%; height:100%; overflow:auto; list-style:none; background:#1c1c1c;"
					class="terminal p-5 m-0 text-white text-left d-block" >
					<li 
						:id="idx" 
						v-for="(line, idx) in lines" 
						:key="idx" 
						:class="{'bg-highlight': highlight[pointer] == idx}"
						v-html="highlight && highlight.indexOf(idx)==-1 ? line : line.replace(toRegex(markKey), '<span class=\'bg-mark\'>$&</span>')">
					</li>
					<li style="color: tomato; margin-left:-1em; margin-bottom:-2em;" v-if="meta[selected].state=='exited'">Service is down ...!</li>
				</pre>
			</div>
		</transition>

	</div>
	<div 
		v-else 
		id="app" 
		class="d-flex flex-column align-items-stretch justify-content-cente my-auto shadow p-5" 
		style="background:#171717; border-radius:1em;" >
        <div class="form-header px-4 py-0 my-3" >
			<p class="header-2 text-primary px-0 py-0 logo-text"> 
				Watch..
				<svg class="mx-2"
					height="50px"
					width="34px"
					version="1.1"
					viewBox="0 0 234 250"
					xmlns="http://www.w3.org/2000/svg"
					xmlns:xlink="http://www.w3.org/1999/xlink" >
					<g fill="tomato" fill-rule="nonzero" id="logo">
						<path id="Shape" d="M232.237912,124.63958 L207.054086,87.2418902 L209.962149,5.64164223 C210.078472,3.66415939 209.08973,1.7448378 207.403054,0.756096381 C205.716377,-0.23264504 203.56441,-0.23264504 201.935895,0.814257642 C182.33555,14.540315 137.493219,48.3901684 122.720259,93.0580162 L145.984763,87.2418902 C145.984763,87.2418902 99.1649485,135.22493 81.6002479,210.369278 C81.6002479,210.369278 79.2156363,220.838305 78.5758624,227.875817 C79.4482813,229.911461 80.3788615,232.063428 81.1931191,234.273556 C81.2512804,234.44804 84.1593434,242.29981 85.3807298,246.312937 C86.078665,248.523064 88.1724703,250.093418 90.4989207,250.093418 L143.309345,250.093418 C145.635795,250.093418 147.729601,248.523064 148.427536,246.312937 C168.318687,181.230487 211.416181,170.528815 213.21918,170.179847 C214.847695,169.772718 216.243565,168.609493 216.9415,167.039139 L232.819524,129.757771 C233.517459,128.071095 233.342976,126.093612 232.237912,124.63958 Z M116.904133,221.012788 L99.4557548,191.932158 L134.352511,191.932158 L116.904133,221.012788 Z M160.17611,151.219276 C144.705215,151.219276 134.352511,151.219276 134.352511,151.219276 L180.881519,122.138646 C180.881519,122.138646 175.705167,151.219276 160.17611,151.219276 Z M87.8235028,52.3451341 C76.6565408,64.6753212 45.7147505,105.620848 36.9324002,151.219276 L53.1012305,151.219276 L49.4370711,173.262394 L47.4595883,185.185452 C43.3883,181.812099 35.5365299,176.752069 35.2457236,176.635747 C26.6960184,171.633879 20.7635699,170.238008 20.5890861,170.179847 C18.9605708,169.772718 17.5647006,168.609493 16.8667654,167.039139 L0.988741422,129.757771 C0.290806301,128.071095 0.465290081,126.093612 1.57035402,124.63958 L26.7541797,87.2418902 L23.8461166,5.64164223 C23.7297941,3.48967561 24.8930193,1.4540315 26.8705022,0.523451341 C28.7316625,-0.407128821 31.0581129,-0.0581612601 32.6866282,1.39587024 L87.8235028,52.3451341 Z M99.4557548,151.219276 C99.4557548,151.219276 89.1204989,151.219276 73.6088908,151.219276 C58.0972827,151.219276 52.9267467,122.138646 52.9267467,122.138646 L99.4557548,151.219276 Z" />
					</g>
				</svg>
				..dog
			</p>
        </div>

        <div class="mx-auto border-0 shadow-sm mb-4 py-0" >
            <b-input-group >
                <b-form-input class="p-4 border-0 font-weight-bold text-white" style="background: #1d1d1d;" v-model="pin" type="password" required placeholder="Enter pin"> </b-form-input>
            </b-input-group>
            
            <b-form-invalid-feedback class="text-center mt-3 mb-0 font-weight-bold" v-if="pinError" :state="false">
                {{pinError}}
            </b-form-invalid-feedback>
        </div>

        <div class="d-flex align-items-center my-4 py-0 border-0 shadow-sm">
            <b-button v-if="!logingIn" class="px-3 py-1 m-0 mx-auto login-btn" @click="start"> LogIn </b-button>
            <b-spinner v-else class="mx-auto bg-primary" small type="grow"></b-spinner>
        </div>
	</div>
	
</template>

<script>
/* eslint-disable */ 
import Convert from 'ansi-to-html';
export default {
	name: 'App',
	data: ()=>{
		return {
			host: '',
			Socket: null,
			meta: null,
			selected: null,
			parser: new Convert(),
			maximized: false,
			removeCounter: 5,
			removing: null,
			lines: [],
			key: "",
			markKey: "",
			pointer: -1,
			highlight: [],
			// downlist: {},
			pin: null,
			unauthorized: true,
			logingIn: false,
			pinError: false,
			caseSensitive: false,
		}
	},
	mounted() {

	},
	methods: {
		start() {
			this.logingIn = true;
			this.Socket = io.connect(this.host+'/meta',{
				query: "pin=" + this.pin,
			});

			this.Socket.on('disconnect', () => { 
				if(this.logingIn) {
					this.unauthorized = true;
					this.logingIn = false; 
					this.pinError = 'Incorrect pin !'; 
				}
			});

			this.Socket.on('connect', () => {
				if(this.logingIn) {
					this.unauthorized = false;
					this.logingIn = false;
					this.pinError = '';
					this.Socket.emit('meta');
				}
			});

			this.Socket.on("meta", (meta)=>{
				meta = this.sort(meta);
				this.meta = meta;
				this.$forceUpdate();
			});

			this.Socket.on("state-change", (data)=> {
				if(this.meta[data.id])
					this.meta[data.id].state = data.state;
				this.$forceUpdate();
			});

			this.Socket.on("added", (container)=> {
				this.meta[container.id] = container;
				this.meta = this.sort(this.meta);
				this.$forceUpdate();
			});

			this.Socket.on("removed", (containerID)=> {
				this.Socket.emit(containerID+'-unsubscribe');
				this.Socket.removeAllListeners(containerID+'-init');
				this.Socket.removeAllListeners(containerID+'-line');
				this.Socket.removeAllListeners(containerID+'-subscribed');
				this.Socket.removeAllListeners(containerID+'-unsubscribed');
				if(this.selected == containerID) {
					var handle = setInterval(() => {
						this.removing = this.selected;
						this.removeCounter--;
						if(this.removeCounter == 0) {
							clearInterval(handle);
							this.removeCounter = 5;
							this.removing = null;
							this.selected = null;
							delete this.meta[containerID];
						}
					}, 1000);
				} else
					delete this.meta[containerID];
				this.$forceUpdate();
			});
		},
		subscribe() {
			this.Socket.emit(this.selected+'-subscribe');

			this.Socket.on(this.selected+'-subscribed', () => {
				this.Socket.on(this.selected+"-init", (data) => {
					data.forEach(line => {
						var log;
						try {
							log = JSON.parse(line).log;
						} catch (err) {
							log = line;
						}
						this.lines.push(this.parser.toHtml(log));
						this.$forceUpdate();
					});
					setTimeout(()=>{
						document.getElementById('console').scrollTop = document.getElementById('console').scrollHeight;
					}, 50);
				});
	
				this.Socket.on(this.selected+"-line", (line) => {
					this.lines.push(this.parser.toHtml(line));
					this.$forceUpdate();
					document.getElementById('console').scrollTop = document.getElementById('console').scrollHeight;
				});
			});
		},
		changeSelected(key) {
			this.pointer = -1;
			this.highlight= [];
			this.key = "";
			this.lines = [];
			this.$forceUpdate();
			if(this.selected) {
				this.Socket.emit(this.selected+'-unsubscribe');
				this.Socket.on(this.selected+'-unsubscribed', () => {
					this.Socket.removeAllListeners(this.selected+'-init');
					this.Socket.removeAllListeners(this.selected+'-line');
					this.Socket.removeAllListeners(this.selected+'-subscribed');
					this.Socket.removeAllListeners(this.selected+'-unsubscribed');
					this.selected = null;
					setTimeout(() => {
						this.selected = key;
						if(this.selected) {
							this.subscribe();
						}
					}, 50);
				});
			}else {
				this.selected = key;
				this.subscribe();
			}
        },
		toTitleCase(str) {
			return str.replace(/\w\S*/g, (txt) => { return txt.charAt(0).toUpperCase() + txt.substr(1).toLowerCase() });
		},
		parseName(fullname) {
			let nameParts = fullname.split('/').filter(x => x.length > 0);
			let name = nameParts.join('-');
			return name.length > 15 ? name.substring(0, 15)+'..' : name;
		},
		mark() {
			this.highlight = [];
			this.markKey = "";
			if(this.key.trim().length) {
				this.lines.forEach((line, idx) => {
					if(line.match(this.toRegex(this.key)))
							this.highlight.push(idx);
				});
				this.markKey = this.key;
			}
			this.$forceUpdate();
		},
		toRegex(key) {
			if(this.caseSensitive) {
				return new RegExp(`${key}`,"g");
			}else {
				return new RegExp(`${key}`,"gi");
			}
		},
		findNext() {
			if(this.pointer == -1)
				this.pointer = 0;
			else
				this.pointer++;

			if(this.pointer >= this.highlight.length)
				this.pointer = 0;

			var el = document.getElementById(this.highlight[this.pointer]);
			el.scrollIntoView(true);
			el.parentElement.scrollTop -= 75;
		},
		findPrev() {
			if(this.pointer == -1)
				this.pointer = this.highlight.length - 1;
			else
				this.pointer--;

			if(this.pointer < 0)
				this.pointer = this.highlight.length - 1;

			var el = document.getElementById(this.highlight[this.pointer]);
			el.scrollIntoView(true);
			el.parentElement.scrollTop -= 75;
		},
		sort(meta) {
			var sortable = [];
			// Convert to array
			Object.values(meta).forEach((container)=>{
				sortable.push(container);
			});
			// Sort
			sortable.sort((a, b) => {
				if ( a.name < b.name )
					return -1;
				if ( a.name > b.name )
					return 1;
				return 0;
			});
			// Convert back
			meta = {}
			sortable.forEach(container => {
				meta[container.id] = container;
			});
			return meta
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
.login-btn {
	border: 1px solid #5874c9 !important;
	color: #5874c9 !important;
	background: unset !important;
}
.login-btn:hover {
	background: #5874c9 !important;
	color: white !important;
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
		transform: scale(.99);
	}
	50% {
		transform: scale(1.01);
	}
	100% {
		transform: scale(.99);
	}
}
.beat {
	color: tomato;
	animation: heartbeat .75s infinite;
}
.bg-mark {
	background: #FF5722;
	color: white;
	font-weight: bold;
}
.bg-highlight {
	background: #0005;
	color: white;
	animation: heartbeat 1s infinite;
}
.logo-text {
	background: linear-gradient(to right, #5874c9, #FF5722, #5874c9);
	-webkit-background-clip: text;
	-moz-background-clip: text;
	-ms-background-clip: text;
	background-clip: text;
	background-size: 200% 200%;
	-webkit-text-fill-color: transparent;
  	animation: logo-text-move ease 5s infinite;
}
@keyframes logo-text-move {
	0%{
		background-position:0% 0%;
	}
    50%{
		background-position:100% 100%;
	}
    100%{
		background-position:200% 200%;
	}
}
</style>
<style scoped>
/* Firefox */
* {
	scrollbar-width: thin;
	scrollbar-color: #aaa4 #0000;
}
/* Chrome, Edge, and Safari */
*::-webkit-scrollbar {
	width: 4px;
	height: 4px;
}
*::-webkit-scrollbar-track {
	background: #0000;
}
*::-webkit-scrollbar-thumb {
	background-color: #aaa4;
	border-radius: 5em;
}
</style>